---
title: How has the COVID-19 pandemic impacted academic output?
---  

I’ve been interested to see if the pandemic has had an effect on the number of published research articles. My University was quick to close down when COVID hit the UK, and for those reliant on laboratory work, research quickly ground to a halt. Conversely, we were spending more time at home, with less distractions in the form of shiny equipment to tinker with. So perhaps, there was finally the time (and motivation) to finalise those articles that had been hanging for so long.
So as we all returned to work, I wanted to look into whether COVID had indeed had an impact on academic output or whether it was unchanged by the pandemic. To me, the best place to do this was to look at the number of publications per year.
To obtain this data I submitted search querries to pubMed, restricting the search to their core collection, returning only articles and those written in English. You can download a csv file of the data here.

Here, I used pymed which is python wrapper for the pubMed API which makes this super easy. We’re going to use the following imports:



```python
from pymed import PubMed
import pandas as pd
import datetime as dt
from dateutil.relativedelta import relativedelta
import matplotlib.pyplot as plt
import numpy as np
import time
from tqdm import tqdm
```

Next we create an instance of the PubMed class. It’s good practive to provide your email and the reasons for using the API so the people at PubMed can contact you if your requests are slowing the server, but assuming you’re reasonable in your use this should never be an issue.



```python
# Good practice to give pubmed some info about us if we are scraping
pubmed = PubMed(tool='Counting publications to study impact of COVID pandemic on academic output', email = 'r.rowanrobinson@gmail.com')
```

I’m going to create a list of dates for which we will extract the numbers of publications for. We look at the number of publications in any 1 month period:


```python
date_list=[]
start_date = dt.date(1965,1,1)
end_date = dt.date(2022,1,2)
curr_date = start_date
while curr_date < end_date:
    date_list.append(curr_date.strftime('%Y/%m/%d'))
    curr_date = (curr_date + relativedelta(months=1))
    
date_list[-3:]
```




    ['2021/11/01', '2021/12/01', '2022/01/01']



Then we loop through that list, extacting the number of results returned the period between each consecutive dates in the date_list:


```python
n_pubs = []
i=0
# Loop through the dates and submit the query to return the number of publications in that period:
for i in tqdm(range(len(date_list)-1)):    
    query = ['''(("{0}"[Date - Completion] : "{1}"[Date - Completion]))
                AND ("journal article"[Publication Type])'''.format(date_list[i], date_list[i+1])]
    result = pubmed.getTotalResultsCount(query)
    n_pubs.append(pubmed.getTotalResultsCount(query))
    
# Create dataframe from the extracted publications counts:
df_pubs = pd.DataFrame([date_list[:-1], n_pubs]).T
df_pubs.rename(columns={0:'date', 1:'n_pubs'}, inplace=True)

# Make the index a datetime:
df_pubs.date = pd.to_datetime(df_pubs.date)
df_pubs.set_index('date', inplace=True)
```

The resulting dataframe (df_pubs) contains the number of publications per 1 month period:


```python
df_pubs.columns = ['date', 'n_publications']
df_pubs.date = pd.to_datetime(df_pubs.date, format='%Y/%m/%d')
df_pubs.set_index('date', inplace=True)
df_pubs.tail(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>n_publications</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2021-08-01</th>
      <td>116308</td>
    </tr>
    <tr>
      <th>2021-09-01</th>
      <td>111007</td>
    </tr>
    <tr>
      <th>2021-10-01</th>
      <td>151343</td>
    </tr>
    <tr>
      <th>2021-11-01</th>
      <td>127128</td>
    </tr>
    <tr>
      <th>2021-12-01</th>
      <td>104484</td>
    </tr>
  </tbody>
</table>
</div>



We first will visualise the time series. The red dashed line indicates when the first lockdown was announced in the UK on the 23rd March 2020. Any dates after this are highlighted in orange. Clearly the number of publications have been increasing steadily over time. At first glance there does seem to be a change in trend during the pandemic, where the post-covid dates seem to indicate a more rapid increase in the number of publications. We now need to establish if it is statistically significant.


```python
fig = plt.figure(figsize=(8,4), dpi=200)

df_pubs_post_covid = df_pubs.iloc[662:]  # Post UK lockdown indexes

plt.plot(df_pubs.index, df_pubs['n_publications'], '.')
plt.plot(df_pubs_post_covid.index, df_pubs_post_covid['n_publications'], '.')
# plt.ylim([0,150000])
plt.xlim([dt.date(1970,1,1), dt.date(2023,1,1)])
plt.ylabel('Number of publications in a month')
plt.xlabel('Date')
plt.plot([dt.date(2020,3,23),dt.date(2020,3,23)],[0, 500e3], '--r', label='First UK lockdown announced')

