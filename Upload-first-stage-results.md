At this stage hopefully all the first stage analyses are completed. We have created some scripts to:

- check that the results look OK
- generate compressed results files
- upload the files to the server

### Checking the analysis

To perform the check, run:

    ./08a-check_results.sh

It will go through the log files and the results files checking that the output is as expected. If there are any problems flagged then please go back to that analyses and re-run. Do contact us if there are any concerns or questions about this.


### Creating files for upload

To generate log file and result file archives:

    ./08b-collect_results.sh

This will create two files: `uploads_results.tgz` and `uploads_logfiles.tgz`.


### Uploading files

    ./08c-upload_results.sh

