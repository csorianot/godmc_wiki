### Genetic data

The genetic data must be
- Imputed to 1000 genomes reference panel, ideally phase 3 (phased haplotypes available [here](https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.html). For guidelines on how to perform imputation see [here](http://genome.sph.umich.edu/wiki/IMPUTE2:_1000_Genomes_Imputation_Cookbook) and [here](https://github.com/explodecomputer/godmc/wiki/Genetic-imputation). A pipeline is also available [here](https://github.com/explodecomputer/imputePipePBS). Please contact us if you have any queries about this.
- Converted to best guess binary plink format
- Filtered to have MAF > 0.01 and imputation quality score > 0.8
- All remaining SNPs combined into a single fileset (*i.e.* not a separate fileset for each chromosome)

Assuming you have used [impute2](https://mathgen.stats.ox.ac.uk/impute/impute_v2.html) to perform the imputation and generated `bgen` dosage files, then the last three steps can be performed using the following bash script:

It is highly recommended that you use `plink1.90` to do this, available for download [here](https://www.cog-genomics.org/plink2)

```bash

#!/bin/bash

for i in {1..22}
do

    # First extract the info scores
    # e.g. If using the output from the -snp-stats flag in qctool:
    awk '{ print $2, $19 }' data_chr${i}.snp-stats | sed 1d > data_chr${i}.info
    # or if you are using the output from the -summary_stats_only flag in snptest
    awk '{ print $2, $9 }' data_chr${i}.summary_stats | sed 1d > data_chr${i}.info

    plink1.90 --bgen data_chr${i}.bgen --sample data.sample --make-bed --qual-scores data_chr${i}.info --qual-threshold 0.8 --maf 0.01 --out data_chr${i}_filtered

    # Rename the SNP IDs if necessary to avoid possible duplicates
    
    cp data_chr${i}_filtered.bim data_chr${i}_filtered.bim.orig
    awk '{
        if (length($5) == "1" && length($6) == "1") 
            print $1, "chr"$1":"$4":SNP", $3, $4, $5, $6;
        else 
            print $1, "chr"$1":"$4":INDEL", $3, $4, $5, $6;
    }' data_chr${i}_filtered.bim.orig > data_chr${i}_filtered.bim

    # For simplicity remove any duplicates

    cp data_chr${i}_filtered.bim data_chr${i}_filtered.bim.orig2
    awk '{
        if (++dup[$2] > 1) { 
            print $1, $2".duplicate".dup[$2], $3, $4, $5, $6 
        } else { 
            print $0 }
    }' data_chr${i}_filtered.bim.orig2 > data_chr${i}_filtered.bim
    grep "duplicate" data_chr${i}_filtered.bim | awk '{ print $2 }' > duplicates.txt
    plink1.90 --bfile data_chr${i}_filtered --remove duplicates.txt --make-bed --out data_chr${i}_filtered

done

# Merge them into one dataset

for i in {2..22}
do 
    echo "data_chr${i}_filtered"
done > mergefile.txt

plink1.90 --bfile data_chr1_filtered --merge-list mergefile.txt --make-bed --out data_filtered

```

If the last step crashes, resulting in a file called `data_filtered.missnp`, then this suggests there is an issue with your data. Please check that you have the same individuals present in each chromosome, and run the analysis again.

The result should be three files: `data_filtered.bed`, `data_filtered.bim`, `data_filtered.fam`. Copy them to `godmc/input_data` and set the following variable in your `config` file:

    bfile_raw="${home_directory}/input_data/data_filtered"


## Sample IDs

**IMPORTANT:** Please ensure that the second column of the `.fam` file (individual IDs) contains unique sample IDs. These sample IDs should be the IDs that are used in the phenotype, covariate, methylation and structural variants data. The first column of the `.fam` file (family IDs) is not important for these analysis. e.g. They can just be the same as the second column. Please also ensure that the individual IDs don't contain any underscores.
