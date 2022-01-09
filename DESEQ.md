#Open RStudio on Thinlinc

library("tidyverse")

#Since BiocManager already available on Thinlinc RStudio, directly did the following. If not, write the following in separate lines.. 
# if (!requireNamespace("BiocManager", quietly = TRUE))
# install.packages("BiocManager")
 
BiocManager::install("DESeq2")

#Load DESeq2
library("DESeq2")

#Load your data
countdata <- read.table("/N/slate/rrkurup/PolyA/Results/counts/counts.tsv", header = TRUE, row.names = 1)

#To view the excel sheet on Rstudio
View(countdata)

#To remove all other columns except the ones containing the read counts i.e. after column 6. 
countdata <- countdata[,6:ncol(countdata)]

# After you type above command, you should see all the remaining columns disappear from the excel sheet that was visible after typing view(countdata). 
#This usually comes on the top of the screen and not on the console/termianl tab of RStudio
#Also, dont worry! The columns are  not removed from the original file. It's just removed from being analysed.

#To display names of all columns present in your file
colnames(countdata)

#To remove additional parts of the column names

colnames(countdata) <- gsub("..Results_GSF2887.aligned.GSF2887.Hundley.","", colnames(countdata))

colnames(countdata) <- gsub("_S1_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_S2_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_S3_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_S4_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_5_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_S5_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_S6_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_S7_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))
colnames(countdata) <- gsub("_S8_R1_001_Aligned.sortedByCoord.out.bam","", colnames(countdata))

colnames(countdata)

countdata <- as.matrix(countdata)

countdata <- countdata[,c("ADAR3_IP_Rep1","ADAR3_IP_Rep2","ADAR3_IP_Rep3","ADAR3_IP_Rep4","PreImmune_IP_Rep1","PreImmune_IP_Rep2","PreImmune_IP_Rep3","PreImmune_IP_Rep4")]

(condition <- factor(c(rep("b",4), rep("a",4))))

(coldata <- data.frame(row.names = colnames(countdata), condition))

dds <- DESeqDataSetFromMatrix( countData = countdata, colData = coldata, design=~condition)

#Deletes rows with 0 in all columns
keep = rowSums(counts(dds)) >=1

dds <- dds[keep,]

dds <- DESeq(dds)

res <- results(dds)

resdata <- merge(as.data.frame(res), as.data.frame(counts(dds,normalized=TRUE)), by="row.names", sort=FALSE)

names(resdata)[1] <- "Gene"

write.csv(resdata, file= "/N/slate/primukh/GSF2887/Results_GSF2887/U373_RIPSeq.csv")

#Secure copy files to your desktop
#Type this on your local desktop terminal

cd Desktop/U373_RIPSeq

scp primukh@carbonate.uits.iu.edu:/N/slate/primukh/GSF2887/Results_GSF2887/U373_RIPSeq.csv .

#For PCA Plot, proceed with following: 
library("ggplot2")

vsd <- vst(dds, blind=FALSE)

pcaData <- plotPCA(vsd, intgroup=c("condition"), returnData=TRUE)

percentVar <- round(100 * attr(pcaData, "percentVar"))

ggplot(pcaData, aes(PC1, PC2, color=condition)) +
  geom_point(size=3) +
  xlab(paste0("PC1: ",percentVar[1],"% variance")) +
  ylab(paste0("PC2: ",percentVar[2],"% variance")) + 
  coord_fixed()
