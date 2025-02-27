[Data Setup](../)  

# Flowsa - Local Industry Lists

We ran the following test to confirm that Flowsa has gaps inherited from the BLS data.  We're using [Eckert County Business Patterns data](https://github.com/modelearth/community-data/tree/master/process/cbp) instead.

Here's our [Google CoLab for exploring NAICS in FLOWSA](https://colab.research.google.com/drive/1HLK4HIUMLlgTR524QoCKvfaNl-La48XU?usp=sharing), create a pipeline step that generates .csv files for the US and individual states using BLS data pulled from the [EPA's Flowsa API](https://github.com/USEPA/flowsa). Create 3 files per state and for the US. One for 2-digit naics, one for 4-digits and one for 6-digits.  Output .csv files with the following [simple column names](https://drive.google.com/drive/u/0/folders/1EoWDvNoaKO8xLclX4fr5exw83jJkkJIy):  


Save [state data files](https://github.com/modelearth/community-data/tree/master/us/state) in the same folder structure as our prior process. The prior BEA data prep process resides at the bottom of the current page. Reuse existing [process for state folders](https://github.com/modelearth/community-data/tree/master/process/python/bea).

For the entire US, output 3 similar files containing 2, 4 and 6-digit naics values in the folder called "all" located at [community-data/us/all](https://github.com/modelearth/community-data/tree/master/us/all).  

Update this [USEEIO_indicators.ipynb Python](https://github.com/modelearth/io/blob/master/charts/bubble/data/USEEIO_indicators.ipynb) used to generate indicators\_sectors\_GA and indicators\_sectors to use the USEEIO 2.0 API.  


## BLS Data Preparation 

The steps below have been applied within our [Google CoLab for exploring NAICS](https://colab.research.google.com/drive/1HLK4HIUMLlgTR524QoCKvfaNl-La48XU?usp=sharing) which can be used to output 6-digit naics by state to a [CSV file](https://drive.google.com/drive/u/0/folders/1EoWDvNoaKO8xLclX4fr5exw83jJkkJIy) called naics-6digit-ga.csv on Google Drive. We've manually output [3 years](https://github.com/modelearth/community-data/tree/master/us/state/GA/naics) using the CoLab script.  The original Python also resides in our [FLOWSA fork](https://github.com/modelearth/flowsa/tree/master/colabs).  

The U.S. Bureau of Labor Statistics (BLS) data is pulled using the [FLOWSA Python script](https://github.com/USEPA/flowsa/blob/master/flowsa/data_source_scripts/BLS_QCEW.py)
maintained by Catherine Birney.
<!--Check if 2017 has been added to master crosswalk  -->

Also [Compare to FactFinder](https://data.census.gov/cedsci/table?g=0400000US13&n=336111&tid=CBP2018.CB1800CBP&hidePreview=true) - Add notes here  

**Flowsa Wiki**  
[Install & Run](https://github.com/USEPA/flowsa/wiki)  
How Flowsa data is prepared - Creating a FlowByActivity Dataset

If you change BLS_QCEW.py, do so in our [FLOWSA fork](https://github.com/modelearth/flowsa).

Check for 6-digit 336111 automobile industry NAICS when outputting using FLOWSA.  
Note that only 4-digit NAICS resides in "By-Industry" in [BLS downloadable files](https://www.bls.gov/cew/downloadable-data-files.htm).  

<br>


# FLOWSA FlowByActivity

Note: The BLS QCEW FlowByActivity datasets include county level data.  
You can use the sample code below to retrieve the datasets.  


Clone using Github Desktop, or run from the terminal:

	pip install git+https://github.com/USEPA/flowsa

	 

In a python IDE (like [Pycharm](https://www.jetbrains.com/pycharm/)) run in a "Python Console", not in a regular Pycharm terminal console.  

If you try to run in the regular command terminal, you'll get `import: command not found`  

IMPORTANT: If using Pycharm, first go to: Tools > "Python or Debug Console"  

	import flowsa

	 

If you want to get the raw data in our flow by activity format. You can subset the data by ‘Employment’ (for number of employees),  ‘Money’ for annual wages and ‘Other’ for number of establishments, or you can set flowclass to any combination of those three. 

	 

	df = flowsa.getFlowByActivity(flowclass=['Employment', 'Money', 'Other'], years=[2018], datasource="BLS_QCEW")

	 

Returns a pandas dataframe that you can subset by NAICS sector in this case it will be the ActivityProducedBy.  

[See the format reference table](https://github.com/USEPA/flowsa/blob/master/format%20specs/FlowByActivity.md) - filter by Location using a county FIPS  as a 5 digit code, e.g. 13001 for Appling County, Georgia.  

You can also create a df for multiple years. There is data for 2010 – 2018. Example:  

	df = flowsa.getFlowByActivity(flowclass=['Employment', 'Money', 'Other'], years=[2015, 2016], datasource="BLS_QCEW")


## Output to CSV

#### # Optional, set the csv output path to …\flowsa\flowsa\output\FlowByActivity
##### # You'll need to manually create a folder at "output\FlowByActivity"

	from flowsa.common import fbaoutputpath  

 
##### # Define parameters for getFlowByActivity

	fc=['Employment', 'Money', 'Other']

	years=[2018]

	ds="BLS_QCEW"

 
##### # Load the FlowByActivity as a parquet (a columnar storage format) - Use one:

	df = flowsa.getFlowByActivity(flowclass=fc, years=years, datasource=ds)

	df = flowsa.getFlowByActivity(flowclass=fc, years=years, datasource=ds, geographic_level='national')

	df = flowsa.getFlowByActivity(flowclass=fc, years=years, datasource=ds, geographic_level='state')

##### # Create new column with first two digits of Location representing state

	df = df.assign(StateFIPS=df['Location'].apply(lambda x: x[0:2]))

##### # Limit the states to Alabama and Georgia (fips 1000 and 13000)

	df_state_sub = df[df['StateFIPS'].isin(['01', '13'])].reset_index(drop=True)

#### # TO DO use the merge function to combine 'Employment', 'Money' and 'Establishments' into one row.

	# Add Function Here
	# Simplify column names to: fips, naics, employees, wages and firms

##### # Limit output to specific columns and rename the column names

	df_col_sub = df[['ActivityProducedBy', 'ActivityConsumedBy', 'Location']]

	df_col_sub = df_col_sub.rename(columns={'ActivityProducedBy': 'APB', 'ActivityConsumedBy': 'ACB'})

##### # To output a file for each US state, loop through the 50 state names.

	state_fips = df['StateFIPS'].unique()

	for s in state_fips:

	    df_name = 'df_' + s

    	vars()[df_name] = df[df['StateFIPS'] == s].reset_index(drop=True)

\# You can link the data to state names using [this function](https://github.com/USEPA/flowsa/blob/master/flowsa/common.py#L374)


##### # Check if employment data is available for any of Georgia's 9 automobile manufacturers (naics 336111)

	df_ga_naics = df[df['StateFIPS'].isin(['13'])]

	df_ga_naics = df_ga_naics[df_ga_naics['Class'] == 'Money']

	df_ga_naics = df_ga_naics[df_ga_naics['ActivityProducedBy'] == '336111'].reset_index(drop=True)

##### # Limit to 6-digit naics, or loop through 2,3,4,5 and 6 to create separate files.

	df_naics6 = df[df['ActivityProducedBy'].apply(lambda x: len(x) == 6)].reset_index(drop=True)


##### # Set output path. Modify to save file elsewhere.

	file_path = fbaoutputpath

##### # Set file name

	file_name = ds + '_' + '_'.join(map(str, years)) + '.csv'


##### # Save dataframe as .csv file.

	df.to_csv(file_path + file_name, index=False)

#### # TO DO - Add script here to save in [community-data repo state folders](https://github.com/modelearth/community-data/tree/master/us/state).  

#### # TO DO - Fix the following so we can see where CSV file is saved.  
#### # Once working, place this before running `df = flowsa.getFlowByActivity`above:

	from flowsa.common import fbaoutputpath

	import log

	log.info(“CSV file saved at " +  file_path + file_name)



<br>

## For Comparison to prior year

TO DO - Additional columns are available to compare to the prior year.  

oty_annual_avg_emplvl_chg  
oty_annual_avg_estabs_chg  
oty_total_annual_wages_chg 

See: [Annual Average Data Slide Layout table](https://data.bls.gov/cew/doc/access/csv_data_slices.htm)  

When Catherine Birney modified the data to include the columns for changes from prior years, pycharm couldn’t handle the size of the files and kept crashing. To include that data, Catherine says you'd need to modify the code in BLS_QCEW.py.  



## Add Zip Code level BLS Data to FLOWSA

TO DO - Explore options for <a href="../../../../community/industries/">zip code level data</a>.


<br>



# BEA GDP by county

[BEA GDP by county](https://www.bea.gov/system/files/2019-12/One_Page_Methodology_v6.pdf) - Could help reconcile Census CBP data with BEA IO data.  

[Detailed methodology](https://apps.bea.gov/scb/2020/03-march/0320-county-level-gdp.htm)  

[GDP by County, Metro, and Other Areas - Main Page](https://www.bea.gov/data/gdp/gdp-county-metro-and-other-areas)  

[Personal Income by County, Metro, and Other Areas (BEA)](https://www.bea.gov/data/income-saving/personal-income-county-metro-and-other-areas)  


## BEA Data Preparation (Discontinued) 

[Prior Data Processing Python Script for BEA data](https://github.com/modelearth/community-data/tree/master/process/python/bea) was edited locally using [Anaconda Jupyter Notebook](https://jupyter.org/install)  

Methodology Discontinued: Some industries lacked payroll estimates at both the county and state level.  This occurs for approximately 80 of 388 industries in Georgia. For example, payroll for Georgia's 9 automotive manufacturers is not included in the U.S. Bureau of Economic Analysis (BEA) industry data. US census privacy protection rules omit company payroll when only 1 or 2 establishments reside in a county.  

The census publishes payroll and employee counts by county for industries with 3 or more establishments. Sometimes industries with up to 27 establishments are also unpublished. Values are available as aggregates within a state total for each industry.  

To fill gaps, we generate an industry estimate for each year in a range of 5 years, then average across years. This allows a sum of each industry’s average to match the state's total for the 5 years spanned.  

To approximate payrolls and employee levels based on the number of establishments, we provide three options under the settings icon when at least one county is selected:  

1. Use average of payrolls not assigned to counties  
2. Use average of all state payrolls (less accurate)  
3. Leave unpublished values blank  

The default display is option 1 since it more accurately fills in the gaps with values that add up to the actual state total. In some cases, option 2 may more accurately match an individual county, so we advise comparing the two while factoring in attributes that may affect the level in the counties you are analysing.  

Option 1 dispersals add up to the state total by pulling their average from other counties with a similar low concentration of an industry.  

For example: With option 1 “Temporary Help Services” is five times higher than the state average for estimated counties using the average of unassigned payrolls - perhaps because agricultural counties with only a couple temp agencies tend to have larger payrolls since as the sole provider of local temp services.  With option 2 (state average), the level is low because overall counties have more temp services with smaller payrolls per establishment.  

This trend flips for industries that have larger revenue in counties where firms are concentrated, hence “Transportation Equipment Manufacturing” is estimated higher for McDuffie County, Georgia when using the average of all state payrolls because payrolls for similar firms in other counties with higher concentrations of equipment manufacturing skew the average upward, exceeding the value that is shared by the unallocated counties which have less concentration.  

[Community Data Github Repo](https://github.com/modelearth/community-data/)

<!--
Removed from project list

## Use of BEA commodities to estimate null industries

To protect the privacy of individual firms, the census omits payroll and empolyee count data for some industries at both the state and county level (like Automobile Manufacturing).  For Georgia, there are [89 industries](../community-data/us/state/ga/industries_state13_naics6_0s.tsv) with only the number of establishments available at both the county and state lever. 

The estimates for these omitted industry values could be generated using the state BEA commodity data with the crosswalk file, or an average from other states could be used (as long as each industry has at least one payroll value in another state). 
-->


