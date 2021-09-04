{%hackmd SybccZ6XD %}
[![hackmd-github-sync-badge](https://hackmd.io/YxHRo0HOQqGMkWA2cahmRQ/badge)](https://hackmd.io/YxHRo0HOQqGMkWA2cahmRQ)
###### tags: `程式語言` `Python` `Kaggle`   
[![hackmd-github-sync-badge](https://hackmd.io/AndI4RZuSyOoIZ88D96aNw/badge)](https://hackmd.io/AndI4RZuSyOoIZ88D96aNw)  
![](https://i.imgur.com/SUcWx1U.jpg)  

# Geospatial Analysis
1. Your First Map  
2. Coordinate Reference Systems  
3. Interactive Maps  
4. Manipulating Geospatial Data  
5. Proximity Analysis  
## [Your First Map](https://www.kaggle.com/alexisbcook/your-first-map)
### Exercise:computer: [Your First Map](https://www.kaggle.com/jadentseng/exercise-your-first-map/edit)
## [Coordinate Reference Systems](https://www.kaggle.com/alexisbcook/coordinate-reference-systems)
### Exercise:computer: [Coordinate Reference Systems](https://www.kaggle.com/jadentseng/exercise-coordinate-reference-systems/edit)
```python=
import pandas as pd
import geopandas as gpd

from shapely.geometry import LineString
```
#### 1) Load the data
```python=5
# Load the data and print the first 5 rows
birds_df = pd.read_csv("../input/geospatial-learn-course-data/purple_martin.csv", parse_dates=['timestamp'])
print("There are {} different birds in the dataset.".format(birds_df["tag-local-identifier"].nunique()))
birds_df.head()
```
```
There are 11 different birds in the dataset.
        timestamp	          location-long  location-lat	  tag-local-identifier
0	2014-08-15 05:56:00	  -88.146014	  17.513049	  30448
1	2014-09-01 05:59:00	  -85.243501	  13.095782	  30448
2	2014-10-30 23:58:00	  -62.906089	  -7.852436	  30448
3	2014-11-15 04:59:00	  -61.776826	  -11.723898	  30448
4	2014-11-30 09:59:00	  -61.241538	  -11.612237	  30448
```
####  "tag-local-identifier" column
There are 11 birds in the dataset, where each bird is identified by a unique value in the "tag-local-identifier" column. Each bird has several measurements, collected at different times of the year.  

Use the next code cell to create a GeoDataFrame birds.  
* birds should have all of the columns from birds_df, along with a "geometry" column that contains Point objects with (longitude, latitude) locations.
* Set the CRS of birds to {'init': 'epsg:4326'}.  
```python=9
# Your code here: Create the GeoDataFrame
birds = gpd.GeoDataFrame(birds_df, geometry=gpd.points_from_xy(birds_df["location-long"], birds_df["location-lat"]))

# Your code here: Set the CRS to {'init': 'epsg:4326'}
birds.crs = {'init': 'epsg:4326'}
```
#### 2) Plot the data
```python=14
# Load a GeoDataFrame with country boundaries in North/South America, print the first 5 rows
world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))
americas = world.loc[world['continent'].isin(['North America', 'South America'])]
americas.head()
```
Use the next code cell to create a single plot that shows both:   
(1) the country boundaries in the americas GeoDataFrame, and   
(2) all of the points in the birds_gdf GeoDataFrame.  
```python=18
# Your code here
ax = americas.plot()
birds.plot(ax=ax, markersize=2)
```
zoom in
```python=21
ax = americas.plot()
birds.plot(ax=ax, markersize=2)
ax.set_xlim([-110, -30])
ax.set_ylim([-30, 60])
```
#### 3) Where does each bird start and end its journey? (Part 1)
* `path_gdf` contains LineString objects that show the path of each bird. It uses the `LineString()`` method to create a LineString object from a list of Point objects.    
* `start_gdf` contains the starting points for each bird.  
```python=25
# GeoDataFrame showing path for each bird
path_df = birds.groupby("tag-local-identifier")['geometry'].apply(list).apply(lambda x: LineString(x)).reset_index()
path_gdf = gpd.GeoDataFrame(path_df, geometry=path_df.geometry)
path_gdf.crs = {'init' :'epsg:4326'}

# GeoDataFrame showing starting point for each bird
start_df = birds.groupby("tag-local-identifier")['geometry'].apply(list).apply(lambda x: x[0]).reset_index()
start_gdf = gpd.GeoDataFrame(start_df, geometry=start_df.geometry)
start_gdf.crs = {'init' :'epsg:4326'}

# Show first five rows of GeoDataFrame
start_gdf.head()
```
To create a GeoDataFrame `end_gdf` containing the final location of each bird.  
* The format should be identical to that of `start_gdf`, with two columns ("tag-local-identifier" and "geometry"), where the "geometry" column contains Point objects.  
* Set the CRS of `end_gdf` to `{'init': 'epsg:4326'}`.  
```python=37
end_df = birds.groupby("tag-local-identifier")['geometry'].apply(list).apply(lambda x: x[-1]).reset_index()
end_gdf = gpd.GeoDataFrame(end_df, geometry=end_df.geometry)
end_gdf.crs = {'init': 'epsg:4326'}
```
#### 4) Where does each bird start and end its journey? (Part 2)
Use the GeoDataFrames from the question above (`path_gdf`, `start_gdf`, and `end_gdf`) to visualize the paths of all birds on a single map. You may also want to use the `americas` GeoDataFrame.  
```python=40
ax = americas.plot(figsize=(10, 10), color='white', linestyle=':', edgecolor='gray')
start_gdf.plot(ax = ax, color='red')
path_gdf.plot(ax = ax, cmap = 'tab20b')
end_gdf.plot(ax = ax, color='green')
```
![](https://i.imgur.com/GiXi9wS.jpg)
#### 5) Where are the protected areas in South America? (Part 1
It looks like all of the birds end up somewhere in South America. But are they going to protected areas?  
In the next code cell, you'll create a GeoDataFrame `protected_areas` containing the locations of all of the protected areas in South America. The corresponding shapefile is located at filepath `protected_filepath`.  
```python=44
# Path of the shapefile to load
protected_filepath = "../input/geospatial-learn-course-data/SAPA_Aug2019-shapefile/SAPA_Aug2019-shapefile/SAPA_Aug2019-shapefile-polygons.shp"

# Your code here
protected_areas = gpd.read_file(protected_filepath)
```
####  6) Where are the protected areas in South America? (Part 2)
Create a plot that uses the `protected_areas` GeoDataFrame to show the locations of the protected areas in South America. (You'll notice that some protected areas are on land, while others are in marine waters.)  
```python=49
# Country boundaries in South America
south_america = americas.loc[americas['continent']=='South America']

# Your code here: plot protected areas in South America
ax = south_america.plot(figsize=(10,10), color='white', edgecolor='gray')
protected_areas.plot(ax=ax, color='red', markersize = 1)
```
![](https://i.imgur.com/CEtdYpd.jpg)

#### 7) What percentage of South America is protected?
You're interested in determining what percentage of South America is protected, so that you know how much of South America is suitable for the birds.  
As a first step, you calculate the total area of all protected lands in South America (not including marine area). To do this, you use the "REP_AREA" and "REP_M_AREA" columns, which contain the total area and total marine area, respectively, in square kilometers.  
```python=55
P_Area = sum(protected_areas['REP_AREA']-protected_areas['REP_M_AREA'])
print("South America has {} square kilometers of protected areas.".format(P_Area))
```
Then, to finish the calculation, you'll use the south_america GeoDataFrame.  
```python=57
south_america.head()
```
Calculate the total area of South America by following these steps:  
* Calculate the area of each country using the `area` attribute of each polygon (with EPSG 3035 as the CRS), and add up the results. The calculated area will be in units of square meters.  
* Convert your answer to have units of square kilometeters.  
```python=58
# Your code here: Calculate the total area of South America (in square kilometers)
totalArea = sum(south_america.geometry.to_crs(epsg=3035).area)/10**3/10**3
```
To calculate the percentage of South America that is protected.  
```python=60
# What percentage of South America is protected?
percentage_protected = P_Area/totalArea
print('Approximately {}% of South America is protected.'.format(round(percentage_protected*100, 2)))
```
```
Approximately 30.39% of South America is protected.
```
#### 8) Where are the birds in South America?
So, are the birds in protected areas?  

Create a plot that shows for all birds, all of the locations where they were discovered in South America. Also plot the locations of all protected areas in South America.  

To exclude protected areas that are purely marine areas (with no land component), you can use the "MARINE" column (and plot only the rows in `protected_areas[protected_areas['MARINE']!='2']`, instead of every row in the `protected_areas` GeoDataFrame).  
```python=63
ax = south_america.plot(figsize=(10,10), color='white', edgecolor='gray')
protected_areas[protected_areas['MARINE']!='2'].plot(ax=ax, alpha=0.4, zorder=1)
birds[birds.geometry.y < 0].plot(ax=ax, color='red', alpha=0.6, markersize=10, zorder=2)
```
![](https://i.imgur.com/0yOrMQ5.jpg)  
## [Interactive Maps](https://www.kaggle.com/alexisbcook/interactive-maps)
### Exercise:computer: [Interactive Maps](https://www.kaggle.com/jadentseng/exercise-interactive-maps/edit)
You are an urban safety planner in Japan, and you are analyzing which areas of Japan need extra earthquake reinforcement. Which areas are both high in population density and prone to earthquakes?  
![](https://i.imgur.com/wvUehS6.jpg)  
Before you get started, run the code cell below to set everything up.  
```python=
import pandas as pd
import geopandas as gpd

import folium
from folium import Choropleth
from folium.plugins import HeatMap
```
We define a function `embed_map()` for displaying interactive maps. It accepts two arguments: the variable containing the map, and the name of the HTML file where the map will be saved.  
This function ensures that the maps are visible [in all web browsers](https://github.com/python-visualization/folium/issues/812).  
```python=7
def embed_map(m, file_name):
    from IPython.display import IFrame
    m.save(file_name)
    return IFrame(file_name, width='100%', height='500px')
```
#### 1) Do earthquakes coincide with plate boundaries?
Run the code cell below to create a DataFrame `plate_boundaries` that shows global plate boundaries. The "coordinates" column is a list of (latitude, longitude) locations along the boundaries.
```python=11
plate_boundaries = gpd.read_file("../input/geospatial-learn-course-data/Plate_Boundaries/Plate_Boundaries/Plate_Boundaries.shp")
plate_boundaries['coordinates'] = plate_boundaries.apply(lambda x: [(b,a) for (a,b) in list(x.geometry.coords)], axis='columns')
plate_boundaries.drop('geometry', axis=1, inplace=True)

plate_boundaries.head()
```
Next, run the code cell below without changes to load the historical earthquake data into a DataFrame `earthquakes`.
```python=16
# Load the data and print the first 5 rows
earthquakes = pd.read_csv("../input/geospatial-learn-course-data/earthquakes1970-2014.csv", parse_dates=["DateTime"])
earthquakes.head()
```
The code cell below visualizes the plate boundaries on a map. Use all of the earthquake data to add a heatmap to the same map, to determine whether earthquakes coincide with plate boundaries.  
```python=19
# Create a base map with plate boundaries
m_1 = folium.Map(location=[35,136], tiles='cartodbpositron', zoom_start=5)
for i in range(len(plate_boundaries)):
    folium.PolyLine(locations=plate_boundaries.coordinates.iloc[i], weight=2, color='black').add_to(m_1)

# Your code here: Add a heatmap to the map
HeatMap(data = earthquakes[['Latitude','Longitude']],radius = 15).add_to(m_1)

# Show the map
embed_map(m_1, 'q_1.html')
```
![](https://i.imgur.com/V1TKJBS.jpg)
#### 2) Is there a relationship between earthquake depth and proximity to a plate boundary in Japan?
You recently read that the depth of earthquakes tells us [important information](https://www.usgs.gov/faqs/what-depth-do-earthquakes-occur-what-significance-depth?qt-news_science_products=0#qt-news_science_products) about the structure of the earth. You're interested to see if there are any intereresting global patterns, and you'd also like to understand how depth varies in Japan.  
```python=29
# Create a base map with plate boundaries
m_2 = folium.Map(location=[35,136], tiles='cartodbpositron', zoom_start=5)
for i in range(len(plate_boundaries)):
    folium.PolyLine(locations=plate_boundaries.coordinates.iloc[i], weight=2, color='black').add_to(m_2)
    
# Your code here: Add a map to visualize earthquake depth
# Custom function to assign a color to each circle
def color_producer(val):
    if val < 50:
        return 'forestgreen'
    elif val < 100:
        return 'darkorange'
    else:
        return 'darkred'

for i in range(0, len(earthquakes)):
    folium.Circle(location=[earthquakes.iloc[i]['Latitude'],earthquakes.iloc[i]['Longitude']], radius = 2000, color=color_producer(earthquakes.iloc[i]['Depth'])).add_to(m_2)


# View the map
embed_map(m_2, 'q_2.html')
```
![](https://i.imgur.com/DpPS4CU.jpg)

Can you detect a relationship between proximity to a plate boundary and earthquake depth? Does this pattern hold globally? In Japan?  
* In the northern half of Japan, it does appear that earthquakes closer to plate boundaries tend to be shallower (and earthquakes farther from plate boundaries are deeper). This pattern is repeated in other locations, such as the western coast of South America. But, it does not hold everywhere (for instance, in China, Mongolia, and Russia).  
#### 3) Which prefectures have high population density?

## Manipulating Geospatial Data  
* Introduction
* Geocoding
* Table Joins
## Proximity Analysis
* Introduction  
* Measuring Distance  
* Creating A Buffer  