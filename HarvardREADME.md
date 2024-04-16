# surplus-equipment
- This file will create a dataset from raw militarization data for both cities and counties. The scripts below are in numerical order: files 1-10 are cleaning files; files 11 and 12 contain the code to replicate both BG and HPBM; files 13 and 14 run the BG and HPBM models on their data, our county data, and our city data; file 15 creates the figures in the main body of the paper; files 16 and 17 create the figures and tables in the supplementary materials. This was run on R version 3.6.1 and 3.6.2. 

### Necessary R Packages
- `plm` for panel data models
- `lfe` for felm function
- `rlang` for formulas
- `tidyverse`; `Hmisc`; `readxl`; `haven`; `foreign`; `lubridate`; `reshape2`; `readstata13`; `readr`; `stringr`; `purrr`; `rlang` for data editing, import, and export
- `openintro` and `cdlTools` to convert state and county names
- `mgsub` for replacing multiple strings with na's 
- `mapproj`; `geosphere` for mapping and other distance calculations
- `broom`; `texreg`; `xtable` for table editing and creation in latex
- `ggpubr`; `cowplot`; `patchwork` for plotting

### lib
These are files with functions
- `arrest-functions.R`: this file takes the arrest data and cleans it for use in file 1 below
- `figures-functions.R`: this file contains the code to create the figures from the models
- `fix-names.R`: this file fixes city and county names for ease of merging across datasets
- `model-functions-BG.R`: this file contains the model functions for the BG replication and extension
- `model-functions-HPBM.R`: this file contains the model functions for the HPBM replication and extension

### 1-create_arrest_data.R

- Arrest Data for Agency level: Uniform Crime Reporting Program Data: Offenses Known and Clearances by Arrest, United States, 2016 (ICPSR 37061)
  
  - Using number of total offenses cleared by arrest
  
- Uses ORI codes from icpsr.umich.edu/icpsrweb/NACJD/studies/35158 
  
  - https://doi.org/10.3886/ICPSR35158.v2 to get City names, counties, states and FIPS merged with arrests
- file merges arrest data of individual agencies to an 
    a) agency-state-year dataset; total observations: 153,257 from years 2009-2015 
    b) city-state-year dataset: Total observations = 91,270 (this is 13,094 unique City/State pairs)
    c) and uses county-level crime data to create a county-state-year dataset: Total observations = 19,064 (3,177-3,178 unique county/state pairs) 
- output `arrests-by-agency.RDS`: agency-state-year dataset with 153,257 obs and 30 variables
- output `arrests-by-city.RDS`: an aggregated city-state-year dataset (aggregated from arrests-by-agency) with 91,270 obs and 8 variables
- output `arrest_county.RData`: a county-state-year dataset with 19,064 obs and 17 variables 

### 2-create_crime_data.R
- this takes the FBI Uniform Crime Reports data for cities and counties and creates two files: a county-state-year crime dataset and a city-state-year crime dataset
- output `crimerates.RData` with crime rates and crime numbers by city-state-year with 56,133 obs and 31 variables
- output `crime_county.RData` with crime rates and crime numbers by county-state-year with 15,683 obs and 18 variables

### 3-create_economic_data.R
- this merges in economic data for cities and counties, cleans it, and preps it for export and merging with variables later on
- output `econ.RData` with economic variables (median household income, unemployment, PCPI, percentinpoverty) by city-state-year with 175,654 obs and 8 variables
- output `econ_county.RData` with economic variables (median household income, unemployment, PCPI, percentinpoverty) by county-state-year with 18,856 obs and 10 variables

### 4-create_demographic_data.R
- this cleans and merges the demographic data for our analysis (population, percent Black, percent males, population 15-19; 20-24; 25-29) collected from the Census Bureau
- output `demog.RData`, a city-state-year dataset with 175,654 obs and 10 variables
- output `demog_county.RData`, a county-state-year dataset with 18,856 obs and 12 variables 

