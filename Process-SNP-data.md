Once the imputed genetic data is in the [correct format](Imputed genetic data) and in the [correct location](Install and setup), the following tasks need to be performed:

- Extract relevant IDs
- HWE and MAF QC
- Re-code SNP IDs
- Re-code INDEL alleles to I,D
- Remove duplicate SNPs
- Construct kinship matrices
- Remove PC outliers
- Remove any cryptic relatedness for 'unrelated' data
- Create a pedigree matrix for 'related' samples
- Calculate principal components
- Strand check
- Flip strand for misaligned SNPs and remove SNPs with mismatched alleles

To do this, run the following script:

    ./02a-snp_data.sh

For a sample size of 100 this script takes less than 5 minutes to run on our computers or 10 minutes on 1800 samples. The relevant files should have been generated and saved in `processed_data/genetic_data`.

Note that for 'unrelated' samples we are generating PCs by removing long LD tracts, extracting HapMap3 SNPs, LD pruning, and then calculating PCs on the data after cryptic relateds have been removed. For 'related' samples we are using a general method for estimating PCs that accounts for any relatedness. This involves estimating a kinship matrix that is robust to population structure and relatedness ([Manichaikil et al 2010](http://bioinformatics.oxfordjournals.org/content/26/22/2867.long)), estimating principal components on an unrelated subset, and then projecting principal components onto the related subset based on estimated kinships ([Conomos et al 2015](http://onlinelibrary.wiley.com/doi/10.1002/gepi.21896/abstract)).

Please check your pca plot `pcaplot.pdf` in the results directory and adjust the SD threshold in the `config` file and rerun `01-check_data.sh` and `02a-snp_data.sh` if necessary. The blue dashed lines define the SD threshold.

    cd results/02

Next, we need to generate the correct format for the MeQTL analysis software (MatrixeQTL). To do this run the following script form your home directory:
   
    ./02b-convert_snp_format.sh

This script will split the genetic dataset into a number of smaller chunks. The number of chunks is determined by the `genetic_chunks` variable in the `config` file. For example:

    genetic_chunks="1000"

is the default and will split all the SNP data into 1000 subsets (e.g. for 8 million SNPs each chunk will have 8000 SNPs). The required files will be saved in `processed_data/genetic_data/tabfile/`. In addition, this script will do the same for the CNV data.

This script takes a few minutes to run for sample size of 100 and 1hour to run ~1800 samples.


### Upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 02 check
```

This should tell you that `Section 02 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload.sh 02 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 02.

This procedure will be repeated at the end of each section.