# Making NTL data analysis ready
## Step 1: Libraries
```python
import pandas as pd
import geemap, ee
import numpy as np
```

To work with the `ee` library, we need to set up an account at https://signup.earthengine.google.com/#!/. 
```python
try:
        ee.Initialize()
except Exception as e:
        ee.Authenticate()
        ee.Initialize()
```

## Step 2: Importing relevant data
In this step, we import GDP data that is splitted up in country, NUTS 2, and NUTS 3 level. This can be any data, but we need this to only find the countries/NUTS levels of importance. In our case, we will cover all EU countries and the CC-5 states (Albania, Montenegro, North Macedonia, Serbia and Turkey).

```python
# GDP country
country_gdp = pd.read_csv("data/GDP constant US$.csv", skiprows=2, header=1, sep=",")

# GDP NUTS 2
nuts2_gdp = pd.read_csv("data/GDP_NUTS2.tsv", sep="\t")

# GDP NUTS 3
nuts3_gdp = pd.read_csv("data/GDP_NUTS3.tsv", sep ="\t")

# GDP growth
gdp_growth = pd.read_csv("data/GDP growth rates_1992-2019.csv", sep=";")

# POP country
country_pop = pd.read_csv("data/Population Total.csv", skiprows=2, header=1, sep=",")

# POP NUTS 2
nuts2_pop = pd.read_csv("data/POP_NUTS2.tsv", sep="\t")

# POP NUTS 3
nuts3_pop = pd.read_csv("data/POP_NUTS3.tsv", sep="\t")

# POP growth
pop_growth = pd.read_excel("data/Population Growth.xlsx", sheet_name="filtered")
```