### 5-clean_city_data.R
- combines city level arrest, economic, crime, and demographic data
- output `city-crime-controls.RData`, a city-state-year dataset with 91,704 obs and 47 variables 

### 6-clean_county_data.R
- combines county level arrest, economic, crime, and demographic data
- output `county-crime-controls.RData` with 19,064 obs and 43 variables
 
### 7-create_militarization_data.R
- Input `DISP_AllStatesAndTerritories_03312018.xlsx` 
    - starts with 155,934 observations
    - output agency level `sum_LESO_agency.RData` (and `sum_LESO_agency.csv`) with 205,494 obs and 29 variables
    - output county level `sum_LESO_county.RData` (and `sum_LESO_county.csv`) with 70,412 obs and 29 variables 
    
### 8-create-HPBM-instruments.R
This file calculates the distance from each city and each county in our dataset to one of 18 disposition centers, as designated by DLA. This file recreates (to the best of our ability) the method described by Harris et al.

1. For calculating city distances load three datasets:
- Location of cities: The cities database is from the 2017 Gazetteer Files from the Census Bureau, found here: https://www.census.gov/geo/maps-data/data/gazetteer2017.html (this is also the source of the lat-long data for counties below)
  * file: `2017CensusPlaceLocations.csv`
  * some of these cities had duplicates which resulted in very different distance calculations. we dropped these
- final instrument is whether the city is a HIDTA (designated as a high intensity drug trafficking area):   * file: `HIDTAcitiesandcounties.csv`
  * Some cities had repeated entries with one being a HIDTA and one was not. Assume then it is a HIDTA.
- The DLA sites are from Harris.
  * `dlaDispSites.csv`
Calculate distances from cities: This approach calculates the Haversine distance in meters from each city to each distribution center. The Haversine distance is the shortest possible distance between the two points, i.e.as the crow flies.
  * Output: `cityHARRISinstruments.csv` with 29,324 obs and 25 variables
  
2. Calculate distances from counties: This approach calculates the Haversine distance in meters from each county to each distribution center. The Haversine distance is the shortest possible distance between the two points, i.e. as the crow flies.
- Input: datafile with lat/long of counties `2017CensusCountyLocations.csv`
- Output: `countyHARRISinstruments.csv` with 3,142 obs and 25 variables
- At the bottom of this file, we also compare our county distance measures with the ones in HPBM

### 9-create_city_aggregate.R

Joins militarization data at agency level with crime and control data:
- Input: `sum_LESO_agency.RData`
- Input: `city-crime-controls.RData`
- Input: `usmilex.csv` from https://www.sipri.org/databases/milex 
- Input: `cityHARRISinstruments.csv` Harris instruments from file 8
The crime and control data has 56,133 but many (32,022 observations) do not have LESO shipments. Mostly due to mismatch in time periods.

Create instruments needed for BG approach:
- milex_iv: This is divided by 6 (instead of 7 as in BG) because we only have data from 2010-2015

Create instruments we will need for the analysis for SUM US ITEMS as DV
- SumUSitems = the sum of all items given to all jurisdictions within a certain year.
- zHUdI1 = the interaction of the inverse distance to the closest distribution center with sumUSitems
- zHUdI6 = the interaction of the inverse distance to the sixth closest distribution center with sumUSitems
- zHUdIland = interaction between  the  availability  of  tactical  items  and  the  jurisdiction’s  land  area  
- zHUdIhidta = interaction between sumUSitems and  an  indicator  for  having  ever  been  designated  as  a  HIDTA

Create instruments we will need for the analysis for SUM US VALUE as DV
- SumUSvalue = the sum of all items given to all jurisdictions within a certain year.
- zHUd1 = the interaction of the inverse distance to the closest distribution center with sumUSitems
- zHUd6 = the interaction of the inverse distance to the sixth closest distribution center with sumUSitems
- zHUdland = interaction between  the  availability  of  tactical  items  and  the  jurisdiction’s  land  area  
- zHUdhidta = interaction between sumUSitems and  an  indicator  for  having  ever  been  designated  as a HIDTA

