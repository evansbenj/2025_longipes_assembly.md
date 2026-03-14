# Make blast db

Here is a script to make a blast db out of a compressed assembly:
```
#!/bin/sh
#SBATCH --job-name=makeblastdb
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=6:00:00
#SBATCH --mem=2gb
#SBATCH --output=makeblastdb.%J.out
#SBATCH --error=makeblastdb.%J.err
#SBATCH --account=rrg-ben


module load StdEnv/2023 gcc/12.3 blast+/2.14.1 

# makeblastdb -in ${1} -dbtype nucl -out ${1}_blastable
gunzip -c ${1} | makeblastdb -in - -dbtype nucl -title ${1}_blastable -out ${1}_blastable
```

# Blast a multifasta query against this db

Here is a script to blast a multifasta query against a blast db:
```
#!/bin/sh
#SBATCH --job-name=makeblastdb
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=6:00:00
#SBATCH --mem=2gb
#SBATCH --output=makeblastdb.%J.out
#SBATCH --error=makeblastdb.%J.err
#SBATCH --account=rrg-ben


module load StdEnv/2023 gcc/12.3 blast+/2.14.1 

blastn -query ${1} -db ${2} -outfmt 6 -out ${1}_to_${2}
```

# Identify perfect matches
```
grep '100.000' long_significant_markers_0.0001.fasta_to_assembly.fasta.gz_blastable > perfect_matches.txt
```
# Now figure out what unique contigs they are on:

The -c argument on the uniq command provides the count (the number of time each contig has a perfect match)
```
cut -d '   ' -f 2 perfect_matches.txt | sort | uniq -c > unique_female_specific_contigs.txt
```

# Annotating contigs

We can use XL CDS to annotate contigs in allo. Using bedtools, I made a fasta file of all CDS from XL here:
```
/home/ben/projects/rrg-ben/ben/2021_XL_v10_refgenome/XL_CDS_only.fasta
```

And I can use this as a query to blast individual contigs. First extract the contig with bedtools:
```
bedtools getfasta -fi allo.fasta.contigs_nobubbles.fasta.gz -bed tig00008587_1_10920835.bed -fo tig00008587_1_10920835.fasta
```

Now I need to make a blast db of the contig of interest, e.g.:
```
makeblastdb -in tig00008587_1_10920835.fasta -dbtype nucl -out tig00008587_1_10920835.fasta_blastable
```

And now blast all coding regiosn as a query against the contig db:
```
blastn -query ../../2021_XL_v10_refgenome/XL_CDS_only.fasta -db tig00008587_1_10920835.fasta_blastable -outfmt 6 -out XL_CDS_to_tig00008587_1_10920835_annotation_out.txt
```

This produces a file that can be downloaded, opened with excel, and sorted by column 9 (sstart, "I") and column 3 (pident "C"). 

# 2026 

I am trying this again using RADsex tags that have P<0.005 and that are female biased. These all have at least 17 females and four or less males.

I blasted these against the longipes flye genome. 24 hit 5 or less contigs; 14 hit 19 or many more contigs.

When I focused on tags that had mostly unique hits (matches to 5 or less contigs), most of the hits were to different contigs (67 in total). There were three contigs that had 2 hits from mostly unique tags: contig_175829, contig_35104, and contig_47634. No contig had 3 hits from mostly unique tags 

```
/home/ben/projects/rrg-ben/ben/2025_allo_PacBio_assembly/Adam_boum_genome_assembly/XL_CDS_only_nospaces.fasta
```

