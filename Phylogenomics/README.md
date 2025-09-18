# Phylogenomics of Crambidae 

### Mitogenome Phylogenetics 

Starting with genes in their own file, we first used MAFFT to align the genes and then made a concatenated matrix. Given that the mitochondria is inherited uniparentally and acts as a single locus, we did not pursue using a coalescent approach for this organelle. 

```
mafft --maxiterate 1000 --localpair --adjustdirectionaccurately [gene] > [gene].aln
concat="/path/to/concat_fasta.pl" 
perl $concat --suffix .aln --outfile mt_concat > mt_partitions.txt
sed -i ‘s/^/DNA, /g’ mt_partitions.txt
tail -n +2 mt_partitions.txt > mt_partitions2.txt
```

We used a maximum-likelihood approach to generate a phylogeny from the concatenated matrix with IQTREE2. 
```
iqtree2 -s mt_concat.fasta -p mt_partitions2.txt -m MFP -B 1000
```

### Nuclear Phylogenomics




### Synteny 

Preperation of data for GENESPACE. 

```
grep "\sgene" [file.gff3] | cut -f 1,4,5,9 > [file.bed]
```
