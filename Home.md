Welcome to the GoDMC pipeline. This wiki will guide you through the analysis. The pipeline has been designed so that minimal coding is required by analysts, which hopefully ensures that it is quick to execute, avoids duplication of efforts, increases reproducibility and reduces potential for error. The main objectives of the pipeline are:

### Perform an meQTL analysis
Perform a GWAS on every methylation probe available in the sample. Variations of this analysis will be performed, including a standard meQTL using bestguess imputed SNPs, a CNV-mQTL analysis using CNVs inferred from methylation arrays, and a var-meQTL analysis which looks for SNPs that influence the variance of a probe. 

### GWAS and GREML analysis
We will be performing a GWAS on age accelerated residuals (AAR), smoking status as predicted by methylation, and cell count proportions.

### EWAS of complex traits
Focussing initially on height and BMI.

---
### Analysisplan
The full analysis plan can be downloaded [here](https://github.com/MRCIEU/godmc/files/197447/GoDMCanalysisplan271116.docx), and we aim to have meta analysed at least 10,000 samples by the end of 2016 (see [here](https://docs.google.com/spreadsheets/d/1iOr0ZyLr8OOmhOsHLCxoBEhanJ2K_LZG-X1YrnDFyRc/edit?usp=sharing) for details of the contributing cohorts).

---

### How to use this wiki

This wiki will guide you through the GoDMC analysis. The first stage is to get your data into the right format and the right place (see the **Pre-requisites** section in the sidebar). Then it's just a matter of running the scripts one by one. At the end of each section we have provided a script that will check and upload the results to the server.