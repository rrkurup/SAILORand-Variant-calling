#!/bin/bash

cd /N/slate/rrkurup/PolyA


#unzip all your  fastq.gz files

gunzip -d *.gz

# Remember to activate your environment.

source activate rnaseq

# Unload system perl for compatability with entrez-direct.

module unload perl

# Assigning Variables.

ASSEMBLY='http://ftp.ensembl.org/pub/release-103/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz'

ANNOTATION='http://ftp.ensembl.org/pub/release-103/gtf/homo_sapiens/Homo_sapiens.GRCh38.103.gtf.gz'

################################
## Generate STAR Genome Index ##
################################

# Make a directory to store the genome files within your working directory.

mkdir -p genome

# Download and unpack the genome assembly.

curl $ASSEMBLY | gunzip > ./genome/assembly.fasta

# Download and unpack the genome annotation.

curl $ANNOTATION | gunzip > ./genome/annotation.gtf

# Create a directory to store the index.

mkdir -p genome/index

# Create the STAR genome index.
# --genomeSAindexNbases 12 was recommended by software.

STAR \
  --runThreadN 4 \
  --runMode genomeGenerate \
  --genomeDir genome/index \
  --genomeFastaFiles genome/assembly.fasta \
  --sjdbGTFfile genome/annotation.gtf \
  --genomeSAindexNbases 12
  
  ###########################
## Align Reads to Genome ##
###########################

# Create an output directory for aligned reads.

mkdir -p Results/aligned

# Align the reads.

FASTQ=$PolyA*

for FASTQ in ${FASTQ[@]}; do
  PREFIX=Results/aligned/$(basename $FASTQ .fastq)_
  STAR \
    --runThreadN 8 \
    --outFilterMultimapNmax 1 \
    --outFilterScoreMinOverLread .66 \
    --outFilterMismatchNmax 10 \
    --outFilterMismatchNoverLmax .3 \
    --runMode alignReads \
    --genomeDir genome/index \
    --readFilesIn $FASTQ \
    --outFileNamePrefix $PREFIX \
    --outSAMattributes All \
    --outSAMtype BAM SortedByCoordinate
done

# Indexing the BAM files.

BAMS=($(find ./Results/aligned -name "*\.bam"))

for BAM in ${BAMS[@]}; do
  samtools index $BAM
  done
  
  ####################
## Count Features ##
####################

# Create an output directory for read counts.

mkdir -p Results/counts

# Count reads.

BAMS=$(find ./Results/aligned -name "*\.bam")

featureCounts \
  -a genome/annotation.gtf \
  -o Results/counts/counts.tsv \
  -t gene \
  -g gene_id \
  --largestOverlap \
  --readExtension3 150 \
  --primary \
  -s 2 \
  -T 8 \
  ${BAMS}
  
  #copy files to local desktop in windows open command prompt
  scp rrkurup@carbonate.uits.iu.edu:/N/slate/rrkurup/PolyA/Results/counts/counts.tsv.summary .
  scp rrkurup@carbonate.uits.iu.edu:/N/slate/rrkurup/PolyA/Results/counts/counts.tsv .
