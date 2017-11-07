---
layout: post
title: Topics suggestion to new drafts on Wikipedia
permalink: drafttopic-detection
tags:
- research
- wikipedia
---

How about an AI service that could classify new drafts on Wikipedia into some
predefined categories? Wouldn't it make the lives of editors easier in
improving the articles?
The project ["Automatic suggestion of topics to new drafts"](https://meta.wikimedia.org/wiki/Research:Automatic_new_article_topics_suggestion) strives to address exactly this. I'll not try to go much into the tech specs of the project which I've explained in detail on my [research page](https://meta.wikimedia.org/wiki/User:Sumit.iitp/Research) rather I'll try to provide a high level motivation for the project and what it entails.

The problem starts with the current state of Wikipedia: too many people
creating new articles but too few to [review](https://meta.wikimedia.org/wiki/Research:New_page_reviewer_impact_analysis) them. The current review process which has been since long is that once a page is created, new page patrollers review the page for any offensive material or violations of any Wikipedia policy. They also help to improve the article by suggesting tags for improvement or adding WikiProjects.

Now WikiProjects on Wikipedia are nothing but certain dedicated topical
namespaces where articles of a similar topic are grouped. For example
WikiProejct: Biography would contain all articles related to biographies of
persons. This dedicated group helps subject experts to pay attention to
articles of their field and improve them.

Therefore, at the core of improving new drafts on Wikipedia as quickly as
possible lies the efficacy with which we can tag these new drafts with
WikiProject topics. 
The above project aims to provide exactly such an efficieny in tagging new
drafts with WikiProjects by building a Machine Learning model that would
**suggest** WikiProjecs for these new articles.
For example, someone might be trying to write a biography
of a recent celebrity but not able to meet the required standards for the
article. If our model is able to predict that the draft **could** belong to
WikiProject:Biography early on, biography subject experts would get the
attention of the article and could provide useful suggestions.