## Step 3: Cleaning the data
Next, we will clean the GDP and population data so it is usable for our analysis.
### Cleaning country level data
```python
# Country
country_gdp = country_gdp[['Country Name', 'Country Code', 'Indicator Name']+list(map(str, list(range(1992,2020))))]
countries = ['Albania','Austria','Belgium','Bulgaria','Cyprus','Czech Republic','Germany','Denmark','Spain','Estonia','Finland','France','Greece','Croatia','Hungary','Ireland','Italy','Lithuania','Luxembourg','Latvia','North Macedonia','Malta','Montenegro','Norway','Poland','Portugal','Romania','Serbia','Slovak Republic','Slovenia','Sweden','Turkey']
country_codes = ['AL','AT','BE','BG','CY','CZ','DE','DK','ES','EE','FI','FR','EL','HR','HU','IE','IT','LT','LU','LV','MK','MT','ME','NO','PL','PT','RO','RS','SK','SI','SE','TR']

country_gdp = country_gdp[country_gdp['Country Name'].isin(countries)]
country_gdp['CC']=country_codes
country_gdp = pd.melt(country_gdp, id_vars=['CC','Country Name','Country Code'], var_name='year', value_name='GDP_constant_2010_USD', value_vars=list(map(str,list(range(1992,2020)))))

gdp_growth = pd.melt(gdp_growth, id_vars=['Code', 'Country Name', 'Country Code'], var_name ='year', value_name ='GDP_annual_growth',\
        value_vars=map(str,list(range(1992,2020))))

gdp_growth['GDP_annual_growth'] = gdp_growth['GDP_annual_growth'].str.replace(',', '.').astype(float)

pop_growth = pd.melt(pop_growth, id_vars=['Code', 'Country Name', 'Country Code'], var_name ='year', value_name ='POP_annual_growth',\
        value_vars=list(range(1992,2020)))
pop_growth['year']=pop_growth['year'].astype(str)

# Population
country_pop = country_pop[['Country Name', 'Country Code', 'Indicator Name']+list(map(str, list(range(1992,2020))))]
country_pop = country_pop[country_pop['Country Name'].isin(countries)]
country_pop = pd.melt(country_pop, id_vars=['Country Name','Country Code'], var_name='year', value_name='total_population', value_vars=list(map(str,list(range(1992,2020)))))
```
### Cleaning NUTS 2 & NUTS 3 level data
```python
def clean_gdp(df):
    # get column names
    cols = list(df.columns)
    
    for col in cols[1:]:
        df[str(col)] = df[str(col)].str.replace('\\s', '', regex=True)
        df[str(col)] = df[str(col)].str.replace(':', '', regex=True)
        df[str(col)] = df[str(col)].str.replace('[A-Za-z]','', regex=True)
    
    
    df["unit"] = df["unit,geo\\time"].str.split(',',1,expand=True)[0]
    df["NUTS"] = df["unit,geo\\time"].str.split(',',1,expand=True)[1]
    df = df.drop(columns="unit,geo\\time")
    
    
    return df


def clean_pop(df):
    # get column names
    cols = list(df.columns)
    
    # clean letters out
    for col in cols[1:]:
        df[str(col)] = df[str(col)].str.replace('\\s', '', regex=True)
        df[str(col)] = df[str(col)].str.replace(':', '', regex=True)
        df[str(col)] = df[str(col)].str.replace('[A-Za-z]','', regex=True)
        
    df["unit"] = df["unit,sex,age,geo\\time"].str.split(',',3,expand=True)[0]
    df["sex"] = df["unit,sex,age,geo\\time"].str.split(',',3,expand=True)[1]
    df["age"] = df["unit,sex,age,geo\\time"].str.split(',',3,expand=True)[2]
    df["NUTS"] = df["unit,sex,age,geo\\time"].str.split(',',3,expand=True)[3]
    df = df.drop(columns="unit,sex,age,geo\\time")
    
    
    return df

nuts2_gdp = clean_gdp(nuts2_gdp)
nuts3_gdp = clean_gdp(nuts3_gdp)
nuts2_pop = clean_pop(nuts2_pop)
nuts3_pop = clean_pop(nuts3_pop)


# filter for NUTS2
nuts2_gdp = nuts2_gdp[nuts2_gdp["NUTS"].str.contains("^[A-Z0-9]{4}$", regex= True)]
nuts2_gdp = nuts2_gdp[~nuts2_gdp["NUTS"].str.contains("[Z]{2}$", regex= True)] # cleaning out the ZZ
nuts2_pop = nuts2_pop[nuts2_pop["NUTS"].str.contains("^[A-Z0-9]{4}$", regex= True)]
nuts2_pop = nuts2_pop[~nuts2_pop["NUTS"].str.contains("[X]{2}$", regex= True)] # cleaning out the XX

# filter for NUTS 3
nuts3_gdp = nuts3_gdp[nuts3_gdp["NUTS"].str.contains("^[A-Z0-9]{5}$", regex= True)]
nuts3_gdp = nuts3_gdp[~nuts3_gdp["NUTS"].str.contains("[Z]{3}$", regex= True)] # cleaning out the ZZZ
nuts3_pop = nuts3_pop[nuts3_pop["NUTS"].str.contains("^[A-Z0-9]{5}$", regex= True)]
nuts3_pop = nuts3_pop[~nuts3_pop["NUTS"].str.contains("[X]{3}$", regex= True)] # cleaning out the XXX

# filter for unit: "MIO_EUR" to get GDP in Mio of Euros
nuts2_gdp = nuts2_gdp[nuts2_gdp["unit"]=="MIO_EUR"]
nuts3_gdp = nuts3_gdp[nuts3_gdp["unit"]=="MIO_EUR"]

# filter for 'TOTAL'
nuts2_pop = nuts2_pop[nuts2_pop['age']=='TOTAL']
nuts3_pop = nuts3_pop[nuts3_pop['age']=='TOTAL']

# filter for 'sex' == 'T'
nuts2_pop = nuts2_pop[nuts2_pop['sex']=='T']
nuts3_pop = nuts3_pop[nuts3_pop['sex']=='T']
```
```python
### NUTS 2
## GDP
# adds 0 instead of empty
nuts2_gdp= nuts2_gdp.replace('',0)

# converts to numeric
nuts2_gdp[[ '{} '.format(x) for x in list(range(2000,2020))[::-1] ]] =\
nuts2_gdp[[ '{} '.format(x) for x in list(range(2000,2020))[::-1] ]].apply(pd.to_numeric)

# melted nuts2_gdp
melt_nuts2_gdp =\
pd.melt(nuts2_gdp, id_vars=['NUTS'], var_name ='year', value_name ='GDP',\
        value_vars=[ '{} '.format(x) for x in list(range(2000,2020))[::-1] ])

# years to numeric
melt_nuts2_gdp[["year"]]=melt_nuts2_gdp[["year"]].apply(pd.to_numeric)

## POPULATION
# adds 0 instead of empty
nuts2_pop= nuts2_pop.replace('',0)

# delete 1991, 1990, 2020
nuts2_pop = nuts2_pop.drop(["1991 ","1990 ", "2020 "], axis=1)

# converts to numeric
nuts2_pop[[ '{} '.format(x) for x in list(range(1992,2020))[::-1] ]] =\
nuts2_pop[[ '{} '.format(x) for x in list(range(1992,2020))[::-1] ]].apply(pd.to_numeric)

# melted nuts2_pop
melt_nuts2_pop =\
pd.melt(nuts2_pop, id_vars=['NUTS'], var_name ='year', value_name ='POP',\
        value_vars=[ '{} '.format(x) for x in list(range(1992,2020))[::-1] ])

# years to numeric
melt_nuts2_pop[["year"]]=melt_nuts2_pop[["year"]].apply(pd.to_numeric)

### NUTS 3
## GDP
# adds 0 instead of empty
nuts3_gdp= nuts3_gdp.replace('',0)

# converts to numeric
nuts3_gdp[[ '{} '.format(x) for x in list(range(2000,2020))[::-1] ]] =\
nuts3_gdp[[ '{} '.format(x) for x in list(range(2000,2020))[::-1] ]].apply(pd.to_numeric)

# melted nuts3_gdp
melt_nuts3_gdp =\
pd.melt(nuts3_gdp, id_vars=['NUTS'], var_name ='year', value_name ='GDP',\
        value_vars=[ '{} '.format(x) for x in list(range(2000,2020))[::-1] ])

# years to numeric
melt_nuts3_gdp[["year"]]=melt_nuts3_gdp[["year"]].apply(pd.to_numeric)

## POPULATION
# adds 0 instead of empty
nuts3_pop= nuts3_pop.replace('',0)

# delete 1991, 1990, 2020
nuts3_pop = nuts3_pop.drop(["1991 ","1990 ", "2020 "], axis=1)

# converts to numeric
nuts3_pop[[ '{} '.format(x) for x in list(range(1992,2020))[::-1] ]] =\
nuts3_pop[[ '{} '.format(x) for x in list(range(1992,2020))[::-1] ]].apply(pd.to_numeric)

# melted nuts2_pop
melt_nuts3_pop =\
pd.melt(nuts3_pop, id_vars=['NUTS'], var_name ='year', value_name ='POP',\
        value_vars=[ '{} '.format(x) for x in list(range(1992,2020))[::-1] ])

# years to numeric
melt_nuts3_pop[["year"]]=melt_nuts3_pop[["year"]].apply(pd.to_numeric)
```

