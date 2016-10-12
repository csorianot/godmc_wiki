Before you move on, please make sure you have run:

```
git pull
```

It will download all phase 2 scripts.

At this point we have generated a candidate SNP-CpG list across a phase 1 cohorts. We have defined cis as 1MB between SNP and CpG. For _cis_ SNP-CpG pairs we have included all SNP-CpG pairs on the candidate list with a pvalue <1e-5 found in at least one phase 1 cohort. For _trans_ we have included all SNP-CpG pairs that were found in at least 2 cohorts with a pvalue <1e-5.
We are now going to perform the meQTL analysis on the candidate SNP-CPG list using a linear mixed model in GCTA.

We need to format the data to get to run in GCTA:

1. Download SNP-CpG candidate lists and probe lists from sftp. 
2. We generate 22 kinship matrices using PLINK (leave one chromosome out each time)
3. Generate 74 subsets of the methylation beta matrix. We remove outliers that are 10 SD from the mean using 3 iterations and conduct an inverse normal transformation on the methylation traits.
4. Generate GCTA covariate files, e.g. age, cell counts and predicted smoking as quantitative variables and Sex and other batch variables as categorical variables.
5. Generate PLINK files for sensitivity analysis in 16c using pc adjusted methylation betas from phase 1.



```
16a-preparegctafiles.sh
```

Before we run the candidate SNP-CpG list we are running the same positive control as we did in 04f.

```
16b-perform_positive_control.sh
```

Please check the manhattan and qqplots which can be found in `./results/16`. Please check the lambdas. Any lambda above 1.08 is concerning.

We are now ready to run the associations using GCTA. As this process is computationally expensive, it is performing an association of subsets of SNP against every probe. So the data has been setup to parallelise by splitting the methylation probes into 74 probesets. For each probe, each chromosome will run separately using one of the 22 generated kinship matrices.

To run the first probeset:
```
    ./16c-gcta.sh 1
```
The script will perform the meQTL analysis using probes from the first probeset, using `nthreads` threads in parallel (also specified in the `config` file). 

Going through 1-74 sequentially is likely to take some time so it is recommended that you parallelise across a cluster or your server. 

If you don't have access to a cluster you can parallelise your jobs using the package GNU parallel.
```
seq 74 | parallel -j 10 --workdir $PWD ./16c-gcta.sh {}
```

If you have access to a cluster, you need to create a job submission script that will work on your cluster. Each cluster is different, but to show an example of how it works on our cluster (which uses a PBS scheduler), we would create a script (e.g. `submit_16.sh`) like this:


```bash

#!/bin/bash

#PBS -N gcta16
#PBS -o gcta16-output
#PBS -e gcta16-error
#PBS -l walltime=100:00:00
#PBS -t 1-74
#PBS -l nodes=1:ppn=2
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/    #EDIT THIS LINE
./16c-gcta.sh ${PBS_ARRAYID}

```

Then, when this is submitted:

    qsub submit_16.sh

it will create a batch of 74 jobs, each running with the variable `$PBS_ARRAYID` set to a value between 1-74, and each individual job further parallelised across 2 threads. 

### Run some sensitivity analysis
We are now ready to run some sensitivity analysis using PLINK. We will only run the first set of probes in this analysis using the PC and covariate adjusted betas generated in script 04d.

```
16d-methQTL.plink.sh
```

The analysis will generate a plot `./results/16/plinkvsgcta.cg0000[0-9].pdf`. This plot compares SNP-CpG associations generated in phase1 against SNP-CpG associations generated in phase 2 (linear mixed model). Please contact us if you see large discrepancies.

### Check and upload the results

The results are in `results/16`.

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