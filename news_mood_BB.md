
## Observations:

1. According to the vader sentiment analysis scatter plot covering the 200 tweet testing period, @CNN sent several of most negative individual tweets. However, they sent many neutral+ tweets, which resulted in @CNN having the highest overall most positive compound sentiment score reflected on the bar chart. This suggests that any organizanization could tweet really negative things and still be perceived as an overall positive contributor on social media if they send enough tweets rated with an above 0 compound score, even if the tweets score in teh highly positive range.

2. According to the vader sentiment analysis scatter plot covering the 200 tweet testing period, @CBSNews sent no neutral- individual tweets, many positive or neutral+ individual tweets, and several negative tweets. The overall data frame indicates they had the highest positive score, but their negative tweets pulled down their compound score below @CNN's even though @CNN had no highly positive tweets.

3. When we make a casual comparison of the positive, negative, neutral, and compound score values contained in the overall scoring data frame, it becomes clear that negative scores make up the highest proportion of each news outlet's tweets. Also, the positive and neutral scores must be weighted more heavily when computing the normalizing compound score. These facts provide a full explanation for the trends identified in observations 1 and 2.

4. As an additional note, I ran the API call two times a day, on 3 seperate days and applied the analytical code each time. For each day the second pull occurred between 1 and 4 hours apart. In each case the scatter and bar charts changed drastically. This assignment called for a minimum of 100 tweets to be analyzed, but I analyzed 200 because I thought 100 was not a large enough sample. Given the widely variable analytics just on a daily basis, I suggest that a still much larger sample or an explanded study method are needed in order to make a clear comparison between each news outlet's tweet sentiment behaviors.


```python
# Modules
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import json
import tweepy

from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()
```


```python
# Please use your own twitter API keys
consumer_key = ""
consumer_secret = ""
access_token = ""
access_token_secret = ""

```


```python
# Tweepy setup
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())

```


```python
# Identify news twitter accounts to be analyzed
# I added @WSJ in order to give print news better representation and specified BBCNorthAmerica
# to better match regional news cycles with the other target users
target_users = ("@BBCNorthAmerica", "@CNN", "@CBSNews", "@FoxNews", "@nytimes", "@WSJ")
```


```python
# setup sentiment variable list for later array setup
sentiment = []
```


```python
# Setup loop of the target list
for user in target_users:
    
    # Set counter to track tweets ago
    counter = 0

    # Setup loop for review of 200 tweets over 10 pages
    for x in range(10):
    
        # Get tweets with the help of tweepy
        public_tweets = api.user_timeline(user)

        # Setup loop of individual tweets for one by one analysis
        for tweet in public_tweets:
            
            # Put each tweet through Vader analysis
            compound = analyzer.polarity_scores(tweet["text"])["compound"]
            pos = analyzer.polarity_scores(tweet["text"])["pos"]
            neu = analyzer.polarity_scores(tweet["text"])["neu"]
            neg = analyzer.polarity_scores(tweet["text"])["neg"]
            tweets_ago = counter
            
            # Append sentiment variable list as array
            sentiment.append({"News Source": user, 
                              "Date": tweet["created_at"], 
                              "Compound": compound, 
                              "Positive": pos, 
                              "Negative": neu, 
                              "Neutral": neg, 
                              "Tweets Ago": counter, 
                              "Tweet Text": tweet["text"]})
            
            # Update counter
            counter = counter + 1

```


```python
# Put all tweet sentiments data into dataframe  
news_tweet_vader = pd.DataFrame.from_dict(sentiment)
news_tweet_vader.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>News Source</th>
      <th>Positive</th>
      <th>Tweet Text</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.8271</td>
      <td>Sun Dec 17 16:55:29 +0000 2017</td>
      <td>0.315</td>
      <td>0.685</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>CIA 'helped stop Russia terror attack' https:/...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.2960</td>
      <td>Sun Dec 17 15:23:05 +0000 2017</td>
      <td>0.879</td>
      <td>0.121</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>"There's no amount of money that can compensat...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>Sun Dec 17 14:41:40 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>Christmas across US Mexico border https://t.co...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>Sun Dec 17 13:50:04 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>It's time to tell you about the family busines...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.7184</td>
      <td>Sun Dec 17 12:19:08 +0000 2017</td>
      <td>0.625</td>
      <td>0.375</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>Barry Sherman: Family disputes reports on myst...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(news_tweet_vader)
```




    1200




