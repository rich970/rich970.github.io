---
title: Using geopandas to plot maternity care scores across England
image: /assets/20221003-imgs/combined-maps.png
---  

On the 21st of September the BBC published an article with analysis of the latest Care Quality
Commission (CQC) results regarding maternity care. You can find it [here](https://www.bbc.co.uk/news/health-62569344).

The article states that more than half of maternity units in England fail consistently to meet safety standards. 

The CQC data can be easily downloaded from their website and I was curious as to the 
regional distribution of the most recent scores (since 2020). So, I created the following 
figure using the [geopandas](https://geopandas.org/en/stable/) package. Notes on how the data was processed prior to plotting
is included in the figure. 

![png](/assets/20221003-imgs/combined-maps.png)

I don't want to delve too deep into the meaning of these results (as I'm really not an
expert in this field), however, the general blueish hue across all CQC rating metrics
(except "Responsiveness" which scores generally higher across the board) reflects that the 
average "Overall" rating for England based on the scores since 2020 is between 
"Requires Improvement" and "Good". This is broadly what was found in the BBC article.
There doesn't seem to be an obvious North-South divide or other disparity, at least at a regional level.

Geopandas is a brilliant open source package for geospatial analysis, especially if you are, like me, 
familiar with pandas, but not so familiar with mapping. All you need is the shape files for
the map you want to produce the plot for and summary statistics for the polygon areas of the map
covered by your shape file.

I'm not going to give a full tutorial on how these figures were generated this time, since most of
what I learnt I pulled from the following introduction from the geopandas docs [here](https://geopandas.org/en/stable/docs/user_guide/mapping.html).

The core process is summarised below though:

```python
import geopandas as gpd

# Read in the shape file for your map as a geopandas dataframe:
shp_file = 'england_regions_shape_file.shp'
england_regions = gpd.read_file(shp_file)

# Create an additional column in this geopandas dataframe with the regional scores:
england_regions['score'] = england_regions['regions'].map(mean_scores)

# Plot the shape file polygons and color using the 'score' column:
england_regions.plot(column='score')

```
