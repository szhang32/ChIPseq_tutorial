# ChIPseq_tutorial

Date used in this tutorial come from:
  Yang, Jie, et al. "Exposure to high-sugar diet induces transgenerational changes in sweet sensitivity and feeding behavior via H3K27me3 reprogramming." Elife 12 (2023): e85365.

Introduction to ChIP-seq data analysis
Shuo Zhang
Penn Epigenetics Institute
xx/xx/2024

Description
This tutorial is designed to help you get familiar with ChIP-seq data analysis. We will describe methods to perform quality control for the raw fastq files, read mapping, and peak calling. We will also learn to create tracks for Integrative Genomics Viewer (IGV). Finally, we will create profile plots at specific genomic regions.

To learn these processes, we will use a dataset that compares H3K27me3 signals between normal diet and high-sugar diet in fruit fly:
	Yang, Jie, et al. "Exposure to high-sugar diet induces transgenerational changes in sweet sensitivity and feeding behavior via H3K27me3 reprogramming." Elife 12 (2023): e85365.


Prerequisites
•	A personal computer (Windows, MacOS, or Linux)
•	A HPC account



Download the dataset
After logging in HPC, you can ask for a private note and download the dataset from github





Step 1: quality control for fastq files

Preprocess fastq files. We will use fastp: https://github.com/OpenGene/fastp. To finish running the command in a reasonable time, only first 250K read pairs are used.




Fastp will perform quality control, remove bad reads, trim adapters etc. 


 
Step 2: alignment with bowtie2
First, we need to build an index for mapping. To save time, we will only build an index for chr2L of the dm6 reference genome. 
The chr2L.fa is stored in the ref directory. We will put index in the ref directory as well.




	two required parameters: 1. Reference 2. dir/basename

Then, we can map the reads to the reference.




	-q: reads are in FASTQ format
	--no-mixed: don’t find alignments for individual mates
	--no-unal: don’t output reads that fail to align
	--phred33: phred score + 33 encoding
	-x: basename of the index
	-1 and -2: reads
	-S: write output to SAM file

More information about bowtie2 parameters: https://bowtie-bio.sourceforge.net/bowtie2/manual.shtml



Step 3: filtering with samtools
We will use samtools to low-quality reads. 






More information about samtools: http://www.htslib.org/doc/samtools.html

We can also remove duplicates using picard (https://broadinstitute.github.io/picard/) and remove blacklist regions using bedtools (https://bedtools.readthedocs.io).




Step 4: call peak with MACS2
One essential step of ChIP-seq data analysis is to identify genomic regions that are enriched with signals of interest, such as histone modifications and transcription factors. MACS (Model-based Analysis of ChIP-seq) is a commonly used to call peaks for ChIP-seq data. We can first install MACS2 using conda, an environment and package manager.






We can run macs2:



	-t: treatment alignment file, can be in different format.
	-f: alignment file format, BAMPE is paired-end bam 
	-g: mappable genome size, there are recommended sizes for model organisms
	--broad: H3K27me3 has broad peaks
	--outdir: output directory
	-n: output name (prefix)

For more information about MACS: https://github.com/macs3-project/MACS

As too few reads (250K) are used, it is no surprise that no broad peak is identified. 

After finish calling peaks, we can exit the macs2 environment by:





Step 5: create an IGV track with deeptools
Visualizing ChIP-seq data is a great way to examine the data quality and find biological patterns. Here, we will create a track to be upload to IGV or genome browser. We can accomplish that using deeptools. To install the package, we create a new environment for it:




the bamCoverage function converts a .bam file to a .bigwig or .bedgraph file






	-b: bam file input
	-o: output file name
	--normalizeUsing: normalization method, Count Per Million (CPM)


Step 6: create profile plots with deeptools
For this task, we will plot H3K27me3 signal around regions that have higher H3K27me3 enrichment in high-sugar diet compared to control. Published data include
GSM6658146_ND_H3K27me3_1.bw: H3K27me3 in control replicate 1
GSM6658159_F1_H3K27me3_1.bw: H3K27me3 in high-sugar diet
chr2L_rep1_up_peaks.bed: up-regulated H3K27me3 regions

They are in the bw directory.

We first compute the signal:




	-S: bigwig files
	-R: regions 
	-a: distance after region end
	-b: distance before region start

Then, we plot the signal:




	-m: input matrix
	-o: output file name
	--perGroup: group samples for each region
	--plotTitle: title for the plot
	--colors: colors for each sample
	--startLabel: start label
	--endLabel: end label


For more information about deeptools: https://deeptools.readthedocs.io/en/develop/index.html
