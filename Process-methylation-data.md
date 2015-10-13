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


### Adjustment of methylation values

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

This will parallelise across `$nthreads` (which was set in your `config` file). For 100 samples of related individuals and using 16 cores this analysis took about 2 hours. See the last section on this page for instructions on how to parallelise this analysis across multiple nodes on a cluster.

In order to perform steps 3 and 4, run:

    ./04c-methylation_pcs.sh

This step took 15 minutes with a sample of 100 individuals. Again, it parallelises across `$nthreads` (for the GWAS stage). 

In order to run step 5, run the following:

    ./04d-methylation_adjustment2.sh

This should take roughly the same amount of time as the `04b-methylation_adjustment1.sh` script, unless you have related data, in which case this will be much faster. Again, it can be split across multiple nodes on a cluster, see below for instructions.

Finally we need to turn the methylation data into the correct format for MatrixeQTL analysis, to do this:

    ./04e-convert_methylation_format.sh

This took a few minutes for a sample size of 100.



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
#PBS -o job_reports/jobname-output
#PBS -e job_reports/jobname-error
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

    resources/aggregate_adjustment1.sh

This will combine your 100 chunks into a single file for use moving forward in the analysis.


Similarly, for the `04d-methylation_adjustment2.sh` script you would create a submission script (e.g. `submit_04d.sh`):

```bash

#!/bin/bash

#PBS -N meth_04d
#PBS -o job_reports/jobname-output
#PBS -e job_reports/jobname-error
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
