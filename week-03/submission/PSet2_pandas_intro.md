# Problem Set 2: Intro to Pandas

Building off the in-class workshop, this problem set will require you to use some of Python's data wrangling functions and produce a few simple plots with Matplotlib. These plots will help us begin to think about how the aggregated GPS data works, how it might be useful, and how it might fall short.

## What to Submit

Create a duplicate of this file (`PSet2_pandas_intro.md`) in the provided 'submission' folder; your solutions to each problem should be included in the `python` code block sections beneath the 'Solution' heading in each problem section.

Be careful! We have to be able to run your code. This means that if you, for example, change a variable name and neglect to change every appearance of that name in your code, we're going to run into problems.

## Graphic Presentation

Make sure to label all the axes and add legends and units (where appropriate).

## Code Quality

While code performance and optimization won't count, all the code should be highly readable, and reusable. Where possible, create functions, build helper functions where needed, and make sure the code is self-explanatory.

## Preparing the Data

You'll want to make sure that your data is prepared using the procedure we followed in class. The code is reproduced below; you should simply be able to run the code and reproduce the dataset with well-formatted datetime dates and no erroneous hour values.

```python
import pandas as pd
import numpy as np
import matplotlib.pylab as plt

# This line lets us plot on our ipython notebook
%matplotlib inline

# Read in the data

df = pd.read_csv('week-03/data/skyhook_2017-07.csv', sep=',')

# Create a new date column formatted as datetimes.
df['date_new'] = pd.to_datetime(df['date'], format='%Y-%m-%d')

# Determine which weekday a given date lands on, and adjust it to account for the fact that '0' in our hours field corresponds to Sunday, but .weekday() returns 0 for Monday.
df['weekday'] = df['date_new'].apply(lambda x: x.weekday() + 1)
df['weekday'].replace(7, 0, inplace = True)

df
# Remove hour variables outside of the 24-hour window corresponding to the day of the week a given date lands on.
for i in range(0, 168, 24):
  j = range(0,168,1)[i - 5]
  if (j > i):
    df.drop(df[
    (df['weekday'] == (i/24)) &
    (
    ( (df['hour'] < j) & (df['hour'] > i + 18) ) |
    ( (df['hour'] > i + 18 ) & (df['hour'] < j) )
    )
    ].index, inplace = True)
  else:
    df.drop(df[
    (df['weekday'] == (i/24)) &
    (
    (df['hour'] < j) | (df['hour'] > i + 18 )
    )
    ].index, inplace = True)
```

## Problem 1: Create a Bar Chart of Total Pings by Date

Your first task is to create a bar chart (not a line chart!) of the total count of GPS pings, collapsed by date. You'll have to use `.groupby` to collapse your table on the grouping variable and choose how to aggregate the `count` column. Your code should specify a color for the bar chart and your plot should have a title. Check out the [Pandas Visualization documentation](https://pandas.pydata.org/pandas-docs/stable/visualization.html) for some guidance regarding what parameters you can customize and what they do.

### Solution

```python

len(df)

gps_count = df.groupby('date_new')['count'].describe()
len(gps_count)
aa = df.groupby('date_new')['count'].sum()
aa.plot(kind = "bar",color = 'purple',title = 'Bar Chart of Total Counts of Pings Group By Date')

#Need to specify color and

```

## Problem 2: Modify the Hours Column

