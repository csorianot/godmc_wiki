An EWAS will be performed on each of the phenotypes provided in the `phenotypes` file. For example, if you have provided a file with `Height` and `BMI` columns, then each of these will be analysed. To perform the EWAS run:

    ./06-ewas.sh

It uses pre-normalised and pre-adjusted methylation levels and phenotypes, so it is very quick to run. Please check that things look as expected by visually inspecting the Q-Q plots created in the `log_files/ewas/` directory.