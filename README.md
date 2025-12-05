# Hexenal_Isomerases_Viridiplantae
Scripts used in Homolog identification and phylogenetic analysis of hexenal isomerases in the green lineage (Viridiplantae)

## 1. Homology search of cupin-domain (PF07883) containing proteins

````bash
wget http://pfam.xfam.org/family/PF07883/hmm -O PF07883.hmm

hmmpress PF07883.hmm

hmmsearch --cpu 8 -E 1e-5 \
  --domtblout PF07883.domtblout \
  PF07883.hmm Viridiplantae.fasta > PF07883.hmmsearch.out

grep -v "^#" PF07883.domtblout | awk '{print $1}' | sort -u > PF07883.protein_ids.txt

seqkit grep -f PF07883.protein_ids.txt Viridiplantae.fasta > CUPIN.fasta

cat CUPIN.fasta HI_all_ref.fasta > CUPIN_all.fasta
````

## 2. Cupin-domain containing proteins phylogenetic analysis

````bash

mafft --auto CUPIN_all.fasta > CUPIN_all.aln.fasta

trimal -in CUPIN_all.aln.fasta -out CUPIN_all_gene.fasta -gappyou

iqtree -s CUPIN_all_gene.fasta -m TEST -n AUTO
````

## 3. Domain screening of potential HI orthologs

````bash
wget -q "https://cloud.uni-konstanz.de/index.php/s/MMzHFEBjZLrpmN7/download" -O "Pf_Sm"
gunzip Pf_Sm.gz
hmmpress Pf_Sm


hmmscan 
  --domtblout HI_orthologs.domtblout \
  Pf_Sam.hmm \
  pHi.fasta > HI_orthologs.hmmscan.out

grep -v "^#" HI_orthologs.domtblout | \
    awk '{print $1"\t"$4"\t"$7"\t"$13}' \
    > HI_orthologs.domains.tsv
````

## 4. Potential HI orthologs phylogenetic analysis

````bash
cat pHi.fasta HI_all_ref.fasta > pHi_all.fasta

mafft --auto pHi_all.fasta > pHi_all.aln.fasta

trimal -in pHi_all.aln.fasta -out pHi_all_gene.fasta -gappyou

iqtree -s pHi_all_gene.fasta -m TEST --bb 1000 -n AUTO

raxml-ng --all \
  --msa pHi_all_gene.fasta \
  --model LG+G4 \
  --bs-trees 100

git clone https://github.com/ObornikLEP/treecombine.git

python treecombine.py -l pHi_all_gene.fasta -m pHi_iq.tree -s pHi_rax.tree -o pHi_merged.tree


````




