---
title: "Sentiment Analysis on Twitter"
date: 2023-04-10T10:26:48+02:00
draft: false
cover:
    image: img/twitter.jpg
    alt: 'Post one image'
    caption: 'This is the caption'
tags: ["API", "Twitter", "Sentiment Analysis"]
categories: ["Python"]
---

## Introduction 

Hey there, fellow data nerds! Today we're going to dive into the world of brand sentiment analysis using Twitter's API. Strap on your thinking caps and let's get started!

First things first, what is brand sentiment analysis? Simply put, it's the process of determining how people feel about a particular brand or product by analyzing their social media interactions. By examining tweets, comments, and other online chatter, we can get a sense of whether people are generally positive, negative, or neutral towards a brand.

So, why use Twitter's API for this? Well, Twitter is a treasure trove of unfiltered opinions and emotions. It's a place where people go to rant, rave, and share their experiences with the world. As such, it's a prime source of data for sentiment analysis.

Now, before we dive into the nitty-gritty of how to use Twitter's API for brand sentiment analysis, let's talk about some of the tools and techniques we'll be using. 

For this project, I'll analyse peoples optnions on a french company called Capgemini. I'll be using Python and accomplishing my task in 4 steps:

- Gather data: We'll use Twitter's API to collect tweets containing relevant keywords or hashtags related to our brand of interest.
- Clean and preprocess the data: We'll remove any irrelevant content (e.g. retweets, spam), tokenize the remaining text into individual words, and perform various other text preprocessing steps (e.g. removing stop words, stemming/lemmatization).
- Perform sentiment analysis: Building an NLP model is relatively complex, we will simply use the 'nltk' package which provides us with several NLP functions, including a function to invoke a pre-trained sentiment analysis model.
- Analyze and visualize the results: We'll use various data visualization tools (e.g. word clouds, bar charts) to explore the sentiment of our data and identify any patterns or trends.

Of course, this is a very high-level overview of the process, and there are countless details and nuances to consider along the way. But hopefully, this gives you a sense of what's involved.

## Preparing our work enviromment
Let's install and import everything we'll need, we'll talk about these packages as we move on. 

### Installing the packages 

```
!pip install tweepy
!pip install oauth2client
!pip install nltk
!pip install emoji
!pip install wordcloud
!pip install geopy
!pip install folium
```
### Importing

General packages
```
import pandas as pd
import numpy as np
import os
import itertools
from tqdm import tqdm
import datetime
```

Text analysis packages
```
from nltk.corpus import stopwords
from nltk.sentiment import SentimentIntensityAnalyzer
import nltk
import re
from collections import Counter
import emoji
```

Graphics packages
```
import matplotlib.pyplot as plt
from PIL import Image
from wordcloud import WordCloud, STOPWORDS
import folium
from folium.plugins import MarkerCluster, HeatMap
from geopy.geocoders import Nominatim
```

API wrappers Twitter
```
import tweepy
```
## Twitter's API

