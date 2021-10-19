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
## Importing
```python
country = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_country_gdp_pop_sol.xlsx")
NUTS2 = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_nuts2_gdp_pop_sol_growth.xlsx")
NUTS3 = pd.read_excel("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_nuts3_gdp_pop_sol_growth.xlsx")
lau = pd.read_csv("/Users/jan/Library/Mobile Documents/com~apple~CloudDocs/WU/Semester IV/_Bachelor Thesis/Analysis/data/final_lau_sol.csv")
```

## Manipulating/adding new columns
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
## Example of scatterplot used for every geographic level 
```python
sns.set(rc={'figure.figsize':(13,10)})
sns.set_style("whitegrid")
plot = sns.scatterplot(x = "SOL", y = "GDP_constant_2010_USD", hue = "CC", alpha=0.4, s=250, data = country)
plot.set_xlabel("SOL")
plot.set_ylabel("GDP")
plot.legend(title='Country', loc = 'center right', bbox_to_anchor=(1.1, 0.5))
plt.savefig('/Users/jan/Downloads/Country_GDP_SOL.png', dpi = 100, bbox_inches='tight')
```

# Part 4: Analysis 2
## Example of scatterplot used for every geographic level 
```python
sns.set(rc={'figure.figsize':(13,10)})
sns.set_style("whitegrid")
plot = sns.scatterplot(x = "SOL", y = "total_population", hue = "CC", alpha=0.4, s=250, data = country)
plot.set_xlabel("SOL")
plot.set_ylabel("Total population")
plot.legend(title='Country', loc = 'center right', bbox_to_anchor=(1.1, 0.5))
#plot.set_title("Population and SOL (1992-2019)", fontsize=15)
plt.savefig('/Users/jan/Downloads/Country_POP_SOL.png', dpi = 100, bbox_inches='tight')
```

# Part 5: Analysis 3

```python
landstrip_list = ["DE80M","DE80O","DE40F","DEE04","DEE07","DEE0D","DEE09","DEG07", "DEG06","DEG09","DEG0P","DEG0B","DEG0E","DEG0H","DEG0I","DEG0K","DED44", "DEF03", "DEF06", "DE935", "DE934", "DE93A", "DE914", "DE917", "DE91B", "DE916", "DE91C", "DE737","DE733", "DE732", "DE266", "DE267", "DE247", "DE24A", "DE249"]
landstrip = NUTS3["NUTS"].isin(["DE80M","DE80O","DE40F","DEE04","DEE07","DEE0D","DEE09","DEG07", "DEG06","DEG09","DEG0P","DEG0B","DEG0E","DEG0H","DEG0I","DEG0K","DED44", "DEF03", "DEF06", "DE935", "DE934", "DE93A", "DE914", "DE917", "DE91B", "DE916", "DE91C", "DE737","DE733", "DE732", "DE266", "DE267", "DE247", "DE24A", "DE249"])


east_border = ["DE80M","DE80O","DE40F","DEE04","DEE0D","DEE07","DEE09","DEG07", "DEG06","DEG09","DEG0P","DEG0B","DEG0E","DEG0H","DEG0I","DEG0K","DED44"]
west_border = [item for item in landstrip_list if item not in east_border]

germany = list(NUTS2["NUTS"][NUTS2["country"]=="DE"].unique())
east = ["DE80","DEE0","DEE0","DEG0","DED5","DED4","DED2","DE40"]
west = [item for item in germany if item not in east]
west.remove('DE30') # exclude Berlin

NUTS2["east"] = np.where(NUTS2["NUTS"].isin(east), "East", np.where(NUTS2["NUTS"].isin(west), "West", None))
NUTS3["east_border"] = np.where(NUTS3["NUTS"].isin(east_border), "East", np.where(NUTS3["NUTS"].isin(west_border), "West", None))
```

### Color Palette
```python
sns.set_palette('Pastel2')
current_palette = sns.color_palette()

first = current_palette[0]
second = current_palette[1]

# create simple linear colormap that maps grey to blue
cmap = LinearSegmentedColormap.from_list(
    'mycmap', [(0, second), (1, first)])
```

## Geopandas graph
```python
plot = NUTS2[(NUTS2['country']=="DE") & (NUTS2['year']==2019)].plot(column="SOL_growth_mean",\
                                           legend=True,\
                                           edgecolor="black", linewidth=0.3,\
                                           cmap='Oranges')
plt.savefig('/Users/jan/Downloads/Germany_NUTS2_SOLgrowth.png', dpi = 100, bbox_inches='tight')
```

