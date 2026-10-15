---
title: "Foundations of Machine Learning Theory"
date: 2026-10-14 16:41:00 +0200
categories: [MachineLearning, Theory]
tags: [SupervisedLearning, HypothesisSpace, LossFunctions, Generalization, TrainTestSplit]
toc: true
math: true
---
# Machine Learning Theory

## Definition

A computer program is said to learn from experience E with respect to some class of
tasks T and performance measure P if its performance at tasks in T, as measured by P,
improves with experience E according to Tom M. Mitchell. 

### Supervised learning

> 
> 
> 
> <img width="561" height="160" alt="image" src="https://github.com/user-attachments/assets/3426e5fb-a60a-433f-aceb-109968334236" />
> 
> This is a decompressed view of supervised learning. It can be decomposed by 2 distinct phases. The training and testing one. 
> 

#### The hypothesis class

The goal of supervised learning is to use $D$ to learn from a function $h$ : $R^d → C$ that can predict $y$ from $x$. Pick a hypothesis class is the first task to do. At first, we have $h$ $\in$ $H$, where $H$ denotes the hypothesis class. $h$ could be for example: 

- Linear Classifiers
- Decision Trees
- Neural Networks
- Support Vector Machines

Be careful, there is no free lunch theorem. No free lunch means that no single learning algorithm or hypothesis class is universally better than any other when averaged across all possible problems.  

**How to choose the best model?** 

Every ML algorithm has to make assumptions. In machine learning, saying that every algorithm has to make assumptions (often called inductive bias) means that a model must possess built-in beliefs or preferences about what the underlying data looks like before it can learn anything useful from it.

Without these assumptions, a learning algorithm cannot generalize to new, unseen data, it would simply memorize the training examples and fail at predicting anything outside of them.

- If your assumptions match the reality of the problem (e.g., using a linear model on linearly separable data), your algorithm performs brilliantly.
- If your assumptions clash with reality (e.g., using a linear model on complex, highly non-linear data), your algorithm will perform terribly.

### The loss function

The second task consists of finding the best function within the hypothesis class $h$ $\in$ $H$ using $D$ is called the learning process. The idea is to pick $h$ making the least mistakes and the simplest as possible. To measure this, we use the loss function:

$l : R → R$ quantifies how well $h(x)$ approximates $y$. The lower the value of $l(y, h(x))$, the better is the approximation. 

- $l(y, y)$ = 0 because we find $h(x) = y$ which is what we search.
- $l(y, h(x)) ≥ 0$ for all $y, h(x)$

**Common loss functions**

| Loss | Function | Task |
| --- | --- | --- |
| $0/1 Loss$ | $l(y, h(x)) = \begin{cases}
1 & \text{if } h(x) \neq y \\
0 & \text{else}
\end{cases}$
 | Classification |
| Quadratic loss | $l(y, h(x)) = (y − h(x))^2$ | Regression |
| Absolute loss | $l(y, h(x)) = |y − h(x)|$ | Regression |

### Loss Minimization

Using the training data $D$, we can compute the average loss over all the data points by using: 

$L = \frac{1}{N} \sum_{i=1}^{N} l(y_i,\, h(x_i))$

Finding the best hypothesis means finding the $h$ that minimizes the loss: 

$h^* = \arg\min_{h \in \mathcal{H}} \frac{1}{N} \sum_{i=1}^{N} l(y_i,\, h(x_i))$ 

### Generalization

The goal is to find $h$ such that, for an unseen point $(x, y) ∼ P, h(x) ≈ y$.
In other words, we want $h$ to generalize.
However, the loss over the training set does not give us information about the generalization
capabilities of the trained model. 

When $h(·)$ has a very low loss, but it does not perform well in unseen data, we say there is
overfitting causing that our model does not generalize well.

#### Generalization loss

**Generalization loss** (often called generalization error) is the difference between how well your machine learning model performs on the data it was trained on versus how well it performs on **unseen, new data** from the same distribution.

In mathematical terms, it represents the discrepancy between your **training error** (loss on your training dataset) and your **true error** (expected loss over the entire underlying probability distribution of all possible inputs).

True error

$\epsilon = \mathbb{E}_{(x,y)\sim\mathcal{P}} \big[\, l(y,\, h(x)) \,\big]$

$\mathbb{E}_{(x, y) \sim \mathcal{P}}$: The expected value taken over the true, underlying probability distribution $\mathcal{P}$ of all possible data points $(x, y)$ in the real world.

### Train/Test splits

During this phase, the dataset $D$ is split into 3 different sets: 

- Training set $D_{TR}$ - Used to learn h
- Validation set $D_{VAL}$ - To check for overfitting
- Test set $D_{TEST}$ - Used to evaluate the chosen h and have an estimate of the generalization
error or loss

Typical sets are 70/10/20, 80/10/10, 60/20/20. It is important to split the data properly to simulate a real life scenario and to avoid data leakage.

### How to use the learning algorithm?

Given a hypothesis class $H$:

Train a model by minimizing the training loss:

$\frac{1}{|\mathcal{D}{TR}|} \sum{(x,y)\in\mathcal{D}_{TR}} l(y,\, h(x))$

Choose among potential models using the validation set

$\frac{1}{|\mathcal{D}{VAL}|} \sum{(x,y)\in\mathcal{D}_{VAL}} l(y,\, h(x))$

Evaluate the testing loss of the chosen model $h$

$\epsilon_{\text{TEST}} = \frac{1}{|\mathcal{D}{TEST}|} \sum{(x,y)\in\mathcal{D}_{TEST}} l(y,\, h^*(x))$
