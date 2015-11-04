We will be performing GWAS on the following phenotypes

- Age accelerated residuals (covariates: sex and smoking)
- Predicted smoking (covariates: age and sex)
- Cellcount entropy (the relative diversity of cell counts in the sample) (covariates: smoking, age and sex)
- Abundance of each cell type (covariates: smoking, age and sex)

These GWASs will be performed using a linear mixed model (LMM) to account for population structure and/or relatedness. Specifically, we will be using the GCTA `--mlma-loco` routine ([Yang et al 2014](http://www.nature.com/ng/journal/v46/n2/abs/ng.2876.html)). 

In addition we have also provided scripts to perform a multivariate LMM (MVLMM) to perform multi-trait GWAS on all cell count estimates jointly. This is performed using GEMMA software ([Zhou and Stephens 2014](http://www.nature.com/nmeth/journal/v11/n4/full/nmeth.2848.html)).

To perform the age accelerated residuals GWAS:

    ./07a-gwas_aar.sh

To perform the predicted smoking GWAS:

    ./07b-gwas_smoking.sh

To perform the cellcount entropy GWAS:

    ./07c-gwas_cellcount_entropy.sh

Each of these may take some time, (approximately 2 minutes each for sample size of 100, but time will increase quadratically with increasing sample size).

To perform the GWAS of each cell type, assuming we have 7 cell types, we need to run:

```
./07d-gwas_cellcounts.sh 1
./07d-gwas_cellcounts.sh 2
./07d-gwas_cellcounts.sh 3
./07d-gwas_cellcounts.sh 4
./07d-gwas_cellcounts.sh 5
./07d-gwas_cellcounts.sh 6
./07d-gwas_cellcounts.sh 7
```

In order to check how many cell types there are, you can check the following file:

```
less results/gwas_cellcounts/cellcounts_columns.txt
```

Finally, in order to perform the MVLMM on cell counts it is likely necessary that you will need a cluster, as this is much slower to run than the standard LMM. The script is setup to split the data into `genetic_chunks` chunks (as specified in the `config` file), and to run you will need to create a submission script that works for your system. For example, on our cluster we would create a script that looks like this (e.g. `submit_07e.sh`, if `genetic_chunks="1000"`):

```
#!/bin/bash

#PBS -N mvlmm
#PBS -o mvlmm-output
#PBS -e mvlmm-error
#PBS -l walltime=12:00:00
#PBS -t 1-1000
#PBS -l nodes=1:ppn=1
#PBS -S /bin/bash

set -e

echo "Running on ${HOSTNAME}"

cd /path/to/godmc/
./07e_gwas_cellcounts_mvlmm.sh ${PBS_ARRAYID}
```

and to submit:

    qsub submit_07e.sh

And this will distribute the entire analysis into 1000 batches.