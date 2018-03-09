# Problem Set 3: Scraping and Cleaning Twitter Data

Now that you know how to scrape data from Twitter, let's extend the exercise a little so you can show us what you know. You will set up the scraper, clean the resulting data, and visualize it. Make sure you get your own Twitter key (AND make sure that you don't accidentally push it to GitHub); careful with your `.gitignore`.

## Graphic Presentation

Make sure to label all your axes and add legends and units (where appropriate)! Think of these graphs as though they were appearing in a published report for an audience unfamiliar with the data.

## Don't Work on Incomplete Data!

One of the dangers of cleaning data is that you inadvertently delete data that is pertinent to your analysis. If you find yourself getting strange results, you can always run previous portions of your script again to rewind your data. See the section called 'reloading your Tweets in the workshop.

## Deliverables

### Push to GitHub

1. A Python script that contains your scraper code in the provided submission folder. You can copy much of the provided scraper, but you'll have to customize it. This should include the code to generate two scatterplots, and the code you use to clean your datasets.
2. Extra Credit: A Python script that contains the code you used to scrape Wikipedia with the BeautifulSoup library.

### Submit to Stellar

1. Your final CSV files---one with no search term, one with your chosen search term---appropriately cleaned.
2. Extra Credit: A CSV file produced by your BeautifulSoup scraper.

## Instructions

### Step 1

Using the Twitter REST API, collect at least 80,000 tweets. Do not specify a search term. Use a lat/lng of `42.359416,-71.093993` and a radius of `5mi`. Note that this will probably take 20-30 minutes to run.

```python

import jsonpickle
import tweepy as tweepy
import pandas as pd
import os
os.chdir('week-04')
from twitter_keys import api_key, api_secret
auth = tweepy.AppAuthHandler(api_key,api_secret)
api = tweepy.API(auth, wait_on_rate_limit = True, wait_on_rate_limit_notify = True)
def auth(key, secret):
  auth = tweepy.AppAuthHandler(key, secret)
  api = tweepy.API(auth, wait_on_rate_limit = True, wait_on_rate_limit_notify = True)
  # Print error and exit if there is an authentication error
  if (not api):
      print ("Can't Authenticate")
      sys.exit(-1)
  else:
      return api

api = auth(api_key, api_secret)

def parse_tweet(tweet):
  p = pd.Series()
  if tweet.coordinates != None:
    p['lat'] = tweet.coordinates['coordinates'][0]
    p['lon'] = tweet.coordinates['coordinates'][1]
  else:
    p['lat'] = None
    p['lon'] = None
  p['location'] = tweet.user.location
  p['id'] = tweet.id_str
  p['content'] = tweet.text
  p['user'] = tweet.user.screen_name
  p['user_id'] = tweet.user.id_str
  p['time'] = str(tweet.created_at)
  return p


def get_tweets(
    geo,
    out_file,
    search_term = '',
    tweet_per_query = 100,
    tweet_max = 150,
    since_id = None, ##what does since_id do?
    max_id = -1,
    write = False
  ):
  tweet_count = 0
  all_tweets = pd.DataFrame()
  while tweet_count < tweet_max:
    try:
      if (max_id <= 0):
        if (not since_id):
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo
          )
        else:
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo,
            since_id = since_id
          )
      else:
        if (not since_id):
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo,
            max_id = str(max_id - 1)
          )
        else:
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo,
            max_id = str(max_id - 1),
            since_id = since_id
          )
      if (not new_tweets):
        print("No more tweets found")
        break
      for tweet in new_tweets:
        all_tweets = all_tweets.append(parse_tweet(tweet), ignore_index = True)
      if write == True:
          with open(out_file, 'w') as f:
              f.write(jsonpickle.encode(tweet._json, unpicklable=False) + '\n')
      max_id = new_tweets[-1].id  ##???
      tweet_count += len(new_tweets)
    except tweepy.TweepError as e:
      # Just exit if any error
      print("Error : " + str(e))
      break
  print (f"Downloaded {tweet_count} tweets.")
#  all_tweets.to_json('')
  return all_tweets

# Set a Lat Lon
latlng = '42.359416,-71.093993' # Eric's office (ish)
# Set a search distance
radius = '5mi'
# See tweepy API reference for format specifications
geocode_query = latlng + ',' + radius
# set output file location
file_name = 'data/Pset3_tweets.json'
# set threshold number of Tweets. Note that it's possible
# to get more than one
t_max = 80000

tweets = get_tweets(
 geo = geocode_query,
 tweet_max = t_max,
 write = True,
 out_file = file_name
)
tweets.shape
tweets.to_json('data/Pset3_8tweets.json')
tweets.to_csv('data/Pset3_8tweets.csv')
tweet_copy = tweets
tweets = tweet_copy
tweet_copy.shape
tweets.shape
```

### Step 2

