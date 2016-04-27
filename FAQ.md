# FAQ

- [What are the main goals of the analysis?](https://github.com/MRCIEU/godmc/wiki/FAQ#what-are-the-main-goals-of-the-analysis)
- [What is the mQTL analysis strategy?](https://github.com/MRCIEU/godmc/wiki/FAQ#what-is-the-mQTL-analysis-strategy)
- [What does the pipeline do?](https://github.com/MRCIEU/godmc/wiki/FAQ#what-does-the-pipeline-do)
- [Deadlines](https://github.com/MRCIEU/godmc/wiki/FAQ#deadlines)
- [How long will it take to run the pipeline](https://github.com/MRCIEU/godmc/wiki/FAQ#how-long-will-it-take-to-run-the-pipeline)
- [Using GitHub](https://github.com/MRCIEU/godmc/wiki/FAQ#using-GitHub)
- [Encountering problems and finding bugs](https://github.com/MRCIEU/godmc/wiki/FAQ#encountering-problems-and-finding-bugs)
- [Publication policy](https://github.com/MRCIEU/godmc/wiki/FAQ#publication-policy)
- [Proposing new analyses](https://github.com/MRCIEU/godmc/wiki/FAQ#proposing-new-analyses)
- [How does the pipeline handle twins and related individuals](https://github.com/MRCIEU/godmc/wiki/FAQ#how-does-the-pipeline-handle-twins-and-related-individuals)
- [What age groups can be included](https://github.com/MRCIEU/godmc/wiki/FAQ#what-age-groups-can-be-included)
- [Which populations can be included](https://github.com/MRCIEU/godmc/wiki/FAQ#which-populations-can-be-included)
- [How should cases and controls be handled](https://github.com/MRCIEU/godmc/wiki/FAQ#how-should-cases-and-controls-be-handled)

## What are the main goals of the analysis?

The primary objective is to perform a cis and trans mQTL meta-analysis. By virtue of the fact that data needs to be harmonised, cleaned and formatted to perform this analysis (and this is likely the most time consuming part), we will capitalise on this time investment by writing extra analysis modules that look at:

- EWAS of BMI and height
- GWAS of epigenetic ageing
- GWAS of cell type composition

The pipeline can be extended to run further analyses proposed by consortium members.


## What is the mQTL analysis strategy?

Performing an mQTL analysis of e.g. 450,000 CpGs against 10 million common genetic variants incurs two main computational problems:

1. Analysis time using, SNPTEST, PLINK, or a full linear mixed model is prohibitively slow for most cohorts
2. Storing the entire surface of results (`450000 x 10000000` associations) would require many terabytes of disk space, this is prohibitively large for most cohorts

To circumvent these problems the analysis is split into two **phases**. 

**Phase 1**: Each cohort uses a fast approximation method (`matrixeqtl`) to perform the full cis and trans scan, storing only results with `p < 1e-5`. These *putative SNP-CpG* associations are collected from each cohort, and a **candidate list** of all unique putative associations from all cohorts is generated. 

**Phase 2**: The **candidate list** is distributed to all cohorts, where a linear mixed model is used to calculate the association for every putative association. The meta analysis is then performed on the results from this analysis. 

This approach should guard against publication bias. For example, if we only meta analysed associations from phase 1 then supposing one cohort has an association at `p = 1e-10`, but the association isn't returned at `p < 1e-5` in any other cohorts then the meta analysis will be potentially unreliable. This approach should also improve power without having to store the entire surface, for example if one cohort identifies an association at `p = 1e-6` and the association is at `p = 1e-3` in all other cohorts, then the second phase allows the meta analysis to identify such scenarios which lead to significant associations where otherwise there would have been none. (Note, just using p-values for simplistic illustrative purposes here, the meta analysis of course will be performed on effect sizes and standard errors).


## What does the pipeline do?

This analysis is unfortunately one of the most complicated meta analyses performed in the GWAS consortium framework, owing to the obstacles described above. The pipeline has three main purposes:

- Reduce the amount of time and efford analysts have to devote to perform the analyses
- Reduce the potential for heterogeneity being introduced into the analyses due to different historical choices made by cohorts
- Provide an expandable platform that allows further analyses to be added with relative ease and then disseminated to the consortium. We hope this will 'democratise' the analysis somewhat.

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


## How long will it take to run the pipeline

This answer will vary from cohort to cohort, depending on

- How much work is required to get the data into the right format
- The sample size
- How large the available computational resources are

Based on our experience with 2000 samples and approximately 100 cores, starting from genetic data in `.bgen` format, and methylation data in `.idat` files:

| Task                                      | Analyst time | Compute time |
|-------------------------------------------|--------------|--------------|
| Create best guess data                    | 2 hours      | 2 hours      |
| Normalise methylation data using `meffil` | 2 hours      | 6 hours      |
| Generate structural variant data          | 1 hour       | 17 hours     |
| Setup pipeline                            | 1 hour       | 1 hour       |
| Run module 01                             | 1 hour       | 1 hour       |
| Run module 02                             | 30 minutes   | 20 minutes   |
| Run module 03                             | 5 minutes    | 5 minutes    |
| Run module 04                             | 1 hour       | 10 hours     |
| Run module 05                             | 30 minutes   | 10 hours     |

If you find these numbers are unrealistic please do let us know and we will update to provide more accurate estimates.


## Using GitHub

GitHub is a cloud based platform for hosting code repositories. We have opted to use it for GoDMC for the following reasons:

- It makes it easy to distribute code
- The web platform makes it easy to host the associated wiki
- If we identify any bugs we can fix the bug, and the fix can be synced across all cohorts easily (the analyst simply runs `git pull` and it will update the scripts to the latest version on the GitHub repository)
- We can use versioning to check that the correct code version is being run for each analysis


## Encountering problems and finding bugs

If you experience unexpected results or crashes from the pipeline please do let us know by emailing us at [ieu-godmc@bristol.ac.uk](mailto:ieu-godmc@bristol.ac.uk).


## Publication policy

The scope and structure of the publications that will arise from these analyses are not yet finalised, and as such making a concrete publication policy at this stage is not possible. However, based on the examples of other large consortia we will abide by the guidelines below for the primary paper(s).

- Each study contributing GWAS data can name 4 authors (including 1 analyst and 1 PI). This can of course be increased if there is a clear case for including extra authors for a particular study.
- Authors will be ordered based on contributions, which will be determined by assigning authors to each of the following tiers:

    1. Central analysis and writing group (junior)
    2. Study-speciflc lead analyst
    3. Study-specific sample collection and data generation (phenotyping/genotyping/supporting analysis)
    4. Study PIs + study design
    5. Central analysis and writing group (senior)

    The studies are ordered according to their sample size, from largest to smallest (tier 2) or smallest to largest (tiers 4, 6). Where there are multiple authors from one study in the same tier, authors are put in alphabetical order.


## Proposing new analyses

Because the pipeline harmonises the data from many cohorts into a single framework, it is our hope that this can be a resource that the community of researchers within the consortium can use to conduct further work involving genetic, methylation and phenotypic data. Members can propose new projects that fall under two different categories:

1. Using the results that are already generated by the pipeline from existing modules
2. Writing new modules to perform extended analyses

For further details, click [here](http://www.godmc.org.uk/information.html).


## How does the pipeline handle twins and related individuals

In phase 1, if it is declared in the `config` file that the study design is based on family relationships (e.g. trios, families, twins etc) we are using the GRAMMAR approach ([Aulchenko et al 2007](http://www.genetics.org/content/177/1/577)) for the purposes of adjusting the methylation data for polygenic effects and then using `matrixeqtl` on the residuals. Phase 2 will then perform a linear mixed model to estimate SNP-CpG associations in the **candidate list** to account for relatedness.

For twin studies MZ and DZ twins can be included. If MZ twins only have genotype data for one of the pairs it is ok to duplicate the genotype data for the second pair.

For data that is declared to comprise only unrelated samples the data will check for any cryptic relatedness and remove samples accordingly.

More details are included in the analysis plan.


## What age groups can be included

Based on recent analysis ([Gaunt et al 2016](http://genomebiology.biomedcentral.com/articles/10.1186/s13059-016-0926-z)) the effects of SNP-CpG associations are highly stable across the life course, so we will be including all age groups in the main mQTL meta-analysis. 

For the secondary analyses (e.g. GWAS of cell counts, GWAS of predicted smoking, EWAS of height and BMI) the pipeline will automatically stratify samples into different age groups (combiend, below 18, above 18).


## Which populations can be included

We are initially focusing on samples with European ancestry. If you have samples that comprise other ancestries then please contact us.


## How should cases and controls be handled

Some cohorts will have collected data that ascertains for cases and controls for a particular phenotype. We request that the cases and controls be combined to analysed together as a single cohort in the pipeline.

