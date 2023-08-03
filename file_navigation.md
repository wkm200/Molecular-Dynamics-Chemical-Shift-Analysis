# Molecular-Dynamics-Chemical-Shift-Analysis
Analyze MD-derived (eg, SHIFTX2) versus NMR-derived chemical shifts. <br>

Recommended to go through all programs in the correct order if working on a new dataset: `str_to_csv.py` to `avg_shifts.py` (optional) to `offset_shifts.py` (optional) to `dCS.py` to `dCS_RMSE.py`. <br>

**Methods adapted from:** <br>
Shrestha, U.R., Smith, J.C. & Petridis, L. Full structural ensembles of intrinsically disordered proteins from unbiased molecular dynamics simulations. Commun Biol 4, 243 (2021). https://doi.org/10.1038/s42003-021-01759-1 <br><br>


## Data Extractor 

This required script converts BMRB NMR-STAR v3 .str files to .csv files formatted as RESNO,CA,CB,H,HA,HA2,HA3,N (same as output created by SHIFTX2). This format is necessary for all other scripts in this repository.

This code is necessary to convert all reference BMRB NMR files to a workable format. Please check the format of your MD-derived chemical shifts. SHIFTX and other servers often save in the format RESNO,CA,CB,H,HA,HA2,HA3,N. However, if they are saved as an NMR-STAR v3 .str file, use this script to convert it. 

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

This optional script calculates the average values per atom per residue, excluding missing values, for multiple CSV files containing chemical shift data in the format RESNO,CA,CB,H,HA,HA2,HA3,N. Since NMR chemical shifts are the average of several conformations existing in solution, it might be helpful to average MD-derived chemical shifts if they were used in different systems. Additionally, one may average several NMR-derived chemical shift values to account for protein status (eg, expression systems with different PTMs) or to account for reference error in CS measurement. 

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

This optional script calculates an offset constant between calculated chemical shift values (from molecular dynamics simulations) and experimental chemical shift values both stored in CSV files containing in the format RESNO,CA,CB,H,HA,HA2,HA3,N. It is based on a method described in the referenced paper, which accounts for the possible offset that may arise from the systematic or referencing error in the NMR CS measurement or calculation. It may or may not approve agreement between the simulated and experimental data. 

### Usage

To run this optional script, use the following command:

```
python offset_shifts.py <input_directory> <reference_directory> <output_directory>
```

- `input_directory` is the directory containing the calculated chemical shift CSV files (input files).
- `reference_directory` is the directory containing the experimental chemical shift CSV files (reference files).
- `output_directory` is the directory where the output files will be saved.
- During execution, the script will ask whether you want to save or print the plots and offset equations. 
  - If you choose to save the plots, they will be saved to an `offset_plots` folder within the output directory, with each plot titled as `<input_file>_vs_<reference_file>_<atom_type>.png`.
  - If you choose to save the offset equations, they will be saved to a CSV file located in an `offset_equations` folder within the output directory, with each row containing the input file name, reference file name, atom type, and offset equation.
  - The offset chemical shift data for each combination of input and reference file is saved to a separate CSV file in an `offset_chemicalshifts` folder within the output directory and is titled `os_<input_file>-_-<reference_file>`.

**Note:** The slope is restricted to 1 to account for a constant systematic offset between calculated and experimental chemical shifts, operating under the assumption that the discrepancy is due to a fixed bias, rather than a scaling or proportional difference.<br><br>



## ΔCS Calculator

This script is used to calculate ΔCS values from input chemical shift files and a random coil chemical shift file when both are CSV files containing chemical shift data in the format RESNO,CA,CB,H,HA,HA2,HA3,N. The ΔCS values are calculated as the difference between the input chemical shift values and the random coil chemical shift values.

$\Delta CS_{\text{{expt}}}(x, i) = CS_{\text{{expt}}}(x, i) - CS_{\text{{RC}}}(x, i)$ <br>
$\Delta CS_{\text{{calc}}}(x, i) = CS_{\text{{calc}}}(x, i) - CS_{\text{{RC}}}(x, i)$ <br>
where atom x ∈ CA,CB,H,HA,HA2,HA3,N and i refers to a specific amino acid. $CS_{\text{{calc}}}$, $CS_{\text{{expt}}}$, $CS_{\text{{expt}}}$ are the chemical shift values from the molecular dynamics simulation, NMR dataset, and random coil dataset respectively. 

Random coil values can be calculated using the [POTENCI webserver](https://st-protein02.chem.au.dk/potenci/). 

ΔCS gives you information on the local chemical environment for an atom and reflects secondary structure. Generally, CA > 0.7 or CB < 0.7 suggests the presence of an α-helix, while CB > 0.7 or CA < 0.7 suggests the presence of a β-sheet. Individual values are assumed to lack independence due to effect of neighboring atoms, which should be accounted for in any statistical analysis. 

### Usage

```
python dCS.py <input_shifts_directory> <random_coil_file> <output_directory>
```

- `<input_shifts_directory>`: The directory containing the input chemical shift CSV files.
    - Use a directory containing only $CS_{\text{{calc}}}$ files for $\Delta CS_{\text{{calc}}}$ or only $CS_{\text{{expt}}}$ files for $\Delta CS_{\text{{expt}}}$.
- `<random_coil_file>`: The path to the random coil chemical shift CSV file. 
- `<output_directory>`: The directory where the output ΔCS CSV files will be saved. The output files will be named 'dCS_<input_file>.csv'.

The output CSV files will contain the calculated ΔCS values for each residue number and atom type. 


