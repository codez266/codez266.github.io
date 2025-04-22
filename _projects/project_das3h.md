---
layout: page
title: Implementing the DAS3H model to model student's learning and retention using spaced repetition
description:
img:
importance: 1
category: work
related_publications: true
github:
---

# Modeling Student Learning with Spaced Repetition: Implementing a DAS3H Model

**This post is an accessible guide to implementing the DAS3H model for modeling student learning and memory using spaced repetition techniques.**

---

## 🎯 Motivation

Understanding how students learn and forget is key to designing intelligent educational systems. The **DAS3H** (Difficulty, Ability, Skill, and Student, with 3-parameter Half-life) model provides a principled way to capture the effects of **time**, **practice history**, and **knowledge component (KC)** difficulty on memory retention. It builds on earlier memory models by explicitly incorporating the idea that memory decays over time unless refreshed.

This makes DAS3H particularly suitable for modeling **spaced repetition**—a technique used by apps like Anki and Duolingo—where practice is scheduled to optimize long-term retention.

---

## 🧠 What is the DAS3H Model?

The DAS3H model estimates the probability that a student will recall a given knowledge component (KC) based on:

- **Student ability**
- **KC difficulty**
- **KC-specific retention/forgetting rate**
- **Time since last correct attempt**
- **Practice history (correct/incorrect responses)**

Mathematically, the probability of a correct response is modeled as a logistic function of the log-time since last practice and other parameters:

\[
P(\text{correct}) = \sigma\left(\theta_s - \delta_k + \gamma_k \cdot \log(t)\right)
\]

Where:
- \( \theta_s \) = student's ability
- \( \delta_k \) = KC difficulty
- \( \gamma_k \) = KC-specific forgetting rate
- \( t \) = time since last correct answer on that KC
- \( \sigma(x) \) = sigmoid function

This model assumes **each KC has its own forgetting curve**, which makes it particularly suited to domains where retention rates vary between concepts (e.g., math formulas vs. vocabulary words).

---

## 🛠️ Implementation Overview

Here’s a quick overview of how to implement DAS3H.

### 1. **Data Format**

The input dataset should contain the following fields:

- `student_id`
- `kc_id` (knowledge component)
- `timestamp` of the attempt
- `is_correct` (binary)
- `attempt_number`

Optionally, store time since last correct response per `(student, KC)` pair.

### 2. **Parameter Initialization**

Each KC has its own:
- Difficulty \( \delta_k \)
- Forgetting rate \( \gamma_k \)

Each student has:
- Ability \( \theta_s \)

You can randomly initialize these parameters or use heuristics from early training performance.

### 3. **Training**

We minimize **binary cross-entropy** loss over the predicted vs. actual correctness using gradient descent.

You can use PyTorch or JAX for a flexible implementation. Here's a simplified PyTorch sketch:

```python
import torch
import torch.nn as nn

class DAS3H(nn.Module):
    def __init__(self, num_students, num_kcs):
        super().__init__()
        self.ability = nn.Embedding(num_students, 1)
        self.difficulty = nn.Embedding(num_kcs, 1)
        self.forgetting_rate = nn.Embedding(num_kcs, 1)

    def forward(self, student_ids, kc_ids, log_time_since_last_correct):
        theta = self.ability(student_ids).squeeze()
        delta = self.difficulty(kc_ids).squeeze()
        gamma = self.forgetting_rate(kc_ids).squeeze()
        logit = theta - delta + gamma * log_time_since_last_correct
        return torch.sigmoid(logit)
```

You’ll need to batch your training data and compute log(t) from timestamps.

## 📊 Evaluating the Model

To evaluate the performance of the DAS3H model, you can use several standard metrics from binary classification tasks, as well as metrics specific to educational data:

- **Accuracy**: Measures the percentage of correctly predicted responses.
- **AUC-ROC (Area Under the ROC Curve)**: Captures the ability of the model to distinguish between correct and incorrect responses.
- **Log-loss (Cross-Entropy Loss)**: Measures the calibration of the predicted probabilities.
- **Mean Absolute Error on Recall Time**: Optional, if you simulate recall prediction over time.

Additionally, you can segment performance by:
- **Time since last attempt** to see if the model degrades reasonably with time.
- **Knowledge component (KC)** to verify KC-specific learning/forgetting curves.
- **Student proficiency levels** to ensure the model generalizes across learner types.

---

## 💡 Insights

Implementing and analyzing the DAS3H model reveals several key insights:

- **Temporal modeling is critical**: Time since last practice is a powerful predictor of recall. DAS3H leverages this directly through the log-time decay term.
- **Different concepts decay differently**: The model learns that not all knowledge components (KCs) are retained equally. Some require more frequent review.
- **Students vary in ability and retention**: The separation of student ability and KC decay rates allows personalization without overfitting.
- **Supports spaced repetition planning**: DAS3H can be used to estimate optimal review times, supporting intelligent tutoring systems that adapt over time.

---

## 🔮 Future Directions

There are many ways to build on and extend the DAS3H model:

- **Meta-learning extensions**: Learn how forgetting rates vary across student profiles to dynamically adjust model parameters.
- **Bayesian DAS3H**: Place priors over student and KC parameters to improve robustness in low-data settings.
- **Item-level effects**: Incorporate question-level embeddings to account for difficulty or bias in specific items.
- **Curriculum planning**: Use predicted recall probabilities to optimize what the student should review next.
- **Active learning for education**: Select the next KC to quiz on based on expected information gain about student knowledge.

---

## 🧰 Related Tools and Libraries

Here are some tools that are helpful for implementing or experimenting with knowledge tracing and memory models:

- [`pyBKT`](https://github.com/CAHLR/pyBKT): A Python library for Bayesian Knowledge Tracing (BKT), suitable for simpler models.
- [`EduMPL`](https://github.com/educational-ml/eduml): Educational Machine Learning library with multi-task learning capabilities.
- [`Torch-KT`](https://github.com/woojihoon/torch-kt): A PyTorch-based framework for implementing knowledge tracing models, including deep KT.
- [`Khan Academy Datasets`](https://www.kaggle.com/c/riiid-test-answer-prediction/data): Useful for large-scale experiments with temporal learning data.

---