```



<img src="/assets/20220726-imgs/Academic-publication-by-year-v3_prod_12_1.png"/>



There is a lot of noise in the data. The months with more than 200,000 publications seem pretty unreasonable and are likely dodgy values returned by the query. It's safe to filter any data points which are more than 4 standard deviations from the average monthly number of publications. We arrive at the  following plot: 


```python
# z-score is the number of standard deviations from the mean value:
df_pubs['z_score'] = (df_pubs.n_publications - df_pubs.n_publications.mean())/df_pubs.n_publications.std()
df_pubs_filt = df_pubs.loc[df_pubs.z_score.abs() < 4].copy()# Filter out datapoints with a z-score more than 4
df_pubs_post_covid = df_pubs_filt.iloc[655:]  # Post UK lockdown indexes on filtered data

fig = plt.figure(figsize=(8,4), dpi=200)
plt.plot(df_pubs_filt.index, df_pubs_filt['n_publications'], '.')
plt.plot(df_pubs_post_covid.index, df_pubs_post_covid['n_publications'], '.')
# plt.ylim([0,150000])
plt.xlim([dt.date(1970,1,1), dt.date(2023,1,1)])
plt.ylabel('Number of publications in a month')
plt.xlabel('Date')
plt.plot([dt.date(2020,3,23),dt.date(2020,3,23)],[0, 500e3], '--r', label='First UK lockdown announced')

```


<img src="/assets/20220726-imgs/Academic-publication-by-year-v3_prod_14_1.png"/>



In addtion, we can group the data by year to reduce the granularity of the data and reduce the impact of any remaining outlier datapoints. We create the 'year' columns for grouping the data and calculate some aggregated statistics within each year. 


```python
df_pubs_filt['year'] = df_pubs_filt.index.year
pubs_by_year = df_pubs_filt.groupby(by='year').agg(total_publications = ('n_publications' , 'sum'),
                                              mean_publications = ('n_publications' , 'mean'),
                                              std_publications = ('n_publications' , 'std'),
                                              count_publications = ('n_publications' , 'count'))
```


```python
pubs_by_year.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>total_publications</th>
      <th>mean_publications</th>
      <th>std_publications</th>
      <th>count_publications</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1965</th>
      <td>20263</td>
      <td>1688.583333</td>
      <td>3966.006016</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1966</th>
      <td>141482</td>
      <td>11790.166667</td>
      <td>3137.332562</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1967</th>
      <td>158751</td>
      <td>13229.250000</td>
      <td>4591.894739</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1968</th>
      <td>217723</td>
      <td>18143.583333</td>
      <td>5285.883180</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1969</th>
      <td>241035</td>
      <td>20086.250000</td>
      <td>4011.041785</td>
      <td>12</td>
    </tr>
  </tbody>
</table>
</div>



If we make the coarse assumption that on the timescale of a year, the number of monthly publications if roughly constant, then we can create an average monthly publications value for each year. With this assumption we can also use the standard deviation to obtain the standard error on this average monthly publications value.


```python
pubs_by_year['error'] = pubs_by_year.std_publications/np.sqrt(pubs_by_year.count_publications)

```

We can calculate the change in this average number of monthly publications for each year. We can also use error propagation to calulcate the errors on this difference value:


```python
pubs_by_year['delta'] = pubs_by_year.mean_publications.diff()
pubs_by_year['delta_error'] = np.sqrt(pubs_by_year['error']**2 + np.roll(pubs_by_year['error'], shift=1)**2) 
```

If we now plot the mean monthly publications for each year, we see that the spikes between 1995 and 2005 are associated with large errors, supporting that these are likely caused by singular outliers point with dramatic variance from the neighbouring monthly publication values within that year. 

The change in publications for the same period between 1995 and 2005 has large errors comparable to the size of the change itself, demonstrating that these values can not be trusted. The mean change in monthly publications is plotted as the grey dashed line, which is 2063 publiations per month, and the red regions indicate the range of values below the 95% confidence interval based on this mean monthly publication value. This further empahsises that the 2021 year had an unusually high publications rate. 

Interestingly, the 2021 data point has a small error, suggesting the variance around the mean value for that year was relatively small, i.e most the monthly counts of publication for that year were above that for the previous year. Likewise, the error on the average change in monthly publications between 2020 and 2021 is small in comparision with the change, suggesting there has been a geniune increase in the number of publications in 2021. 




