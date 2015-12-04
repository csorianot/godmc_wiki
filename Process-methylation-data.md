You have created the necessary files for the SNP data. Now we will create the methylation related files.

### Generate covariates and derived variables

First we will generate the following:
- Predict smoking ([Zeilinger et al 2013](http://www.ncbi.nlm.nih.gov/pubmed/23691101) and [Elliott et al 2014](http://www.ncbi.nlm.nih.gov/pubmed/24485148))
- Estimate cell counts ([Houseman reference method](http://www.biomedcentral.com/1471-2105/13/86) as impPredict smoking ([Zeilinger et al 2013](http://www.ncbi.nlm.nih.gov/pubmed/23691101) and [Elliott et al 2014](http://www.ncbi.nlm.nih.gov/pubmed/24485148))
- Generate necessary covariate file formats for upcoming analyses

To do this simply run

    ./04a-methylation_variables.sh

This script should run very quickly (e.g. less than 5 minutes for 100 individuals, I took me 6 min on ~1800 samples), and generate the necessary files in the `processed_data/methylation_data` folder.

Please check the following graphs:
- `results/04/smoking_prediction.pdf` - shows smoking predictor distribution
- `results/04/age_prediction.pdf` - shows correlation between predicted and actual ages
- `results/04/cellcounts_plot.pdf` - shows cell count distributions. Please note that there maybe zeros in the cell counts (CD8T and eosinophils) so the transformations don't look great. 


### Adjustment of methylation values

Next we will normalise the methylation data. This is a more computationally intensive process and you have the option to split it across multiple threads on a single computer, or across multiple threads and multiple nodes on a cluster (though this second option requires some script input from you).

In order to maximise the meQTL analysis speed we will pre-adjust the methylation values for all possible covariates. In order to improve power we will be adjusting for as much non-genetic variance as possible, and to reduce the possibility of type 1 errors we will be transforming the methylation values using inverse normal transformation. The steps we are about to perform are as follows:

1. Set 10 SD outliers to NA
2. Inverse-normal transformation of each methylation probe
3. Fit age, sex, predicted smoking, (predicted) cell counts, genetic principal components as fixed effects, and family relatedness as a random effect if it is family data, against each methylation probe and keep the normalised residuals.
4. Set NAs to probe mean.
5. Estimate the methylation principal components using the 20000 most variable methylation probes, retain the first `n` PCs that individually explain at least 1% of the methylation variation. Remove any PCs associated with height or BMI from this list.
6. Run a GWAS against each of the retained methylation PCs and discard any PCs that have evidence for a genetic effect (p < 1e-6).
7. Fit the remaining non-genetic methylation PCs against each of the methylation probes from (2) and retain the residuals.

These residuals have now been adjusted for measured covariates, estimated cell counts, estimated smoking, genetic relatedness and structure, and estimates of unmeasured confounders. Therefore the residual variance should be minimised as much as possible without losing genetic variance.

In order to perform this normalisation perform the following. To perform steps 1 and 2, run:

    ./04b-methylation_adjustment1.sh

This will parallelise across `$nthreads` (which was set in your `config` file). For 100 samples of related individuals and using 16 cores this analysis took about 2 hours. It is faster for unrelated samples. See the last section on this page for instructions on how to parallelise this analysis across multiple nodes on a cluster.

In order to perform steps 3 and 4, run:

    ./04c-methylation_pcs.sh

This step took 15 minutes with a sample of 100 individuals. Again, it parallelises across `$nthreads` (for the GWAS stage). 

In order to run step 5, run the following:

    ./04d-methylation_adjustment2.sh

This should take roughly the same amount of time as the `04b-methylation_adjustment1.sh` script. Again, it can be split across multiple nodes on a cluster, see below for instructions.

Finally we need to turn the methylation data into the correct format for MatrixeQTL analysis, to do this:

    ./04e-convert_methylation_format.sh

This also produces data of the squared residuals for use in variance meQTL analysis. This script took a few minutes for a sample size of 100.



### Optional: Running on the cluster

We can run the `04b-methylation_adjustment1.sh` and `04d-methylation_adjustment2.sh` in parallel on a cluster. In the `config` file you can specify how many chunks to break the entire analysis into, e.g. 

    meth_chunks="100"

will split the ~480k probes into 100 chunks of ~4.8k each. If you run

    ./04b-methylation_adjustment1.sh

Then the script runs the entire 480k probes using `nthreads` threads. However if you specify a number between 1 and 100 as an argument to this script, e.g.:

    ./04b-methylation_adjustment1.sh 1

then the script will run only the 1st chunk of the entire dataset, i.e. approximately, probes 1-4800.

To utilise this on a cluster requires two steps:

1. Creating a submission script that works for your cluster
2. Aggregating the individual results from each chunk into a single file at the end.


An example of how a submission script, e.g. `submit_04b.sh` for our cluster would be written is as follows:


```bash

#!/bin/bash

#PBS -N meth_04b
#PBS -o meth_04b-output
#PBS -e meth_04b-error
#PBS -l walltime=12:00:00
#PBS -t 1-100
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./04b-methylation_adjustment1.sh ${PBS_ARRAYID}

```

then to run:

    qsub submit_04b.sh

On our cluster this creates a batch of 100 jobs, each with the `${PBS_ARRAYID}` variable set to a value between 1 and 100. 

After these jobs have completed, assuming there have been no errors, there will be 100 separate `.RData` files that need to be combined together. To do this simply run:

    resources/methylation/aggregate_adjustment1.sh

This will combine your 100 chunks into a single file for use moving forward in the analysis.


Similarly, for the `04d-methylation_adjustment2.sh` script you would create a submission script (e.g. `submit_04d.sh`):

```bash

#!/bin/bash

#PBS -N meth_04d
#PBS -o meth_04d-output
#PBS -e meth_04d-error
#PBS -l walltime=12:00:00
#PBS -t 1-100
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./04d-methylation_adjustment.sh ${PBS_ARRAYID}

```

Then:

    qsub submit_04d.sh

and once the jobs are completed:

    resources/aggregate_adjustment2.sh


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