# Flye

```
#!/bin/sh
#SBATCH --job-name=long_flye
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=192
#SBATCH --time=166:00:00
#SBATCH --mem=512gb
#SBATCH --output=long_flye.%J.out
#SBATCH --error=long_flye.%J.err
#SBATCH --account=rrg-ben

module load StdEnv/2023 python/3.13.2

/home/ben/projects/rrg-ben/ben/2025_bin/Flye/bin/flye --nano-hq /home/ben/scratch/longipes_flye_assembly/run1.fq.gz /home/ben/scratch/longipes_flye_assembly/run2.fq.gz /home/ben/scratch/longipes_flye_assembly/run3.fq.gz /home/ben/scratch/longipes_flye_assembly/run4.fq.gz /home/ben/scratch/longipes_flye_assembly/run5.fq.gz --threads 40 --iterations 4 --resume --asm-coverage 20 --genome-size 10.2g --out-dir /home/ben/scratch/longipes_flye_assembly/longipes_flye_assembly_2
```
