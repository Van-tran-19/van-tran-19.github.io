---
title: "K-Nearest Neighbors"
date: 2026-10-15 20:46:00 +0200
categories: [MachineLearning, Algorithms]
tags: [KNN, DistanceMetrics, InstanceBasedLearning, Classification, Regression]
toc: true
math: true
---
# K-nearest neighbors

K-nearest neighbors is the oldest learning method. 

## Definition

Given a test point $x^*$, let us denote $S_{x^*}$ as its set of k nearest neighbors.

**Formally:** 

$S_{x^*} \subseteq D \;\text{s.t.}\; |S_{x^*}| = k$

$\forall (x', y') \in D \setminus S_{x^*},\quad 
\text{dist}(x^*, x') \ge 
\max_{(x'', y'') \in S_{x^*}} \text{dist}(x^*, x'')$

It means that you pick a subset $S_{x^*}$ from your dataset $D$. This subset must contain exactly $k$ elements. These elements are supposed to be the neighbors of $x^*$. Then, the second line says that take any point $(x’, y’)$ not in the chosen subset $S_{x^*}$. Its distance to $x^*$ must be greater or equal to the distance of the farthest point inside $S_{x^*}$. 

### Regression

If the label $y$ are numbers, k-NN predicts by averaging the values of neighbors:

$\hat{y} = h(x^*) = \frac{1}{k} \sum_{(x_i'', y_i'') \in S_{x^*}} y_i''$

### Classification

If the label $y$ are numbers, k-NN predicts by majority vote: 

$\hat{y} = h(x^*) = \text{mode}\big(\{\, y_i'' : (x_i'', y_i'') \in S_{x^*} \,\}\big)$

### Distance Metric

The definition of the set $S_{x^*}$ is strongly dependent on the distance metric that is used. The better the chosen metric reflects the similarity among points the better the resulting model
will be. One of the most common metrics used is the Minkowski distance:

$\text{dist}(x, z) = \left( \sum_{j=1}^{D} |x_j - z_j|^p \right)^{1/p}$

**What Minkowski distance is?** 

It’s a **general formula** that measures how far two points

$x=(x_1,x_2,…,x_D), z=(z_1,z_2,…,z_D)$

are from each other in a $D$‑dimensional space.

**Why it matters in ML?** 

Because k‑NN depends **entirely** on how you measure distance.

- With p=1, neighbors are chosen by Manhattan geometry.
- With p=2, neighbors are chosen by Euclidean geometry.
- With p=∞, neighbors depend only on the worst coordinate difference.

Different p → different neighbors → different predictions. This is why choosing the right distance metric is crucial.

## Instance based learning

The algorithm doesn’t learn an explicit model. It stores the training instances ****and uses them directly at prediction time. Most ML algorithms learn parameters:

- linear regression learns weights
- neural networks learn millions of parameters
- decision trees learn splits

k‑NN learns nothing like that. It keeps the dataset and waits. 

When you give k‑NN a new point x∗, it doesn’t use a formula or a learned model.
Instead, it:

1. Looks at all stored training points
2. Finds the **k closest ones**
3. Uses their labels to produce the prediction

If the labels are numbers → it averages them (regression).
If the labels are categories → it takes the majority vote (classification).

**What is the difference between an instance based learning algorithm and a model based learning algorithm?** 

Model‑based algorithms compress knowledge into parameters.
Instance‑based algorithms keep the raw data and compare new inputs to it.
