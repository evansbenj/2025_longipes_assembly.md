# Longipes RNAseq

We haev RNAseq data from 19 individuals

location of the trimmed RNAseq data:
```
/home/ben/projects/rrg-ben/ben/2025_longipes/2026_longipes_RNAseq/trimmed
```

location of the nanopore flye genome:
```
/home/ben/projects/rrg-ben/ben/2025_longipes/flye_assembly
```

# Index the nanopore flye genome:
```
#!/bin/sh
#SBATCH --job-name=star_index
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=24:00:00
#SBATCH --mem=256gb
#SBATCH --output=star_index.%J.out
#SBATCH --error=star_index.%J.err
#SBATCH --account=rrg-ben

module load star/2.7.11b

STAR --runMode genomeGenerate --genomeDir /home/ben/projects/rrg-ben/ben/2025_longipes/flye_assembly --genomeFastaFiles /home/ben/projects/rrg-ben/ben/2025_longipes/flye_assembly/assembly.fasta --runThreadN 8 --limitGenomeGenerateRAM=124544990592
```
