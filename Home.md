# GoDMC pipeline

This wiki will guide you through using the GoDMC pipeline. 

The objectives of the pipeline are:

1. **Perform an meQTL analysis:** Perform a GWAS on every methylation probe available in the sample. Variations of this analysis will be performed, including a standard meQTL using bestguess imputed SNPs, a CNV-mQTL analysis using CNVs inferred from methylation arrays, and a var-meQTL analysis which looks for SNPs that influence the variance of a probe. 
2. **GWAS and GREML analysis of various methylation-related phenotypes:** We will be performing a GWAS on age accelerated residuals (AAR), smoking status as predicted by methylation, and cell count proportions.
3. **EWAS of complex traits:** Focussing initially on height and BMI.
