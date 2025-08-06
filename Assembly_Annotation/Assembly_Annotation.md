# Assembly and Annotation of <i>Neomusotima</i> Genome


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