**final output**: `city-aggregate.RData` with 48,122 obs and 144 variables
- this also outputs a stata file (`agency.dta`) to be able to calculate f-statistics

### 10-create-county-aggregate.R

- Input: `sum_LESO_agency.RData` county level militarization
- Input: `usmilex.csv` from https://www.sipri.org/databases/milex 
- Input: `county-crime-controls.RData` county level control variables 
- Input: `countyHARRISinstruments.csv` Harris instruments from file 8
- CREATE INSTRUMENTS WE WILL NEED FOR THE ANALYSIS FOR SUM US ITEMS AS DV
  * SumUSitems = the sum of all items given to all jurisdictions within a certain year.
  * zHUdI1 = the interaction of the inverse distance to the closest distribution center with sumUSitems
  * zHUdI6 = the interaction of the inverse distance to the sixth closest distribution center with sumUSitems
  * zHUdIland = interaction between  the  availability  of  tactical  items  and  the  jurisdiction’s  land  area  
  * zHUdIhidta = interaction between sumUSitems and  an  indicator  for  having  ever  been  designated  as  a  HIDTA
- CREATE INSTRUMENTS WE WILL NEED FOR THE ANALYSIS FOR SUM US ITEMS AS DV. --------
  * SumUSitems = the sum of all items given to all jurisdictions within a certain year.
  * zHUdI1 = the interaction of the inverse distance to the closest distribution center with sumUSitems
  * zHUdI6 = the interaction of the inverse distance to the sixth closest distribution center with sumUSitems
  * zHUdIland = interaction between  the  availability  of  tactical  items  and  the  jurisdiction’s  land  area  
  * zHUdIhidta = interaction between sumUSitems and  an  indicator  for  having  ever  been  designated  as  a  HIDTA
  
Create instruments needed for BG approach:
- milex_iv: This is divided by 5 (instead of 7 as in BG) because we only have data from 2010-2014


**final output**: `county-aggregate.RData` with 15,685 obs and 133 variables
- this also outputs a stata file (`county.dta`) to be able to calculate f-statistics

