#!/bin/bash

# in genome directory
# copy genome file

cp assembly.fasta /N/u/rrkurup/Carbonate

# In PolyA directory

mkdir Sailor

# In aligned directory
# create merged bam file

module load samtools

samtools merge -@ 8 ../../sailor/GSF2848-EE-N2.merged.bam GSF2848-EE-N2-rep1_S11_R1_001_Aligned.sortedByCoord.out.bam GSF2848-EE-N2-rep2_S12_R1_001_Aligned.sortedByCoord.out.bam GSF2848-EE-N2-rep3_S13_R1_001_Aligned.sortedByCoord.out.bam

samtools merge -@ 8 ../../sailor/GSF2848-EE-adr-2.merged.bam GSF2848-EE-adr-2-rep1_S14_R1_001.Aligned/sortedByCoord.out.bam GSF2848-EE-adr-2-rep2_S15_R1_001.Aligned/sortedByCoord.out.bam GSF2848-EE-adr-2-rep3_S16_R1_001.Aligned/sortedByCoord.out.bam

# in Carbonate (not in a directory)

cp /N/slate/emierdma/GSF2848/sailor/* .

# in Desktop

scp c.elegans.WS275.snps.sorted.bed emierdma@carbonate.uits.iu.edu:

scp sailor-1.0.4 emierdma@carbonate.uits.iu.edu:

# in Carbonate
# should have bam files, snps.sorted.bed file, assembly.fasta file
# load singularity module

module load singularity

chmod +x sailor-1.0.4

./sailor-1.0.4

# open .yaml file

vim ce11_example.yaml

i

# update filenames using arrow keys

# save and quit

Esc :wq

# rename .yaml file

mv ce11_example.yaml N2.yaml

# run sailor

./sailor-1.0.4 N2.yaml

#proceed to SAILOR annotation
