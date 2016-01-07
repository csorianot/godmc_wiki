We have created a script which will check the data you have deposited and the `config` file parameters to make sure it all looks as expected. 
It checks:
 - necessary R libraries are present 
 - creates a list of IDs that are in common between the different data sources 
 - a number of graphs to visualise the raw data. 

To run:

    ./01-check_data.sh

**PLEASE NOTE:** It is important to monitor this script as it runs - it will stop with errors if it encounters problems. Please fix any errors/warnings that are encountered, and re-run the script until it completes without errors.

The script produces the following plots:

- the number of SNPs per chromosome
- the SNP quality metrics
- age distributions

Please visually check that these look as expected, they are located in the `results/01/` directory. 

It also generates summary statistics of the input data (`cohort_descriptives.RData`). Please check there are no NAs in there.

### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 01 check
```

This should tell you that `Section 01 has been successfully completed!`. Now please go to the Results directory and check your results.
```
cd results/01
```


Now please go back to the godmc directory and upload the scripts like this:

```
./check_upload.sh 01 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 01`.

This procedure will be repeated at the end of each section.