```python
# Save bulk data dataframe as a csv
news_tweet_vader.to_csv("news_source_tweet_sentiment.csv", encoding="utf-8", index=False)
```


```python
# Split mixed data into individual news source dataframes

bbc_vader_df = news_tweet_vader.loc[news_tweet_vader["News Source"] == "@BBCNorthAmerica", :]
bbc_vader_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>News Source</th>
      <th>Positive</th>
      <th>Tweet Text</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.8271</td>
      <td>Sun Dec 17 16:55:29 +0000 2017</td>
      <td>0.315</td>
      <td>0.685</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>CIA 'helped stop Russia terror attack' https:/...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.2960</td>
      <td>Sun Dec 17 15:23:05 +0000 2017</td>
      <td>0.879</td>
      <td>0.121</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>"There's no amount of money that can compensat...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.0000</td>
      <td>Sun Dec 17 14:41:40 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>Christmas across US Mexico border https://t.co...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.0000</td>
      <td>Sun Dec 17 13:50:04 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>It's time to tell you about the family busines...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.7184</td>
      <td>Sun Dec 17 12:19:08 +0000 2017</td>
      <td>0.625</td>
      <td>0.375</td>
      <td>@BBCNorthAmerica</td>
      <td>0.0</td>
      <td>Barry Sherman: Family disputes reports on myst...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
cnn_vader_df = news_tweet_vader.loc[news_tweet_vader["News Source"] == "@CNN", :]
cnn_vader_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>News Source</th>
      <th>Positive</th>
      <th>Tweet Text</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>200</th>
      <td>0.4588</td>
      <td>Sun Dec 17 17:45:04 +0000 2017</td>
      <td>0.850</td>
      <td>0.000</td>
      <td>@CNN</td>
      <td>0.150</td>
      <td>Here's what's in the tax bill for homeowners:\...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>201</th>
      <td>0.0164</td>
      <td>Sun Dec 17 17:30:31 +0000 2017</td>
      <td>0.685</td>
      <td>0.156</td>
      <td>@CNN</td>
      <td>0.159</td>
      <td>Their missions vary greatly: To provide loving...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>202</th>
      <td>0.2500</td>
      <td>Sun Dec 17 17:22:18 +0000 2017</td>
      <td>0.895</td>
      <td>0.000</td>
      <td>@CNN</td>
      <td>0.105</td>
      <td>"Do I need a British accent?" former President...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>203</th>
      <td>0.4019</td>
      <td>Sun Dec 17 17:00:47 +0000 2017</td>
      <td>0.881</td>
      <td>0.000</td>
      <td>@CNN</td>
      <td>0.119</td>
      <td>Alabama Senator-elect Doug Jones says it's tim...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>204</th>
      <td>0.2584</td>
      <td>Sun Dec 17 16:45:10 +0000 2017</td>
      <td>0.855</td>
      <td>0.000</td>
      <td>@CNN</td>
      <td>0.145</td>
      <td>UK Prime Minister Theresa May 'will not be der...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
cbs_vader_df = news_tweet_vader.loc[news_tweet_vader["News Source"] == "@CBSNews", :]
cbs_vader_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>News Source</th>
      <th>Positive</th>
      <th>Tweet Text</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>400</th>
      <td>0.2732</td>
      <td>Sun Dec 17 17:34:28 +0000 2017</td>
      <td>0.909</td>
      <td>0.000</td>
      <td>@CBSNews</td>
      <td>0.091</td>
      <td>Sen. John McCain is "doing well" and is "looki...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>401</th>
      <td>0.4019</td>
      <td>Sun Dec 17 17:20:01 +0000 2017</td>
      <td>0.863</td>
      <td>0.000</td>
      <td>@CBSNews</td>
      <td>0.137</td>
      <td>Sen. Bernie Sanders joined @FaceTheNation to d...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>402</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:04:41 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@CBSNews</td>
      <td>0.000</td>
      <td>White House @PressSec Sarah Huckabee Sanders c...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>403</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:00:02 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@CBSNews</td>
      <td>0.000</td>
      <td>Michael McDonald is back with "Wide Open" and ...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>404</th>
      <td>-0.3182</td>
      <td>Sun Dec 17 16:40:01 +0000 2017</td>
      <td>0.776</td>
      <td>0.142</td>
      <td>@CBSNews</td>
      <td>0.082</td>
      <td>Treasury Sec. Mnuchin says lobbyists "absolute...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
fox_vader_df = news_tweet_vader.loc[news_tweet_vader["News Source"] == "@FoxNews", :]
fox_vader_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>News Source</th>
      <th>Positive</th>
      <th>Tweet Text</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>600</th>
      <td>-0.6597</td>
      <td>Sun Dec 17 17:41:19 +0000 2017</td>
      <td>0.694</td>
      <td>0.306</td>
      <td>@FoxNews</td>
      <td>0.000</td>
      <td>'Batman' Actor Christian Bale: Trump's America...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>601</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:28:40 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@FoxNews</td>
      <td>0.000</td>
      <td>'Take Them Out In Cuffs': Pirro Doubles Down o...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>602</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:26:01 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@FoxNews</td>
      <td>0.000</td>
      <td>.@AmbJohnBolton: "I think the fact that Secret...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>603</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:22:14 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@FoxNews</td>
      <td>0.000</td>
      <td>Trump, Putin Spoke Today via Phone https://t.c...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>604</th>
      <td>0.3612</td>
      <td>Sun Dec 17 17:15:00 +0000 2017</td>
      <td>0.872</td>
      <td>0.000</td>
      <td>@FoxNews</td>
      <td>0.128</td>
      <td>Jones supports DACA, says it’s time for GOP op...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
nyt_vader_df = news_tweet_vader.loc[news_tweet_vader["News Source"] == "@nytimes", :]
nyt_vader_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>News Source</th>
      <th>Positive</th>
      <th>Tweet Text</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>800</th>
      <td>0.1280</td>
      <td>Sun Dec 17 17:45:05 +0000 2017</td>
      <td>0.741</td>
      <td>0.116</td>
      <td>@nytimes</td>
      <td>0.143</td>
      <td>President Trump's lawyer accused the special c...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>801</th>
      <td>-0.0258</td>
      <td>Sun Dec 17 17:30:06 +0000 2017</td>
      <td>0.558</td>
      <td>0.200</td>
      <td>@nytimes</td>
      <td>0.242</td>
      <td>“If we treat the dead like garbage, then the l...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>802</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:20:41 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@nytimes</td>
      <td>0.000</td>
      <td>RT @nytimesworld: Chile’s presidential electio...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>803</th>
      <td>0.7609</td>
      <td>Sun Dec 17 17:11:43 +0000 2017</td>
      <td>0.600</td>
      <td>0.102</td>
      <td>@nytimes</td>
      <td>0.298</td>
      <td>For women, the burden of caring for children c...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>804</th>
      <td>-0.8779</td>
      <td>Sun Dec 17 17:10:05 +0000 2017</td>
      <td>0.641</td>
      <td>0.359</td>
      <td>@nytimes</td>
      <td>0.000</td>
      <td>When this baby died, severe malnutrition was l...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
wsj_vader_df = news_tweet_vader.loc[news_tweet_vader["News Source"] == "@WSJ", :]
wsj_vader_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Compound</th>
      <th>Date</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>News Source</th>
      <th>Positive</th>
      <th>Tweet Text</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1000</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:40:02 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@WSJ</td>
      <td>0.000</td>
      <td>A small club in suburban Chicago bought a ski ...</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1001</th>
      <td>-0.2960</td>
      <td>Sun Dec 17 17:22:31 +0000 2017</td>
      <td>0.891</td>
      <td>0.109</td>
      <td>@WSJ</td>
      <td>0.000</td>
      <td>No federal law requires home sellers to disclo...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1002</th>
      <td>0.0000</td>
      <td>Sun Dec 17 17:00:02 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@WSJ</td>
      <td>0.000</td>
      <td>Dianne Feinstein says she sees another ‘year o...</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1003</th>
      <td>0.0000</td>
      <td>Sun Dec 17 16:45:09 +0000 2017</td>
      <td>1.000</td>
      <td>0.000</td>
      <td>@WSJ</td>
      <td>0.000</td>
      <td>Taxes look to be just one of many factors in A...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1004</th>
      <td>-0.3182</td>
      <td>Sun Dec 17 16:30:07 +0000 2017</td>
      <td>0.670</td>
      <td>0.196</td>
      <td>@WSJ</td>
      <td>0.134</td>
      <td>Banks offer to spice up some deals with levera...</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
# plot the polarity of news source tweets

fig, ax = plt.subplots(figsize=(15, 10))

ax.scatter(bbc_vader_df["Tweets Ago"], bbc_vader_df["Compound"], 
           color = "c", alpha=0.75, label="BBC North America") 

ax.scatter(cnn_vader_df["Tweets Ago"], cnn_vader_df["Compound"], 
           color = "b", alpha=0.75, label="CNN")

ax.scatter(cbs_vader_df["Tweets Ago"], cbs_vader_df["Compound"], 
           color = "gold", alpha=0.75, label="CBS News")

ax.scatter(fox_vader_df["Tweets Ago"], fox_vader_df["Compound"], 
           color = "r", alpha=0.75, label="Fox News")

ax.scatter(nyt_vader_df["Tweets Ago"], nyt_vader_df["Compound"], 
           color = "g", alpha=0.75, label="NY Times")

ax.scatter(wsj_vader_df["Tweets Ago"], wsj_vader_df["Compound"], 
           color = "k", alpha=0.75, label="Wall Street Journal")

plt.grid()
plt.xlim(200, 0)

plt.xlabel("Tweets Ago")
plt.ylabel("Tweet Polarity")

plt.title("Sentiment Analysis of News Source Tweets")
plt.legend(bbox_to_anchor=(1.01, 1), loc=2, borderaxespad=0.)

plt.savefig("news_individual_tweet_sentiment.png")
plt.show()
```


