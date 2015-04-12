---
layout: post
title: A comparison of NLP open source tools for sentiment analysis
date: '2015-01-20T17:40:00.000+02:00'
author: kabamaru
tags:
- nlp
- python
- java
- open source
---

# Goal

The objective of this project is to apply various NLP sentiment analysis techniques on reviews of the [Yelp Dataset](http://www.yelp.com/dataset_challenge) and assess their effectivenes on correctly identifiyng them as positive or negative.

# Yelp Dataset Challenge

Yelp has released an anonymized part of their stored data to the public. This was accompanied by [a challenge](http://www.yelp.com/dataset_challenge) with various awards in order to incentivize research and generate insights for the use of the data. 

Here follows a brief explanation of the dataset, from their website.

```
The Challenge Dataset includes data from Phoenix, Las Vegas, Madison, Waterloo and Edinburgh:

* 42,153 businesses
* 320,002 business attributes
* 31,617 check-in sets
* 252,898 users
* 955,999 edge social graph
* 403,210 tips
* 1,125,458 reviews
```

This project is not a participation in the challenge. 

# Review extraction

The reviews in the Yelp dataset are stored in a single json file. To extract the reviews a parser has been implemented. It takes as input a business category and a number of samples for each category (positive/negative). 

First I extracted the anonymized ids for all the businesses that belong to the requested category. These ids are found in a different file than the reviews. Then I parsed the reviews and extract the requested amount of positive and negative reviews. A review is considered positive when the rating is of 4 or 5 stars and negative when it has 1 or 2 start. *Ratings of 3 stars are not taken into account.* The reviews are saved for future processing to two distinct files, one for each class.

# Classification Techniques

Three different techniques will be assesed and compared:

* Training a classifier using samples from the dataset 
* Classify using generic lexicons
* Pretrained state-of-the-art system

Only a subset of the dataset will be used.

## Train a classifier using the dataset

The training on the dataset is done using the *bag of words* model and a NaiveBayes classifier. The evaluation of the classifier is done by stratified k-fold cross validation. Also the samples are randomly shuffled for each run.

Three different feature extraction algorithms where implemented: 

* Single words
* Single words after the removal of stopwords
* Bigrams (two words appearing together)
* Bigrams after the removal of stopwords

For the bigrams extraction all the bigrams are computed and the *n* best are selected according to an association measure. By default *n = 500* and the metric is based on the [Chi-square test](http://en.wikipedia.org/wiki/Chi-square_test).

## Classify using generic lexicons

Sentiment analysis can be done through the lexical analysis of text. **Pros:** The advantage of this technique is that it doesn't require any training and can be applied to any text without any *a priori* knowledge of the domain. **Cons:** On the other hand this technique utilizes heavier sentence manipulation and requires pre-constructed lexicons.

The main steps of this techique are shown in the following diagram. This procedure was formulated in ["Reviews Classification Using SentiWordNet Lexicon"](http://www.academia.edu/1336655/Reviews_Classification_Using_SentiWordNet_Lexicon) by A. Hamouda and M. Rohaim.

![flow of sentiwordnet](/assets/images/nlp/sentiwordnet-flowchart.png){: style="width: 100%"}

The sentence is tokenized and each token is assigned a part-of-speech tag. The POS tag can be adjective, noun, verb etc. A comprehensive list can be found [here](https://www.ling.upenn.edu/courses/Fall_2003/ling001/penn_treebank_pos.html). 

Since a word can have muliple senses(or meanings), a word sense disambiguation(WSD) process is needed. A technique for doing this was adapted from a [twitter sentiment analysis tool](www.slideshare.net/faigg/tutotial-of-sentiment-analysis). It makes use of WordNet which provides various senses for a word and each sense's definition. For example for the word *good* as a noun WordNet contains the following senses and their definitions:

```
  Sense        Description
 ('good.n.01', 'benefit'),
 ('good.n.02', 'moral excellence or admirableness'),
 ('good.n.03', 'that which is pleasing or valuable or useful'),
 ('commodity.n.01', 'articles of commerce')
```

For each word it checks all the meanings and the selects the one whose definition matches closer the context of the word. That matching is done by simply comparing the frequency of common words between the context and the definition. 

In the original implementation the context taken into account for that was the whole text. This would be ok in the case of a small text, e.g. a twitter. Indeed the original author was trying to analyze tweets. But a whole review can be too broad of a context. We tried to improved the algorithm by evaluating each sentence of the review seperately and aggregating over the scores. This provides the WSD algorithm with a compact, more relevat context to work with.

After the meaning has been identified then SentiWordNet is used to derive the final negative or positive score of the word. The score for the text is the summation over the scores of all of it's words. 

Next you can see an example of the objective, positive and negative score for each sense of the word *good* as a noun.

```
 Sense				Obj		Pos		Neg
('good.n.01', 		0.5,    0.5, 	0.0),
('good.n.02', 		0.125,	0.875,  0.0),
('good.n.03', 		0.375,	0.625,  0.0),
('commodity.n.01', 	1.0,	0.0, 	0.0)
```

Another WSD technique was adapted from another open source [sentiment classifier](http://pythonhosted.org/sentiment_classifier) for twitter. This one was very slow and wasn't considered at all in the results.

## CoreNLP: A pretrained state-of-the-art system

Stanford's [CoreNLP](http://nlp.stanford.edu/sentiment/code.html) integrates many tools for doing NLP in a cohesive library. It is written in Java and provides a part-of-speech (POS) tagger, a named entity recognizer (NER), a parser, a coreference resolution system, sentiment analysis, and bootstrapped pattern learning tools.

It's [sentiment analysis](http://nlp.stanford.edu/sentiment/code.html) tool is using deep learning techniques and is trained on 215k phrases extracted from 12k sentences (Penn Treebank format). 

For our purposes the provided parser is used to extract the sentences of the text and then the sentiment analysis tool scores them one by one. The score provided is in the range of 0 (very negative) to 4 (very positive) with 2 being neutral. 

The estimation of the whole review's sentiment score is done by aggregating the score of it's individual sentences. I've tried three different techniques on this one. The first one is averaging the score of the sentences, the second one is a weighted average where the weight is the length of the sentence and the third being a count of how many sentences have been positive/negative. 


# Results

All the results regard the reviews of restaurants. The reviews are extracted in series from the dataset. That means that the experiment with 2000 samples contains the first 1000 positive and the first 1000 negative reviews and so on.

## Naive Bayes

We ran the naive bayes on various sample quantities and for the four differenct feature extraction algorithms. The results of the accuracy can be found in the following table and graph.

| Samples  | Single Words  | Stopwords Removal  | Bigrams | Bigrams & Stopwords |
| -------------: |:-------------:| :-----:| :--: | :-:|
|1000|	0.703|	0.774|	0.740| 0.804
|2000|	0.707|	0.786|	0.757| 0.815
|4000	|0.661|	0.733|	0.757| 0.796
|10000	|0.740|	0.816	|0.811| 0.861
|20000	|0.780|	0.850|	0.852| 0.894

![naive bayes accuracy chart](/assets/images/nlp/naive-bayes-accuracy.png){: style="width: 100%"}

We can see that using stopwords improves the results in the whole range of number of samples. Also using stopword and bigrams provide similar results when the sample number is high. The best results are achieved by the combination of stopwords removal and bigrams.

From the graph we note that there is a small decline for 4000 samples. This could be attributed to overfitting in low sample numbers. After that the classifier improves and stabilizes as more data is provided.

We have also timed the run for each feature extraction algorithm.
As seen in the following table each run of the single words and the stopword removal take more or less the same time. Bigram extraction is much more time consuming resulting about 7 times slower. All the times refer to the training and testing of all the five datasets.

| Feature  | Time  | 
| :------------- |:-------------|
|Single Words|	1m 6s|
|Stopwords Removal|	1m 3s|	
|Bigrams	| 7m 51s|	
|Bigrams & Stopwords| 6m 13s|

It's also interesting to see which where the most informative features in each case. These are the feature with the most discrimitative power.

**Single Words**

```
Most Informative Features
              flavorless = True              neg : pos    =     65.0 : 1.0
              disgusting = True              neg : pos    =     49.0 : 1.0
               tasteless = True              neg : pos    =     35.6 : 1.0
               microwave = True              neg : pos    =     31.0 : 1.0
                  filthy = True              neg : pos    =     26.3 : 1.0
                     YUM = True              pos : neg    =     25.0 : 1.0
                    zero = True              neg : pos    =     25.0 : 1.0
                   Gross = True              neg : pos    =     24.3 : 1.0
                inedible = True              neg : pos    =     23.8 : 1.0
                   worst = True              neg : pos    =     22.6 : 1.0
```
                  
**Stopwords Removal**

```
Most Informative Features
                   awful = True              neg : pos    =     42.3 : 1.0
               tasteless = True              neg : pos    =     39.3 : 1.0
                   Worst = True              neg : pos    =     36.3 : 1.0
                response = True              neg : pos    =     31.0 : 1.0
                     Meh = True              neg : pos    =     31.0 : 1.0
              flavorless = True              neg : pos    =     28.7 : 1.0
                   worst = True              neg : pos    =     27.3 : 1.0
                 refused = True              neg : pos    =     24.3 : 1.0
               redeeming = True              neg : pos    =     22.3 : 1.0
                   lousy = True              neg : pos    =     22.3 : 1.0

```

**Bigrams**

```
Most Informative Features
       (u'rude', u'and') = True              neg : pos    =     50.3 : 1.0
     (u'terrible', u'.') = True              neg : pos    =     40.2 : 1.0
                inedible = True              neg : pos    =     39.0 : 1.0
      (u'was', u'bland') = True              neg : pos    =     37.4 : 1.0
      (u'the', u'worst') = True              neg : pos    =     37.3 : 1.0
                downhill = True              neg : pos    =     35.0 : 1.0
                   awful = True              neg : pos    =     34.5 : 1.0
     (u'well', u'worth') = True              pos : neg    =     33.7 : 1.0
        (u'was', u'dry') = True              neg : pos    =     33.4 : 1.0
         (u'rude', u'.') = True              neg : pos    =     33.0 : 1.0
```

**Bigrams & Stopwords**

```
               tasteless = True              neg : pos    =     48.6 : 1.0
              disgusting = True              neg : pos    =     47.8 : 1.0
     (u"won't", u'back') = True              neg : pos    =     37.4 : 1.0
                   awful = True              neg : pos    =     35.7 : 1.0
              flavorless = True              neg : pos    =     35.4 : 1.0
(u'perfectly', u'cooked') = True              pos : neg    =     33.0 : 1.0
          (u'.', u'Yum') = True              pos : neg    =     32.3 : 1.0
                     YUM = True              pos : neg    =     31.7 : 1.0
     (u'inedible', u'.') = True              neg : pos    =     30.3 : 1.0
                inedible = True              neg : pos    =     28.6 : 1.0
```


It's evident that most of the features characterize negative sentiments. 

## WordNet & SentiWordNet

The results for the SentiWordNet approach can be found in the following table.

| Samples  | WSD on review | Time | WSD on sentence  | Time |
| :------------- |:-------------| :-- | :--| :--|
|1000|	0.621| 4m 42| 0.644 | 4m 59s| 
|2000|	0.605|	 11m 34s| 0.638| 9m 44s|
|4000| 0.602|	 59m 16s | 0.638| 54m 9s|

Comparing the two WSD approaches it becomes apparent that using the sentence as the context the results are better and more robust. This confirms the original hypothesis that the sentence provides more relevant information for word sense disambiguation than the whole review.

## Stanford's CoreNLP

The samples (positive and negative) are read from the same files that were created by ```python```. Since they are formated in json there is no need to recreate them. 

The accuracy is compared for the three methods of score aggregation mentioned before (average, weighted, counts). Since this was implemented a while after the first two methods I also considered computing the accuracy of each class (pos/neg) and not only the average one. 

| Samples  | Average (pos/neg/avg) | Weighted (pos/neg/avg) | Counts (pos/neg/avg)  | Time |
| :------------- |:-------------| :-- | :--| :--|
|1000|	0.544 / 0.884 / 0.714 | 0.552 / 0.930 / 0.741 | 0.520 / 0.886 / 0.703 | 41m 3s | 
|2000|	0.545 / 0.899 / 0.722 | 0.532 / 0.939 / 0.736  | 0.521 / 0.898 / 0.710 | 1h 33m|
|4000| 0.565 / 0.910 / 0.737| 0.543 / 0.943 / 0.743 | 0.538 / 0.911 / 0.724 | 2h 52m|

From the results it becomes evident that CoreNLP gives better results for the negative class rather than the positive. Whether this is a bias needs further investigation. 

Comparing the three aggregation methods we see that the weighted averages gives the best overall results and also the best results for the negative class. The simple average gives better results for the positive class. 

# Comparison

![comparison](/assets/images/nlp/comparison-0.5.png){: style="width: 100%"}

In these experiments it seems that the the Naive Bayes classifier gave the best overall results. This is expected since it was trained with samples from the the Yelp dataset. The SentiWordNet results where somewhat dissapointing, barely better than random guessing. CoreNLP gave rather good results, given that it came pretrained, but it's accuracy for the positive class was marginally better than random guessing.

All the methods were timed. Naive Bayes was much faster (even including training time), SentiWordNet was slower by an order of magnitude and CoreNLP was even slower by another order of magnitude.