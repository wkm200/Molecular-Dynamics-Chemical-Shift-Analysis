# Molecular-Dynamics-Chemical-Shift-Analysis
Analyze MD-derived versus NMR-derived chemical shifts <br><br>

**Methods adapted from:** <br>
Shrestha, U.R., Smith, J.C. & Petridis, L. Full structural ensembles of intrinsically disordered proteins from unbiased molecular dynamics simulations. Commun Biol 4, 243 (2021). https://doi.org/10.1038/s42003-021-01759-1 <br><br>


## Data Extractor 

This required script converts BMRB NMR-STAR v3 .str files to .csv files formatted as RESNO,CA,CB,H,HA,HA2,HA3,N (same as output created by SHIFTX2). This format is necessary for all other scripts in this repository.

### Usage

To run this script, use the following command: 

```
python str_to_csv.py <input_directory> <output_directory> <desired_starting_RESNO>
```

- `input_directory` is the directory containing the CSV files with simulation data.
- `output_directory` is the directory where the output file will be saved.
- `desired_starting_RESNO` is an optional specifier that will change the starting residue number.

**Note:** This script will iterate through all .str files in input directory and convert each to shift list .csv. <br><br>


## Average Calculator

This optional script calculates the average values per atom per residue, excluding missing values, for multiple CSV files containing chemical shift data in the format RESNO,CA,CB,H,HA,HA2,HA3,N. 

### Usage

To run this optional script, use the following command:

```
python avg_shifts.py <input_directory> <output_directory>
```

- `input_directory` is the directory containing the CSV files with simulation data.
- `output_directory` is the directory where the output file will be saved.
- During execution, the script will print a numbered list of each file in the input directory available for averaging. Type the file numbers you would like to be used for the calculation when prompted, then press enter. 

The output file name includes all of the names of the input files that were used, separated by `_-_` and starts with `avg_`. 

**Note:** The script reads each selected CSV file, groups the data by `RESNO`, calculates the sum and count for each atom (`CA`, `CB`, `H`, `HA`, `HA2`, `HA3`, `N`), and adds these values to cumulative sum and count DataFrames. It then calculates the average values per atom per residue by dividing the cumulative sum DataFrame by the cumulative count DataFrame, and saves these average values to the output CSV file.<br><br>


## Offset Calculator

This optional script calculates an offset constant between calculated chemical shift values (from molecular dynamics simulations) and experimental chemical shift values. It is based on a method described in the referenced paper, which accounts for the possible offset that may arise from the systematic or referencing error in the NMR CS measurement or calculation. It may or may not approve agreement between the simulated and experimental data. 

### Usage

To run this optional script, use the following command:

```
python offset_calculator.py <input_directory> <ref_directory> <output_directory>
```

- `input_directory` is the directory containing the calculated chemical shift CSV files (input files).
- `ref_directory` is the directory containing the experimental chemical shift CSV files (reference files).
- `output_directory` is the directory where the output files will be saved.
- During execution, the script will ask whether you want to save or print the plots and offset equations. 
  - If you choose to save the plots, they will be saved to an `offset_plots` folder within the output directory, with each plot titled as `<input_file>_vs_<ref_file>.png`.
  - If you choose to save the offset equations, they will be saved to a CSV file within the output directory, with each row containing the input file name, reference file name, atom type, and offset equation.
  - The offset data for each combination of input and reference file is saved to a separate CSV file within the output directory.

**Note:** The slope is restricted to 1 to account for a constant systematic offset between calculated and experimental chemical shifts, operating under the assumption that the discrepancy is due to a fixed bias, rather than a scaling or proportional difference.<br><br>


