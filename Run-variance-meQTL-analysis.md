We have now performed the standard meQTL analysis. We are now going to perform the same kind of analysis again except this time we will look for genetic effects against the squared residuals of the methylation probes, hence testing for genetic influences on the variance of methylation. 

The procedure is exactly the same, the SNPs have been split into `genetic_chunks` chunks which can each be run independently on a different node on the cluster. An example job submission script (e.g. `submit_vmqtl.sh`) would be:

```bash

#!/bin/bash

#PBS -N vmqtl
#PBS -o vmqtl-output
#PBS -e vmqtl-error
#PBS -l walltime=12:00:00
#PBS -t 1-500
#PBS -l nodes=1:ppn=16
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./06-vmqtl.sh ${PBS_ARRAYID}

```

then to submit to the cluster:

    qsub submit_vmqtl.sh

And this will distribute the entire surface across 500 nodes, each node parallelising across 16 cores.


### Upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 06 check
```

This should tell you that `Section 06 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 06 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 06.

This procedure will be repeated at the end of each section.