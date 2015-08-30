---
layout: post
title: End of GSoC 2015
permalink: End_of_GSoC2015
tags:
- GSoC
- summerintern
---
Aug 28th, 2015 marked the end of my Google Summer of Code intern at Wikimedia Foundation.
My project targeted the [banner expedition](https://en.wikivoyage.org/wiki/Wikivoyage:Banner_Expedition) of [English Wikivoyage](http://en.wikivoyage.org/wiki/). Here's a small walkthrough of my project, what led to it and how it came out to be.

## Background:

English Wikivoyage and some other Wikivoyage projects had developed a culture to render page-wide banners on articles to enhance the overall appeal of the pages by portraying some attractive location of the place in the banner.
The banners were originally rendered through the use of a [Mediawiki Template](https://www.mediawiki.org/wiki/Help:Templates) which however had some issues with it:

1. Banners were not responsive. Since the banners were 2100px X 300px, downloading a large sized banner on small screens became a significant aspect.
2. Table of Contents were displayed in a linear fashion inside the banner, which however did not support multiple levels.
3. Banners on mobile devices clashed with page heading.
4. Due to a 7:1 aspect ration, banners on small screens became too small to leave a visual impact upon the reader.

## The project

Therefore, my project involved developing a Mediawiki extension to resolve the above issues and pack the feature inside a single unit so that its usage can be extended uniformly across all wikis, if needed.
I worked under the mentorship of [Jon Robson](http://www.mediawiki.org/wiki/User:Jdlrobson) and [Nicolas Raoul](https://www.mediawiki.org/wiki/User:Syced). We formed a great team and I really enjoyed working with them!
We had regular meetings and I never was blocked on any issue for a long time. Their ideas and suggestions helped a lot in both refining the code as well as the aesthetics part.

Here's the final demo of what the extension does:

[![Final Demo](/images/wikivoyage.png "Banner on English Wikivoyage")](/images/wikivoyage.png)

## Phases of the project(in brief)

### Community Bonding Period:

This is a common period for all GSoC projects where the intern is required to get more familiar with his/her organization and his/her mentors. I was already in touch with my mentors from before this period so it involved taking the first steps towards building the extension. It involved:

* Bootstrapping the extension from a bootatrap extension Template.
* Setting up basic [Mediawiki Hooks](http://www.mediawiki.org/wiki/Manual:Hooks) to insert banners by default on all pages.
* Starting the development of a [parser function](http://www.mediawiki.org/wiki/Manual:Parser_functions) {{PAGEBANNER}} to insert custom page banners on articles.

### Coding phase:

This phase marked the beginning of actual coding.

* The parser function {{PAGEBANNER}} was completed.
* The feature to fetch a banner from [Wikidata](http://www.wikidata.org) was implemented
* Feature to add icons to banners was added.
* The banners were made to display on preview.
* Php unit tests were added to cover server-side code.
* Multiple level Table of contents were added.
* The existig code was refactored to allow better testing.
* Issues of styling and Flash of unstyled content were resolved.
* The work to add mobile compatibility was added.
* Table of contents generation was shifted to server side.
* "Srcset" attribute used to trigger responsiveness.
* A module to focus specific areas of banner on small screens was added.
* The extension has been deployed on [English Wikivoyage](http://en.wikivoyage.org/wiki/) and is in use there.

The above is a brief overview and complete report can be found at [Phabricator](https://phabricator.wikimedia.org/T101227)

### English Wikivoyage Community support:

[English Wikivoyage](http://en.wikivoyage.org/wiki/) community is great and very open to new ideas. A lot of discussions took place on [Traveller's Pub](en.wikivoyage.org/wiki/Traveller's_Pub) which led to the refinement of the extension. The support extended by them in making this a success from feedback to criticism to pointing out bugs has been invaluable!

### Challenges and Takeaways:

* Maintaining a testable code was a challenge that I learnt and also the need for having good test cases, as they allow preemptive bug monitoring.
* Ensuring community involvement and making suitable changes to satisfy the beneficiary community needs was an important aspect that I learnt.
* Writing code conforming to standards and also keeping it self-explanatory.
* Weekly meetings with my mentors helped me keep everything on track and things never got too far out of hand.

## A cool new feature!

*Due to banner cropping on smaller devices, a new feature has been added which would focus the banner on specific areas.
This can have a huge impact on banner creation as now it would no longer be mandatory to create specific size banners. Just pick a suitable image and focus it!*

<a href="/images/mobile1.png"><img src="/images/mobile1.png" style="float:left;width:47%;margin:5px" title="focus on left"></a>
<a href="/images/mobile2.png"><img src="/images/mobile2.png" style="width:47%;margin:5px" title="focus on right"></a>

### Final Words

The developed extension's documentation is at [Extension:WikidataPageBanner](http://mediawiki.org/wiki/Extension:WikidataPageBanner).

It was a great experience to work with such an organization which aims to bring free knowledge to everyone. The people are great!
For anyone thinking of contributing to it, I'd say, Just Go for it!
