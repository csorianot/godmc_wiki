# 1000G imputation protocol
### Based on GIANT/DIAGRAM/MAGIC analysis plan

### 1.1 1000G Imputation reference haplotype panel
Please use haplotypes excluding monomorphic and singleton sites for chromosomes 1-22. from the phase one integrated variant release v3, to enable the best overall imputation quality. Cohorts with overlapping GWAS should contact us before proceeding.

### 1.2 Initial sample and variant quality control
Each study will be responsible for their sample and variant quality control and should apply their own filters as appropriate. Typically, samples with low call rate, extreme heterozygosity, gender mismatch with chromosome X variants, duplicates, first or second degree relatives (unless by design), or outlying ethnic ancestry are removed. After sample quality, perform SNV qulatiy control by using exclusion cut-offs for SNPs call rate, HWE and MAF. The choice of MAF cut-point should reflect the genotyping accuracy of lower frequency SNPs.

A good source describing data quality assessment and control can be found in [Anderson et al] (http://www.ncbi.nlm.nih.gov/pubmed/21085122). 

### 1.3 Convert your genotype data to NCBI Build 37 (hg19).
Current releases of 1000G Project Data use NCBI build37 (hg19) and, before you start imputation you need to ensure that all your genotypes are reported using build37 coordinates and on the forward strand of the reference genome (see 1.4 below). Additional information on how to convert data from earlier genome builds to build 37 can be found on the [liftover] (http://genome.sph.umich.edu/wiki/LiftOver) page.

### 1.4 Align your SNP alleles to the forward (“+”) strand of the reference genome. 
All of the 1000 Genomes reference panels have had their SNP alleles annotated to the + strand of the human genome reference sequence and so genotype data from study samples will also need alleles expressed relative to the + strand.

The following link contains [strand] (http://www.well.ox.ac.uk/~wrayner/strand/) files for the common genotyping chips. 

[GTOOL] (http://www.well.ox.ac.uk/~cfreeman/software/gwas/gtool.html) has options to re-orient genotype data (.GEN format) according to a strand file.

# 2. Imputation strategy

## 2.1 Follow the minimac and IMPUTE protocols for imputation.

IMPUTE2: use standard settings with pre-calculated build 37 recombination maps
([IMPUTE2 1000G cookbook] (http://genome.sph.umich.edu/wiki/IMPUTE2:_1000_Genomes_Imputation_Cookbook))

Minimac: use standard settings which calculate recombination maps on the fly ([Minimac 1000G cookbook] (http://genome.sph.umich.edu/wiki/Minimac:_1000_Genomes_Imputation_Cookbook))

## 2.2 Use the pre-made 1000G reference haplotype panels.
You can find reference panels for [Minimac] (https://mathgen.stats.ox.ac.uk/impute/1000GP%20Phase%203%20haplotypes%206%20October%202014.html) and for [IMPUTE2](https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.html).

## 2.3 Imputation by genome chunks
Imputations by genome chunks is standard in IMPUTE2. As there are considerable time savings, imputations by genome chunks should also be used for Minimac imputation (2500 marker chunks, with 500 marker overhang on ech side of the chunk). See section on Further time savings in minimac web protocol.

## 2.4 Logfiles
Check logfiles from chromosome 20 after running if available. For example, SNP identifier mapping problems between target and reference panel, lift over and parameter setting problems.
















