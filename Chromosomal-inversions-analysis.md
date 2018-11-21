We aim to investigate the influence of 19 genetic inversions on DNA methylation and on different phenotypes. First, we will explore the association between inversions and DNA methylation in cis in CpGs located 1 Mb up and down the inversion and in trans effects. Second, we will associate chromosomal inversions to BMI and height and to the phenotypes inferred from DNA methylation data: epigenetic age and cell composition. 

## 1. Requirements and setup
This analysis reuses the processed data generated in the main GoDMC analysis. Therefore, we will also use the same covariates used by the cohorts to adjust their data. Please, ensure that you have successfully run steps 1-4 before running this analysis.

This analysis uses the same programs of the main GoDMC pipeline. The only exception is scoreInvHap, an R Bioconductor package that should be installed by the analysts. You can install scoreInvHap with:

    install.packages("devtools")
    library(devtools)
    install_github("isglobal-brge/scoreInvHap")
	

**Important**: the recommended way to install scoreInvHap is using the github repository. This ensures having the last version of the package with the latest updates. To install scoreInvHap from github, the server should have at least R3.4. If you have an older R version in your server, please, contact us. 

	
## Inversions module contact

If you have any issues or questions about inversion pipeline then please contact us: [carlos.ruiz@isglobal.org](mailto:carlos.ruiz@isglobal.org)