![png](output_16_0.png)



```python
# Create dataframe of mean sentiment data for each news source to be graphed
news_tweet_means = news_tweet_vader.groupby("News Source").mean()

# Remove problematic symbol from news source names
#news_tweet_means["News Source"] = ["BBCNorthAmerica", "CBSNews", "CNN", "FoxNews", "NYTimes", "WSJ"]
news_tweet_overall = news_tweet_means.reset_index()
news_tweet_overall
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>News Source</th>
      <th>Compound</th>
      <th>Negative</th>
      <th>Neutral</th>
      <th>Positive</th>
      <th>Tweets Ago</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>@BBCNorthAmerica</td>
      <td>-0.148150</td>
      <td>0.88405</td>
      <td>0.10320</td>
      <td>0.01280</td>
      <td>99.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>@CBSNews</td>
      <td>0.016045</td>
      <td>0.82525</td>
      <td>0.08985</td>
      <td>0.08490</td>
      <td>99.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>@CNN</td>
      <td>0.055955</td>
      <td>0.86330</td>
      <td>0.05795</td>
      <td>0.07870</td>
      <td>99.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>@FoxNews</td>
      <td>-0.030330</td>
      <td>0.85820</td>
      <td>0.07885</td>
      <td>0.06295</td>
      <td>99.5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>@WSJ</td>
      <td>-0.087390</td>
      <td>0.83530</td>
      <td>0.09765</td>
      <td>0.06705</td>
      <td>99.5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>@nytimes</td>
      <td>-0.034960</td>
      <td>0.83350</td>
      <td>0.09655</td>
      <td>0.06995</td>
      <td>99.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
# plot bar chart of overall sentiment for each news source

c = ["c", "gold", "b", "r", "k", "g"]
ec= ["k", "k", "k", "k", "k", "k"]
fig, ax = plt.subplots(figsize=(10, 7))

ax.bar(news_tweet_overall["News Source"], news_tweet_overall["Compound"], color=c, width=1, 
       edgecolor=ec, alpha=.75)

for i, j in enumerate(news_tweet_overall["Compound"]):
    ax.annotate((str(np.round(j, decimals=4))), xy=(i,-.007), ha="center", 
                color="k", fontweight="bold")

# set title and axis labels
plt.title("Overall News Source Sentiment Based on Tweets")
plt.xlabel("News Source")
plt.ylabel("Tweet Polarity")
plt.xlim(-0.5, len(news_tweet_overall)-0.5)
plt.grid()

plt.savefig("overall_news_sentiment.png")
plt.show()
```


![png](output_18_0.png)

