
### Convert to gemma format
Next, we need to generate the correct format for the multi-variate linear mixed model (MVLMM) on cell counts that we run in GEMMA. 

**IMPORTANT:** We don't run this analysis on cord blood samples.

To do this run the following script form your home directory:
``` 
    ./13a-convert_gemma_format.sh
```

### Running a multi-variate linear mixed model (MVLMM) on cell counts
In order to perform the multi-variate linear mixed model (MVLMM) on cell counts it is likely necessary that you will need a cluster, as this is much slower to run than the standard LMM. The script is setup to split the data into `genetic_chunks` chunks (as specified in the `config` file), and to run you will need to create a submission script that works for your system. 

**WARNING**: Some cohorts have experienced problems running this module. It might therefore useful to test the script on just one chunk: `./13-gwas_cellcounts_mvlmm.sh 1` and check the logfile here: `./results/13/logs/log.txt1`. If you see something like this, you probably can't fit the model and you won't be able to run this module.
```
se(Ve): 
0.0000	
nan	0.0000	
nan	nan	nan	
0.0000	nan	nan	0.0000	
nan	nan	nan	nan	nan	
0.0000	nan	nan	0.0000	0.0000	nan	
nan	nan	nan	nan	nan	nan	0.0000
```

 If it runs correctly you might generate a submission that looks like this (e.g. `submit_mvlmm.sh`, if `genetic_chunks="500"`):

```
#!/bin/bash

#PBS -N mvlmm
#PBS -o mvlmm-output
#PBS -e mvlmm-error
#PBS -l walltime=12:00:00
#PBS -t 1-500
#PBS -l nodes=1:ppn=1
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./13-gwas_cellcounts_mvlmm.sh ${PBS_ARRAYID}
```

and to submit:

    qsub submit_mvlmm.sh

And this will distribute the entire analysis into 500 batches.

If it seems that this is going to be too slow, or if some sections are crashing, please let us know.

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 13 check
```

This should tell you that `Section 13 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 13 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 13`.

This procedure will be repeated at the end of each section.