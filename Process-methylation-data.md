You have created the necessary files for the SNP data. Now we will create the methylation related files.

### Generate covariates and derived variables

First we will generate the following:

- Estimate cell counts
- Estimate age accelerated residuals
- Predict smoking
- Generate necessary covariate file formats

To do this simply run

    04a-methylation_variables.sh

This script should run very quickly, and generate the necessary files in the `processed_data/methylation_data` folder.


#### Adjustment of methylation values

Next we will normalise the methylation data. This is a more computationally intensive process and you have the option to split it across multiple threads on a single computer, or across multiple threads and multiple nodes on a cluster (though this second option requires some script input from you).

In order to maximise the meQTL analysis speed we will pre-adjust the methylation values for all possible covariates. In order to improve power we will be adjusting for as much non-genetic variance as possible, and to reduce the possibility of type 1 errors we will be transforming the methylation values using inverse normal transformation. The steps we are about to perform are as follows:

1. Inverse-normal transformation of each methylation probe
2. Fit age, sex, smoking, cell counts, genetic principal components as fixed effects, and family relatedness as a random effect if it is family data, against each methylation probe and keep the normalised residuals
3. Estimate the methylation principal components using the 20000 most variable methylation probes, retain the first n PCs that cumulatively explain 80% of the methylation variation. Remove any PCs associated with height or BMI from this list.
4. Run a GWAS against each of the retained methylation PCs and discard any PCs that have evidence for a genetic effect (p < 1e-8).
5. Fit the remaining non-genetic methylation PCs against each of the methylation probes from (2) and retain the residuals.

These residuals have these been adjusted for measured covariates, estimated cell counts, estimated smoking, genetic relatedness and structure, and estimates of unmeasured confounders. Therefore the residual variance should be minimised as much as possible without losing genetic variance.

In order to perform this normalisation perform the following. To perform steps 1 and 2, run:

    ./04b-methylation_adjustment1.sh

This will parallelise across `$nthreads` (which was set in your `config` file). For 100 samples of related individuals and using 16 cores this analysis took about 15 minutes. See [here]() for instructions on how to parallelise this analysis across multiple nodes on a cluster.

In order to perform steps 3 and 4, run:

    ./04c-methylation_pcs.sh

This step took 15 minutes with a sample of 100 individuals. Again, it parallelises across `$nthreads` (for the GWAS stage). 

In order to run step 5, run the following:

    ./04d-methylation_adjustment2.sh

This should take roughly the same amount of time as the `04b-methylation_adjustment1.sh` script, unless you have related data, in which case this will be much faster. Again, it can be split across multiple nodes on a cluster, see [here]() for instructions.

Finally we need to turn the methylation data into the correct format for MatrixeQTL analysis, to do this:

    ./04e-convert_methylation_format.sh

This took a few minutes for a sample size of 100.


