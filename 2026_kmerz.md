# Plotting fem_specific kmerz in genomic windows

I'm going to generate a db of fem specifci kmers by subtracting the iridian male from the nanopore female. Then...
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
```	perl
#!/usr/bin/env perl

# Usage
#meryl print female_kmers.meryl > female_kmers.txt
#perl count_kmers_per_record.pl female_kmers.txt genome_windows.fa > counts.tsv

use strict;
use warnings;

my $k = 29;

my ($kmerfile, $fastafile) = @ARGV;

die "Usage: $0 kmers.txt windows.fa\n"
    unless defined $fastafile;

# Load kmers into hash
my %km;

open(my $KF, '<', $kmerfile)
    or die "Cannot open $kmerfile: $!\n";

while (<$KF>) {
    chomp;
    next unless /\S/;
    my ($mer) = split;
    $mer = uc($mer);
    $km{$mer} = 1;
}
close($KF);


sub canonical {
    my ($mer) = @_;
    my $rc = reverse($mer);
    $rc =~ tr/ACGT/TGCA/;
    return ($mer lt $rc) ? $mer : $rc;
}

sub process_seq {
    my ($id, $seq, $k, $kmref) = @_;
    return unless defined $id;
    my $len = length($seq);
    my $sum = 0;
    for (my $i = 0; $i <= $len - $k; $i++) {
        my $mer = substr($seq, $i, $k);
        next if $mer =~ /[^ACGT]/;
        $mer = canonical($mer);
        $sum++ if exists $kmref->{$mer};
    }
    print "$id\t$sum\n";
}

open(my $FA, '<', $fastafile)
    or die "Cannot open $fastafile: $!\n";
my $id;
my $seq = '';

while (<$FA>) {
    chomp;
    if (/^>(.*)/) {
        process_seq($id, $seq, $k, \%km);
        $id  = $1;
        $seq = '';
    }
    else {
        $seq .= uc($_);
    }
}

process_seq($id, $seq, $k, \%km);
close($FA);
```
