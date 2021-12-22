<!-- omit in toc -->
# Topic Modeling Approach to Improve Twitter’s Search Algorithm 

<!-- omit in toc -->
# Contents
- [Motivation](#motivation)
- [Problem Statement](#problem-statement)
- [Datasets](#datasets)
- [Modeling Methodology](#modeling-methodology)
- [Results](#results)
- [Conclusions](#conclusions)
- [Directory Tree](#directory-tree)
- [References](#references)

# Motivation 

Twitter is one of the most popular social media platforms, and it currently has over [396.5 million](https://backlinko.com/twitter-users) users. Twitter is a microblogging and social networking service. Users post *tweets*, short text messages to interact with their followers. The text content of a tweet can contain up to 280 characters. Users can post and talk about any topic they wish to, provided they do not violate the Twitter rules. Given the open platform nature of the service, Twitter attracts a wide range and variety of users and topics, and users of the platform use it as an outlet to discuss and inform their viewpoints. Twitter is a valuable text mining data pool that can be leveraged to discover the underlying themes of conversations happening at any given time. E.g., an organization would like to understand the discussions around their products. However, manually reading and making sense of all the tweets to identify the themes and topics of discussions is not always practical. A text mining approach can be adopted to get a *big picture* idea of general discussion topics. Topic modeling can help distill information into a more usable and actionable format.<br>
I analyze the Twitter activity of users from **India** over the last two years (Sep-2019 to Sep-2020). *Topic models* are used to discover and identify trends in the underlying themes in the topics being discussed on the platform. COVID played a big part in the last two years and greatly influenced how users interacted on social media platforms. Topic models can provide insights into any specific trends that could correlate with the rise and the fall in the number of COVID cases in India. Since topic modeling is an unsupervised machine learning technique, our analysis and results are driven by how well the topic models capture the relevant topics. 

[Back to Top](#Contents)

# Problem Statement

As a data science engineer at Twitter-India, I am tasked to improve Twitter's search algorithm to show more context-based results rather than just query-based results. I use topic models to discover latent themes in the users' posts specifically originating from India to improve the search engine's performance. Helping users find relevant information will lead to more engagement on the platform and drive user monetization.

[Back to Top](#Contents)

# Datasets

I use **Snscrape API** to collect data which in this case are tweets posted by users in India. Snscrape is a scraper for social networking services (SNS). It has support for several social media services such as Twitter, Facebook, Instagram, etc. The full list of its supported services and its associated functionalities can be found on its [Github](https://github.com/JustAnotherArchivist/snscrape) page.

**Tweet Scrapper**
Snscrape has a Python wrapper for Twitter with support for users, user profiles, hashtags, searches, threads, and list posts. It has 2 distinct advantages over Tweepy:
- You do not need an API key to scrape tweets.
- Snscrape's search function works the same way as Twitter's search.
  
 I used the `near` `Geo` operator to filter out tweets by location around the top 25 cities by population in India. A typical search query to scrape tweets around a given city will look something like this:
`f'near:{city} within:200km since:{since} until:{until} lang:en -filter:retweets -filter:replies filter:verified')`. The `city`, `since` and `date` parameters represent a given city name, start date and end date of search query respectively. I applied additional filters to the search results to limit the number of search results.

I ended up collecting more than million tweets for the two year time period. However, after accounting for duplicates there were approximately 650k tweets in the dataset. These duplicates were due to overlapping search radius around the cities that was used in the search query.


[Back to Top](#Contents)


# Modeling Methodology

**LDA Models**

Gensim is designed to process raw, unstructured digital texts (”plain text”) using unsupervised machine learning algorithms. In this project, I use the LDA algorithm of the gensim library. The primary steps in building a LDA model using Gensim are:
- Building a dictionary object which is assigning a unique id to each token. To do so, convert the texts to a list of tokens and pass it to Dictionary object.
- Building a document term matrix which is the gensim corpus object. It contains the word id and its frequency in each document.
- Building a model by passing on the corpus, dictionary, and the number of topics. There are other hyperparameters which can be used to fine tune the model results. 
- LDAMulticore is the parallelized implementation of the LDA model.
- Investigate the topics and words associated with the topics
- Calculate model metrics
- Visualize the topics using pyLDAvis which is a python library for interactive topic model visualization.

A three step modeling approach for the LDA model is adopted

1. Base LDA model for the entire corpus
2. Tuning the Hyperparameters to improve over the base model
3.  Build a model for each month by limiting the corpus for that particular month by tuning their hyperparameters

The first two steps provide models that reveal topics of interest over the entire two year period. However, since topics evolve over time a monthly model should be better able to capture relevant topics for that given month instead of generalizing over a longer timeframe. 

**Short Text Topic Models-GSDMM**

LDA models do not work very well with short texts such as tweets. LDA models identifies multiple topics in a given document. However, short texts such as tweets are usually focussed on a single topic. GSDMM models works on the premise that one topic in one document.
To evaluate the performance of GSDMM, a single model was fitted to our corpus. This model takes 6 hours to run for 50 iterations on Google Colab. Hence, even though this model reveals much more coherent topics than the LDA model, it could not be more extensively used as the LDA model due to its slow run time.

**Evaluation of Models**

*Coherence* - It measures how the texts are semantically meaningful. It is the implementation of the four stage topic coherence pipeline. There are 4 different coherence models in `gensim`, `u_mass`, `c_v`, `c_uci`, `c_npmi`. I have used the `c_v` measure which is based on a sliding window, a one-set segmentation of the top words and an indirect confirmation measure that uses normalized pointwise mutual information (NPMI) and the cosinus similarity. Higher `c_v` is better and its value is between 0 and 1.

[Back to Top](#Contents)

# Results

**LDA Models**
- The coherence score is for base LDA model with 25 topics 0.45
- The best LDA model as ranked by their coherence scores after tuning the hyperparameters has a number of topics of 15 and a decay of 0.5
- The distribution of word counts for all documents show that the document length is too short due to the removal of most occuring tokens
- The top 2 topics contribute almost 50% of documents
- The topics that can be identified by creating one model for every month are as follows:

| Mon-Year   | Topic Label                                                        |
|------------|--------------------------------------------------------------------|
| Sep - 2019 |  very small dataset so not a coherent topic                        |
| Oct - 2019 |  Diwali wishes                                                     |
| Nov - 2019 |  Thanking for Birthday wishes                                      |
| Dec - 2019 |  Student protests                                                  |
| Jan - 2020 |  Film promotion                                                    |
| Feb - 2020 |  Film promotion                                                    |
| Mar - 2020 |  Stay home covid messages                                          |
| Apr - 2020 |  Lockdown                                                          |
| May - 2020 |  Lockdown and Migrant worker crisis                                |
| Jun - 2020 |  China-India border skirmishes                                     |
| Jul - 2020 |  Family+Student, possibly related to school shutdowns due to covid |
| Aug - 2020 |  Family+Student, possibly related to school shutdowns due to covid |
| Sep - 2020 |  Thank you tweets                                                  |
| Oct - 2020 |  Thank you tweets                                                  |
| Nov - 2020 |  Diwali wishes                                                     |
| Dec - 2020 |  Farmers protest                                                   |
| Jan - 2021 |  Farmers protest                                                   |
| Feb - 2021 |  Birthday wishes to guru                                           |
| Mar - 2021 |  Woman+Thank you messages                                          |
| Apr - 2021 |  Covid 2nd wave, Oxygen and Hospital help                          |
| May - 2021 |  Covid 2nd wave, Oxygen and Hospital help                          |
| June -2021 |  End of 2nd Covid wave, Happy messages                             |
| Jul - 2021 |  Birthday wishes                                                   |
| Aug - 2021 |  India England Cricket Series                                      |

**Short Text Topic Modeling**

The input number of clusters was 50 and the model ended up assigning the corpus to all the 50 clusters with 30 iterations. But the documents assigned to ~15 clusters is very low. Probably with more iterations the model would have assigned fewer than 50 clusters to our corpus. The Cluster-28 is most dominant cluster with almost 35,000 documents from the corpus.

| Cluster #   | Topic Label                                             |
|-------------|---------------------------------------------------------|
| Cluster 28  |  Wishing the best                                       |
| Cluster 1   |  Social Media and Fake News                             |
| Cluster 24  |  Birthday wishes and congratulations                    |
| Cluster 6   |  Invitations to join live stream                        |
| Cluster 26  |  Religion based Politics discussion                     |
| Cluster 0   |  Cricket and sports                                     |
| Cluster 49  |  Covid messages specifically wishing well               |
| Cluster 32  |  Woman, education and students                          |
| Cluster 15  |  Covid affecting students due to lockdown               |
| Cluster 39  |  Bengal Elections and two national parties              |
| Cluster 30  |  Stocks and companies                                   |
| Cluster 37  |  Film promotions                                        |
| Cluster 44  |  Audience review                                        |
| Cluster 11  |  Covid lockdown and case fatalities                     |
| Cluster 17  |  Ruling party                                           |
| Cluster 2   |  Visiting places                                        |
| Cluster 4   |  Law enforcement                                        |
| Cluster 8   |  Public protests                                        |
| Cluster 9   |  Covid deaths and condolences                           |
| Cluster 27  |  Anniversary and possibly about Indian independence day |

# Conclusions

- Snscrape API uses the public Twitter search results which severely limits the number and the quality of tweets that it can scrape. Most of the scraped tweets had overlapping themes.
- GSDMM model assigned more relevant topics as judged from a human interpretation point of view. 
- GSDMM model's performance could be improved further by tuning the `alpha` and `beta` hyperparameters.
- LDA models identified distinct topics with overlapping themes
- LDA Multicore implementation makes running and fine tuning the hyperparameters a much faster process and is a definite advantage over the GSDMM


[Back to Top](#Contents)

# Directory Tree

| **Directory** | **Contents**                                                                                                              |
|---------------|---------------------------------------------------------------------------------------------------------------------------|
| code          | jupyter-notebooks                                                                                                         |
| models        | trained models                                                                                                            |
| output        | csv files for cleaned tweets dataset and model evaluation<br> https://1drv.ms/u/s!Ar52d2HxEkbnioYmVXXLJcWwBEXvUA?e=jPj5lX |
| presentations | presentation                                                                                                              |
| tweets_data   | raw tweets data for each city and India<br> https://1drv.ms/u/s!Ar52d2HxEkbnkK0-WKfS6ZgDqi7gyA?e=OhInfQ                   |

[Back to Top](#Contents)

# References
[Advanced Search Twitter Cheatsheet](https://github.com/igorbrigadir/twitter-advanced-search)<br>
[Topic Modeling Visualization](https://www.machinelearningplus.com/nlp/topic-modeling-visualization-how-to-present-results-lda-models/)<br>
[GSDMM Modeling](https://towardsdatascience.com/short-text-topic-modelling-lda-vs-gsdmm-20f1db742e14)<br>
[LDA Model Evaluation](https://towardsdatascience.com/evaluate-topic-model-in-python-latent-dirichlet-allocation-lda-7d57484bb5d0)<br>
[Parallelizing Spacy Pipeline](https://prrao87.github.io/blog/spacy/nlp/performance/2020/05/02/spacy-multiprocess.html)<br>

[Back to Top](#Contents)