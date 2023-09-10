# <span style="font-family: Arial;"> An Exploratory Data Analysis of Shipwrecks from Antiquity through the 15th century. </span>



### This is a quick and dirty exploration of data from the _Summary Geodatabase of Shipwrecks AD 1-1500, Status 2008_, available on Harvard University's Dataverse, using Geopandas and Plotly Express to explore and render the geographic information wthin the dataset.

Dataset: McCormick, Michael; Ataoguz, J. Kirsten; Gibson, Kelly L.; Grigoli, Leland; Maione-Downing, Brendan; More, Alexander M.; Reich, Robin; Turnator, Ece; Wang, Julia, 2013, "Summary Geodatabase of Shipwrecks AD 1-1500, Status 2008", https://doi.org/10.7910/DVN/ZIQPDG, Harvard Dataverse, V2



```python
#import pandas and geopandas for the duration
import pandas as pd
import geopandas as gpd
```


```python
# load data, create dataframe, and see first five rows
df = pd.read_csv('Shipwrecks.csv')
df.head()
```


```python
# Drop two columns that immediately have no value
df = df.drop(['Name 2', 'Depth Q'], axis=1)
```


```python
# Create new dataframe from selected columns
df = df[['Name', '2008 Wreck ID', 'Latitude', 'Longitude', 'Start Date', 'End Date', 'Year Found', 'Cargo 1', 'Type 1', 'Cargo 2', 'Type 2', 'Comments']]
```


```python
# Fill NaN values to 0. Not ideal, but need quick fix. 
df = df.fillna(0)
```


```python
# Transform dataframe to geodataframe.
gdf = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df.Longitude, df.Latitude))
# Check out what it looks like.
gdf
```


```python
# Import plotly express library and mapbox to render map.
import plotly.express as px
px.set_mapbox_access_token(open('access.mapbox_token').read())
gdf['size'] = 1

# What I want to see in the map.
fig = px.scatter_mapbox(gdf, 
                  lat=gdf.geometry.y,
                  lon=gdf.geometry.x,
                  hover_name='Name',
                  color='Start Date',
                  hover_data=['Cargo 1', 'Type 1', 'Cargo 2', 'Type 2', 'Comments', 'Start Date', 'End Date'],
                  mapbox_style='dark',
                  size='size',
                  size_max=8,      
                  zoom=1)
# Make map. 
fig.show()
```

### The dataset indicates that the timeframe is 1-1500 AD, but the rendered legend on that map has negative values, which should not be the case. 


```python
#Taking another look at the dates
gdf
```


```python
gdf1 = gdf['Start Date']
```


```python
gdf2 = gdf1.sort_values(ascending=True)
gdf2
```

### Ok, seems to be correct and that some BCE dates are in the dataset. Now out of curiosity let's look at the cargo by goupings and frequency.


```python
cargo = gdf[['Cargo 1', 'Cargo 2']]
```


```python
cargo = cargo['Cargo 1'].value_counts()
cargo
```


```python
#Drop second row since it is 0
cargo.drop(0, inplace=True)
```


```python
cargo.plot.bar();
```

### All in all, an interesting first-pass with geopandas and plotly express. The dataset is a challenge to work with becuase of its incompleteness and the variety of data, which is hard to render programmatically without a lot of substantial effort. But this is the case that all historical datasets will have. So to understand how to work within their limitations, and understand this as a morphological approximation of the dataset itself, is important.   
