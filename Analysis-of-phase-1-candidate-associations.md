In phase 1 each cohort returned a list of SNP-CpG associations that showed nominal evidence of association at threshold p < 1e-5. We used the lists from each cohort to create a combined list of putative associations. Now, each cohort needs to provide estimates of the effects for this list of putative associations so that they can be meta-analysed.

This analysis is performed in three steps - setting up the analysis, performing a control test, and running the full analysis. Please make sure you have finished module 1-4 before you start this analysis.


### Setup

The setup will download the list of putative associations, and generate the genotype file formats required for the analysis. To run:

    ./16a-setup.sh

For a sample size of 2000 this runs in about 20 minutes and doesn't require large memory.


### Control

This performs a GWAS on a single CpG with a known large cis-effect. We check that the cis-effect is identified, and then we check that there is no genomic inflation amongst the trans-SNPs. To run:

    ./16b-control.sh

For a sample size of 2000 this runs in about 50 minutes and doesn't require large memory.

**If you get an error about no strong association, please contact the developers.**

It generates Manhattan and Q-Q plots, located at `results/16/control`. Please check them and the lambda values in the output to check they look sensible. If you need assistance then contact the developers.


### Running the analysis

There are more than 120 million putative associations to test. This list has been split into 962 chunks which allows parallelisation across a cluster.

If we run

    ./16c-run.sh 1

The script will perform the meQTL analysis of the first batch of candidate associations. 

Going through 1-962 sequentially could take some time so you can parallelise across a cluster or your server. 

If you don't have access to a cluster you can parallelise your jobs using the package GNU parallel.

```
seq 962 | parallel -j 10 --workdir $PWD ./16c-run.sh {}
```

If you have access to a cluster, you need to create a job submission script that will work on your cluster. For ~2000 samples, one chunk took 20 minutes to run using a single core. Each cluster is different, but to show an example of how it works on our cluster (which uses a PBS scheduler), we would create a script (e.g. `submit_16.sh`) like this:


```bash

#!/bin/bash

#PBS -N godmc16
#PBS -o godmc16-output
#PBS -e godmc16-error
#PBS -l walltime=12:00:00
#PBS -t 1-962
#PBS -l nodes=1:ppn=1
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/    #EDIT THIS LINE
./16c-run.sh ${PBS_ARRAYID}

```

Then, when this is submitted:

    qsub submit_16.sh

it will create a batch of 638 jobs, each running with the variable `$PBS_ARRAYID` set to a value between 1-962. 


### Check and upload the results

The results are in `results/16/results_*.gz`.

Please check that these look ok (e.g. `zless results/16/results_1.gz`, they should be in GWAMA format with the following headers:

```
MARKERNAME PHENOTYPE CISTRANS BETA SE PVAL N EA NEA EAF SNP
```

To check that everything ran successfully, please run:

```
./check_upload.sh 16 check
```

This should tell you that `Section 16 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 16 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 16.

This procedure will be repeated at the end of each section.