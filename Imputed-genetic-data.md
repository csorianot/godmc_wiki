### Genetic data

The genetic data must be
- Imputed to 1000 genomes reference panel, ideally phase 3. Phased haplotypes available [here](https://mathgen.stats.ox.ac.uk/impute/1000GP_Phase3.html). For guidelines on how to perform imputation see [here](http://genome.sph.umich.edu/wiki/IMPUTE2:_1000_Genomes_Imputation_Cookbook) and [here](https://github.com/explodecomputer/godmc/wiki/Genetic-imputation). A pipeline is also available [here](https://github.com/explodecomputer/imputePipePBS). Please contact us if you have any queries about this.
- Converted to best guess binary plink format
- Filtered to have MAF > 0.01 and imputation quality score > 0.8
- All remaining SNPs combined into a single fileset (*i.e.* not a separate fileset for each chromosome)

We also require imputation quality scores for each SNP. Some instructions on how to get imputed data into the desired format are below.


#### `Impute2` imputed data 

Assuming you have used [impute2](https://mathgen.stats.ox.ac.uk/impute/impute_v2.html) to perform the imputation and generated `bgen` dosage files, and generated info scores using the `-snp-stats` flag in [qctool](http://www.well.ox.ac.uk/~gav/qctool/#overview), then the last three steps can be performed using the following bash script.

It is recommended that you use `plink1.90` to do this, available for download [here](https://www.cog-genomics.org/plink2)

```bash

#!/bin/bash

for i in {1..22}
do

    # First extract the info scores (renaming any duplicated SNP IDs)
    # e.g. If using the output from the -snp-stats flag in qctool:

    sed 1d data_chr${i}.snp-stats | awk '{
        if (++dup[$2] > 1) {
            print $2".duplicate."dup[$2], $19 
        } else { 
            print $2, $19 
        }
    }' > data_chr${i}.info

    # Now extract the best guess data from the bgen files, filtering on info score and MAF

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
            print $1, $2".duplicate."dup[$2], $3, $4, $5, $6 
        } else { 
            print $0 }
    }' data_chr${i}_filtered.bim.orig2 > data_chr${i}_filtered.bim
    grep "duplicate" data_chr${i}_filtered.bim | awk '{ print $2 }' > duplicates.txt
    plink1.90 --bfile data_chr${i}_filtered --exclude duplicates.txt --make-bed --out data_chr${i}_filtered

done

# Merge them into one dataset

for i in {2..22}
do 
    echo "data_chr${i}_filtered"
done > mergefile.txt

plink1.90 --bfile data_chr1_filtered --merge-list mergefile.txt --make-bed --out data_filtered


# Create the correct info score format
# Column 1: SNP ID
# Column 2: MAF
# Column 3: Quality score

for i in {1..22}
do
    # Extract the relevant SNPs
    # Assumes column 2 is the SNP ID

    awk '{ print $2 }' data_chr${i}_filtered.bim.orig > chr${i}_filtered.snplist
    fgrep -wf chr${i}_filtered.snplist data_chr${i}.snp-stats > chr${i}_filtered.snp-stats


    # Relabel the SNP IDs and extract relevant columns
    # Assumes column 2 is the SNP ID
    # Assumes column 4 is the position
    # Assumes columns 7 and 8 are the allele names
    # Assumes column 15 is the MAF
    # Assumes columns 19 is the info score

    awk -v chr=$i '{
        if (length($7) == "1" && length($8) == "1") 
            print "chr"chr":"$4":SNP", $15, $19;
        else 
            print "chr"chr":"$4":INDEL", $15, $19;
    }' chr${i}_filtered.snp-stats > chr${i}_filtered.info

    # Remove duplicates

    awk '{ if (++dup[$1] == 1) { print $0 }}' > temp${i}
    mv temp${i} chr${i}_filtered.info

done

# Combine into a single file

for i in {1..22}
do
    cat chr${i}_filtered.info
done > data_filtered.info

```


The result should be three plink files: `data_filtered.bed`, `data_filtered.bim`, `data_filtered.fam`, and the info file: `data_filtered.info`. Copy them to `godmc/input_data` and set the following variable in your `config` file:

    bfile_raw="${home_directory}/input_data/data_filtered"

and

    quality_scores="${home_directory}/input_data/data_filtered.info"
    quality_type="impute2"


#### Sample IDs

**IMPORTANT:** Please ensure that the second column of the `.fam` file (individual IDs) contains unique sample IDs. These sample IDs should be the IDs that are used in the phenotype, covariate, methylation and structural variants data. The first column of the `.fam` file (family IDs) is not important for these analysis. e.g. They can just be the same as the second column. Please also ensure that the individual IDs don't contain any underscores.