## Step 4: Extracting SOL for any geographic level
In this step, we extract the Sum Of Lights (SOL) of any given area. In this code **example**, we will do this for NUTS 2 areas. To perform our analysis, we need to consider a couple of parameters. First, we want need to specify the start and end year -- in our case 1992 to 2019. 

Then, ```ee.FeatureCollection()``` needs a path to the shapefile of the geographic level that you want to use. This path should be obtained on https://code.earthengine.google.com/. We uploaded the necessary country, NUTS 2 and NUTS 3 shapefiles, as well as the NTL data, for personal use, so that we can now easily access the data within this interface.

The ```.ee.Filter.eq()``` function requires the column name of the shapefile where the information can be found, and the identification code/name of the area for which the SOL should be then later filtered. We save this as ```aoi``` (Area of Interest).

Due to the long process of obtaining the SOL for a lot of areas and years, we included a error handling that would show us whether the SOL for a certain area could not be gained. We also included a counter of how many percent of the areas are already completed.

Another for-loop iterates over the relevant years, and ```ee.ImageCollection()``` saves the relevant picture (NTL GeoTIFF file), which is then cut-out to the relevant area ```aoi``` and the sum of light is calculated by ```.reduceRegion()```. Then we adress the relevant (and only layer, in this case) of the GeoTIFF file by ```.get()```, and get the SOL by ```.getInfo()```.

