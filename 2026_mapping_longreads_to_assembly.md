# Index genome for minimap2

```
#!/bin/sh
#SBATCH --job-name=minimap2
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=4:00:00
#SBATCH --mem=16gb
#SBATCH --output=minimap2.%J.out
#SBATCH --error=minimap2.%J.err
#SBATCH --account=rrg-ben

# sbatch 2024_minimap2.sh ref.fasta query.fasta

module load StdEnv/2023 minimap2/2.26
minimap2 -d ${1}.mmi ${1}.fasta
```
# Now map the longread data
```
#!/bin/sh
#SBATCH --job-name=minimap2
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=144:00:00
#SBATCH --mem=128gb
#SBATCH --output=minimap2.%J.out
#SBATCH --error=minimap2.%J.err
#SBATCH --account=rrg-ben

# sbatch 2024_minimap2.sh ref.fasta query.fasta

module load StdEnv/2023 minimap2/2.26
#minimap2 -x asm10 --secondary=no -t8 ${1} ${2} > ${2}_alignments.paf

#minimap2 -ax lr:hq -t 12 reference.fa reads.fastq > alignment.sam
minimap2 -ax lr:hq -t 12 ${1} ${2} > ${3}_alignment.sam
```