In order to accomplish this project, you will need to create an account on [Twitters Developers Platform](https://developer.twitter.com/en/docs/platform-overview).
I personnally find this API rather difficult to use. Even getting an account could be tricky for new users, so we may comeback to this on a future post.

### Creation your API's keys

You will need to start a project on this platform and get the following information: API_KEY, API_SECRET_KEY, BEARER_TOKEN

image: twitter_API.png

Replace those on the following code:

```
API_KEY = 'XXXXXXXXX'
API_SECRET_KEY = 'XXXXXXXX'
BEARER_TOKEN = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
```

### API Client Authentication and Creation

```
auth = tweepy.AppAuthHandler(API_KEY, API_SECRET_KEY)
api = tweepy.API(auth, wait_on_rate_limit=True)
```

This code is using the Tweepy library, which is a Python wrapper for the Twitter API, to authenticate and establish a connection to the Twitter API.

The first line creates an instance of the AppAuthHandler class provided by Tweepy. This class handles the OAuth2 authentication method for the Twitter API. The API_KEY and API_SECRET_KEY are the keys provided by Twitter to identify the application using the API.

The second line creates an instance of the API class provided by Tweepy, which allows you to interact with the Twitter API. The auth parameter is set to the AppAuthHandler instance that was created in the previous line. The wait_on_rate_limit parameter is set to True, which means that Tweepy will automatically wait if the API rate limit is reached, instead of throwing an error.

With these two lines of code, you should be able to make authenticated requests to the Twitter API using the api instance.

### Getting the data

First, you define a string variable query with the value "capgemini -filter:retweets". This is the search query that you want to use to search for tweets. notice that we only want to get retweets: this is a way of filtering tweets made by other accounts and not by Capgemini account itself.

```
query ="capgemini -filter:retweets"
```
Next, you create a Cursor object using the tweepy.Cursor method, passing in the following parameters:

- api.search_tweets: This is the search method of the Twitter API that you want to use to search for tweets.
- q=query: This is the search query that you defined earlier (in my case "capgemini").
- lang="en": This specifies that you only want to search for tweets that are written in English.
- tweet_mode="extended": This specifies that you want to retrieve the full text of each tweet (up to 280 characters).
- ).items(50): This specifies that you want to retrieve up to 50 tweets that match your search query.

```
tweets = tweepy.Cursor(api.search_tweets,
                       q=query,
                       lang="en",
                       tweet_mode="extended",
                       #since="2021-06-18", # put here the last week
                       #until="2021-06-25",
                      ).items(50)
tweets = list(tweets)

You could also specify dates with the parameter 'since' and 'until'.
```

After creating the Cursor object, you convert it to a list using the list() function and assign it to the variable tweets. This retrieves the tweets that match your search query and stores them in a list.

### Understanding our results

We have retrivied a list of 50 that match our search query. Let's take a look at what elements are contained in this json response of 1 single tweet:

```
test = list(tweets)[4]
json_test = test._json
json_test
```

This is my full response for 1 single tweet:

