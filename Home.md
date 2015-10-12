# GoDMC pipeline

This wiki will guide you through using the GoDMC pipeline. 

The objectives of the pipeline are:

1. **Perform an meQTL analysis:** Perform a GWAS on every methylation probe available in the sample. Variations of this analysis will be performed, including a standard meQTL using bestguess imputed SNPs, a CNV-mQTL analysis using CNVs inferred from methylation arrays, and a var-meQTL analysis which looks for SNPs that influence the variance of a probe. 
2. **GWAS and GREML analysis of various methylation-related phenotypes:** We will be performing a GWAS on age accelerated residuals (AAR), smoking status as predicted by methylation, and cell count proportions.
3. **EWAS of complex traits:** Focussing initially on height and BMI.

---

## Table of contents

1. [Pre-requisites - Normalised methylation data](Pre-requisites---Normalised-methylation-data.md)
2. [Pre-requisites - Imputed genetic data](Pre-requisites---Imputed-genetic-data.md)
3. [Pre-requisites - Structural variant data](Pre-requisites---Structural-variant-data.md)
4. [Pre-requisites - Phenotype and covariate data](Pre-requisites---Phenotype-and-covariate-data.md)
5. [System requirements]
6. [Setup-and-installation](Setup-and-installation.md)
7. [System check]
8. [Processing SNP data]
9. [Processing methylation data]
10. [Processing phenotypes and covariates]
11. [Running meQTL analysis]
12. [Running CNV-meQTL analysis]
13. [Running var-meQTL analysis]
14. [Performing GWAS]
15. [Performing EWAS]
16. [Transferring first stage results to Bristol]


## Contact

If you have any issues or questions about this analysis then please contact us:

- Gibran Hemani at [g.hemani@bristol.ac.uk](mailto:g.hemani@bristol.ac.uk)
- Josine Min at [josine.min@bristol.ac.uk](mailto:josine.min@bristol.ac.uk)
