To run the association between inversions and BMI:

    ./27-inv_gwas_BMI.sh
    
    
### Now upload the results

To check that everything ran successfully, please run:

```
./check_upload.sh 27 check
```

This should tell you that `Section 27 has been successfully completed!`. Now please upload the results like this:

```
./check_upload.sh 27 upload
```

It will make sure everything looks correct and connect to the sftp server. It will request your password (this should have been provided to you along with your username). Once you have entered your password it will upload the results files from `section 27`.

This procedure will be repeated at the end of each section.

## Inversions module contact

If you have any issues or questions about inversion pipeline then please contact us: [carlos.ruiz@isglobal.org](mailto:carlos.ruiz@isglobal.org)