### 11-B_and_G-replication
- This file completely replicates the results (tables and Figures) from the B&G paper using their data. We save results from their analyses to later combine with our results.
- input `militarization.dta` this is from B&G (https://www.aeaweb.org/articles?id=10.1257/pol.20150478 and the ICPSR link: https://www.openicpsr.org/openicpsr/project/114670/version/V1/view)
- input `police.dta` this is from B&G
- output `BG_milit.RData`, an edited version of the militarization dataset above with calculated IV's for analysis; 24,601 obs with 263 variables

### 12-HPBM_replication.R
 - Table 1 replicated from `aggregates.dta` file
 - Input `HPBM-AEJ-1033.dta` from HPBM (aeaweb.org/articles?id=10.1257/pol.20150525 and the ICPSR link: https://www.openicpsr.org/openicpsr/project/114674/version/V1/view)
 - output `harris.RData`, an edited version of the dataset above; 43,512 obs with 206 variables 
 - This file replicates all tables (Tables 1 through 10) in HPBM
  
### 13-run_BG_models.R
This file re-runs BG results and our replication of BG at agency and county level and stores all of the results
- Input: `city-aggregate.RData` for our city analyses
- Input: `county-aggregate.RData` for our aggregated county analyses
- Input: `BG_milit.RData` for the data used in BG for exact replication (from `replication-B_and_G.R`)

1. Agency level of BG and reduced form
2. County level of BG and reduced form
3. Direct replcation of BG

- Output: `results-BG-for-tables.RData` which is the data used to create the tables in our main paper and supplemental analyses
- This file also reproduces subsets of the BG data (for controlled items only; our year subset; keeping city outliers included; and dropping problematic crime observations)

- For controlled items set to TRUE; for the year subset set to TRUE; for city outliers included set to FALSE; for the exclusion of the problematic crime observations set to TRUE 

### 14-run_HPBM_models.R
This file re-runs HPBM results and our replication of HPBM at agency and county level and stores all of the results
- Input: `city-aggregate.RData` for our city analyses
- Input: `county-aggregate.RData` for our aggregated county analyses
- Input: `harris.RData` for the data used in BG for exact replication (from `replication-HPBM.R`)

1. Agency level of HPBM and reduced form
2. County level of HPBM and reduced form
3. Direct replicatioin of HPBM

- Output: `results-HPBM-for-tables.RData` which is the data used to create the tables in our main paper and supplemental analyses
- This file also reproduces subsets of the HPBM data (for controlled items only; our year subset; keeping city outliers included; and dropping problematic crime observations) 
- For controlled items set to TRUE; for the year subset set to TRUE; for city outliers included set to FALSE; for the exclusion of the problematic crime observations set to TRUE; to scale the HPBM instruments, set to TRUE

### 15-create_figures_main.R

- Output: Fig 1 map of militarization equipment by city, 2010 to 2015 (`citiesmap.pdf`)
  * Input: `city-aggregate.RData` and `2017CensusPlaceLocations.csv` for militarization data and latitude and longitude for plotting, respectively
- Output: Fig 2 Item discrepancies between LESO and NPR FOIA data (`NSNdifferencesaggandyear.pdf`)
  * Input: `DISP_AllStatesAndTerritories_03312018.xlsx`; `clean-countiesandfips.csv`; `LESO data - all states.csv` for raw LESO data, agencies geocoded to counties, NPR FOIA data, respectively
  * Additional Output: SI Figure 8 showing value discrepancies within NSN (`NSNdifferencesvariation.pdf`)
- Output: Fig 3 Replication BG (`plot7by2_facets_BG.pdf`)
  * Input: `results-BG.RData` for BG replication results; our county results; our city results
  * Additional Output: `allBG_results.RData` for creating figure captions for the plot
- Output: Fig 4 Replication HPBM (`plot7by2_facets_harris.pdf`)
  * Input: `results-HPBM.RData` for HPBM replication results; our county results; our city results
  * Additional Output: `allHPBM_results.RData` for creating figure captions for the plot

### 16-create_SI_tables.R 

- Output: Supplementary Table 1: The Effect of Military Aid on Substantiated Crime Rates (Table 2 in BG) (`BGtable2-partA-new.tex`; `BGtable2-partB-new.tex`; `BGtable2-partC-new.tex`)
  * Input: `results-BG-for-tables.RData` from file 11
- Output: Supplementary Tables 2-4: BG first stage and reduced form (`BG-BG-reduced-form.tex`, `BG-county-reduced-form.tex`, `BG-agency-reduced-form.tex`)
  * Input: `results-BG-for-tables.RData`
- Output: Supplementary Table 5: The Effect of Receiving Tactical Items on Substantiated Crime Rates (Table 8 in HPBM) (`HPBMtable8-partA1-new.tex`, `HPBMtable8-partA2-new.tex`, `HPBMtable8-partB1-new.tex`, `HPBMtable8-partB2-new.tex`)
  * Input: `results-HPBM-for-tables.RData`
- Output: Supplementary Tables 6-8: HPBM first stage and reduced form (`HPBM-HPBM-reduced-form.tex`, `HPBM-county-reduced-form.tex`, `HPBM-agency-reduced-form.tex`)
  * Input: `results-HPBM-for-tables.RData` 
- Output: Supplementary Table 10: Equivalence test
- Output: Supplementary Table 11: BG results for controlled items only (`BGtable2-partA-new-controlled.tex`, `BGtable2-partB-new-controlled.tex`, `BGtable2-partC-new-controlled.tex`)
  * Input: `results-BG-for-tables-controlled.RData`
- Output: Supplementary Table 12: HPBM results for controlled items only (`HPBMtable8-partA1-controlled.tex`, `HPBMtable8-partA2-controlled.tex`, `HPBMtable8-partB1-controlled.tex`, `HPBMtable8-partB2-controlled.tex`)
  * Input: `results-HPBM-for-tables-controlled.RData`
- Output: Supplementary Table 13: BG subset analysis, 2010 to 2012 (`BGtable2-partA-new-subset.tex`, `BGtable2-partB-new-subset.tex`, `BGtable2-partC-new-subset.tex`)
  * Input: `results-BG-for-tables-year-subset.RData`
- Output: Supplementary Table 14: HPBM subset analysis, 2010 to 2012 (`HPBMtable8-partA1-subset.tex`, `HPBMtable8-partA2-subset.tex`, `HPBMtable8-partB1-subset.tex`, `HPBMtable8-partB2-subset.tex`)
  * Input: `results-HPBM-for-tables-year-subset.RData`
- Output: Supplementary Table 15: BG analysis, no outliers eliminated (`BGtable2-partA-new-outliers.tex`, `BGtable2-partB-new-outliers.tex`, `BGtable2-partC-new-outliers.tex`)
  * Input: `results-BG-for-tables-with-outliers.RData`
- Output: Supplementary Table 16: HPBM analysis, no outliers eliminated (`HPBMtable8-partA1-outliers.tex`, `HPBMtable8-partA2-outliers.tex`, `HPBMtable8-partB1-outliers.tex`, `HPBMtable8-partB2-outliers.tex`)
  * Input: `results-HPBM-for-tables-with-outliers.RData` 
- Output: Supplementary Table 17: BG analysis, dropping problematic crime observations (`BGtable2-partA-new-crime-obs-dropped.tex`, `BGtable2-partB-new-crime-obs-dropped.tex`, `BGtable2-partC-new-crime-obs-dropped.tex`)
  * Input: `results-BG-for-tables-crime-obs-dropped.RData` 
- Output: Supplementary Table 18: HPBM analysis, dropping problematic crime observations (`HPBMtable8-partA1-crime-obs-dropped.tex`, `HPBMtable8-partA2-crime-obs-dropped.tex`, `HPBMtable8-partB1-crime-obs-dropped.tex`, `HPBMtable8-partB2-crime-obs-dropped.tex`)
  * Input: `results-HPBM-for-tables-crime-obs-dropped.RData`
- Output: Supplementary Table 19: Alternative approach to calculating quantity and value for HPBM
- Output: Supplementary Table 20: summary statistics (`summary-stats.tex`)
  * Input: `BG_milit.RData`; `harris.RData`; `city-aggregate.RData`; `county-aggregate.RData`
- Output: Supplementary Table 21: Summary stats restricted years (`summary-stats-subset.tex`)
  * Input: `BG_milit.RData`; `harris.RData`; `city-aggregate.RData`; `county-aggregate.RData` (all are then filtered to 2010 to 2012 alone)

### 17-create_SI_figures.R 

- Output: Supplementary Figure 1: Distribution of logged total value of yearly transfers (`DistOfAidPerCap.pdf`)
  * Input: `city-aggregate.RData`
- Output: Supplementary Figure 2: Total crime rate per 100,000 and annual logged military transfers (`crimeAidCoplot.pdf`)
  * Input: `city-aggregate.RData`
- Supplementary Figure 3 is a screenshot of our FOIA request to the DLA
- Output: Supplementary Figures 4-7: mapping the differences between BG and HPBM (`bg_harris_map_comparison.pdf`, `bg_harris_map_comparisonNOABS.pdf`, `bg_harris_map_comparisonaverage.pdf`, `bg_harris_map_comparisonNOABSaverage.pdf`)
  * Input: `militarization.dta` (BG source data) and `HBPM-AEJ-fips-unlocked.dta` (HPBM source data)
- Supplementary Figure 8 is created in file 15 (the item value variance)
- Output: Supplementary Figure 9: visualization of BG first stage coefficients (`first-stage-plot-BG.pdf`)
  * Input: `results-BG.RData` 
- Output: Supplementary Figure 10: visualization of HPBM first stage coefficients (`figures/first-stage-plot-HPBM-facet.pdf`)
  * Input: `results-HPBM.RData`  
