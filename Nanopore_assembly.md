# Nanopore assembly

We have ~20X coverage for longipes. Here is advice we got from Jill Wegrzyn at UConn:
```
I would avoid Flye for this case and move toward hifiasm, which allows you to specify ploidy directly. Before running the assembly, confirm that you have SUP base calls (rather than HAC) and, if needed, re-basecall with Dorado to achieve SUP quality. This should bring your read error rates close to HiFi-equivalent levels.
 
It is worth checking your read length distribution (N50) beforehand, as this will give a good indication of how well the assembly will perform at lower coverage. After read QC, you can use a tool such as Centrifuge to identify and remove any potential contaminant reads prior to assembly.
 
When running hifiasm, make sure to provide the estimated ploidy as a parameter. Given the genome size (~11 Gb), I expect the assembly should assemble with this tool with 512 Gb of RAM.

Following assembly, you may be able to use the relatives for scaffolding with RagTag - it's unlikely that you can phase the genome at this coverage but you may be able to get to a chromosome structure.
```

To figure out kit information, I needed to use a python wheel called `pod5`:

```
module load arrow/14.0.1 python/3.11
virtualenv --no-download --clear ~/ENV && source ~/ENV/bin/activate
pip install --no-index --upgrade pip
pip install --no-index pod5
pip freeze --local > pod5-reqs.txt
deactivate 
rm -r ~/ENV
```
