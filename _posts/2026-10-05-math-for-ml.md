---
title: "Mathematics for Machine Learning"
date: 2026-10-05 20:53:00 +0200
categories: [Machine Learning, Mathematics]
tags: [Calculus, LinearRegression, CostFunction]
toc: true
math: true
---
# Mathematics for Machine Learning

## Differential calculus

If you've ever wondered how machine learning models actually learn from data, the secret often boils down to a fundamental tool: **differential calculus**. It allows us to train one of the most classic models in machine learning: **linear regression,** by finding the optimal line of best fit.

### Measuring our mistakes: the cost function

Lets take $(x_1, y_1), ..., (x_n, y_n)$ a scatter plot of data points. 

The goal is to fit a straight line defined by the equation:

$y = mx + b$

Real world data is often noisy, then, a line rarely passes through every single point. To measure how wrong the line is, we use squared error for each point: 

$SE = [(mx_i + b) - y_i]^2$

This is the predicted value minus the true value, all squared. By summing up all these individual errors, we get a total error function, often called the **cost function** or loss function: $J(m, b)$. 

$J(m, b) = \sum_{i=1}^{n}[(mx_i + b) - y_i]^2$

Our goal is simple: find the exact values of $m$ (slope) and $b$ (intercept) that **minimize** this error function $J$.

## Sources

1. https://users.umiacs.umd.edu/~hal3//courses/2013S_ML/math4ml.pdf