```python
fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(12,7), dpi=200, sharex=True)

ax[0].errorbar(x=pubs_by_year.index, y=pubs_by_year['mean_publications'], yerr=pubs_by_year['error'],
             fmt='.',
             capsize=2,
             label='Total publications that year')
ax[1].bar(x=pubs_by_year.index, height=pubs_by_year['delta'], yerr=pubs_by_year['delta_error'],
        color='orange',
        capsize=2,
        label='change in publications from previous year')

mn = pubs_by_year.delta.mean()
# Remove any years where the error is great than 50,000
ci_95 = 1.96*(pubs_by_year.delta.std())

ax[0].plot([2020,2020],[0, 200e3],'--r')
ax[1].plot([2020,2020],[-180e3, 180e3],'--r')
ax[1].plot([1970, 2023], [mn,mn], 'k--', alpha=0.5)

ax[1].fill_between([1970, 2023], [mn-ci_95, mn-ci_95], color='red', alpha=0.2)
ax[1].fill_between([1970, 2023], [mn+ci_95, mn+ci_95], color='red', alpha=0.2)

ax[0].grid()
ax[0].set_xlim([1970, 2023])
ax[0].set_ylim([0, 150e3])
ax[0].set_ylabel('Average monthly publications')

ax[1].grid()
ax[1].set_xlim([1970, 2023])
ax[1].set_ylim([-68e3, 68e3])
ax[1].set_ylabel('Change in average monthly publications')
ax[1].set_xlabel('Date')
plt.subplots_adjust(hspace=0)
```


<img src="/assets/20220726-imgs/Academic-publication-by-year-v3_prod_23_0.png"/>


To further test whether 2021 was a bumper year for publications, we calculate the z-score for the delta columns as follows:



```python
pubs_by_year['delta_z_score'] = (pubs_by_year.delta - pubs_by_year.delta.mean())/pubs_by_year.delta.std()
pubs_by_year.sort_values(by='delta_z_score', ascending=False)[['delta', 'delta_error', 'delta_z_score']].head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>delta</th>
      <th>delta_error</th>
      <th>delta_z_score</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2021</th>
      <td>38877.750000</td>
      <td>7434.241562</td>
      <td>3.946041</td>
    </tr>
    <tr>
      <th>2010</th>
      <td>26053.916667</td>
      <td>11923.284850</td>
      <td>2.571504</td>
    </tr>
    <tr>
      <th>2004</th>
      <td>25874.750000</td>
      <td>16035.038797</td>
      <td>2.552300</td>
    </tr>
    <tr>
      <th>2002</th>
      <td>18203.000000</td>
      <td>12212.361678</td>
      <td>1.729995</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>14486.666667</td>
      <td>4153.076608</td>
      <td>1.331656</td>
    </tr>
  </tbody>
</table>
</div>



Here, 2021 comes out top with a z-score on our filtered data of 3.95. Assuming a normal distribution for the null hypothesis, this would correspond to a one-sided p-value of 0.00004 which is P < 0.05 indicating that it is statistically significant. We take one-sided as we are interested in significantly high numbers of publications. 

Equally, we have some high z-scores for 2010, 2004 and 2002 which are unexplained and equally signifiant based on this analysis. One question is whether the normal distribution is truly valid. The limited data however means that it is hard to conclude for sure. However, we can plot a histogram of the delta_z_score column and we see visually that the 2021 value is indeed an extreme value


```python
fig, ax = plt.subplots(figsize=(8,4), dpi=200)

ax.annotate('2021', xy=(3.8, 1.2), xytext=(3.8, 3),
            arrowprops=dict(facecolor='black', shrink=0.05),
            )
pubs_by_year.delta_z_score.hist(bins=20)
plt.xlabel('Change in average monthly publications as a z-score')
plt.ylabel('Count')
```




    Text(0, 0.5, 'Count')





<img src="/assets/20220726-imgs/Academic-publication-by-year-v3_prod_27_1.png"/>



# Conclusions

From this analysis, it would seem that 2021 was a bumper year for publications. We can't say whether this was directly because of the pandemic, perhaps there were indirect effects e.g. perhaps due to homeworking the review process was quicker during that pandemic papers and papers were just published more quickly rather than more papers actually being written. 

2021 is just one datapoint of course, so we mustn't read into it too much! It would be interesting to see how the trend behaves in the coming years, or even better, develop a forecasting model for the number of publications. 

