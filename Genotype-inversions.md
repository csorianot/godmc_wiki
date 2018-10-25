Chromosomal inversions generate different haplotypes. Thus, standard chromosomes have haplotypes different from the haplotypes found in inverted chromosomes. Thus, we can use haplotype differences to get the inversions status of an individual. We have developed scoreInvHap, an R package available at Bioconductor, which gets the inversion status of an individual based on its genotypes. scoreInvHap classifies individuals based on reference haplotypes generated using individuals from European ancestry. 

**Important**: Inversion haplotypes vary depending on the ancestry, **we will only consider individuals of European ancestry**. Therefore, we ask the cohorts to **remove individuals of non-European ancestry from the genotype data**. If you have any doubts, please, contact the leading group. 

## Inversion genotyping
Inversion genotyping involves the following steps:

- Subset genetic data to SNPs present in inversion references
- Annotate SNPs to dbSNP identifiers
- Get inversion genotypes with scoreInvHap

To do this, run the following script:

    ./20-genotypeInversion.sh

For a sample size of 400 this script takes around 15 minutes to run on our computers. The relevant files should have been generated and saved in `processed_data/inversions`.
