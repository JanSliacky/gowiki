---
title: "Example"
date: 2023-03-15T21:12:44+01:00
draft: false
---

# Get BiocManager
if (!"BiocManager" %in% rownames(utils::installed.packages()))
  install.packages("BiocManager")

# Get Bioc libs
BiocManager::install(pkgs = c("IRanges", "GenomicRanges", "BSgenome", "GenomeInfoDb"))

BiocManager::install(pkgs = "BSgenome.Hsapiens.UCSC.hg19")

# Required libs
library(dplyr)
library(reshape2)
library(tidyverse)
library(magrittr)
library(kableExtra)
library(ggplot2)
library(gridExtra)
library(BSgenome.Hsapiens.UCSC.hg19)

# Load mutSignatures
library(mutSignatures)

# prep hg19
hg19 <- BSgenome.Hsapiens.UCSC.hg19

# load data
install.packages("vcfR")
library(vcfR)
vcfexample <- read.vcfR("C://Users/sliacky/Downloads/P22.fc876eec-10b6.WGS_entirely.raw.vcf")

#convert vcf into data.frame
# https://gist.github.com/sephraim/3d352ba4893df07a2c35d8f227ab17ac
vcfexample.df <- cbind(as.data.frame(getFIX(vcfexample), INFO2df(vcfexample)))
# vcfexample.df$CHROM <- mapvalues(vcfexample.df$CHROM, from = chromosomes_new, to = chromsomes)
# https://stackoverflow.com/questions/20565949/replace-values-in-data-frame-with-other-values-according-to-a-rule


# Import data (VCF-like format)
z <- mutSigData$inputC

# Filter non SNV
x <- filterSNV(dataSet = vcfexample.df,  seq_colNames = c("REF", "ALT"))

# Visualize head
head(x) %>% kable() %>% kable_styling(bootstrap_options = "striped")

# Attach context
x <- attachContext(mutData = x,
                   chr_colName = "CHROM",
                   start_colName = "POS",
                   end_colName = "POS",
                   nucl_contextN = 3,
                   BSGenomeDb = hg19)

# Visualize head
head(x) %>% kable() %>% kable_styling(bootstrap_options = "striped")

# Remove mismatches
x <- removeMismatchMut(mutData = x,                  # input data.frame
                       refMut_colName = "REF",       # column name for ref base
                       context_colName = "context",  # column name for context
                       refMut_format = "N")
# Compute mutType
x <- attachMutType(mutData = x,                      # as above
                   ref_colName = "REF",              # column name for ref base
                   var_colName = "ALT",              # column name for mut base
                   context_colName = "context")
# Visualize head
head(x) %>% kable() %>% kable_styling(bootstrap_options = "striped")

# Count
blca.counts <- countMutTypes(mutTable = x,
                             mutType_colName = "mutType",
                             sample_colName = "SAMPLEID")
# Mutation Counts
print(blca.counts)

# how many signatures should we extract?
num.sign <- 5

# Define parameters for the non-negative matrix factorization procedure.
# you should parallelize if possible
blca.params <-
  mutSignatures::setMutClusterParams(
    num_processesToExtract = num.sign,    # num signatures to extract
    num_totIterations = 20,               # bootstrapping: usually 500-1000
    num_parallelCores = 4)                # total num of cores to use (parallelization)
# Extract new signatures - may take a while
blca.analysis <-
  decipherMutationalProcesses(input = blca.counts,
                              params = blca.params)

# Retrieve signatures (results)
blca.sig <- blca.analysis$Results$signatures

# Retrieve exposures (results)
blca.exp <- blca.analysis$Results$exposures

# Plot signature 1 (standard barplot, you can pass extra args such as ylim)
msigPlot(blca.sig, signature = 4, ylim = c(0, 0.10))

# Plot exposures (ggplot2 object, you can customize as any other ggplot2 object)
msigPlot(blca.exp, main = "BLCA samples") +
  scale_fill_manual(values = c("#1f78b4", "#cab2d6", "#ff7f00", "#a6cee3"))

# Export Signatures as data.frame
xprt <- coerceObj(x = blca.sig, to = "data.frame")
head(xprt) %>% kable() %>% kable_styling(bootstrap_options = "striped")

# Get signatures from data (imported as data.frame)
# and then convert it to mutSignatures object
cosmixSigs <- mutSigData$blcaSIGS %>%
  dplyr::select(starts_with("COSMIC")) %>%
  as.mutation.signatures()

blcaKnwnSigs <- mutSigData$blcaSIGS %>%
  dplyr::select(starts_with("BLCA")) %>%
  as.mutation.signatures()

# Compare de-novo signatures with selected COSMIC signatures
msig1 <- matchSignatures(mutSign = blca.sig, reference = cosmixSigs,
                         threshold = 0.45, plot = TRUE)
msig2 <- matchSignatures(mutSign = blca.sig, reference = blcaKnwnSigs,
                         threshold = 0.45, plot = TRUE)
# Visualize match
# signature 1 is similar to COSMIC ;
# signatures 2 and 3 are similar to COSMIC
# Here, we should probably extract only 2 mutational signatures
hm1 <- msig1$plot + ggtitle("Match to COSMIC signs.")
hm2 <- msig2$plot + ggtitle("Match to known BLCA signs.")

# Show
grid.arrange(hm1, hm2, ncol = 2)
