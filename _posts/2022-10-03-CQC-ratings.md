---
title: Maternity care across England
image: /assets/20221003-imgs/combined-maps.png
---  

On the 21st of September the BBC published an article analysis the latest Care Quality
Commission (CQC) results regarding maternity care. You can find it [here](https://www.bbc.co.uk/news/health-62569344).

As an new father to be, it made for some rather uneasy reading. The article states that 
more than half of maternity units in England fail consistently to meet safety standards. 

The CQC data can be easily downloaded from their website and I was curious as to the 
regional distribution of the most recent scores (since 2020). So, I created the following 
figure using the geopandas package. Notes on how the data was processed prior to plotting
is included in the figure. 

![png](/assets/20221003-imgs/combined-maps.png)

The general blueish hue across all CQC rating metrics (except "Responsiveness" which scores
generally higher across the board) reflects that the average "Overall" rating for England
based on the scores since 2020 is between "Requires Improvement" and "Good". 

It's important to add that personally I don't find these scores are a reflection of the staff,
all of which have so far been fantastic during the hospital and midwife appointments myself and my
wife have attended. 

I'm not going to give a full tutorial on how these figures were generated, but most of
what I learnt I pulled from the following introduction from the geopandas docs [here](https://geopandas.org/en/stable/docs/user_guide/mapping.html).

The core process is summarised below:

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
