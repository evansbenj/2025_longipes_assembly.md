# Plotting fem_specific kmerz in genomic windows

I'm going to generate a dbof fem specifci kmers by subtracting the iridian male from the nanopore female. Then...
* Make genomic windows:
```
cut -f1,2 longipes_hifiasm.bp.p_ctg.fa.fai > genome.sizes
WINDOW=100000
bedtools makewindows -g genome.sizes -w $WINDOW > windows.bed
bedtools getfasta -fi longipes_hifiasm.bp.p_ctg.fa -bed windows.bed -fo windows.fa
```
* Count the femlspecific kmerz in each window:
```
meryl-lookup \
    -exist \
    -sequence windows.fa \
    fem_specific_kmers.meryl
```
* summarize by window
```	
	awk '
	/^>/ {
	    win=substr($0,2)
	    next
	}
	{
	    counts[win]+=1
	}
	END{
	    for(i in counts)
	        print i,counts[i]
	}' lookup.out > window_counts.txt

```
