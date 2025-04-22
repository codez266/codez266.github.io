---
layout: page
title: Learning Bayesian user model from 8 M Census records to design an efficient preference elicitation agent.
description: Understanding privacy preferences of users with a personalization AI agent
img: assets/img/personalization_overview.jpg
importance: 1
category: work
related_publications: true
---

> 📘 **This post is an accessible read of the publication:**
>
> Asthana, Sumit, et al. *"I know even if you don't tell me": Understanding Users' Privacy Preferences Regarding AI-based Inferences of Sensitive Information for Personalization.*
> *Proceedings of the 2024 CHI Conference on Human Factors in Computing Systems.* 2024.
> [https://doi.org/10.1145/3613904.3642153](https://doi.org/10.1145/3613904.3642153)

This post distills the motivation, methods, and key insights from our research on privacy and AI-based personalization. It’s written for a broader audience — so whether you're a designer, privacy researcher, or just AI-curious, this is for you!

## Project Overview

**Personalization** offers undeniable benefits to user experience—recommendations and services that feel tailor-made based on individual preferences. But what happens when those personalizations come from AI-inferred data users never knowingly provided?

In this project, I explored how users react when they become aware of AI-generated inferences based on the data they willingly provide. The work led to a paper that combines large-scale experimentation with sophisticated modeling approaches to investigate privacy and consent in the age of personalization.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/personalization_overview.png" title="Method Overview" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Overview of our method: Learn a user preference model from US Census data, and adaptively ask questions to users to maximize their preference knowledge, then ask privacy sensitive questions.
</div>

---

## Skills Demonstrated

- Probabilistic modeling (Bayesian Networks via `pgmpy`, structure learning via `ARACNE`)
- Scalable data processing and augmentation using US Census datasets
- Designing and executing large-scale online experiments (n > 800)
- Human-centered design for privacy and AI consent
- Empirical evaluation of personalization and inference transparency

---

### Core Motivation

Today, personalization engines routinely make inferences about users’ interests and attributes—from their online behavior, purchase history, and more. While helpful, these inferences are often invisible and made without explicit consent. This can lead to discomfort, mistrust, and, in some cases, real harm—especially when the inferred data is sensitive or incorrect.

Our goal was to understand how awareness of AI inferences changes user consent behaviors and to build computational systems that support transparency and agency.

---

## Key Contributions

- **Quantified Privacy Behavior**: We conducted two large-scale experiments (N=877 total) to measure how people’s consent decisions change when shown AI inferences of varying sensitivity (e.g., public interest, income level, or protected attributes like race).
- **Measured Impact of Incorrect Inferences**: Participants were less likely to consent when they saw incorrect inferences, even if the original data was provided willingly.
- **Highlighted Need for Granular Consent Interfaces**: Our work provides empirical support for designing new kinds of AI interfaces that foreground inferences and ask for enthusiastic, informed consent.

---

## Technical Approach

### Learning a Bayesian User Model from Census Data

To simulate realistic AI inferences, I trained a Bayesian network on public data from the **Supplement of Public Arts Participation (SPPA)** dataset, drawn from the US Census Bureau's **Current Population Survey (CPS)**. The data included ~125,000 respondents and covered:

- **Demographics** (e.g., age, gender)
- **Household information** (e.g., marital status, children)
- **Engagement in public arts** (e.g., attending museums, concerts)

After pre-processing and filtering the data (removing minors, incomplete interviews, and variables irrelevant for modeling), I used **Census person weights** to upsample records and produce a representative dataset of ~8.2 million US adults.

Each CPS question was treated as a variable in the user model. To learn the structure of the Bayesian network, I applied the **ARACNE algorithm** via the `bnlearn` package. I then estimated conditional probability tables using the `pgmpy` library.

This allowed us to simulate realistic, data-driven AI inferences—like predicting a person’s income bracket or cultural preferences—based on the same data they explicitly provided.

#### Code snippet: Parameter learning

```python
from pgmpy.estimators import StructureEstimator, BayesianEstimator
from pgmpy.models import BayesianModel
from pgmpy.inference import VariableElimination

# First load the graph structure and then learn the parameters
dag = None
with open('../datasets/{}_network_aug8.pk'.format(dataset), 'rb') as fin:
    dag = pickle.load(fin)
model = BayesianNetwork(ebunch=dag.edges())
mle = MaximumLikelihoodEstimator(model=model, data=data)
cpds = mle.get_parameters(weighted=True)
model.add_cpds(*cpds)
with open("../datasets/cps_model_aug8.pk", "wb") as fout:
    pickle.dump(model, fout)
```

---

## Insights for Industry

This project demonstrates how to:

- **Use RNN-style personalization scenarios** without sacrificing user agency.
- **Train Bayesian user models** at scale using public data while maintaining interpretability.
- **Build AI systems that prioritize transparency** and give users the opportunity to meaningfully consent—or opt out—of AI-based inferences.

As AI-based personalization becomes ubiquitous, this work emphasizes a vital point: **the line between data explicitly shared and data inferred is not only blurry—it’s consequential**.

---

## Future Directions

I'm interested in expanding this work into:

- Real-time interfaces where users can audit or revoke AI inferences
- Generalizable Bayesian/RNN hybrid models for interactive systems
- Deployable consent frameworks that adapt as AI learns from more user data

This project lays the groundwork for AI systems that are not just powerful—but **aligned with user values**.

