# Filter-VCF
A notebook demonstrating different ways to filter VCF files.

##### AUTHOR : Anuj Guruacharya
##### DATE: September 13, 2017
##### Performed on a machine with Ubunutu 14.04 LTS, Memory 62 GB, Intel® Xeon(R) CPU E5-2650 v3 @ 2.30GHz × 16
##### TASK : Obtain a VCF file and filter it based on some metric of your own choosing, and output the VCF file. 
##### TASK: DO NOT USE ANY PACKAGE THAT MANIPULATES VCF FILES DIRECTLY
##### TIME TAKEN FOR TASK: Approximately an hour
##### COPYRIGHT: Anuj Guruacharya
#####  Written for Intermountain Healthcare


#######################
####OBTAINING DATA
#####################
# to obtain the VCF file from NCBI 
wget ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp/pilot_data/release/2010_07/exon/snps/CEU.exon.2010_03.genotypes.vcf.gz
# to unzip the compressed VCF file
gunzip CEU.exon.2010_03.genotypes.vcf.gz

#######################
#METHOD 0 USING VCFTOOLS
######################
sudo apt-get install vcftools
vcftools --vcf CEU.exon.2010_03.genotypes.vcf  --minDP 10 --recode --out filter1
vcftools --vcf CEU.exon.2010_03.genotypes.vcf --freq --chr 1 --out filter2
vcftools --vcf CEU.exon.2010_03.genotypes.vcf --recode --out filter3 --min-meanDP 30

######################
###METHOD 1 USING AWK
#instructions for use: type the commands directly into the shell. 
#Note: Since awk does not require loading the VCF file onto memory, this is the quickest method compared to R or Python
#####################
#filter by chromosome number 2 
awk '$1 == 2  { print }' CEU.exon.2010_03.genotypes.vcf > awk_filterByChromosome.vcf
#filter by chromosome number 2 and all reference alleles that have T nucleotide
awk '$1 == 2 && $4 == "T" { print }' CEU.exon.2010_03.genotypes.vcf > awk_filterByChromosomeAndReference.vcf
# filter by chromosome number 2 and all which PASS
awk '$1 == 2 && $7 == "PASS" { print }' CEU.exon.2010_03.genotypes.vcf > awk_filterByChromosomeAndPASS.vcf

########################
#### METHOD 2 USING R
#instructions for use: type "R" on the command line and then copy paste the following code
#instructions for use: your output file will be in same directory as your home directory for R
######################
#importing the VCF file into R
CEU_exon_2010_03_genotypes <- read.delim("~/Anuj/ProjectIntermountain/CEU.exon.2010_03.genotypes.vcf", sep="\t", skip = 10,header=TRUE)
#filter by PASS
filterByPASS <- subset(CEU_exon_2010_03_genotypes, FILTER=="PASS")
#filter all SNPs that have T in their reference
filterByReference <- subset(CEU_exon_2010_03_genotypes, REF=="T")
# filter all SNPs that are obtained from chromosome 2
filterByChromosome <- subset(CEU_exon_2010_03_genotypes,X.CHROM==2)
# output the vcf file into the hard drive
write.table(filterByChromosome,file="R_filterByChromosome.vcf",sep="\t")
# output the vcf file into the hard drive
write.table(filterByPASS,file="R_filterByPass.vcf",sep="\t")
# output the vcf file into the hard drive
write.table(filterByReference,file="R_filterByReference.vcf",sep="\t")

#########################
###METHOD 3 USING PYTHON
#instructions for use: type "python" on the command line and then copy paste the following code
#instructions for use: your output file will be in same directory as your home directory for python
########################
#using the pandas package in python which is used for table manipulation
import pandas as pd
# reading in the VCF file into python
mytable=pd.read_table("/home/dnarules/Anuj/ProjectIntermountain/CEU.exon.2010_03.genotypes.vcf",skiprows=10)
# filter based on chromosome number 1
filtertable1=mytable[mytable['#CHROM'] ==1]
# filter based on reference allele T
filtertable2=mytable[mytable['REF'] == "T"]
# filter all SNPs which have PASS
filtertable3=mytable[mytable['FILTER'] == "PASS"]
# output the vcf file into the hard drive
filtertable1.to_csv('python_Filter_Chromosome_is_1.vcf', sep='\t', index=False)
# output the vcf file into the hard drive
filtertable2.to_csv('python_Filter_Reference_is_T.vcf', sep='\t', index=False)
# output the vcf file into the hard drive
filtertable3.to_csv('python_Filter_Filter_is_PASS.vcf', sep='\t', index=False)
