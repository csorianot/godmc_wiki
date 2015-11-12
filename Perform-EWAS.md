An EWAS will be performed on each of the phenotypes provided in the `phenotypes` file. For example, if you have provided a file with `Height` and `BMI` columns, then each of these will be analysed. To perform the EWAS run:

    ./08-ewas.sh

It uses pre-normalised and pre-adjusted methylation levels and phenotypes, so it is very quick to run. Please check that things look as expected by visually inspecting the Q-Q plots created in the `results/08/` directory.


### Upload the results

To check that everything ran successfully, please run:

```
./check_upload 08 check
```

This should tell you that `Section 08 has been successfully completed!`. Now please upload the scripts like this:

```
./check_upload 08 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from section 08.

This procedure will be repeated at the end of each section.