---
layout: page
title: Modeling indoor behaviors using deep-RL
description: Modeling indoor behaviors of people using deep-RL and simulating virus transmissions to evaluate health interventions
img:
importance: 1
category: work
related_publications: false
---

> 📘 **This post is an accessible read of a work under publication**
>
> I am happy to share the code on request for any evaluation of skills, or discussion about the project. After the submission of the work, I will open-source the code, and the manuscript on arxiv.

## Simulating Indoor Behavior and Viral Transmission with Deep RL

This project simulates human behavior in an indoor home environment using Deep Q-Learning to model rational decision-making based on physiological and routine-driven needs. Each agent (represented as a circle in the simulation) has internal state variables such as `needs_food`, `needs_bathroom`, and others that drive its movement across rooms (e.g., kitchen, bathroom, bedroom).

The goal is to train agents to behave in a way that satisfies their needs efficiently while interacting with others in a shared space—capturing both individual rationality and population-level emergent behavior.

### A video demo

Below is a short clip describing the simulation. One of the agents is infected with covid-19 and the other is uninfected. They move about, occassionally going to the bathroom or kitchen depending on the next.

<video width="100%" controls>
  <source src="{{ site.baseurl }}/assets/video/behavior_modeling.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

---

### Feature Representation

Each agent’s state is encoded as a one-hot vector concatenating both **need states** and **room location**. Specifically:

Let:
- $$\mathbf{n} \in \{0,1\}^N $$ be a one-hot vector representing the current active need (e.g., hunger, bathroom).
- $$\mathbf{r} \in \{0,1\}^R $$ be a one-hot vector representing the agent’s current room (e.g., kitchen, bathroom, bedroom, etc.).

The full input to the Q-network is:

$$\mathbf{s} = [\mathbf{n} | \mathbf{r}]$$

where $$ | $$ denotes concatenation.

This vectorized state is passed through a neural network (Q-network) to produce Q-values for each possible room transition action.

---

### Learning

Agents are trained using the standard Deep Q-Learning algorithm. The objective is to approximate the optimal action-value function $$ Q^*(s, a) $$, which represents the expected cumulative reward from state $$ s $$, taking action $$ a $$, and following the optimal policy thereafter.

At each step, we minimize the temporal difference loss:

$$
\mathcal{L}(\theta) = \mathbb{E}_{(s, a, r, s')} \left[ \left( r + \gamma \max_{a'} Q_\theta^{-}(s', a') - Q_\theta(s, a) \right)^2 \right]
$$

where:
- $$ \theta $$ are the parameters of the current Q-network.
- $$ Q_\theta^{-} $$ is a target network with periodically updated weights.
- $$ \gamma \in [0,1] $$ is the discount factor.
- $$ r $$ is the reward received for satisfying a need.

Rewards are sparse and only provided when a need is satisfied (e.g., an agent eats when `needs_food` is active and they enter the kitchen).


### Virus Transmission Dynamics

In addition to behavioral modeling, the simulation includes a probabilistic model of airborne virus transmission, visualized as red particles that diffuse through space.

Each infected agent emits viral particles following a Gaussian spatial distribution centered at their current location:

$$
P(x, y) = \frac{1}{2\pi\sigma^2} \exp\left(-\frac{(x - x_0)^2 + (y - y_0)^2}{2\sigma^2}\right)
$$

where:
- $$ (x_0, y_0) $$ is the agent's current position,
- $$ \sigma $$ is the spread parameter that increases with infection intensity,
- $$ P(x, y) $$ is the probability density of viral concentration at point $$ (x, y) $$.

The **intensity of infection** determines both the **frequency** of particle emission and the **spread** of the Gaussian. This intensity is a time-dependent function that peaks between 4–7 days post-infection and then decays:

$$
I(t) = \exp\left(-\frac{(t - \mu)^2}{2\sigma_t^2} \right)
$$

with:
- $$ t $$ as the number of days since contact,
- $$ \mu = 5.5 $$ (days),
- $$ \sigma_t = 1.0 $$.

Transmission risk is computed by integrating exposure over time and proximity to infected individuals. When multiple agents are in the same room, this allows the simulation to capture how behavioral patterns—like congregating in the kitchen—can influence disease spread at both the individual and population levels.

---

### Applications

This system provides a testbed for studying:
- The effect of household layouts on transmission risk.
- The impact of behavioral interventions (e.g., staggered routines).
- Emergent coordination among rational agents in confined, shared spaces.

By combining deep reinforcement learning with epidemiological modeling, this simulation allows for a rich analysis of decision-making under constraints, with applications in public health, smart environments, and human-AI interaction design.

### Next Steps

Future directions aim to enrich the simulation's realism and policy relevance by incorporating adaptive behavioral and environmental responses:

- **Avoidance Behavior**: Extend agent behavior policies to include infection-aware actions, such as avoiding rooms recently visited by infected individuals or dynamically rerouting to meet needs while minimizing risk. These behaviors can be trained via modified reward structures that penalize risky proximity.

- **Intervention Modeling**: Simulate public health interventions such as room-based ventilation improvements, masking, scheduled access to shared spaces (e.g., staggered kitchen use), or isolation protocols. These can be encoded as either constraints on agent actions or environmental parameters influencing transmission probability.

- **Dynamic Policy Optimization**: Implement meta-RL or multi-agent coordination mechanisms to explore how global objectives (e.g., minimizing overall infection) can be balanced against individual needs through learned policies.

These additions will allow the simulation to serve as a testbed for evaluating the effectiveness of behavioral and policy interventions in indoor spaces.
