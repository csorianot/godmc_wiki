Welcome to the GoDMC pipeline. This wiki will guide you through the analysis. The pipeline has been designed so that minimal coding is required by analysts, which hopefully ensures that it is quick to execute, avoids duplication of efforts, increases reproducibility and reduces potential for error. The main objectives of the pipeline are:

### Perform a methQTL analysis

Perform a GWAS on every methylation probe available in the sample. Variations of this analysis will be performed, including a standard methQTL using bestguess imputed SNPs, a CNV-methQTL analysis using CNVs inferred from methylation arrays, and a var-methQTL analysis which looks for SNPs that influence the variance of a probe. 

**IMPORTANT: Please note in the first phase of GoDMC we are focusing on blood samples of Europeans only. Datasets should have at least 100 individuals.** However, after we have established our methQTL catalogue we would definitely like to do a transethnic and intertissue comparison.

### GWAS and GREML analysis
We will be performing a GWAS on age accelerated residuals (AAR), smoking status as predicted by methylation, and cell count proportions.

### EWAS of complex traits
Focussing initially on height and BMI.

---
### Analysis plan
The full analysis plan can be downloaded [here](https://github.com/MRCIEU/godmc/files/197447/GoDMCanalysisplan271116.docx), and we aim to have meta analysed at least 10,000 samples by the end of 2016 (see [here](https://docs.google.com/spreadsheets/d/1iOr0ZyLr8OOmhOsHLCxoBEhanJ2K_LZG-X1YrnDFyRc/edit?usp=sharing) for details of the contributing cohorts).

---

### How to use this wiki

This wiki will guide you through the GoDMC analysis. The first stage is to get your data into the right format and the right place (see the **Pre-requisites** section in the sidebar). Then it's just a matter of running the scripts one by one. At the end of each section we have provided a script that will check and upload the results to the server.