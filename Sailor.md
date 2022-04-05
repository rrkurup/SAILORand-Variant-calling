#!/bin/bash
mkdir Sailor
# in genome directory
# copy genome file

cp GRCh37.primary_assembly.genome.fa /N/slate/rrkurup/hg19/Sailor
cp u87.snp.bed /N/slate/rrkurup/hg19/Sailor


# In aligned directory
# copy bam file

cp *.Aligned/sortedByCoord.out.bam /N/slate/rrkurup/hg19/Sailor 
cd /N/slate/rrkurup/hg19/Sailor 
module load samtools
samtools merge -@ 8 412-11-14.merged.sorted.rmdup.readfiltered.bam PolyA_412-11-14.merged.fwd.sorted.rmdup.readfiltered.bam PolyA_412-11-14.merged.rev.sorted.rmdup.readfiltered.bam
# in Desktop

scp sailor-1.0.4 rrkurup@carbonate.uits.iu.edu:

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

mv ce11_example.yaml 412-11-14.merged.yaml

# run sailor
#For small data
./sailor-1.0.4 name.yaml
#To submit job make a script file 412-11-14.merged.script with following commands
#!/bin/bash

#SBATCH -J 412.11.14.merged
#SBATCH -p general
#SBATCH -o 412.11.14.merged_%j.txt
#SBATCH -e 412.11.14.merged_%j.err
#SBATCH --mail-type=ALL
#SBATCH --mail-user=rrkurup@iu.edu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --cpus-per-task=1
#SBATCH --time=12:00:00

module load singularity

cd /N/slate/rrkurup/hg19/Sailor

srun -n 16 ./sailor-1.0.4 412-11-14.merged.yaml

To submit job sbatch 412-11-14.merged.script


#proceed to SAILOR annotation

