This workflow aims at aligning a list of gRNAs generated from all or selected target sequences
(e.g. promoter or coding sequences) in a genome to all or selected 
target sequences to identify gRNAs that have promiscuous targets.

## Prerequisites
The workflow was created in R 3.4.0 under Mac OS X 10.11.6 (x86_64-apple-darwin13.4.0 (64-bit))
using RMarkdown (rmarkdown_1.5 library). 
Besides RMarkdown, the following R libraries must be installed:
* reshape2_1.4.2                             
* ggplot2_2.2.1                              
* bindrcpp_0.1                               
* dplyr_0.7.0                                
* seqinr_3.3-6                               
* BSgenome.Scerevisiae.UCSC.sacCer3_1.4.0    
* TxDb.Scerevisiae.UCSC.sacCer3.sgdGene_3.2.2
* BSgenome_1.44.0                            
* rtracklayer_1.36.3                         
* Biostrings_2.44.1                          
* XVector_0.16.0                             
* GenomicFeatures_1.28.3                     
* AnnotationDbi_1.38.1                       
* Biobase_2.36.2                             
* GenomicRanges_1.28.3                       
* GenomeInfoDb_1.12.2                        
* IRanges_2.10.2                             
* S4Vectors_0.14.3                           
* BiocGenerics_0.22.0                        
* knitr_1.16   

## Running the workflow from an R environment (R library-dependent)
An example workflow can be directly run by sourcing main.Rmd in an R environment e.g. RStudio. 
Note that this requires a working installation of R Markdown and that dependent R libraries
are installed within the R enrivonment (e.g. RStudio). The user can change the default parameters
by editing the values at the beginning of the R Markdown document. See below for parameters' description.

## Running the workflow from command line (R library-independent)
The R Markdown script can be called with user-defined arguments from command line by calling main.R:

> R CMD BATCH --no-save --no-restore '--args nmismatches=1' main.R log.out

All dependent R libraries will be installed automatically using packrat (https://rstudio.github.io/packrat/).
This will produce a html report called main.html. Interim and final results will be saved in default folders unless
otherwise specified. Additional arguments can be specified in the command above. See below for parameters' description.

## Parameters
The following parameters can be set:
*  nmismatches : integer. Indicates maximum allowed number of mismatches when a gRNA is aligned
to the target region of each gene. (Default: 0)
*  chunk       : integer. gRNAs are split into a number of chunks indicated by nchunks and chunk identifies
the index of the chunk analyzed by the current script. Useful for parallelization (Default: 1)
*  nchunks     : integer. Number of chunks in which gRNAs are split. See chunk (Default: 1)
*  region      : character. Either "coding" or "promoter" or "chromosome". Indicates the gene region or entire chromosomes from which gRNAs are first
extracted and then aligned back. (Default: "coding")
*  toprank     : integer. Total number of promiscuous gRNAs returned as output (Default: 10)
*  maxgRNA     : integer. Maximum number of target per promiscuous gRNA (Default: 2*10^5)
*  interimdir  : character. Path to directory where interim files are stored (Default: "interim/")
*  minGC       : float. Minimum fraction of GC content in extracted gRNA (Default: 0.1)
*  selgenes    : comma-separated characters. List of gene names where gRNAs are extracted and aligned back. Must 
overlap with list of genes in the loaded genome. If NULL, uses all genes in genome. Ignored if region is "chromosome". (Default: 'YOR317W,YER015W,YIL009W,YMR246W,YMR008C,YMR006C,YOL011W').
*  genome      : character. Genome to be loaded. Must match abbreviations for makeTxDbFromUCSC function in R library 
GenomicFeatures 1.24.5. (Default: "sacCer3").

# src

This folder contains source files used by the workflow. The user can :

1. retrievepromoterseq.R: it retrieves all target region sequences in DNA. It outputs a list containing
a character vector of all target sequences and a character vector for the associated genes.
It accepts 3 args:
	1. region: a character. Either "promoter" or "coding" or "chromosome".
	2. filename (optional): a character. If not NULL, then saves output into filename.

2. retrievegRNAseq.R : it extracts all potential gRNAs given the PAM sequences in the target sequences.
It outputs a character vector containing all gRNAs. 
It accepts 7 args:
	1. promoters: a character vector of target sequences, as received from the first element of the list generated by retrievepromoterseq.R.
	2. genes:  a character vector of associated genes, as received from the second element of the list generated by retrievepromoterseq.R.
	3. promfile:  if promoters and genes are NULL, then it points to the path where the list generated by retrievepromoterseq.R is. Can be directly the arg "filename" to retrievepromoterseq.R
	4. gRNA.size (default=20): length of the gRNAs
	5. PAM.size (default=3): length of the PAM sequence
	6. minGC (default=0.1): fraction of minimum GC content in gRNAs
	7. filename (optional): a character. If not NULL, then saves output into filename.

3. aligngRNAtopromoters.R : it aligns all extracted gRNAs to the target sequences and scores the number of hits.
It outputs the number of hits per target sequence for each gRNAs. gRNAs list can be split into chunks for parallelization.
It accepts 8 args:
	1. promoters: a character vector of target sequences, as received from the first element of the list generated by retrievepromoterseq.R.
	2. gRNAs.ext: a character vector of extracted gRNAs, as generated by retrievegRNAseq.R.
	3. promfile: if promoters and genes are NULL, then it points to the path where the list generated by retrievepromoterseq.R is. Can be directly the arg "filename" to retrievepromoterseq.R
	4. gRNAfile: if promoters and gRNAs.ext are NULL, then it points to the path where the file generated by retrievegRNAseq.R is. Can be directly the arg "filename" to retrievegRNAseq.R
	5. nmismatches: maximum number of mismatches in the alignment
	6. nchunks (default=1): number of chunks in which the gRNAs.ext list is split
	7. chunk (default=1): chunk index analysed by the script
	8. filename (optional): a character. If not NULL, then saves output into filename.

4. analyzealignments.R : analyses the output of aligngRNAtopromoters.R.
It accepts 2 args:
	1. hitsindexes: a matrix of hits per target sequences, as generated by aligngRNAtopromoters.R.
	2. nmismatches: maximum number of mismatches in the alignment

