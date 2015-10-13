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

    ./03b-convert_snp_format.sh

This script will split the genetic dataset into a number of smaller chunks. The number of chunks is determined by the `meth_chunks` variable in the `config` file. For example:

    meth_chunks="100"

is the default and will split all the SNP data into 100 subsets (e.g. for 8 million SNPs each chunk will have 80000 SNPs). The required files will be saved in `processed_data/genetic_data`. In addition, this script will do the same for the CNV data.

This script takes a few minutes to run for sample size of 100.