>{'created_at': 'Fri Apr 14 11:35:01 +0000 2023',
 'id': 1646839472493871104,
 'id_str': '1646839472493871104',
 'full_text': 'Through this initiative, we aim to empower the next generation of leaders &amp; help them get the future they want. Please join us in giving them a warm welcome &amp; wishing them good luck as they embark on a new journey with us at Capgemini. #GetTheFutureYouWant #LifeAtCapgemini https://t.co/SJVmxP6q4j',
 'truncated': False,
 'display_text_range': [0, 281],
 'entities': {'hashtags': [{'text': 'GetTheFutureYouWant',
    'indices': [244, 264]},
   {'text': 'LifeAtCapgemini', 'indices': [265, 281]}],
  'symbols': [],
  'user_mentions': [],
  'urls': [],
  'media': [{'id': 1646839435839799296,
    'id_str': '1646839435839799296',
    'indices': [282, 305],
    'media_url': 'http://pbs.twimg.com/media/Ftq_vddWAAAFSkS.jpg',
    'media_url_https': 'https://pbs.twimg.com/media/Ftq_vddWAAAFSkS.jpg',
    'url': 'https://t.co/SJVmxP6q4j',
    'display_url': 'pic.twitter.com/SJVmxP6q4j',
    'expanded_url': 'https://twitter.com/CapgeminiIndia/status/1646839472493871104/photo/1',
    'type': 'photo',
    'sizes': {'thumb': {'w': 150, 'h': 150, 'resize': 'crop'},
     'medium': {'w': 1024, 'h': 768, 'resize': 'fit'},
     'large': {'w': 1024, 'h': 768, 'resize': 'fit'},
     'small': {'w': 680, 'h': 510, 'resize': 'fit'}}}]},
 'extended_entities': {'media': [{'id': 1646839435839799296,
    'id_str': '1646839435839799296',
    'indices': [282, 305],
    'media_url': 'http://pbs.twimg.com/media/Ftq_vddWAAAFSkS.jpg',
    'media_url_https': 'https://pbs.twimg.com/media/Ftq_vddWAAAFSkS.jpg',
    'url': 'https://t.co/SJVmxP6q4j',
    'display_url': 'pic.twitter.com/SJVmxP6q4j',
    'expanded_url': 'https://twitter.com/CapgeminiIndia/status/1646839472493871104/photo/1',
    'type': 'photo',
    'sizes': {'thumb': {'w': 150, 'h': 150, 'resize': 'crop'},
     'medium': {'w': 1024, 'h': 768, 'resize': 'fit'},
     'large': {'w': 1024, 'h': 768, 'resize': 'fit'},
     'small': {'w': 680, 'h': 510, 'resize': 'fit'}}},
   {'id': 1646839436766765057,
    'id_str': '1646839436766765057',
    'indices': [282, 305],
    'media_url': 'http://pbs.twimg.com/media/Ftq_vg6WYAE0cyW.jpg',
    'media_url_https': 'https://pbs.twimg.com/media/Ftq_vg6WYAE0cyW.jpg',
    'url': 'https://t.co/SJVmxP6q4j',
    'display_url': 'pic.twitter.com/SJVmxP6q4j',
    'expanded_url': 'https://twitter.com/CapgeminiIndia/status/1646839472493871104/photo/1',
    'type': 'photo',
    'sizes': {'thumb': {'w': 150, 'h': 150, 'resize': 'crop'},
     'large': {'w': 1600, 'h': 1200, 'resize': 'fit'},
     'medium': {'w': 1200, 'h': 900, 'resize': 'fit'},
     'small': {'w': 680, 'h': 510, 'resize': 'fit'}}}]},
 'metadata': {'iso_language_code': 'en', 'result_type': 'recent'},
 'source': '<a href="https://mobile.twitter.com" rel="nofollow">Twitter Web App</a>',
 'in_reply_to_status_id': 1646839468219850752,
 'in_reply_to_status_id_str': '1646839468219850752',
 'in_reply_to_user_id': 44105494,
 'in_reply_to_user_id_str': '44105494',
 'in_reply_to_screen_name': 'CapgeminiIndia',
 'user': {'id': 44105494,
  'id_str': '44105494',
  'name': 'Capgemini India',
  'screen_name': 'CapgeminiIndia',
  'location': '13 major cities across India',
  'description': 'We explore the endless possibilities of tech-driven services enabled by the human touch. Follow @JoinCapgeminiIN for live hiring updates & #GetTheFutureYouWant!',
  'url': 'https://t.co/ykRGVlR5JI',
  'entities': {'url': {'urls': [{'url': 'https://t.co/ykRGVlR5JI',
      'expanded_url': 'https://www.capgemini.com/in-en/',
      'display_url': 'capgemini.com/in-en/',
      'indices': [0, 23]}]},
   'description': {'urls': []}},
  'protected': False,
  'followers_count': 170079,
  'friends_count': 955,
  'listed_count': 788,
  'created_at': 'Tue Jun 02 12:01:57 +0000 2009',
  'favourites_count': 10943,
  'utc_offset': None,
  'time_zone': None,
  'geo_enabled': True,
  'verified': True,
  'statuses_count': 42683,
  'lang': None,
  'contributors_enabled': False,
  'is_translator': False,
  'is_translation_enabled': False,
  'profile_background_color': 'C0DEED',
  'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png',
  'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png',
  'profile_background_tile': False,
  'profile_image_url': 'http://pbs.twimg.com/profile_images/1610246748349550592/gm0U6AqU_normal.jpg',
  'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1610246748349550592/gm0U6AqU_normal.jpg',
  'profile_banner_url': 'https://pbs.twimg.com/profile_banners/44105494/1671516631',
  'profile_link_color': '3B94D9',
  'profile_sidebar_border_color': 'C0DEED',
  'profile_sidebar_fill_color': 'DDEEF6',
  'profile_text_color': '333333',
  'profile_use_background_image': True,
  'has_extended_profile': False,
  'default_profile': False,
  'default_profile_image': False,
  'following': None,
  'follow_request_sent': None,
  'notifications': None,
  'translator_type': 'none',
  'withheld_in_countries': []},
 'geo': None,
 'coordinates': None,
 'place': None,
 'contributors': None,
 'is_quote_status': False,
 'retweet_count': 0,
 'favorite_count': 1,
 'favorited': False,
 'retweeted': False,
 'possibly_sensitive': False,
 'lang': 'en'}

