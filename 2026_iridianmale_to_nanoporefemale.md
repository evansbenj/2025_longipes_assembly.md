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

# Make this into intervals

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
