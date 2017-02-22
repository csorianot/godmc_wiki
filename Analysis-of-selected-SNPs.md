This analysis calculates all associations between a subset of SNPs and a subset of CpGs. Efforts have been made to minimise disk space usage but, it will still require approximately 40Gb of space. If this is an issue then please contact the developers.

**Background:** Mendelian randomisation can be used to evaluate the causal relationship between traits using genetic associations. In this analysis we will use selected SNPs known to influence specific complex traits to evaluate the causal influence of those complex traits on methylation levels. This requires obtaining summary data of the trait SNPs' associations with all relevant methylation levels.

This analysis is performed in two steps - setting up the analysis and running the full analysis. 


### Setup

The setup will download the lists of selected SNPs and CpGs, and generate the genotype and phenotype file formats required for the analysis. To run:

	./17a-setup.sh

For a sample size of 2000 this runs in about 15 minutes and doesn't require large memory.


### Running the analysis

The CpG list has been split into 100 chunks which allows parallelisation across a cluster.

If we run

```
./17b-run.sh 1
```

The script will perform the meQTL analysis of the first batch of CpGs. It takes approximately 20 minutes for a sample size of 2000, and doesn't require large memory.

Going through 1-100 sequentially could take some time so you can parallelise across a cluster or your server. 

If you don't have access to a cluster you can parallelise your jobs using the package GNU parallel.

```
seq 100 | parallel -j 10 --workdir $PWD ./17b-run.sh {}
```

If you have access to a cluster, you need to create a job submission script that will work on your cluster. For ~2000 samples, one chunk took 20 minutes to run using a single core. Each cluster is different, but to show an example of how it works on our cluster (which uses a PBS scheduler), we would create a script (e.g. `submit_17.sh`) like this:


```bash

#!/bin/bash

#PBS -N godmc17
#PBS -o godmc17-output
#PBS -e godmc17-error
#PBS -l walltime=12:00:00
#PBS -t 1-100
#PBS -l nodes=1:ppn=1
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/    #EDIT THIS LINE
./16-run.sh ${PBS_ARRAYID}

```

Then, when this is submitted:

    qsub submit_17.sh

it will create a batch of 100 jobs, each running with the variable `$PBS_ARRAYID` set to a value between 1-100. 


### Check and upload the results

The results are in `results/17/results_*.txt.gz`.

Please check that these look ok (e.g. `zless results/16/results_1.txt.gz`, they should be in GWAMA format with the following headers:

```
MARKERNAME EA NEA EAF BETA SE
1706_4145 C T 0.4936 0.741768 0.127785
1082_4077 T C 0.3506 -0.837446 0.1562
1082_7262 A C 0.3766 -0.800838 0.151384
325_13470 T C 0.1908 -0.967334 0.184061
```

To check that everything ran successfully, please run:

```
./check_upload.sh 17 check
```

This should tell you that `Section 17 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 17 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 17.

This procedure will be repeated at the end of each section.
