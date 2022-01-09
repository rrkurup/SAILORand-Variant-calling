genelist <- read.csv("/Users/priyankamukherjee/Desktop/U373_RIPSeq/U373_RIPSeq_Copy.csv", header = TRUE, row.names = 1)

colnames(genelist)

library("tidyr")
library("dplyr")
library("rlang")
library("magrittr")

ensembl <- useDataset("hsapiens_gene_ensembl", mart = useMart("ensembl"))

annotations <- getBM(attributes=c("ensembl_gene_id", "hgnc_symbol", "description", "gene_biotype"), filters = "ensembl_gene_id", values = genelist$Gene, mart = ensembl)

final_genelist <- inner_join(genelist, annotations, by = c("Gene"="ensembl_gene_id"))

write.csv(final_genelist,file="/Users/priyankamukherjee/Desktop/U373_RIPSeq/Annotated_U373_RIPSeq.csv")
