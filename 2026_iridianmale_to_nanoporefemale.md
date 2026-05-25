# Get positions with zero coverage

Directory
```
/home/ben/projects/rrg-ben/ben/2025_longipes/iridian
```

```
#!/bin/sh
#SBATCH --job-name=samtools_depth
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=24:00:00
#SBATCH --mem=8gb
#SBATCH --output=samtools_depth.%J.out
#SBATCH --error=samtools_depth.%J.err
#SBATCH --account=rrg-ben

module load StdEnv/2023  gcc/12.3 samtools/1.20
samtools depth -aa ${1} | grep '	0' >> ${1}_zerodepth_aa.txt
```

# Make this in to intervals that have zero coverage

```pl
#!/usr/bin/perl
use strict;
use warnings;

# run like this:
# make_intervals.pl <temp.txt >temp.out

my ($prev_contig, $prev_pos);
my ($start, $end);

while (<STDIN>) {
    chomp;
    my ($contig, $pos) = (split /\t/)[0,1];  # ignore 3rd column

    if (!defined $prev_contig) {
        # first line
        ($prev_contig, $prev_pos) = ($contig, $pos);
        ($start, $end) = ($pos, $pos);
        next;
    }

    # check if still consecutive
    if ($contig eq $prev_contig && $pos == $prev_pos + 1) {
        $end = $pos;
    } else {
        # print previous interval
        print join("\t", $prev_contig, $start, $end), "\n";

        # start new interval
        ($start, $end) = ($pos, $pos);
    }

    ($prev_contig, $prev_pos) = ($contig, $pos);
}

# print last interval
if (defined $prev_contig) {
    print join("\t", $prev_contig, $start, $end), "\n";
}
```
# Identify coding regions by blasting XL_CDS to nanopore genome:
```
#SBATCH --time=6:00:00
#SBATCH --mem=32gb
#SBATCH --output=blastn.%J.out
#SBATCH --error=blastn.%J.err
#SBATCH --account=rrg-ben


module load StdEnv/2023 gcc/12.3 blast+/2.14.1 

blastn -query ${1} -db ${2} -outfmt "6 std qlen" | awk '($4/$13) >= 0.75' > ${1}_to_${2}
```
output is here:
```
/home/ben/projects/rrg-ben/ben/2025_longipes/flye_assembly/XL_CDS_only.fasta_to_assembly.fasta.gz_blastable
```
# Make an interval file that has all of the coding regions
```
awk '{print $2, $9, $10, $1}' XL_CDS_only.fasta_to_assembly.fasta.gz_blastable > XL_CDS_only.fasta_to_assembly.fasta.gz_blastable.bed
```

# Change the spaces to tabs
```
sed -ie 's/ /    /g' XL_CDS_only.fasta_to_assembly.fasta.gz_blastable.bed
```

# Load Bioconductor on ComputeCanada
```
module load  StdEnv/2023  gcc/12.3 r-bundle-bioconductor/3.21
```

# Identify overlapz
```R
#!/usr/bin/env Rscript

# To run:
# script Overlap_Genomic_ranges.R fileA.txt fileB.txt output.txt

# Generate the first file from a blast output like this:
# awk '{print $2, $9, $10, $1}' blastoutput > fileA.txt

# Generate the second file using a perl script to make intervals
# out of the output of: amtools depth -aa ${1} | grep '	0' >> ${1}_zerodepth_aa.txt

suppressPackageStartupMessages(library(GenomicRanges))

# ---- Read input arguments ----
args <- commandArgs(trailingOnly = TRUE)

if (length(args) < 2) {
  stop("Usage: script.R fileA.txt fileB.txt [output.txt]")
}

fileA <- args[1]
fileB <- args[2]
outfile <- ifelse(length(args) >= 3, args[3], NA)

# ---- Read files ----
A <- read.table(fileA, header = FALSE, sep = "",
                stringsAsFactors = FALSE, quote = "")

B <- read.table(fileB, header = FALSE, sep = "",
                stringsAsFactors = FALSE, quote = "")

# Assign column names
colnames(A)[1:4] <- c("chr", "start", "end", "label")
colnames(B)[1:3] <- c("chr", "start", "end")

# fix order of start stop


# Ensure numeric
A$start <- as.numeric(A$start)
A$end   <- as.numeric(A$end)
B$start <- as.numeric(B$start)
B$end   <- as.numeric(B$end)

# Remove NA rows
A <- A[!is.na(A$start) & !is.na(A$end), ]
B <- B[!is.na(B$start) & !is.na(B$end), ]

# Fix coordinate order properly
A_start <- pmin(A$start, A$end)
A_end   <- pmax(A$start, A$end)
A$start <- A_start
A$end   <- A_end

B_start <- pmin(B$start, B$end)
B_end   <- pmax(B$start, B$end)
B$start <- B_start
B$end   <- B_end

# Safety check
if (any(A$end < A$start)) stop("A still has invalid intervals")
if (any(B$end < B$start)) stop("B still has invalid intervals")


# ---- Convert to GRanges ----
grA <- GRanges(seqnames = A$chr,
               ranges = IRanges(start = A$start, end = A$end),
               label = A$label)

grB <- GRanges(seqnames = B$chr,
               ranges = IRanges(start = B$start, end = B$end))


# ---- Find overlaps ----
hits <- findOverlaps(grB, grA)

# Compute overlap widths
ov_width <- width(pintersect(grB[queryHits(hits)], grA[subjectHits(hits)]))

# Compute width of A intervals
A_width <- width(grA)[subjectHits(hits)]

# Fraction of A covered by B
frac_overlap_A <- ov_width / A_width

# Keep only overlaps >= 75%
keep <- frac_overlap_A >= 0.75

hits <- hits[keep]
ov_width <- ov_width[keep]
frac_overlap_A <- frac_overlap_A[keep]


# ---- Extract results ----
result <- data.frame(
  B_chr   = as.character(seqnames(grB))[queryHits(hits)],
  B_start = start(grB)[queryHits(hits)],
  B_end   = end(grB)[queryHits(hits)],

  A_chr   = as.character(seqnames(grA))[subjectHits(hits)],
  A_start = start(grA)[subjectHits(hits)],
  A_end   = end(grA)[subjectHits(hits)],

  A_label = mcols(grA)$label[subjectHits(hits)],

  overlap_bp = ov_width,
  frac_of_A  = frac_overlap_A

)

# ---- Output ----
if (is.na(outfile)) {
  write.table(result, stdout(), sep = "\t", quote = FALSE, row.names = FALSE)
} else {
  write.table(result, file = outfile, sep = "\t", quote = FALSE, row.names = FALSE)
}

```
# Check out the names of genez

```
cut -f7 Coding_intervals_in_lenduflyefem_with_no_coverage_in_lenduiridianmal.txt | cut -f3 -d '_' | sort | uniq
```
