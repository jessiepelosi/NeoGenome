# Assembly and Annotation of <i>Neomusotima</i> Genome

### k-mer Spectrum and Genome Size Estimation

We used KMC to generate k-mer frequency spectra for k=21, 31, 41, 51, and 61. The k-mer histogram was read into GenomeScope2 and a custom R script to estimate genome size and heterozygosity. 

```
kmc -kXX -t16 -ci1 -cs1000 $reads XXmers
kmc_tools transform XXmers histogram XXmer.hist
```

### Nuclear Genome Assembly

We used hifiasm to assemble the nuclear genome. 

```
hifiasm -t 96 --hg-size 500m --n-hap 8 -o Neo_asm ../m84082_250722_081851_s1.hifi_reads.bc2074.fasta
awk '/^S/{print ">"$2;print $3}' Neo_asm.bp.p_ctg.gfa > Neo_asm.bp.p_ctg.fasta
```

To visualize the assembly in Bandage, we need to change the depth flag in the graph file. 
```
sed 's/rd:i:/dp:f:/g' Neo_asm.bp.p_ctg.gfa > Neo_asm.bp.p_ctg.bandage.gfa
```

We used purge_haplotigs to remove duplicated sequences, possibly representing un-purged haplotypic contigs, from the assembly. We also retained contigs that were at least 1Mb in length using seqkit.  

```
minimap2 -ax map-hifi -t 36 $ref $reads --secondary=no | samtools sort -m 1G -o Neo_k21.nhap8.reads.bam -T tmp.ali
purge_haplotigs hist -b Neo_k21.nhap8.reads.bam -g Neo_k21nhap8.bp.p_ctg.fasta -t 16 -d 500
purge_haplotigs cov -i Neo_k21.nhap8.reads.bam -l 8 -m 65 -h 300
purge_haplotigs purge -g Neo_k21nhap8.bp.p_ctg.fasta -c coverage_stats.csv -b Neo_k21.nhap8.reads.bam -I 1G
seqkit seq -m 1000000 curated.fasta > Neo_k21.nhap8.purgeHaps.l8m65h300.fasta
```

Assess completeness of the assembly with compleasm. 

```
compleasm run -a $asm -l lepidoptera_odb10 -o $asm.busco -t 8 --miniprot_execute_path ./miniprot/miniprot
```

Teleomeric repeats were identified with tidk using the Lepidoptera repeat AACCT.  

```
tidk find --clade Lepidoptera --output asm_tidk --dir . $asm 
tidk plot --tsv [asm_tidk_telomeric_repeat_windows.tsv]
```

### Nuclear Genome Annotation 

We first generated a species-specific repeat library and then masked repeats in the genome prior to generating gene model predictions. We used the dfam-tetools-latest singularity image.  

```
singularity exec dfam-tetools-latest.sif BuildDatabase -name Neomusotima $asm
singularity exec dfam-tetools-latest.sif RepeatModeler -database Neomusotima -pa 96 -LTRStruct
singularity exec dfam-tetools-latest.sif -pa 96 -norna -lib Neomusotima-families.fa -no_is -gff -a -xsmall $asm
```

We then used the BRAKER3 pipeline to generate gene model predictions using the soft-masked genome as input. 
```
singularity exec braker3.sif braker.pl --genome=$asm --species=Neo --prot_seq=$proteins --threads=8 \
  --AUGUSTUS_CONFIG_PATH=/blue/barbazuk/jessiepelosi/scripts/BUSCO_scripts/Genome/config \
  --GENEMARK_PATH=/blue/barbazuk/jessiepelosi/local_prgms/GeneMark-ETP/
```

The longest isoform for each gene was extracted from the annotation using AGAT. 
```
gtf2gff.pl <braker.gtf --out=braker.gff
agat_sp_keep_longest_isoform.pl -gff braker.gff -o braker.longest.gff
```

### Mitogenome Assembly and Annotation

We used MitoHiFi to first identify the closest reference mitogenome available on NCBI and then assemble and annotate the mitogenome for <i>Neomusotima</i> using this as a reference genome. 

```
mitohifi="/path/to/mitohifi.sif" 
singularity exec $mitohifi findMitoReference.py --species "Neomusotima conspurcatalis" --outfolder ./ --min_length 14000
singularity exec $mitohifi mitohifi.py -r m84082_250722_081851_s1.hifi_reads.bc2074.fasta -f PV255021.1.fasta -g PV255021.1.gb -t 8 -a animal 
```

There were several mitogenomes within Crambidae that we wanted to use but were not annotated. We used the MitoFinder annotation feature in MitoHiFi to annotate these genomes. 
```
singularity exec $mitohifi mitohifi.py -c [genome.fasta] -f PV255021.1.fasta -g PV255021.1.gb -t 4 -o 5
```

To extract the coding sequences from each of the annotated mitogenomes, we used GBSeqExtractor. 
```
gbextractor="/path/to/gbseqextractor.py"
python $gbextractor -f [genome.gb] -prefix [genome]
```

### Removal of Mitogenome and Contamination in Nuclear Assembly

Minimap2 was used to map contigs in the nuclear assembly to the mitogenome assembled above. These contigs were removed from the final assembly. 

```
minimap2 -ax asm5 Neo_v1.XXX.fasta final_mitogenome.fasta > ref_to_mt.sam 
```