Your second task is to further clean the data. While we've successfully cleaned our data in one way (ridding it of values that are outside the 24-hour window that correspond to a given day of the week) it will be helpful to restructure our `hour` column in such a way that hours are listed in a more familiar 24-hour range. To do this, you'll want to more or less copy the structure of the code we used to remove data from hours outside of a given day's 24-hour window. You'll then want to use the [DataFrame's `replace` method](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.replace.html). Note that you can use lists in both `to_replace` and `value`.

After running your code, you should have either a new column in your DataFrame or new values in the 'hour' column. These should range from 0-23. You can test this out in a couple ways; the simplest is probably to `df['hour'].unique()`; if you're interested in seeing sums of total pings by hour, you can run `df.groupby('hour')['count'].sum()`.

### Solution

```python
df['hour_new'] = 0
df.loc[df['hour'] <=18, 'hour_new'] = df['hour']+5
df.loc[(df['hour'] >18), 'hour_new'] = (df['hour'] - 19)%24
df.loc[df['hour'] >=163, 'hour_new'] = df['hour']-163

df['hour_new'].unique()
df.groupby('hour_new')['count'].sum()
df
```

## Problem 3: Create a Timestamp Column

Now that you have both a date and a time (stored in a more familiar 24-hour range), you can combine them to make a single timestamp. Because the columns in a `pandas` DataFrames are vectorized, this is a relatively simple matter of addition, with a single catch: you'll need to use `pd.to_timedelta` to convert your hours columns to a duration.

### Solution

```python
len(df['date'].unique())
base = pd.to_datetime(df["date_new"], format="%Y %H:%M")
ts = base + pd.to_timedelta(df['hour_new'], unit='h')
df['ts'] = ts
```

## Problem 4: Create Two Line Charts of Activity by Hour

Create two more graphs. The first should be a **line plot** of **total activity** by your new `timestamp` field---in other words a line graph that displays the total number of GPS pings in each hour over the course of the week. The second should be a **bar chart** of **summed counts** by hours of the day---in other words, a bar chart displaying the sum of GPS pings occurring across locations for each of the day's 24 hours.

### Solution

```python
df.groupby('ts')['count'].describe()
count_ts = df.groupby(ts)['count'].sum()
count_ts.plot(color = 'purple', title = 'Line Chart of Counts of Pings')


sum_ts = df.groupby('hour_new')['count'].sum()
sum_ts.plot(kind = 'bar',color = 'purple',title = 'Bar Chart of Sum of Counts of Pings')

```

## Problem 5: Create a Scatter Plot of Shaded by Activity

Pick three times (or time ranges) and use the latitude and longitude to produce scatterplots of each. In each of these scatterplots, the size of the dot should correspond to the number of GPS pings. Find the [Scatterplot documentation here](http://pandas.pydata.org/pandas-docs/version/0.19.1/visualization.html#scatter-plot). You may also want to look into how to specify a pandas Timestamp (e.g., pd.Timestamp) so that you can write a mask that will filter your DataFrame appropriately. Start with the [Timestamp documentation](https://pandas.pydata.org/pandas-docs/stable/timeseries.html#timestamps-vs-time-spans)!

```python
##time1: specifying the exact time at 2017-01-01 08:00:00
time1 = df[df['ts'] == '2017-07-01 08:00:00']

time1.plot.scatter(x='lon', y='lat', s=time1['count']*0.1, title = "Scatter Plot of GPS Pings at 2017-07-01 08:00:00")


#time2: specifying a time range on 2017-07-02 AM

time2 = df[(df['ts'] > '2017-07-02 00:00:00') & (df['ts'] < '2017-07-02 12:00:00')]
time2.plot.scatter(x='lon', y='lat', s=time2['count']*0.01, title = "Scatter Plot of GPS Pings at 2017-07-02 AM",color = 'darkgreen')


#time3: specifying a time range after 2017-07-31 12:00:00
time3 = df[df['ts'] > '2017-07-30 17:00:00']
time3.plot.scatter(x='lon', y='lat', s=time3['count']*0.01, title = "Scatter Plot of GPS Pings  after 2017-07-31 12:00:00", color = 'pink')
#ax = plt.gca()
#ax.set_xlim(ax.get_xlim()[::-1])

```

## Problem 6: Analyze Your (Very) Preliminary Findings

For three of the visualizations you produced above, write a one or two paragraph analysis that identifies:

1. A phenomenon that the data make visible (for example, how location services are utilized over the course of a day and why this might be).
2. A shortcoming in the completeness of the data that becomes obvious when it is visualized.
3. How this data could help us identify vulnerabilities related to climate change in the greater Boston area.

I tried different hours on a day; it seems that hours in the early morning (like 8am on 7/1 in plot 1) have few pins in general and those pings do not spread out through the city but rather concentrate at certain places, like Allston and Harvard. During the peak hours I could tell that people move within the city more frequently and intensely, as the pings scatter across the city and show patterns of the major roads. If we increase the time range to cover more hours, we could clearly see a cityscape pattern constituted by pings. The mostly travelled parts in Boston are the Highways such as I90 and I93. This result makes sense as there are more travels on the arterial roads.
One shortcoming I noticed is that at the peripheral areas there are large empty lands where no pings are shown. It is reasonable they are less utilized parts of the city, but could it also be that less towers are built nearby so that signals at some grids are not captured? Also, I guess because of the scraping algorithm we can only scrape data within a lat/lon boundary, which does not collect comprehensive data for the entire Great Boston area. Furthermore, I think this visualization helps us identify infrastructure that are heavily used and are under the threat of global warming. For example, the I93 interstate highway has parts very close to the coast and shoulders heavy traffic. To my understanding, it does have vulnerabilities potentially like erosion and runoff related to climate change which would be harmful.
