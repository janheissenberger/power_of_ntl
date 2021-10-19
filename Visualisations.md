# Part 1: Libraries

```python
import matplotlib.pyplot as plt
import matplotlib.colors as col
import matplotlib as mpl
import numpy as np
import pandas as pd
import seaborn as sns
import geopandas as gpd
from mpl_toolkits.axes_grid1 import make_axes_locatable
from matplotlib.colors import LinearSegmentedColormap
```

# Part 2: Import & manipulate the data

```python
country = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_country_gdp_pop_sol.xlsx")
NUTS2 = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_nuts2_gdp_pop_sol_growth.xlsx")
NUTS3 = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_nuts3_gdp_pop_sol_growth.xlsx")
lau = pd.read_csv("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_lau_sol.csv")
```


```python
country["ln_SOL"]=np.log(country["SOL"])
country["ln_GDP"]=np.log(country["GDP_constant_2010_USD"])
country["ln_POP"]=np.log(country["total_population"])
NUTS2["ln_SOL"]=np.log(NUTS2["SOL"])
NUTS2["ln_GDP"]=np.log(NUTS2["MANGDP"])
NUTS2["ln_POP"]=np.log(NUTS2["MANPOP"])
NUTS3["ln_SOL"]=np.log(NUTS3["SOL"])
NUTS3["ln_GDP"]=np.log(NUTS3["MANGDP"])
NUTS3["ln_POP"]=np.log(NUTS3["MANPOP"])
```

    /Library/Frameworks/Python.framework/Versions/3.9/lib/python3.9/site-packages/pandas/core/arraylike.py:358: RuntimeWarning: divide by zero encountered in log
      result = getattr(ufunc, method)(*inputs, **kwargs)



```python
country["Area"] = pd.to_numeric(country["Area"], errors='coerce')
NUTS2["Area"] = pd.to_numeric(NUTS2["Area"], errors='coerce')
NUTS3["Area"] = pd.to_numeric(NUTS3["Area"], errors='coerce')

# SOL per capita, $km2$, population density
NUTS2["SOL_percapita"] = NUTS2["SOL"]/NUTS2["MANPOP"]
NUTS2["SOL_perkm2"] = NUTS2["SOL"]/NUTS2["Area"]
NUTS2["pop_density"] = NUTS2["MANPOP"]/NUTS2["Area"]
NUTS2["SOL_perpopdensity"] = NUTS2["SOL"]/NUTS2["pop_density"]
NUTS3["SOL_percapita"] = NUTS3["SOL"]/NUTS3["MANPOP"]
NUTS3["SOL_perkm2"] = NUTS3["SOL"]/NUTS3["Area"]
NUTS3["pop_density"] = NUTS3["MANPOP"]/NUTS3["Area"]
NUTS3["SOL_perpopdensity"] = NUTS3["SOL"]/NUTS3["pop_density"]

# country
country_growth = country.groupby(['CC']).agg(np.mean)[['GDP growth', 'POP growth','SOL growth']]
country_growth = country_growth.rename(columns = {'GDP growth': 'GDP_growth_mean', 'POP growth': 'POP_growth_mean', 'SOL growth': 'SOL_growth_mean'})
country = country.merge(country_growth, left_on='CC', right_on='CC')

# NUTS 2
NUTS2_growth = NUTS2.groupby(['NUTS']).agg(np.mean)[['GDP_growth', 'POP_growth','SOL_growth']]
NUTS2_growth = NUTS2_growth.rename(columns = {'GDP_growth': 'GDP_growth_mean', 'POP_growth': 'POP_growth_mean', 'SOL_growth': 'SOL_growth_mean'})
NUTS2 = NUTS2.merge(NUTS2_growth, left_on='NUTS', right_on='NUTS')

# NUTS 3
NUTS3_growth = NUTS3.groupby(['NUTS']).agg(np.mean)[['GDP_growth', 'POP_growth','SOL_growth']]
NUTS3_growth = NUTS3_growth.rename(columns = {'GDP_growth': 'GDP_growth_mean', 'POP_growth': 'POP_growth_mean', 'SOL_growth': 'SOL_growth_mean'})
NUTS3 = NUTS3.merge(NUTS3_growth, left_on='NUTS', right_on='NUTS')

country_shp = gpd.read_file('/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/undone data/other data/NUTS_RG_01M_2021_4326_LEVL_0.shp/NUTS_RG_01M_2021_4326_LEVL_0.shp')
country = country_shp.merge(country,how= 'left',left_on='NUTS_ID',right_on='CC')

nuts2_shp = gpd.read_file('/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/undone data/other data/2016_NUTS2_RG_01M_2016_4326_LEVL_2.shp/NUTS_RG_01M_2016_4326_LEVL_2.shp')
NUTS2 = nuts2_shp.merge(NUTS2,how= 'left',left_on='NUTS_ID',right_on='NUTS')

nuts3_shp = gpd.read_file('/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/undone data/other data/2016_NUTS_RG_01M_2016_3035_LEVL_3.shp/NUTS_RG_01M_2016_3035_LEVL_3.shp')
NUTS3 = nuts3_shp.merge(NUTS3,how= 'inner',left_on='NUTS_ID',right_on='NUTS')

lau["name"]=lau["LAU_NAME"]
lau_shp = gpd.read_file('/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/undone data/other data/ref-lau-2019-01m.shp/LAU_RG_01M_2019_4326.shp/LAU_RG_01M_2019_4326.shp')
lau_shp = lau_shp[lau_shp["CNTR_CODE"]=="DE"]
lau = lau_shp.merge(lau, how="left", left_on="LAU_NAME", right_on="LAU_NAME")
```


# Part 3: Analysis 1

# Part 4: Analysis 2

# Part 5: Analysis 3

# Part 6: Analysis 4
