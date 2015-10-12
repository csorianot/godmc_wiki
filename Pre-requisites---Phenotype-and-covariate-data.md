
## Covariates
Your covariates file should be a text file (white space separated) with `IID` in the first column followed by headed columns for covariates. The following covariates are **required**:

- `Sex`: A column of `1`s for males and `2`s for females
- `Age`: In years

The following covariates are strongly recommended (but not necessary):

- `Slide`: The slide ID for the methylation array
- Any other important batches

The pipeline will create genetic and methylation principal components for you so you don’t need to add PCs to this file. An example of what your file should look like is below:

    IID Sex Age Slide
    id1 2 30 12345678
    id2 1 42 12345678
    id3 1 76 87654321

## Phenotypes

We are expecting that you will provide `height` and `BMI` phenotypes. Please provide them in the same format as the covariates, e.g.:

    IID Height BMI
    id1 1.62 23.1
    id2 1.89 20.6
    id4 1.70 22.7


**IMPORTANT NOTE:** Any missing values for covariates or phenotypes will lead to the individual being excluded. Please ensure that this is kept to a minimum!