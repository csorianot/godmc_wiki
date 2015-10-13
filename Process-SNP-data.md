Once the imputed genetic data is in the [correct format](Imputed genetic data) and in the [correct location](Install and setup), the following tasks need to be performed:

- Extract relevant IDs
- HWE and MAF QC
- Re-code SNP IDs
- Construct kinship matrices
- Calculate principal components
- Remove PC outliers
- Remove any cryptic relatedness for 'unrelated' data
- Create a pedigree matrix for 'related' samples

To do this, run the following script:

    ./03a-snp_data.sh

For a sample size of 100 this script takes less than 5 minutes to run on our computers. The relevant files should have been generated and saved in `processed_data/genetic_data`


Next, we need to generate the correct format for the MeQTL analysis software (MatrixeQTL). To do this run the following script:

    ./04a-convert_snp_format.sh

The required files will be saved in `processed_data/genetic_data`. This script takes a few minutes to run for sample size of 100.