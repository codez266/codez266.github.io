---
title: UG-Research-Project
date: 2017-05-01
permalink: /posts/2017/05/ug-research-project
tags:
- research
- ug-project
---
A lot of times I had to explain my seemingly "complicated" Undergradute-research project
or as it may seem because of the relative novelty of the topic. Hence I finally
take time to pen this down once and for all for any references for myself or
anyone else in future, with an aim to comprehensively define my topic and research conducted
as part of UG Thesis project at IIT Patna, 2016-17.
 For a concise and brief
overview of my project, head over to the poster
__[here](/resources/ug_thesis_poster.pdf)__.

## Background

The title of my UG-Research Project is __"Entity Relation ranking from
unstructured text"__. Lets first see what are entity relations.

### Entity Relations

These are the fundamental blocks of structured knowledge bases. Example, any
knowledge base has data in the form of _entities_ which may be representation of any real world object or any concept.
These are linked to each other by **subject-predicate-object** triples
where _predicate_ is some property linking _subject_ and _object_ each of which
forms an _entity_ in knowledge bases.
Example, _cat_ is an entity and _animal_ is an entity. _Cat_ __is an__ _animal_
is a relation which links _cat_ and _animal_ through the _is_ relation.

Now my project was about ranking these "entity relations" which means ranking
amongst several __subject-predicate-object__ triple where each such relation has
a common subject. Its best illustrated through a snapshot of the dataset I'm using.

<img src="/images/profession_train.png" width="700px" title="Train file snapshot">

Here, each relation is a _Person_ - _Profession_ relation to which my project is
restricted. Clearly, each _person_ can have several different _professions_ and
the goal is to rank these relations based on the relevance of the _profession_
to the _person_ its associated with. 

E.g. _William Shakespeare_ makes more sense with _Playwright_ than with
_Lyricist_.

The quantitative definition of _"relevance"_ is taken to be the __amount of
information about the profession present in Wikipedia related to that person__.

## Dataset

The dataset used was from [WSDM Cup 2017](https://wsdm-cup-2017.org/triple-scoring.html) competition. I only used the profession entities for my project.
Each person-profession pair had a label between 0-7 with 7 being most relevant
and a similar labels had to be predicted for test samples based on the relevancy
information.

## Approach

During the course of my project I undertook two approaches, one feature based,
and another _deep learning based_ which was a __novel proposal__ from my side.

* [Feature Based Method](#Feature-Based-Method)
* [Deep Learning Method](#Deep-Learning-Based-Method)

### _Feature Based Method_

#### __Query Expansion__:

This was done to get more context out of individual profession words. E.g. for
'actor', words like _'acting', 'acted', 'starred', 'cast', 'played'_ were
extracted. This was done for each of the 200 professions from the dataset using
a Logistic Regression classifier where the positive samples were Wikipedia
articles of persons who only worked in that profession and negative samples were
Wikipedia articles of persons who never worked in that profession. Then after
training, the above words would come up as _top features_.

#### __Features__:

Using the above expanded query for each profession, a set of features was
formulated on the Wikipedia article of the person for the _person-profession_
pair.

![](/images/wiki_features.png "Features on Wikipedia Article")

#### __Ranking__:

These sets of features were then fed to an SVM or Random Forest Classifier and
ranking was done by following a hierarchical classification strategy, i.e.,
first classifying on the top level as 0 or 1, then on the next level as 0 or 1
then on the next level as 0 or 1. This way we get a tree with eight leaves
corresponding to the 8 different labels between 0-7. The results are in the
[poster](/resources/ug_thesis_poster.pdf).

### _Deep Learning Based Method_

For this approach, the above feature selection strategy was replaced by a CNN
network and following the classification of a profession as relevant or non-relevant with respect to a person
based on a set of features automatically identified by the CNN network. A
__single__ classifier was used across all __professions__ unlike the above case
and the sample selection strategy for each profession was similar to above.

__CNN Statistics:__

|Parameters|Values|
|----------|------|
|Samples|22638|
|Hidden Layers|2|
|Hidden Neurons|512,64|
|Embedding Dimension|300|
|Precision|82%|

![](/images/cnn_btp.png "CNN Structure for text classification Ref. Yoon Kim's -
CNN for Sentence classification")

## Future Work

A lot of work could be taken up in the domain of "ranking structured knowledge
base entities from unstructured text". I've shown the viability of the deep
learning method to the problem but not shown the final rankings produced by the
CNN as I had to stop at the stage of predicting a single profession of a person
as relevant or non-relevant due to time constraints. This could be taken up as
an interesting research problem.