Clean up the data so that variations of the same user-provided location name are replaced with a single variation. Once you've cleaned up the locations, create a pie chart of user-provided locations. Your pie chart should strive for legibility! Let the [`matplotlib` documentation](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.pie.html) be your guide!
```python
df = pd.read_json('data/Pset3_8tweets.json')
df.shape
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

df.drop_duplicates(subset = 'content', keep = False, inplace = True)
df.shape
import re as re
#use re to clean multiple locations
clean = ["boston", "cambridge", "texas", "allston", "brooklyn", "California", "brighton", "Watertown", "washington", "somerville", "florida", "New York", "chelsea","brookline"]
replace = ["Boston, MA", "Cambridge, MA", "Texas", "Allston, MA", "Brooklyn, MA", "California","Brighton, MA", "Watertown, MA", "Washington, DC", "Somerville, MA", "Florida", "New York", "Chelsea, MA","Brookline, MA"]
for i in range(0,len(clean)):
  pattern = re.compile(clean[i], flags=re.IGNORECASE)
  list_tobereplaced = df[df['location'].str.contains(pattern)]['location']
  df['location'].replace(list_tobereplaced, replace[i], inplace = True)

#BOS
bos_list = df[df['location'].str.contains("BOS")]['location']
df['location'].replace(bos_list, 'Boston, MA', inplace = True)

#USA
usa_list = df[df['location'].str.contains("USA")]['location']
df['location'].replace(usa_list, 'United States', inplace = True)

#CA
ca_list = df[df['location'].str.contains("CA")]['location']
df['location'].replace(ca_list, 'California', inplace = True)

#use of df
loc_tweets = df[df['location'] != '']
count_tweets = loc_tweets.groupby('location')['id'].count()
df_count_tweets = count_tweets.to_frame()
df_count_tweets.columns
df_count_tweets.columns = ['count']
#df_count_tweets.sort_index()
```
explanation:
The next step is to create the pie chart. But since there are too many locations and some of the content of locations are irrelevant, I only selected a subset of the locations whose counts are between 40 and 31103. The lower bound 40 helped me eliminate some of the noise and the upper bound 31103 helped me eliminate Boston. Because the number of tweets with location Boston dominates, a pie chart including Boston will skew the percentages and make the parts of other locations small. There are 15 locations that have counts 40~31103 and the names look reasonable.
```python
#df_count_tweets
choose = df_count_tweets[(df_count_tweets['count']>40) & (df_count_tweets['count']<31103)]

choose

# Create a list of colors (from iWantHue)
colors=    ["#87659d",
          "#5bdddc",
          "#9b5a79",
          "#5bcde9",
          "#db96c1",
          "#47a3ad",
          "#bcace6",
          "#92dee3",
          "#4e70a9",
          "#dcbdd0",
          "#3b7383",
          "#abcbe1",
          "#65637a",
          "#65a9de",
          "#8995af"]
# Create a pie chart
plt.pie(choose['count'], labels=choose.index.get_values(), shadow=False, colors=colors)
plt.title("Locations Shown over 40 Times in Tweets \n within 5 Mile Radius of 42.359416,-71.093993")
plt.axis('equal')
plt.tight_layout()
plt.show()
```


### Step 3

Create a scatterplot showing all of the tweets are that are geolocated (i.e., include a latitude and longitude).

```Python
tweets_geo = tweets[tweets['lon'].notnull() & tweets['lat'].notnull()]
len(tweets_geo)
len(tweets)
plt.scatter(tweets_geo['lon'], tweets_geo['lat'], s = 40,c='darkgreen')
plt.title('Scatterplot of 198 Geolocated Tweets')
plt.show()

```


### Step 4

Pick a search term (e.g., "housing", "climate", "flood") and collect tweets containing it. Use the same lat/lon and search radius for Boston as you used above. Depending on the search term, you may find that there are relatively few available tweets.

```python
#I used the search term "snow"
def parse_tweet(tweet):
  p = pd.Series()
  if tweet.coordinates != None:
    p['lat'] = tweet.coordinates['coordinates'][0]
    p['lon'] = tweet.coordinates['coordinates'][1]
  else:
    p['lat'] = None
    p['lon'] = None
  p['location'] = tweet.user.location
  p['id'] = tweet.id_str
  p['content'] = tweet.text
  p['user'] = tweet.user.screen_name
  p['user_id'] = tweet.user.id_str
  p['time'] = str(tweet.created_at)
  return p


def get_tweets(
    geo,
    out_file,
    search_term = 'snow',
    tweet_per_query = 100,
    tweet_max = 150,
    since_id = None, ##what does since_id do?
    max_id = -1,
    write = False
  ):
  tweet_count = 0
  all_tweets = pd.DataFrame()
  while tweet_count < tweet_max:
    try:
      if (max_id <= 0):
        if (not since_id):
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo
          )
        else:
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo,
            since_id = since_id
          )
      else:
        if (not since_id):
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo,
            max_id = str(max_id - 1)
          )
        else:
          new_tweets = api.search(
            q = search_term,
            rpp = tweet_per_query,
            geocode = geo,
            max_id = str(max_id - 1),
            since_id = since_id
          )
      if (not new_tweets):
        print("No more tweets found")
        break
      for tweet in new_tweets:
        all_tweets = all_tweets.append(parse_tweet(tweet), ignore_index = True)
      if write == True:
          with open(out_file, 'w') as f:
              f.write(jsonpickle.encode(tweet._json, unpicklable=False) + '\n')
      max_id = new_tweets[-1].id  ##???
      tweet_count += len(new_tweets)
    except tweepy.TweepError as e:
      # Just exit if any error
      print("Error : " + str(e))
      break
  print (f"Downloaded {tweet_count} tweets.")
#  all_tweets.to_json('')
  return all_tweets

# Set a Lat Lon
latlng = '42.359416,-71.093993' # Eric's office (ish)
# Set a search distance
radius = '5mi'
# See tweepy API reference for format specifications
geocode_query = latlng + ',' + radius
# set output file location
file_name = 'data/Pset3_tweets_searchterm.json'
# set threshold number of Tweets. Note that it's possible
# to get more than one
t_max = 80000

tweets_term = get_tweets(
 geo = geocode_query,
 tweet_max = t_max,
 write = True,
 out_file = file_name
)

tweets_term.shape
tweets_term.to_json('data/Pset3_tweets_term.json')
tweets_term.to_csv('data/Pset3_tweets_term.csv')
tweet_term_copy = tweets_term
#tweets = tweet_copy
#tweet_copy.shape
#tweets.shape

```
### Step 5

