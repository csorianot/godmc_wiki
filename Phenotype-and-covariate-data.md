
## Covariates
Your covariates file should be a text file (white space separated) with `IID` in the first column followed by headed columns for covariates. The following covariates are **required**:

- `Sex`: A column of `M`s for males and `F`s for females
- `Age`: In years

The following covariates are strongly recommended (but not necessary):

- `Slide`: The slide ID for the methylation array
- Any other important batches

The pipeline will create genetic and methylation principal components for you so you don’t need to add PCs to this file. An example of what your file should look like is below:

    IID Sex Age Slide
    id1 F 30 12345678
    id2 M 42 12345678
    id3 M 76 87654321

Please ensure there are no missing values in the `Sex` and `Age` columns, even if your data is all the same age or all the same sex. Please also ensure that the capitalisation of column headers in the example above is followed.

## Phenotypes

We are expecting that you will provide `Height` and `BMI` phenotypes. Please provide them in the same format as the covariates, e.g.:

    IID Height BMI
    id1 1.62 23.1
    id2 1.89 20.6
    id4 1.70 22.7

Note the capitalisation!!

We are expecting height in metres and BMI in kg/m2 units.

**IMPORTANT NOTE:** Any missing values for covariates or phenotypes will lead to the individual being excluded. Please ensure that this is kept to a minimum!