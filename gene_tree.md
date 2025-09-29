## Aligning sequences
Aligned sequences with macse (https://www.agap-ge2pop.org/macse/). Best run in array with SLURM script.
```
$java -jar macse_v2.07.jar -prog alignSequences -seq "$file".headers.cds
```

## Triming aligned sequences 
Replace ! or * with -.
```
sed 's/!/-/g' "$file"_NT.fa > "$file"_NT_clean.fa
sed 's/*/-/g' "$file"_NT.fa > "$file"_NT_clean.fa
```
Trimed aligned sequences to remove columns with less than 50% data. Run as SLURM script. 
```
$trimal -in "$file"_NT_clean.fa -out "$file"_NT_clean.fa.trim -gt 0.5 
```

## Generating trees 
Used iqtree3 (https://iqtree.github.io/) to generate phylogenies. Best run in array with SLURM script.
```
$iqtree3 -s $file -m MFP -B 1000 --alrt 1000 --redo
```

## Rename IDs
Renamed species IDs so all are consistent. 
```
#!/bin/sh 

while getopts e: flag
do
    case "${flag}" in
        e) ext=${OPTARG};;
    esac
done

for file in *.$ext; do
        sed -i -r 's/Parotis_xtbg_[0-9]*-RA/PACH/g' "$file"
        sed -i -r 's/MECY_[A-Za-z0-9\.]*/MEFL/g' "$file"
        sed -i -r 's/MUNI_[A-Za-z0-9\.]*/MUNI/g' "$file"
        sed -i -r 's/PAST_[A-Za-z0-9\.]*/PAST/g' "$file"
        sed -i -r 's/NECO_[A-Za-z0-9\.]*/NECO/g' "$file"
        sed -i -r 's/ELNY_[A-Za-z0-9\.]*/ELNY/g' "$file"
        sed -i -r 's/SCIN_[A-Za-z0-9\.]*/SCIN/g' "$file"
        sed -i -r 's/SCGI_[A-Za-z0-9\.]*/SCGI/g' "$file"
        sed -i -r 's/PATR_[A-Za-z0-9\.]*/PACH/g' "$file"
        sed -i -r 's/MVIT_[A-Za-z0-9\.]*/MVIT/g' "$file"
        sed -i -r 's/rna-gnl\|WGS_[A-Za-z]*_Lst[A-Za-z0-9\.]*/LOST/g' "$file"
        sed -i -r 's/OSNU_[A-Za-z0-9\.]*/OSNU/g' "$file"
        sed -i -r 's/AMTR_[A-Za-z0-9\.]*/AMTR/g' "$file"
        sed -i -r 's/GAME_[A-Za-z0-9\.]*/GAME/g' "$file"
        sed -i -r 's/HESA_[A-Za-z0-9\.]*/HESA/g' "$file"
        sed -i -r 's/Hvi[A-Za-z0-9\_\-]*/HEVI/g' "$file"
        sed -i -r 's/CHILSU_[A-Za-z0-9\.]*/CHILSU/g' "$file"
        sed -i -r 's/DIATSA_[A-Za-z0-9\.]*/DISA/g' "$file"
        sed -i -r 's/SCAM_[A-Za-z0-9\.]*/SCAM/g' "$file"
        sed -i -r 's/EULA_[A-Za-z0-9\.]*/EULA/g' "$file"
done
```

## Coalescence tree
Concatenated iqtree .treefiles
```
cat *.treefile > moth_trees
```
Used astral (https://github.com/chaoszhang/ASTER) to generate coalescence tree
```
$astral -i  moth_tree_sco100_cleaner.tre -t 4 -u 2
```


