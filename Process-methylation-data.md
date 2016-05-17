You have created the necessary files for the SNP data. Now we will create the methylation related files.

### Generate covariates and derived variables

First we will generate the following:
- Predict smoking ([Zeilinger et al 2013](http://www.ncbi.nlm.nih.gov/pubmed/23691101) and [Elliott et al 2014](http://www.ncbi.nlm.nih.gov/pubmed/24485148))
- Estimate cell counts from normalized betas ([Houseman reference method](http://www.biomedcentral.com/1471-2105/13/86)) if you haven't provided these.
- Transform houseman predicted cellcounts for the GWA analysis
- Generate necessary covariate file formats for upcoming analyses

To do this simply run

    ./04a-methylation_variables.sh

This script should run very quickly (e.g. less than 5 minutes for 100 individuals, I took me 6 min on ~1800 samples), and generate the necessary files in the `processed_data/methylation_data` folder.

**IMPORTANT** Please check the following graphs:
- `results/04/smoking_prediction.pdf` - shows smoking predictor distribution
- `results/04/age_prediction.pdf` - shows correlation between predicted and actual ages
- `results/04/cellcounts_plot.pdf` - shows cell count distributions. Please note that there maybe zeros in the cell counts (CD8T and eosinophils) so the transformations don't look great. Please note we are not running a GWAS on eosinophils but only use the untransformed eosinophils as a covariate in the analysis. 
- `results/04/logs_a/log.txt` -the last sentence should say 'Successfully created methylation-related variables'


### Adjustment of methylation values

Next we will normalise the methylation data. This is a more computationally intensive process and you have the option to split it across multiple threads on a single computer, or across multiple threads and multiple nodes on a cluster (though this second option requires some script input from you).

In order to maximise the meQTL analysis speed we will pre-adjust the methylation values for all possible covariates. In order to improve power we will be adjusting for as much non-genetic variance as possible, and to reduce the possibility of type 1 errors we will be transforming the methylation values using inverse normal transformation. The steps we are about to perform are as follows:

1. Set 10 SD outliers to NA using 3 iterations
2. Inverse-normal transformation of each methylation probe
3. Fit age, sex, predicted smoking, (predicted) cell counts, genetic principal components as fixed effects, and family relatedness as a random effect if it is family data, against each methylation probe and keep the normalised residuals.
4. Set NAs to probe mean.
5. Estimate the methylation principal components using the 20000 most variable methylation probes, retain the first `n` PCs that individually explain at least 1% of the methylation variation. Remove any PCs associated with height or BMI from this list.
6. Run a GWAS against each of the retained methylation PCs and discard any PCs that have evidence for a genetic effect (p < 1e-7).
7. Fit the remaining non-genetic methylation PCs against each of the methylation probes from (2) and retain the residuals.
8. Convert files to matrixQTL format.
9. Run a methQTL association as positive control.

These residuals have now been adjusted for measured covariates, estimated cell counts, estimated smoking, genetic relatedness and structure, and estimates of unmeasured confounders. Therefore the residual variance should be minimised as much as possible without losing genetic variance.

In order to perform this normalisation perform the following. To perform steps 1, 2, 3 and 4 run:

    ./04b-methylation_adjustment1.sh

This will parallelise across `$nthreads` (which was set in your `config` file). For 100 samples of related individuals and using 16 cores this analysis took about 2 hours. It is faster for unrelated samples. 
See [here](https://github.com/MRCIEU/godmc/wiki/Running-script-4b-and-4d-on-a-cluster) for instructions on how to parallelise this analysis across multiple nodes on a cluster. You can check your log files on /results/04/logs_b. In ARIES, where we have related data of ~1800 samples, it took one hour to run one of the 100 chunks on 16 cores.

In order to perform steps 5 and 6, run:

    ./04c-methylation_pcs.sh

This step took 15 minutes with a sample of 100 individuals and 15 minutes with a sample of ~1800 individuals. Again, it parallelises across `$nthreads` (for the GWAS stage). In the `config` file we have currently set the GWA significance threshold to 1e-7 (`meth_pc_threshold="1e-7"`). This means that any GWA signals with a p<1e-7 are considered as genetic signals. These "genetic" PCs are ignored and not regressed out from the methylation data. The maximum number of "non-genetic" PCs that is regressed out is set to 20 in the `config` file (`n_meth_pcs="20"`).

It is important that you check the log file generated in 4c: 
```
head -n20 ./results/04/logs_c/log.txt
tail ./results/04/logs_c/log.txt
```
If you get warning below, you need to change settings in your config file eg. increase `n_meth_pcs` to 30 if you have more than 20 PCs that explain 0.8 of methylation variance or increase `meth_pc_cutoff` to 0.9 if you have 20 PCs or less that explain 0.8 of methylation variance

```
Identified 20 out of 20 PCs with significant genetic component
Error in main() : It appears that all the PCs have a genetic component
This is a little worrying because it suggests family effects or stratification
Please check that 04b is adjusting for these factors
You could also try increasing the 'n_meth_pcs and/or meth_pc_cutoff' value in the config file.
Execution halted
```


In order to run step 7, run the following:

    ./04d-methylation_adjustment2.sh

This should take roughly the same amount of time as the `04b-methylation_adjustment1.sh` script on unrelated data. Again, it can be split across multiple nodes on a cluster, see [here](https://github.com/MRCIEU/godmc/wiki/Running-script-4b-and-4d-on-a-cluster) for instructions.  In ARIES (~1800 samples), it took one hour to run one of the 100 chunks on 16 cores.

We need to turn the methylation data into the correct format for MatrixeQTL analysis (step 8), to do this:

    ./04e-convert_methylation_format.sh

This also produces data of the squared residuals for use in variance meQTL analysis. This script took a few minutes for a sample size of 100. It took about 30 minutes on ~1800 samples.

Finally we run one CpG GWA as a positive control as a final check (step 9). 

    ./04f-perform_positive_control.sh

**IMPORTANT please check**
- `cg07959070` should give a signal on chromosome 22 as this cis methQTL was found in at least 3 cohorts (`p<1e-8`, index SNP `chr22:50053871:SNP`). This CpG survives stringent probe filtering based on Price et al, Chen et al and Naeem et al and no 1000 Genomes SNP was found in the probe sequence or CpG site. Obviously the strength of the association will depend on the sample size of your cohort and is currently set to a pvalue of 0.001.
- In addition this GWA is used to check the lambda (inflation statistic) of your data. Please check the qqplots and lambdas (in the plot) `positive_control_pcadjusted_cg07959070_qqplot.png` and `positive_control_pcunadjusted_cg07959070_qqplot.png`  in the results section. The difference between the two plots is the adjustment for methylation pcs. It is important to check lambda here before going on and any lambda above 1.08 is worrying. In general for a standard GWA you would expect to see a lambda below 1.08.

### Upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 04 check
```

This should tell you that `Section 04 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 04 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 04.

This procedure will be repeated at the end of each section.