As you can see, a tweet contains much for the it's text. Now we can extract the information we need for our analyses using the .attribute syntax (.full_text, .user.screen_name, etc.) to extract the text of the tweet, its creation date, hashtags, user's location, user's follower count, user's status count, tweet's retweet count, and tweet's favorite count. For exemple:

```
test.full_text
```

>'Through this initiative, we aim to empower the next generation of leaders &amp; help them get the future they want. Please join us in giving them a warm welcome &amp; wishing them good luck as they embark on a new journey with us at Capgemini. #GetTheFutureYouWant #LifeAtCapgemini https://t.co/SJVmxP6q4j'

### Extracting the diffrent elements of our tweets

The attributes we want to extract are the following: full_text, created_at, entities.hashtags, user.location, user.followers_count, user.statuses_count, retweet_count, favorite_count

```
tweets_infos = []
for tweet in tqdm(tweets):
    tweet_info = [tweet.full_text, 
                    tweet.created_at, 
                    [h['text'] for h in tweet.entities['hashtags']], 
                    tweet.user.location, 
                    tweet.user.followers_count, 
                    tweet.user.statuses_count, 
                    tweet.retweet_count, 
                    tweet.favorite_count] 
    tweets_infos.append(tweet_info)
```

The code provided extracts information from a list of tweets retrieved from the Twitter API. It iterates through each tweet in the list and extracts the full text of the tweet, creation date, hashtags, user location, follower count, status count, retweet count, and favorite count using various attributes of the tweet object. The extracted information is stored in a list of lists called tweets_infos.

Note that:
- tqdm is a Python library for creating progress bars, and it is used here to provide a visual indication of the progress of the loop.
- [h['text'] for h in tweet.entities['hashtags']]: This extracts the hashtags used in the tweet. The entities attribute of the tweet object contains metadata about the tweet, including hashtags. This line uses a list comprehension to extract the text of each hashtag in the tweet.

## Pre-processing

### Converting our results to a dataframe

Now let's create a dataframe and take a look at our results.

```
df = pd.DataFrame(tweets_infos)
df.columns = ['full_text', 'created_at', 'hashtags', 'location',
              'followers', 'statuses', 'retweets', 'favorites']
df
```

|    | full_text                                                                                                                                                                    | created_at                | hashtags                                                            | location                |   followers |   statuses |   retweets |   favorites |
|---:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------|:--------------------------------------------------------------------|:------------------------|------------:|-----------:|-----------:|------------:|
|  0 | @rahulphatangare Same thing happened with my relative but Its capgemini                                                                                                      | 2023-04-14 12:51:44+00:00 | []                                                                  |                         |         629 |      48890 |          0 |           0 |
|  1 | 🔔 New Job Alert! Check Out 👉Capgemini Invent - Graduate Business Technology Accelerate Programme. Apply Now! 🎓                                                            | 2023-04-14 12:01:34+00:00 | ['graduate', 'jobs', 'graduatejobs', 'capgemini', 'grb']            | Brighton and London, UK |       18763 |      16081 |          0 |           0 |

### Downloading stopwords from nltk

Stopwords are common words such as "the", "and", "is", etc., that are often removed from text data during text processing tasks like text analysis or natural language processing (NLP) to filter out noise and reduce computational overhead. We are goint to download a list of stopwords using the NLTK (Natural Language Toolkit) package in Python. 

```
nltk.download('stopwords')
stop_words = set(stopwords.words('english'))
```

### Defining a cleaning fonction

```
def clean_tweet(txt):
    return re.sub("(http\S+)|(@\S+)", "", txt).replace('#', '')
```
The clean_tweet() function will be called to clean the tweet text by removing any unnecessary characters or symbols, and the resulting cleaned text is converted to lowercase using the .lower() function.

