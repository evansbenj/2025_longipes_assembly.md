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
```
cut -d '   ' -f 2 perfect_matches.txt | sort | uniq > unique_female_specific_contigs.txt
```
