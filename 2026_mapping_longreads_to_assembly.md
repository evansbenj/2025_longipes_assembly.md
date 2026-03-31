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