Lastly, we store the SOL with the respective area and add it to a dataframe.

```python
start = 1992 # start year
end = 2019 # end year

NUTS_dict={}
nuts2_sol = pd.DataFrame()

# counter to see how far the code has already processed while executing
counter = 0
total = len(list(nuts2_gdp["NUTS"]))

for nut in list(nuts2_gdp["NUTS"]):
    counter = counter +1
    
    # takes area of interest (aoi) 
    aoi = ee.FeatureCollection("users/janheissenberger/NUTS2_2016").filter(ee.Filter.eq('FID',str(nut))).geometry()
    
    try:
        NUTS_dict["NUTS"]= nut

        for year in range(start, end+1):
            
            date_start = str(year)+'-01-01'
            date_end = str(year)+'-12-31'
            
            # takes the whole image collection
            image = ee.ImageCollection("users/janheissenberger/harmonized_dataset").filterDate(date_start,date_end)
            
            # takes the first image of collection (in our case it is only one image anyways)
            image = image.first()

            # takes the SOL of the given area of interest
            NUTS_dict[year]= image.reduceRegion(reducer=ee.Reducer.sum(), geometry=aoi, scale=1000, maxPixels=1e9).get('b1').getInfo()
    
        NUTS_dict["unit"] = "SOL"

        nuts2_sol = nuts2_sol.append(NUTS_dict, ignore_index=True)
        print(str(nut)+" successfully added "+ str(round(counter/total * 100, 2)) + " %", end="\r")
    
    except:
        print("\n"+str(nut)+" could not be added "+ str(round(counter/total * 100, 2)) + " %", end="\n")

print("\nTask successfully finished")
```

## Step 5: Prepare SOL data
We manipulate our dataframe to only have two columns, namely the country/NUTS code and the respective SOL.
```python
country_sol = pd.melt(country_sol,id_vars=['NUTS'], var_name='year', value_name='SOL', value_vars=list(map(str,list(range(1992,2020)))))
country_sol = country_sol.rename(columns={'NUTS':'CC'})

melt_nuts2_sol = pd.melt(nuts2_sol, id_vars=['NUTS'], var_name ='year', value_name ='SOL', value_vars=list(map(str,list(range(1992,2020)))))
melt_nuts2_sol[["year"]]=melt_nuts2_sol[["year"]].apply(pd.to_numeric) # years to numeric

melt_nuts3_sol = pd.melt(nuts3_sol, id_vars=['NUTS'], var_name ='year', value_name ='SOL', value_vars=list(map(str,list(range(1992,2020)))))
melt_nuts3_sol[["year"]]=melt_nuts3_sol[["year"]].apply(pd.to_numeric) # years to numeric
```

## Step 6: Merge and save GDP, population and SOL data for each geographic level
Then, we merge the SOL data with the GDP and population data and are then ready

