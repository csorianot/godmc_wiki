For the EWAS we need to normalise the BMI and height phenotypes. The following will be done:

- Normalise height using inverse-normal transformation separately for males and females
- Adjust height for age and age squared (if they are nominally associated)
- Perform the same for BMI

To do this, run:

    ./02-phenotype_data.sh

This script also generates plots of the phenotypes.