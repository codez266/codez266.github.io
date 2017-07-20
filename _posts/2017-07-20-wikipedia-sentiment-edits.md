---
layout: post
title: Wikipedia - Sentiment in Wikipedia Drafts and Edits
permalink: wikipedia-sentiment-edits
tags:
- research
- Wikimedia
---

My exciting work with WMF is continuing and this is a brief account of that
as I got some time to breathe between my other commitments. This post is about a
sentiment feature I implemented using SentiWordnet on Wikipedia drafts/edits for the
purpose of classification of an edit into damaging or not. The reason for using
drafts and edits together is because this started with [draftquality research](https://meta.wikimedia.org/wiki/Research:Automated_classification_of_draft_quality)
but is now showing promises for [editquality](https://meta.wikimedia.org/wiki/Research:Automated_classification_of_edit_quality).

## Background

Wikipedia has a webservice called [ORES](https://www.mediawiki.org/wiki/ORES)
that reviews edits made on Wikipedia and gives a score to them on based on damaging,
goodfaith and reverted models at the backend. These models are curated using
Machine Learning techniques on trained data acquired by labeling of edits by
Wikipedia editors. Each Wikipedia language has its on models based on their wiki
and language.

The [revscoring](https://github.com/wiki-ai/revscoring) library is the one doing the
heavy lifting when it comes to building the models. For those new to the
it, it is a very convenient piece of software written to ease development
of Machine Learning models around Wikipedia by making featching of revisions and
their scoring super easy through command line arguments. It also provides a
framework to define new features to extract from edtis and already has an
extensive list of features that it extracts from articles.

## Sentiment feature on drafts and edits

An edit or a draft on Wikipedia, according to the damaging criteria is one which harms the
content to which it is added or the encyclopedia in general. I came up with a
hypothesis that if the overall sentiment of the edit or an article is positive
or negative it could play a role in determining the damaging nature of an edit.

Edits are reverted while new drafts are deleted based on [Speedy deletion
criteria](https://en.wikipedia.org/wiki/Wikipedia:Criteria_for_speedy_deletion)
Hence, starting from [feature engineering](https://github.com/wiki-ai/revscoring/blob/master/ipython/feature_engineering.ipynb) on the revscoring library
I set out to implement a feature to provide an overall sentiment of the edit. My
research is recorded at the [phab task](https://phabricator.wikimedia.org/T167305) but here are the few interesting bits and pieces:
The terminology of spam, vandalism and attack is explained in a [previous
post]({ %post_url 2017-05-20-article-draft-quality }).

### Advertising is SPAM!

A quick look at the below image will clearly show that the spam class created by
fake editors to promote stuff clearly shows a spike in positive sentiment:

<img src="/images/polarity_scores_draft.png" width="700px" title="Polarity scores on drafts">

### From where do these come ?

These polarity scores are generated using SentiWordnet. In reality there are
tons of resources out there to get polarity of sentences or documents but here I
had to adhere to certain constraints:
* The code should be open source
* The scoring should be fast - typically in milliseconds otherwise ORES would
  timeout
* It should be easily portable or usable in python.

[Sentiwordnet](http://sentiwordnet.isti.cnr.it/) seemed to be the best fit. Its just a database of common words and
their corresponding polarities per the word sense from wordnet. 

### Attempt 1

I did a simple task of aggregating the positive and negative score of each word
after doing its WSD using simple-lesk. To my despair, the scoring was taking
seconds per draft! strict **NO**

### Attempt 2

I had to somehow prune the scoring time at the same time retaining the viability of my
hypothesis. I decided to sacrifice accuracy for time. 
I was earlier using a library from kevincobain called sentiment_classifier that
does WSD and then uses the sense to query the polarity. A profiling of the
method revealed:

```
 ncalls  tottime  percall  cumtime  percall filename:lineno(function)
 1125/1    0.076    0.000  171.370  171.370 {built-in method builtins.exec}
      1    0.000    0.000  171.370  171.370 emot.py:1(<module>)
      1    0.000    0.000  168.748  168.748 emot.py:19(get_polarity)
      2    0.000    0.000  168.740   84.370 /home/sumit/venv/lib/python3.6/site-packages/sentiment_classifier-0.7-py3.6.egg/senti_classifier/senti_classifier.py:234(polarity_scores)
      2    0.006    0.003  168.740   84.370 /home/sumit/venv/lib/python3.6/site-packages/sentiment_classifier-0.7-py3.6.egg/senti_classifier/senti_classifier.py:202(classify)
   1084    1.248    0.001  168.734    0.156 /home/sumit/venv/lib/python3.6/site-packages/sentiment_classifier-0.7-py3.6.egg/senti_classifier/senti_classifier.py:173(disambiguateWordSenses)
 731495    0.795    0.000  152.014    0.000 /home/sumit/venv/lib/python3.6/site-packages/nltk/corpus/reader/wordnet.py:1547(path_similarity)
 731495    2.020    0.000  151.219    0.000 /home/sumit/venv/lib/python3.6/site-packages/nltk/corpus/reader/wordnet.py:728(path_similarity)
 731495    7.519    0.000  119.911    0.000 /home/sumit/venv/lib/python3.6/site-packages/nltk/corpus/reader/wordnet.py:658(shortest_path_distance)
1447733   34.959    0.000  102.647    0.000 /home/sumit/venv/lib/python3.6/site-packages/nltk/corpus/reader/wordnet.py:634(_shortest_hypernym_paths)
```
Clearly, the code was spending most of the time in disambiguateWordSenses and
path_similarity which are necessary for WSD.

**No WSD:** Hence I decided to completely skip WSD and go ahead with the
sentiment of the most dominant sense which is the sense of a word with index 0 in nltk.
This is a crude approximation but *something that worked*! the results retaining
the above shape of the graph somewhat. And it took only milliseconds to do the
scoring.

The benchmarking on the complete dataset is going on but this is something
showing an almost balance between computational capacity and the need for
accuracy. I'm not allowed to touch the deleted dataset because of
confidentiality restrictions.

### Sentiment on Edits

However, I could still apply this feature to the [editquality research](https://meta.wikimedia.org/wiki/Research:Automated_classification_of_edit_quality).
 and see
the stats there.

After generating the tuning reports, the Gradient Boosting classifier, (best one) gave almost a jump of 2% while BernoulliNB jumped
from 66% to 77% showing some promises.