## Geopandas graph
```python
missing_kwds = dict(color='0.95', label="Berlin")

plot = NUTS2[(NUTS2['country'] =="DE") & (NUTS2['year']==2019)].plot(column="east",\
                                           missing_kwds=missing_kwds,\
                                           legend=True,\
                                           edgecolor="black", linewidth=0.3,\
                                           cmap=cmap)
                                                                    
plt.savefig('/Users/jan/Downloads/Germany_NUTS2_eastwest.png', dpi = 100, bbox_inches='tight')
```

## Geopandas graph
```python
missing_kwds = dict(color='0.95', label="_nolabel")

plot = NUTS3[(NUTS3['country'] =="DE") & (NUTS3['year']==2019)].plot(column="east_border",\
                                           legend=True,\
                                           edgecolor="black", linewidth=0.3,\
                                           missing_kwds=missing_kwds,\
                                           cmap=cmap)
plot.set_xlim((4000000,4700000))
plot.set_ylim((2600000, 3600000))
plt.savefig('/Users/jan/Downloads/Germany_NUTS3_eastwest.png', dpi = 100, bbox_inches='tight')
```
## Line graphs
```python
# SOL
plot = sns.lineplot(x="year",y="SOL", hue="east", linewidth=5, data = NUTS2)
plot.set_xlabel("Year")
plot.set_ylabel("SOL")
plt.legend(title='Area in Germany')

# SOL per capita
plot = sns.lineplot(x="year",y="SOL_percapita", hue="east", linewidth=5, data = NUTS2)
plot.set_xlabel("Year")
plot.set_ylabel("SOL per capita")
plt.legend(title='Area in Germany')
plt.savefig('/Users/jan/Downloads/Analysis3_NUTS2_SOL_percapita.png', dpi = 100, bbox_inches='tight')

# SOL per KM2
plot = sns.lineplot(x="year",y="SOL_perkm2", hue="east", linewidth=5, data = NUTS2)
plot.set_xlabel("Year")
plot.set_ylabel("SOL per $km^2$")
plt.legend(title='Area in Germany')
plt.savefig('/Users/jan/Downloads/Analysis3_NUTS2_SOL_perkm2.png', dpi = 100, bbox_inches='tight')

# SOL per SOL_perpopdensity
plot = sns.lineplot(x="year",y="SOL_perpopdensity", hue="east", linewidth=5, data = NUTS2)
plot.set_xlabel("Year")
plot.set_ylabel("SOL per Population Density")
plt.legend(title='Area in Germany')
plt.savefig('/Users/jan/Downloads/Analysis3_NUTS2_SOL_perpopdensity.png', dpi = 100, bbox_inches='tight')

```

# Part 6: Analysis 4
```python
lau_filtered = lau.dropna(subset=['name'])
lau_filtered['is_elsterberg'] = np.where(lau_filtered['name']== 'Elsterberg, Stadt', 'Elsterberg, Stadt', 'Others')
```
## Line graphs
```python
# SOL
plot = sns.lineplot(x="year",y="SOL", linewidth=5, hue="is_elsterberg", palette=['#2ec4b6'], data = lau_filtered[lau_filtered["name"]=="Elsterberg, Stadt"])
plot.set_xlabel("Year")
plot.set_ylabel("SOL")
plot.axvline(x=2009, ymin=0.0, ymax=2009, color='r', label="Factory closed")
plt.legend(title='Town in Germany')
plt.savefig('/Users/jan/Downloads/Analysis4_lineplot_els.png', dpi = 100, bbox_inches='tight')
```
```python
# SOL
plot = sns.lineplot(x="year",y="SOL", hue="is_elsterberg", linewidth=5, palette=['#ffbf69', '#2ec4b6'], data = lau_filtered)
#plot.set_title("SOL in Elsterberg and similar towns 1992-2019")
plot.set_xlabel("Year")
plot.set_ylabel("SOL")
plot.axvline(x=2009, ymin=0.0, ymax=2009, color='r', label="Factory closed")
plt.legend(title='Town in Germany')
plt.savefig('/Users/jan/Downloads/Analysis4_lineplot_all.png', dpi = 100, bbox_inches='tight')
```

## Geopandas graph
```python
missing_kwds = dict(color='lightgray', label="_nolabel")

plot = lau.plot(edgecolor="black", linewidth=0.001, column = 'name', missing_kwds=missing_kwds)

plt.savefig('/Users/jan/Downloads/Germany_NUTS3_towns.png', dpi = 100, bbox_inches='tight')
```