Clean the search term data as with the previous data.


```Python
df1 = pd.read_json('data/Pset3_tweets_term.json')
df1.shape
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline

df1.drop_duplicates(subset = 'content', keep = False, inplace = True)
df1.shape

import re as re
#use re to clean multiple locations
clean = ["boston", "cambridge", "texas", "allston", "brooklyn", "California", "brighton", "Watertown", "washington", "somerville", "florida", "New York", "chelsea","brookline"]
replace = ["Boston, MA", "Cambridge, MA", "Texas", "Allston, MA", "Brooklyn, MA", "California","Brighton, MA", "Watertown, MA", "Washington, DC", "Somerville, MA", "Florida", "New York", "Chelsea, MA", "Brookline, MA"]
for i in range(0,len(clean)):
  pattern = re.compile(clean[i], flags=re.IGNORECASE)
  list_tobereplaced = df1[df1['location'].str.contains(pattern)]['location']
  df1['location'].replace(list_tobereplaced, replace[i], inplace = True)

#BOS
bos_list = df1[df1['location'].str.contains("BOS")]['location']
df1['location'].replace(bos_list, 'Boston, MA', inplace = True)

#USA
usa_list = df1[df1['location'].str.contains("USA")]['location']
df1['location'].replace(usa_list, 'United States', inplace = True)

#CA
ca_list = df1[df1['location'].str.contains("CA")]['location']
df1['location'].replace(ca_list, 'California', inplace = True)


#use of df
loc_tweets = df1[df1['location'] != '']
count_tweets = loc_tweets.groupby('location')['id'].count()
df_count_tweets = count_tweets.to_frame()
#df_count_tweets
df_count_tweets.columns

df_count_tweets.columns = ['count']

df_count_tweets
#(df_count_tweets['count']>20) & (df_count_tweets['count']<31103)]

choose = df_count_tweets[(df_count_tweets['count']>4) & (df_count_tweets['count']<2461) ]
choose
len(choose)

# Create a list of colors (from iWantHue)
colors=  ["#ccdcc5","#e67151",
"#70d76c",
"#6d97e2",
"#b8ca45",
"#78b6cd",
"#db9e33",
"#69deba",
"#ca9b83",
"#6aa452",
"#b99c56",
"#74a387",
"#d2de8c"]
# Create a pie chart
plt.pie(choose['count'], labels=choose.index.get_values(), shadow=False, colors=colors)
plt.title("Locations Shown >=5 Times in Tweets about SNOW \n within 5 Mile Radius of 42.359416,-71.093993")
plt.axis('equal')
plt.tight_layout()
plt.show()
```



### Step 6

Create a scatterplot showing all of the tweets that include your search term that are geolocated (i.e., include a latitude and longitude).
```Python
tweets_geo = df1[df1['lon'].notnull() & df1['lat'].notnull()]
len(tweets_geo)
len(df1)
plt.scatter(tweets_geo['lon'], tweets_geo['lat'], s = 40, c='darkgreen')
plt.title('Scatterplot of 198 Geolocated Tweets')
plt.show()

```



### Step 7

Export your scraped Twitter datasets (one with a search term, one without) to two CSV files. We will be checking this CSV file for duplicates and for consistent location names, so make sure you clean carefully!


```Python
##the df here refers to the first df with no search term
df.to_csv("data/Pset3_Xuenan_no_search.csv")

#the df1 here refers to the 2nd df with search term "snow"
df1.to_csv("data/Pset3_Xuenan_with_search.csv")

```

## Extra Credit Opportunity

Build a scraper that downloads and parses the Wikipedia [List of Countries by Greenhouse Gas Emissions page](https://en.wikipedia.org/wiki/List_of_countries_by_greenhouse_gas_emissions) using BeautifulSoup and outputs the table of countries as as a CSV.
