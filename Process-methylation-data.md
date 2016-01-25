You have created the necessary files for the SNP data. Now we will create the methylation related files.

### Generate covariates and derived variables

First we will generate the following:
- Predict smoking ([Zeilinger et al 2013](http://www.ncbi.nlm.nih.gov/pubmed/23691101) and [Elliott et al 2014](http://www.ncbi.nlm.nih.gov/pubmed/24485148))
- Estimate cell counts from normalized betas ([Houseman reference method](http://www.biomedcentral.com/1471-2105/13/86) if you haven't provided these (http://www.ncbi.nlm.nih.gov/pubmed/24485148))
- Transform houseman predicted cellcounts for the GWA analysis
- Generate necessary covariate file formats for upcoming analyses

To do this simply run

    ./04a-methylation_variables.sh

This script should run very quickly (e.g. less than 5 minutes for 100 individuals, I took me 6 min on ~1800 samples), and generate the necessary files in the `processed_data/methylation_data` folder.

**IMPORTANT** Please check the following graphs:
- `results/04/smoking_prediction.pdf` - shows smoking predictor distribution
- `results/04/age_prediction.pdf` - shows correlation between predicted and actual ages
- `results/04/cellcounts_plot.pdf` - shows cell count distributions. Please note that there maybe zeros in the cell counts (CD8T and eosinophils) so the transformations don't look great. 


### Adjustment of methylation values

Next we will normalise the methylation data. This is a more computationally intensive process and you have the option to split it across multiple threads on a single computer, or across multiple threads and multiple nodes on a cluster (though this second option requires some script input from you).

In order to maximise the meQTL analysis speed we will pre-adjust the methylation values for all possible covariates. In order to improve power we will be adjusting for as much non-genetic variance as possible, and to reduce the possibility of type 1 errors we will be transforming the methylation values using inverse normal transformation. The steps we are about to perform are as follows:

1. Set 10 SD outliers to NA using 3 iterations
2. Inverse-normal transformation of each methylation probe
3. Fit age, sex, predicted smoking, (predicted) cell counts, genetic principal components as fixed effects, and family relatedness as a random effect if it is family data, against each methylation probe and keep the normalised residuals.

4. Set NAs to probe mean.
5. Estimate the methylation principal components using the 20000 most variable methylation probes, retain the first `n` PCs that individually explain at least 1% of the methylation variation. Remove any PCs associated with height or BMI from this list.
6. Run a GWAS against each of the retained methylation PCs and discard any PCs that have evidence for a genetic effect (p < 1e-6).
7. Fit the remaining non-genetic methylation PCs against each of the methylation probes from (2) and retain the residuals.

These residuals have now been adjusted for measured covariates, estimated cell counts, estimated smoking, genetic relatedness and structure, and estimates of unmeasured confounders. Therefore the residual variance should be minimised as much as possible without losing genetic variance.

In order to perform this normalisation perform the following. To perform steps 1 and 2, run:

    ./04b-methylation_adjustment1.sh

This will parallelise across `$nthreads` (which was set in your `config` file). For 100 samples of related individuals and using 16 cores this analysis took about 2 hours. It is faster for unrelated samples. 
See [here](https://github.com/MRCIEU/godmc/wiki/Running-script-4b-and-4d-on-a-cluster) for instructions on how to parallelise this analysis across multiple nodes on a cluster.

In order to perform steps 3 and 4, run:

    ./04c-methylation_pcs.sh

This step took 15 minutes with a sample of 100 individuals and 15 minutes with a sample of ~1800 individuals. Again, it parallelises across `$nthreads` (for the GWAS stage). 

In order to run step 5, run the following:

    ./04d-methylation_adjustment2.sh

This should take roughly the same amount of time as the `04b-methylation_adjustment1.sh` script. Again, it can be split across multiple nodes on a cluster, see [here](https://github.com/MRCIEU/godmc/wiki/Running-script-4b-and-4d-on-a-cluster) for instructions.

We need to turn the methylation data into the correct format for MatrixeQTL analysis, to do this:

    ./04e-convert_methylation_format.sh

This also produces data of the squared residuals for use in variance meQTL analysis. This script took a few minutes for a sample size of 100.

Finally we run one CpG GWA as a positive control. It should give a signal on chromosome 22 as this methQTL was found in at least 3 cohorts (p<1e-8).

    ./04f-convert_methylation_format.sh

Please check the plots and lambda in the results section. It is important to check lambda here before going on and any lambda above 1.08 is worrying. In general for a standard GWA you would expect to see a lambda below 1.08.