### Cleaning the data

First, the tweets_text variable is created to store the text of the tweets by extracting the first element (i.e., the full text) from the tweets_infos list. Next, a corpus list is initialized to store the processed text data after pre-processing steps.

```
tweets_text = [t[0] for t in tweets_infos]
corpus = []
for tweet in tweets_text:
    clean = clean_tweet(tweet).lower()
    clean = re.sub(r'[^\w\s\U00010000-\U0010ffff]', ' ', clean)
    clean = clean.split()
    collection_words = ['capgemini']
    clean = [word for word in clean if not word in collection_words]
    clean = [word for word in clean if not word in stop_words]
    corpus.extend(clean)
```

- The re.sub() function is used to substitute any characters or symbols that are not alphanumeric or whitespace with a space character. This step helps in removing any special characters or emojis from the text data.
- clean = clean.split(): The tweet text is then split into a list of words using the .split() function, which separates words based on whitespace.
- collection_words = ['capgemini']: A list of collection words, which are words that are known to be irrelevant or redundant in the context of the analysis, is created. 

A list comprehension is used to filter out the collection words from the list of words in the tweet text. Then another list comprehension is used to remove the stop words.

## Analysis

So what can we say about these tweets?

### Single words analysis

Let's count how many single words we have:

```
print(f"There are {len(set(corpus))} words in the combination of all searched tweets.").
```

There are 533 words in the combination of all searched tweets.

Which ones are the most frequent?

```
top10_words = Counter(corpus).most_common(10)
top10_words
```

>[('apply', 8),
 ('2023', 7),
 ('us', 7),
 ('getthefutureyouwant', 7),
 ('new', 6),
 ('job', 6),
 ('join', 6),
 ('journey', 5),
 ('data', 5),
 ('next', 4)]

We can also show this single word analysis in a bar plot:

```
def plot_frequencies(words, top=15):
    
    # Create dataframe for plotting
    frequency_df = pd.DataFrame(Counter(words).most_common(top),
                             columns=['words', 'count'])
    
    # Plot
    frequency_df.sort_values(by='count').plot.barh(x='words',
                                                   y='count',
                                                   color="red")
    plt.title("Most Common Words in Tweets", fontweight='semibold')
    plt.show()

plot_frequencies(corpus, top=10)
```

![Plot Frequencies](static/img/plot_frequencies.png)

### Wordcloud

```
def wordcloud(corpus, title=None, mask=None, figsize=(10, 10)):
     
    # preprocess data
    corpus = str(corpus).replace("'", "")
    
    wordcloud = WordCloud(
        background_color = 'white',
        max_font_size = 45,
        min_font_size = 5,
        contour_width = 0.1,
        contour_color = 'silver',
        repeat = True,
        stopwords=STOPWORDS,
        random_state = 1).generate(str(corpus))
    
    fig = plt.figure(1, figsize=figsize)
    plt.axis('off')
    if title:
        fig.suptitle(title, fontsize=18, y=0.75)
    plt.imshow(wordcloud)

wordcloud(corpus, title="Most Common Words in Tweets")
```

![Wordcloud](static/img/wordcloud.png)

## Conclusion
So, what are some potential applications of brand sentiment analysis using Twitter's API? Here are a few:

Market research: By understanding how people feel about a particular brand or product, companies can make more informed decisions about marketing strategies, product development, and customer service.

Crisis management: If a company experiences a PR crisis (e.g. a product recall, a viral negative review), sentiment analysis can help them gauge the public's response and take appropriate action.

Competitive analysis: By comparing the sentiment of their brand to that of their competitors, companies can gain insights into their relative strengths and weaknesses in the marketplace.

Overall, brand sentiment analysis using Twitter's API is a powerful tool for gaining insights into the public's perceptions of a particular brand or product. With the right tools, techniques, and a healthy dose of data science know-how, you can unlock valuable insights that can inform business decisions and drive growth. So get out there and start analyzing those tweets!

[def]: TechTales/static/img/plot_frequencies.png