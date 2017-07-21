**General questions**

- [What are the main goals of the analysis?](#what-are-the-main-goals-of-the-analysis)
- [What is the mQTL analysis strategy?](#what-is-the-mqtl-analysis-strategy)
- [What does the pipeline do?](#what-does-the-pipeline-do)
- [Deadlines](#deadlines)
- [How long will it take to run the pipeline](#how-long-will-it-take-to-run-the-pipeline)
- [Using GitHub](#using-github)
- [Encountering problems and finding bugs](#encountering-problems-and-finding-bugs)
- [Proposing new analyses](#proposing-new-analyses)
- [Publication policy](#publication-policy)
- [Who is the lead person for the secondary analyses](#who-is-the-lead-person-for-the-secondary-analyses)

**Technical questions**

- [How does the pipeline handle twins and related individuals](#how-does-the-pipeline-handle-twins-and-related-individuals)
- [What age groups can be included](#what-age-groups-can-be-included)
- [Which populations can be included](#which-populations-can-be-included)
- [How should cases and controls be handled](#how-should-cases-and-controls-be-handled)
- [How should cord blood samples be handled](#how-should-cord-blood-samples-be-handled)
- [Which tissues can be included](#which-tissues-can-be-included)
- [How do you code INDELs](#how-do-you-code-indels)
- [What significance thresholds will be used in the meta-analysis](#what-significance-thresholds-will-be-used-in-the-meta-analysis)
- [Can we use combat variables](#can-we-use-combat-variables)
- [Are NAs allowed in the methylation matrix](#are-nas-allowed-in-the-methylation-matrix)
- [Are X chromosome SNPs required](#are-x-chromosome-snps-required)
- [Are X chromosome CpGs required](#are-x-chromosome-cpgs-required)
- [Which methylation arrays can be included](#which-methylation-arrays-can-be-included)
- [How to run the pipeline on two different datasets](#running-multiple-datasets-with-the-pipeline) 

* * *

## What are the main goals of the analysis?

The primary objective is to perform a cis and trans mQTL meta-analysis. By virtue of the fact that data needs to be harmonised, cleaned and formatted to perform this analysis (and this is likely the most time consuming part), we have tried to capitalise on this time investment by writing extra analysis modules that look at:

- Sex stratified mQTL analysis
- Variance mQTL analysis (testing the influence of SNPs on the variance of methylation levels)
- Structural variant mQTLs
- EWAS of BMI and height
- GWAS of epigenetic ageing
- GWAS of cell type composition
- GWAS of predicted smoking levels

The pipeline can be extended to run further analyses proposed by consortium members.


## What is the mQTL analysis strategy?

Performing an mQTL analysis of e.g. 450,000 CpGs against 10 million common genetic variants incurs two main computational problems:

1. Analysis time using, SNPTEST, PLINK, or a full linear mixed model is prohibitively slow for most cohorts
2. Storing the entire surface of results (`450000 x 10000000` associations) would require many terabytes of disk space, this is prohibitively large for most cohorts

To circumvent these problems the analysis is split into two **phases**. 

**Phase 1**: Each cohort uses a fast approximation method (`matrixeqtl`) to perform the full cis and trans scan, storing only results with `p < 1e-5`. These *putative SNP-CpG* associations are collected from each cohort, and a **candidate list** of all unique putative associations from all cohorts is generated. 

**Phase 2**: The **candidate list** is distributed to all cohorts, where a linear model is used to calculate the association for every putative association. The meta analysis is then performed on the results from this analysis. 

This approach should guard against publication bias. For example, if we only meta analysed associations from phase 1 then supposing one cohort has an association at `p = 1e-10`, but the association isn't returned at `p < 1e-5` in any other cohorts then the meta analysis will be potentially unreliable. This approach should also improve power without having to store the entire surface, for example if one cohort identifies an association at `p = 1e-6` and the association is at `p = 1e-3` in all other cohorts, then the second phase allows the meta analysis to identify such scenarios which lead to significant associations where otherwise there would have been none. (Note, just using p-values for simplistic illustrative purposes here, the meta analysis of course will be performed on effect sizes and standard errors).


## What does the pipeline do?

This analysis is unfortunately one of the most complicated meta analyses performed in the GWAS consortium framework, owing to the obstacles described above. The pipeline has three main purposes:

- Reduce the amount of time and effort analysts have to devote to perform the analyses
- Reduce the potential for heterogeneity being introduced into the analyses due to different historical choices made by cohorts
- Provide an expandable platform that allows further analyses to be added by members of the consortium with relative ease and then disseminated to the other centres.

The pipeline requires data in the following formats:

- Genotypes in best guess plink format data.
- Methylation as a beta matrix in `.RData` format
- Covariates and phenotypes as text files in a specified format

We have provided scripts and packages to help get the data into these formats. 

The analyst is required to create a `config` file which sets the parameters and file locations for the pipeline to use. Then each module can be run sequentially. At the end of each module the pipeline checks the outputs and uploads them to the server in Bristol. This means that any problems can be identified early on, and hopefully minimise having to ask analysts to go back and re-perform sections if there were issues.

Where necessary the modules can be parallelised across cores or across nodes on a cluster.


## Deadlines

We request that analysts finish up to module `05` in the pipeline by **13th May 2016**. This will complete phase 1 - sending each cohort's putative mQTLs to the Bristol server. However we must make two important points about this:

1. **We are very aware that this is not the primary work for most analysts** and we tried to set a deadline that reflected this, based on the experiences of having it already run in 4 different centres. However, we will continue to communicate with analysts about how realistic this deadline is and we may push it back if necessary.
2. It is not necessary for every cohort to perform phase 1. If a sufficiently large number of cohorts have completed phase 1 then we will be able to create a comprehensive **candidate list** of SNP-CpG associations for phase 2. So if you can't complete by this deadline then the cohort can still be involved by contributing to phase 2.

3. We would now like to invite you to contribute to the next phase of the analysis. We request that analysts finish 16 and 17 by **10th July 2017**. Module 16 will test candidate list of SNP-CpG associations from section 5. Module 17 will test a candidate list of SNP-CpG list using biologically interesting SNPs and CpGs. See [here](https://github.com/MRCIEU/godmc/issues/17) for the list.

## How long will it take to run the pipeline

This answer will vary from cohort to cohort, depending on

- How much work is required to get the data into the right format
- The sample size
- Computational resources

Based on our experience with 2000 samples and approximately 100 cores, starting from genetic data in `.bgen` format, and methylation data in `.idat` files:

| Task                                      | Analyst time | Compute time |
|-------------------------------------------|--------------|--------------|
| Create best guess data                    | 2 hours      | 4 hours      |
| Normalise methylation data using `meffil` | 2 hours      | 6 hours      |
| Generate structural variant data          | 1 hour       | 15 hours     |
| Setup pipeline                            | 1 hour       | 1 hour       |
| Run module 01*                            | 1 hour       | 1 hour       |
| Run module 02                             | 30 minutes   | 30 minutes   |
| Run module 03                             | 5 minutes    | 5 minutes    |
| Run module 04 (with unrelated samples     | 2 hours      | 15 hours     |
| Run module 04 (with related samples)      | 2 hours      | 25 hours     |
| Run module 05                             | 30 minutes   | 35 hours     |
| Run module 16                             | 20 minutes   | 10 hours     |
| Run module 17                             | 20 minutes   | 10 hours     |


*This step could potentially be more time consuming for the analyst - it is checking that the various datasets are in the correct format and won't complete until things have been fixed.

If you find these numbers are unrealistic please do let us know and we will update to provide more accurate estimates.


## Using GitHub

GitHub is a cloud based platform for hosting code repositories. We have opted to use it for GoDMC for the following reasons:

- It makes it easy to distribute code
- The web platform makes it easy to host the associated wiki
- If we identify any bugs we can fix the bug, and the fix can be synced across all cohorts easily (the analyst simply runs `git pull` and it will update the scripts to the latest version on the GitHub repository)
- We can use versioning to check that the correct code version is being run for each analysis


## Encountering problems and finding bugs

If you experience unexpected results or crashes from the pipeline please do let us know by emailing us at [ieu-godmc@bristol.ac.uk](mailto:ieu-godmc@bristol.ac.uk).


## Proposing new analyses

Because the pipeline harmonises the data from many cohorts into a single framework, it is our hope that this can be a resource that the community of researchers within the consortium can use to conduct further work involving genetic, methylation and phenotypic data. Members can propose new projects that fall under two different categories:

1. Using the results that are already generated by the pipeline from existing modules
2. Writing new modules to perform extended analyses

The executive committee has developed application procedures for both 1) and 2) including a publication policy for these proposals. You can find these [here](http://www.godmc.org.uk/projects.html)   

## Publication policy

There is a broad level publication policy written into the [Codes of conduct](http://www.godmc.org.uk/codesofconduct.pdf). You can find a more detailed publication policy [here](https://github.com/MRCIEU/godmc/files/487937/authorship160505_final.docx).


## Who is the lead person for the secondary analyses

The lead person for the EWAS and GWAS analyses is yet to be determined, if you have any queries about specific analyses please address them to [ieu-godmc@bristol.ac.uk](mailto:ieu-godmc@bristol.ac.uk).


## How does the pipeline handle twins and related individuals

In phase 1, if it is declared in the `config` file that the study design is based on family relationships (e.g. trios, families, twins etc) we are using the GRAMMAR approach ([Aulchenko et al 2007](http://www.genetics.org/content/177/1/577)) for the purposes of adjusting the methylation data for polygenic effects and then using `matrixeqtl` on the residuals. Phase 2 will then perform a linear mixed model to estimate SNP-CpG associations in the **candidate list** to account for relatedness.

For twin studies MZ and DZ twins can be included. If MZ twins only have genotype data for one of the pairs it is ok to duplicate the genotype data for the second pair.

For data that is declared to comprise only unrelated samples the data will check for any cryptic relatedness and remove samples accordingly.

More details are included in the analysis plan.


## What age groups can be included

Based on recent analysis ([Gaunt et al 2016](http://genomebiology.biomedcentral.com/articles/10.1186/s13059-016-0926-z)) the effects of SNP-CpG associations are highly stable across the life course, so we will be including all age groups in the main mQTL meta-analysis. 

For the secondary analyses (e.g. GWAS of cell counts, GWAS of predicted smoking, EWAS of height and BMI) the pipeline will automatically stratify samples into different age groups (combiend, below 18, above 18).


## Which populations can be included

We are initially focusing on samples with European ancestry. If you have samples that comprise other ancestries then please contact us as these samples could be valuable further down the line for making comparisons to Europeans.


## How should cases and controls be handled

Some cohorts will have collected data that ascertains for cases and controls for a particular phenotype. We request that the cases and controls be combined to analysed together as a single cohort in the pipeline.


## How should cord blood samples be handled

For cord blood samples please set age to 0, and use a cord blood reference for cell count prediction. There are three cord blood references implemented in meffil including the reference provided in [Bakulski et al](http://www.ncbi.nlm.nih.gov/pubmed/27019159). Note - **please do not use Houseman for cell count prediction on cord blood samples**. For cord blood samples you don't need to prepare phenotypes and you don't need to run script 03, 08 and 09.

Please also add maternal smoking as covariate in the analysis if this is available.


## Which tissues can be included

In phase 1 and 2 we are only focusing on whole blood samples (e.g. no buccal cells or other tissue types). If you have other tissue types please contact us as these could be valuable further down the line for comparing differences between tissues.


## How do you code INDELs

We recode INDELS to I and D following the easyQC convention. If you have indels coded like this in your genotype data then this is fine. If you have I/R/D then the pipeline will recode to I/D so you don't need to change this coding. The same is true if you have allele names - the pipeline will recode these automatically so you don't need to make adjustments here.


## What significance thresholds will be used in the meta-analysis

This is a question for which many different opinions exist, ranging from the most conservative (e.g. Bonferroni for the entire surface) to more liberal (e.g. FDR for cis and trans separately). 

Upon performing the meta analysis of the **candidate list** we will discuss with the consortium where in this spectrum we should be for reporting the numbers of "significant" mQTLs, but we will make all available tested associations publicly available without imposing a threshold.


## Can we use combat variables

The main issue with combat variables is they may regress out genetic and/or phenotypic variation of interest. The pipeline conducts principal components analysis and only regress out non-genetic PCs (and PCs not associated to the phenotypes). Therefore, **please don't include combat variables as covariates.**


## Are NAs allowed in the methylation matrix

Initially we wanted to avoid allowing NAs because these could mean different things in different cohorts, and sometimes can introduce bias due to non-random missingness. 

However, we have now relaxed this requirement for simplicity, and if your methylation matrix has NAs it then it should pass the data check.

Note that the pipeline does set NAs itself - it iteratively sets values to NA if they are >10 SD from the mean of the probe.

## Are X chromosome SNPs required

We have made the X chromosome optional as there are many groups who won't have this imputed yet, and the main imputation servers don't yet accommodate this. The pipeline does allow for X chromosome data though, and for some downstream analyses it could be a valuable resource (e.g. identifying sex differences).

## Are X chromosome CpGs required

The pipeline analyses X chromosome methylation data. In module 14 and 15 we examine chrX CpGs in females and males only. 

## Which methylation arrays can be included

We are focusing on Illumina HumanMethylation450 (HM450) BeadChip data. However, methylation data measured with MethylationEPIC (EPIC) BeadChips are also included. Meffil can deal with both 450k as EPIC data.

## How to run the pipeline on two different datasets

You need to download the repository separately for each dataset. In the config file you need to specify the cohortname. All results that will be uploaded will have the following format {cohortname}_01.tgz. So you can use the same sftp details to upload results from multiple cohorts.