### Country
```python
# merge GDP with POP
country_gdp_pop = pd.merge(country_gdp, country_pop, on=['Country Name','Country Code','year'])

# merge GDP/POP with SOL
country_gdp_pop_sol = pd.merge(country_gdp_pop, country_sol,on=['CC', 'year'])

# CSV
country_gdp_pop_sol.to_csv("final_country_gdp_pop_sol.csv", sep=",", index=False)
```
### NUTS 2
```python
# merge GDP with SOL
final_nuts2_gdp_sol = pd.merge(melt_nuts2_gdp, melt_nuts2_sol,  how='right', left_on=['NUTS','year'], right_on = ['NUTS','year'])

# adds country code
final_nuts2_gdp_sol["country"]=final_nuts2_gdp_sol["NUTS"].str[0:2]

# merge GDP/SOL with POPULATION
final_nuts2_gdp_pop_sol = pd.merge(melt_nuts2_pop, final_nuts2_gdp_sol,  how='right', left_on=['NUTS','year'], right_on = ['NUTS','year'])

# CSV
final_nuts2_gdp_pop_sol.to_csv('final_nuts3_gdp_sol.csv', index=False)
```

### NUTS 3
```python

# merge GDP with SOL
final_nuts3_gdp_sol = pd.merge(melt_nuts3_gdp, melt_nuts3_sol,  how='right', left_on=['NUTS','year'], right_on = ['NUTS','year'])

# adds country code
final_nuts3_gdp_sol["country"]=final_nuts3_gdp_sol["NUTS"].str[0:2]

# merge GDP/SOL with POPULATION
final_nuts3_gdp_pop_sol = pd.merge(melt_nuts3_pop, final_nuts3_gdp_sol,  how='right', left_on=['NUTS','year'], right_on = ['NUTS','year'])

# CSV
final_nuts3_gdp_pop_sol.to_csv('final_nuts3_gdp_sol.csv', index=False)
```

## Step 7: Adding growth variables
The growth variables of GDP, population, and SOL are added via MS Excel.

## Step 8: Adding area
For our analysis, we also require the km2 of every area in our dataset. After adding this, we have our datasets ready for analysis. The final datasets can be retrieved here https://doi.org/10.5281/zenodo.5579761.
```python
# reading in area data
nuts_area = pd.read_csv("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/undone data/AREA_NUTS.tsv", sep="\t")

# cleaning
nuts_area[['landuse','unit', 'geo\\time']] = nuts_area['landuse,unit,geo\\time'].str.split(',',expand=True)
nuts_area = nuts_area.drop(columns=['landuse,unit,geo\\time', "2016 "])
nuts_area = nuts_area[nuts_area["landuse"]=="TOTAL"]
nuts_area = nuts_area.drop(columns=['landuse','unit'])
nuts_area = nuts_area.rename(columns={"2013 ": "Area"})

# reading in merged GDP/POP/SOL/growth data again
country = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_country_gdp_pop_sol.xlsx", sheet_name = "final")
NUTS2 = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_nuts2_gdp_pop_sol_growth.xlsx", sheet_name = "final")
NUTS3 = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_nuts3_gdp_pop_sol_growth.xlsx", sheet_name = "final")
lau = pd.read_csv("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_lau_sol.csv")

# merging
country_merge = country.merge(nuts_area, how="left", left_on="CC", right_on = "geo\\time")
country_merge = country_merge.drop(columns='geo\\time')
NUTS2_merge = NUTS2.merge(nuts_area, how="left", left_on="NUTS", right_on = "geo\\time")
NUTS2_merge = NUTS2_merge.drop(columns='geo\\time')
NUTS3_merge = NUTS3.merge(nuts_area, how="left", left_on="NUTS", right_on = "geo\\time")
NUTS3_merge = NUTS3_merge.drop(columns='geo\\time')

# saving as excel files ready for analysis
country_merge.to_excel('data/final_country_gdp_pop_sol.xlsx', index=False)
NUTS2_merge.to_excel('data/final_nuts2_gdp_pop_sol_growth.xlsx', index=False)
NUTS3_merge.to_excel('data/final_nuts3_gdp_pop_sol_growth.xlsx', index=False)
```

