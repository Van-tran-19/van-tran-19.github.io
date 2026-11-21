---
title: "Gradient Descent, Regularization, and Classifier Evaluation in Machine Learning"
date: 2026-11-05 18:00:00 +0200
categories: [MachineLearning, Theory]
tags: [GradientDescent, RidgeRegression, Lasso, Regularization, LogisticRegression, Precision, Recall, Classification]
toc: true
math: true
---
# Gradient descent, regularization and evaluating a classifier

## Objectives

- explain how the learning rate and feature scaling affect gradient descent,
- explain what an L2 and an L1 penalty do to the coefficients, and choose between them,
- choose a hyperparameter on the validation set and use the test set once,
- compute and interpret a confusion matrix, precision and recall,
- explain why accuracy fails when one class is rare, and choose a decision threshold from costs.

## Setup

In the first lab, we use the Auto MPG dataset: 392 cars (after removing the rows with missing values), and the goal is to predict their fuel consumption in miles per gallon (mpg) from six numeric features (cylinders, displacement, horsepower, weight, acceleration, model year).

The cars are split into a training set (60%, 234 cars) to fit the models, a validation set (20%, 79 cars) to compare models and choose their hyperparameters, and a test set (20%, 79 cars), used once at the end to estimate the performance of the chosen model. In Part C we also use a synthetic dataset of car components inspected at the end of a production line.

I will run this code to install and load libraries and data. 

```python
# %pip install pandas numpy matplotlib scikit-learn seaborn

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression, Ridge, Lasso, LogisticRegression
from sklearn.preprocessing import PolynomialFeatures
from sklearn.exceptions import ConvergenceWarning
from sklearn.metrics import (r2_score, confusion_matrix, precision_score, recall_score, roc_curve, roc_auc_score,
                             precision_recall_curve, average_precision_score)

pd.set_option("display.float_format", lambda v: f"{v:.3g}")
np.set_printoptions(precision=3, suppress=True)
# Lasso warns when it stops before full convergence for very small lambda; we silence it
import warnings
warnings.filterwarnings("ignore", category=ConvergenceWarning)

df = sns.load_dataset("mpg")
df_clean = df.dropna().copy()

numeric_cols = ["cylinders", "displacement", "horsepower", "weight", "acceleration", "model_year"]

# Split: 60% training, 20% validation, 20% test
trainval_df, test_df = train_test_split(df_clean, test_size=0.2, random_state=42)
train_df, val_df = train_test_split(trainval_df, test_size=0.25, random_state=1)

X_train, y_train = train_df[numeric_cols].values.astype(float), train_df["mpg"].values
X_val, y_val = val_df[numeric_cols].values.astype(float), val_df["mpg"].values
X_test, y_test = test_df[numeric_cols].values.astype(float), test_df["mpg"].values

# Standardization: subtract the mean and divide by the standard deviation, both computed on the training set
mu, sigma = X_train.mean(axis=0), X_train.std(axis=0)
X_train_s, X_val_s, X_test_s = [(X - mu) / sigma for X in (X_train, X_val, X_test)]

def add_ones(X):
    '''Adds the column of ones, so that w_0 is the first entry of w.'''
    return np.c_[np.ones(len(X)), X]

print("Training:", X_train.shape, " validation:", X_val.shape, " test:", X_test.shape)
#output
Training: (234, 6)  validation: (79, 6)  test: (79, 6)
```

## **Part A. Gradient descent for linear regression**

In this part, we use gradient descent to solve linear regression, instead of computing the closed form solution $\hat{w}=(X^TX)^{−1}X^Ty$ directly. 

This is the analytical, exact formula to find the optimal weights $w$ in one direct calculation (using Ordinary Least Squares). However, computing the inverse of the matrix $(X^T X)^{-1}$ takes cubic time complexity $\mathcal{O}(D^3)$ with respect to the number of features $D$, making it computationally infeasible or extremely slow for massive datasets with millions of features or samples. Instead of solving it all at once, gradient descent is an iterative optimization algorithm. It starts with an initial guess and gradually tweaks the weights step-by-step until they minimize the error.

Recall that we fit $\hat{y}=w^Tx$ (linear prediction for a single data point), where $x=(1,x_1,…,x_D)^T$ (the feature vector for a data point) contains the column of ones, by minimizing the mean squared error (MSE):

$J(w)=1/N∑_{i=1}^N(y_i−w^Tx_i)^2$

This function measures how "wrong" our model is. $y_i$ is the true target value, and $w^T x_i$ is our predicted value for the $i$-th data point out of $N$ total samples. 

$$
w^{(\tau+1)} = w^{(\tau)} - \alpha \nabla_w J(w^{(\tau)})
$$

The gradient descent update rule is defined as: 

- $w^{(\tau)}$ represents the weights at the current iteration step $\tau$ and $w^{(\tau+1)}$ are the updated weights for the next step.
- $\alpha$: the learning rate is a small positive constant that controls how large of a step we take in each iteration. If $\alpha$ is too large, the algorithm might overshoot the min. If it’s too small, it will take too long to converge.
- $\nabla_w J(w^{(\tau)})$ is the gradient of the cost function with respect to the weights.

$$
\nabla_w J = -\frac{2}{N} X^T (y - Xw)
$$

The gradient formula is a matrix-vector form of the derivative of the MSE cost function. $X$ is the design matrix containing all feature vectors, and $y$ is the vector of true target values. $(y - Xw)$ represents the **residuals.** 

The algorithm doesn’t run forever, it stops when either: 

- $\Vert{}\nabla_w J\Vert{} \le \epsilon$
- or it reaches a safety limite

We will deploy it into code: 

```python
def mse(X, y, w):
    '''Mean squared error J(w). X has the column of ones.'''
    return np.mean((y - X @ w) ** 2)

def gradient(X, y, w):
    '''Gradient of J at w.'''
    return -2 / len(X) * X.T @ (y - X @ w)

def gradient_descent(X, y, alpha, max_iter=1000, eps=1e-3):
    '''Gradient descent from w = 0. Returns the final w and the values of J, one per iteration.'''
    w = np.zeros(X.shape[1])
    history = []
    for tau in range(max_iter):
        history.append(mse(X, y, w))
        g = gradient(X, y, w)
        if np.linalg.norm(g) <= eps:
            break
        w = w - alpha * g
    return w, history
```

We run gradient descent for 1000 iterations with four learning rates, on the **standardized** features and on the **raw** features, and plot J against the iteration (log scale).

```python
Xtr, Xtr_raw = add_ones(X_train_s), add_ones(X_train)
w_ols = np.linalg.solve(Xtr.T @ Xtr, Xtr.T @ y_train)    # closed form solution, for comparison
J_min = mse(Xtr, y_train, w_ols)
print(f"Minimum of J (closed form solution): {J_min:.4g}")

alphas_std = [0.01, 0.1, 0.2, 0.25]
alphas_raw = [0.01, 0.1, 0.2, 0.25]    # add smaller learning rates here

fig, axes = plt.subplots(1, 2, figsize=(11, 4), sharey=True)
for ax, X, alphas, title in [(axes[0], Xtr, alphas_std, "Standardized features"),
                             (axes[1], Xtr_raw, alphas_raw, "Raw features")]:
    print(f"\n{title}")
    for a in alphas:
        with np.errstate(over="ignore", invalid="ignore"):
            w, h = gradient_descent(X, y_train, alpha=a, max_iter=1000, eps=0)
        ax.plot(h, label=f"alpha = {a:g}")
        J_end = h[-1]
        print(f"  alpha = {a:<8g} J after 1000 iterations: " + (f"{J_end:.4g}" if np.isfinite(J_end) and J_end < h[0] else "diverges"))
    ax.axhline(J_min, color="k", ls=":", label="minimum (closed form solution)")
    ax.set_yscale("log"); ax.set_ylim(5, 1e6); ax.set_xlim(0, 1000)
    ax.set_xlabel("iteration"); ax.set_title(title); ax.legend(fontsize=8)
axes[0].set_ylabel("J (training MSE)")
plt.tight_layout(); plt.show()

print("\nRange of the raw features (training set):")
display(pd.DataFrame({"min": X_train.min(axis=0), "max": X_train.max(axis=0)}, index=numeric_cols).style.format("{:g}"))
#output
Minimum of J (closed form solution): 12.17

Standardized features
  alpha = 0.01     J after 1000 iterations: 12.26
  alpha = 0.1      J after 1000 iterations: 12.17
  alpha = 0.2      J after 1000 iterations: 12.17
  alpha = 0.25     J after 1000 iterations: diverges

Raw features
  alpha = 0.01     J after 1000 iterations: diverges
  alpha = 0.1      J after 1000 iterations: diverges
  alpha = 0.2      J after 1000 iterations: diverges
  alpha = 0.25     J after 1000 iterations: diverges

Range of the raw features (training set):
 	min	max
cylinders	3	8
displacement	70	455
horsepower	46	230
weight	1613	5140
acceleration	8	23.7
model_year	70	82

```

<img width="1089" height="390" alt="download" src="https://github.com/user-attachments/assets/9446ab0a-6864-44dc-a509-3370dc8e836d" />

**Interpreting the results**

Scenario A with standazdized features (Xtr), for $\alpha = 0.01, 0.1, 0.2$: succesfully converge and for 0.1 and 0.2 hit the exact closed form minimum (12.17). For $\alpha$ = 0.25: it diverges. 

Scenario B with raw features, all learning rate diverge. 

**Why Do Raw Features Cause Gradient Descent to Diverge?**

The range of the raw features are on widly different scales, the cost function landscape becomes over-control. Standardizing data brings all features onto a comparable numerical scale (e.g., mean 0, variance 1). This makes the error surface spherical, allowing gradient descent to take smooth, direct steps straight to the minimum without diverging.

I start again to see until which value, J tends to diverge. 

```python
Standardized features
  alpha = 0.01     J after 1000 iterations: 12.26
  alpha = 0.1      J after 1000 iterations: 12.17
  alpha = 0.2      J after 1000 iterations: 12.17
  alpha = 0.21     J after 1000 iterations: 12.17
  alpha = 0.22     J after 1000 iterations: 12.17
  alpha = 0.23     J after 1000 iterations: 12.17
  alpha = 0.231    J after 1000 iterations: 12.17
  alpha = 0.232    J after 1000 iterations: 12.17
  alpha = 0.233    J after 1000 iterations: 12.17
  alpha = 0.234    J after 1000 iterations: 12.17
  alpha = 0.235    J after 1000 iterations: diverges
  alpha = 0.24     J after 1000 iterations: diverges
  alpha = 0.25     J after 1000 iterations: diverges
  
  Raw features
  alpha = 1e-07    J after 1000 iterations: 93.26
  alpha = 1e-06    J after 1000 iterations: diverges
  alpha = 0.0001   J after 1000 iterations: diverges
  alpha = 0.001    J after 1000 iterations: diverges
  alpha = 0.01     J after 1000 iterations: diverges
  alpha = 0.1      J after 1000 iterations: diverges
  alpha = 0.2      J after 1000 iterations: diverges
  alpha = 0.25     J after 1000 iterations: diverges
```

It diverges when $\alpha ≥ 0.234$ for the standardized features and for the raw features, it diverges all the time. 

For α=0.01: J decreases smoothly and monotonically, but the step size is too small to fully converge within 1000 iterations, stopping at 12.26. 

For α=0.1 and α=0.2: Both curves decrease rapidly and reach the exact closed-form minimum before the 1000th iteration, with α=0.2 converging in roughly half the iterations of α=0.1. 

For α=0.25: The learning rate exceeds the stability threshold causing J diverges to infinity. 

Among the tested values, α=0.2 is the optimal choice because it achieves the fastest convergence to the exact minimum 12.17 without diverging. 

For the raw features, All four learning rates cause gradient descent to diverge immediately. 10−7 is the largest that does not diverge where J = 93.26. Looking at the table, the raw features span vastly different orders of magnitude. 

Feature scaling(standardization) is essential before running gradient descent. It makes the loss landscape well-conditioned, allowing us to use a much larger learning rate and converge orders of magnitude faster to the global minimum.

## **Part B. Regularization: L2 (ridge) and L1 (lasso)**

Regularization modifies the standard linear regression objective to prevent **overfitting** by penalizing large model weights.

**The Problem with Ordinary Least Squares (OLS)**

In standard linear regression, the only goal is to minimize the Residual Sum of Squares (RSS measures the total squared prediction error of regression model across all data points), $\text{RSS} = \sum_{i=1}^N (y_i - \hat{y}_i)^2 = \Vert{}y - Xw\Vert{}^2$, on the training set. This fails in two common scenarios:

- High dimensionality ($D$ is large relative to $N$)
- Multicollinearity (strongly correlated features). OLS can assign a massive positive weight (e.g., $+1000$) to one feature and a massive negative weight (e.g., $-998$) to a correlated feature so they cancel each other out on the training set. On unseen test data, slight variations in those features cause wild prediction errors high variance and poor generalization.

**The Regularization Solution & Hyperparameter $\lambda$**

To stop the weights from exploding, we change the objective function to:

$\text{Total Cost} = \text{Data Fit Error } (\Vert{}y - Xw\Vert{}^2) + \lambda \cdot \text{Weight Penalty}$

Here, $\lambda \ge 0$ is a **hyperparameter** (chosen via cross-validation, not learned directly by gradient descent) that controls the bias-variance trade-off:

$\lambda = 0$**:** The penalty disappears, recovering standard Ordinary Least Squares (OLS).
$\lambda \to \infty$**:** The penalty dominates completely, forcing all feature weights $w_1, \dots, w_D$ toward 0
**Intermediate** $\lambda$**:** Balances fitting the training data with keeping the weights small and stable.

**L2 (Ridge) vs. L1 (Lasso) Penalties**

Ridge Regression ($L_2$ norm): Penalizes the sum of squared weights, $\lambda \sum_{j=1}^D w_j^2$. Because the derivative of $w_j^2$ is $2w_j$, the penalty pushes large weights down aggressively, but as a weight approaches zero, the push gets weaker. Consequently, Ridge shrinks correlated weights together smoothly toward zero, but never sets them to exactly zero. It keeps all features in the model. 

Lasso Regression ($L_1$ norm): Penalizes the sum of absolute weights, $\lambda \sum_{j=1}^D \vert{}w_j\vert{}$. Because the slope of $\vert{}w_j\vert{}$ is a constant $\pm 1$ all the way down to zero, Lasso applies a constant pull toward zero regardless of how small the weight already is. This creates sparsity: for sufficiently large $\lambda$, Lasso drives the weights of less useful or redundant features to exact zero ($w_j = 0$), acting as an automatic feature selection method.

**Ridge Regression's Closed-Form Solution**

$$
\hat{w} = (X^T X + \lambda I_0)^{-1} X^T y
$$

This form can be written as $\lambda w^T I_0 w$. We can take the gradient of $J_{\text{ridge}}(w)$ with respect to $w$, set it to $0$, and solve for $w$ directly using linear algebra just like in Ordinary Least Squares (OLS).

$$
I_0 = \begin{pmatrix} 0 & 0 & \cdots & 0 \\ 0 & 1 & \cdots & 0 \\ \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & \cdots & 1 \end{pmatrix}
$$

We can add that $(X^T X + \lambda I_0)$ is always invertible for $\lambda > 0$. 

**Why not use the training set?**

If you tried to pick $\lambda$ by minimizing the training error, you would always pick $\lambda = 0$ (OLS), because adding any penalty restricts the weights and increases the error on the training set itself.

**Ridge regression**

We fit ridge regression to the Auto MPG data and study how the weights change as λ grows, in particular for features that are strongly correlated.

```python
def ridge_fit(X, y, lam):
    '''Ridge from the closed form. X: N x D WITHOUT the column of ones. Returns w of size D+1, w[0] = intercept.'''
    Xa = add_ones(X)
    I0 = np.eye(Xa.shape[1])
    I0[0, 0] = 0
    w = np.linalg.solve(Xa.T @ Xa + lam * I0, Xa.T @ y)
    return w
sk = Ridge(alpha=10).fit(X_train_s, y_train)
assert np.allclose(ridge_fit(X_train_s, y_train, 10), np.r_[sk.intercept_, sk.coef_]), "ridge_fit does not match scikit-learn"
print("ridge_fit matches scikit-learn.")
#output
ridge_fit matches scikit-learn.
```

Before using ridge, we look at how the six features relate to each other. The table below gives the correlation between each pair of features on the training set: a value close to 1 (or to −1) means that the two features vary together and carry almost the same information.

```python
display(pd.DataFrame(np.corrcoef(X_train_s.T), index=numeric_cols, columns=numeric_cols).round(2))
```

|  | cylinders | displacement | horsepower | weight | acceleration | model_year |
| --- | --- | --- | --- | --- | --- | --- |
| cylinders | 1 | 0.95 | 0.85 | 0.9 | -0.51 | -0.36 |
| displacement | 0.95 | 1 | 0.9 | 0.93 | -0.55 | -0.38 |
| horsepower | 0.85 | 0.9 | 1 | 0.86 | -0.7 | -0.41 |
| weight | 0.9 | 0.93 | 0.86 | 1 | -0.42 | -0.3 |
| acceleration | -0.51 | -0.55 | -0.7 | -0.42 | 1 | 0.25 |
| model_year | -0.36 | -0.38 | -0.41 | -0.3 | 0.25 | 1 |

#### Evaluating a regression model

To compare models, we need a measure of how well they predict. For regression, a common measure is the coefficient of determination

$$
R^2 = 1 - \frac{\sum_{i=1}^{N}(y_i - \hat{y}_i)^2}{\sum_{i=1}^{N}(y_i - \bar{y})^2},
$$

where $\hat{y_i}$ is the prediction of the model and $\bar{y}$ the mean of the $y_i$ in the set considered. It compares the squared errors of the model with those of a model that always predicts the mean. $R^2=1$ for perfect predictions, $R^2=0$ for a model no better than the mean, and $R^2$ can be negative for a model worse than the mean. Computed on the training set, $R^2$ measures how well the model fits the data it was trained on; computed on the validation set, it estimates how well the model generalizes to new data. A training $R^2$ much higher than the validation $R^2$ is a sign of overfitting.

We now fit ridge regression for increasing values of λ. Each row of the table gives the weights $w_1,…,w_6$ , and the $R^2$on the training and on the validation set. The first row, λ=0, is ordinary least squares.

```python
rows = {}
for lam in [0, 1, 10, 50, 100, 300]:
    w = ridge_fit(X_train_s, y_train, lam)
    rows[lam] = list(w[1:]) + [r2_score(y_train, add_ones(X_train_s) @ w), r2_score(y_val, add_ones(X_val_s) @ w)]
display(pd.DataFrame(rows, index=numeric_cols + ["training R²", "validation R²"]).T.rename_axis("lambda"))
```

|  | cylinders | displacement | horsepower | weight | acceleration | model_year | training R² | validation R² |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| lambda |  |  |  |  |  |  |  |  |
| 0 | -0.361 | -0.67 | 0.412 | -5.18 | 0.0794 | 2.66 | 0.805 | 0.817 |
| 1 | -0.411 | -0.74 | 0.285 | -4.97 | 0.0218 | 2.63 | 0.805 | 0.816 |
| 10 | -0.73 | -1.07 | -0.362 | -3.85 | -0.241 | 2.43 | 0.802 | 0.811 |
| 50 | -1.15 | -1.34 | -0.94 | -2.49 | -0.284 | 2.03 | 0.789 | 0.792 |
| 100 | -1.22 | -1.35 | -1.04 | -2.02 | -0.131 | 1.75 | 0.775 | 0.775 |
| 300 | -1.11 | -1.17 | -0.998 | -1.41 | 0.172 | 1.2 | 0.719 | 0.715 |

#### **Lasso and feature selection**

We now study lasso regularization. It is particularly useful when some features do not provide valuable information. By setting their weights exactly to zero, lasso removes them from the model.

As a reminder, lasso has no closed form solution. Therefore, the function `lasso_fit` relies on scikit-learn to solve it.

```python
def lasso_fit(X, y, lam):
    '''Lasso, solved with scikit-learn. X: N x D WITHOUT the column of ones. Returns w of size D+1, w[0] = intercept.'''
    m = Lasso(alpha=lam / (2 * len(X)), max_iter=100000).fit(X, y)
    return np.r_[m.intercept_, m.coef_]
```

To study the properties of lasso, we add 20 columns of pure noise to the original six features of Auto MPG, so that the model now has 26 features, of which 20 are useless. We added those 20 fake random columns (`Z_train`) strictly as a **controlled experiment (a stress test)** to compare how Ridge and Lasso behave when junk features are mixed in with real ones. 

We compare the **coefficient paths** of ridge and lasso, i.e. the value of each coefficient as λ grows. The cell below plots such paths, showing the noise features in grey. As their coefficients are small compared with those of the real features, the top row of figures plots all the coefficients and the bottom row only the noise coefficients, on their own scale.

```python
rng = np.random.default_rng(7)
add_noise = lambda X: np.c_[X, rng.normal(size=(len(X), 20))]
Z_train, Z_val = add_noise(X_train_s), add_noise(X_val_s)

lams = np.logspace(0, 3.5, 40)
paths = {"Ridge": np.array([ridge_fit(Z_train, y_train, l)[1:] for l in lams]),
         "Lasso": np.array([lasso_fit(Z_train, y_train, l)[1:] for l in lams])}
n_real = 6    # number of real features: columns 0 to 5 are real, the 20 noise columns come after
fig, axes = plt.subplots(2, 2, figsize=(11, 7), sharex=True, sharey="row")
for k, (name, P) in enumerate(paths.items()):
    # top row: all the coefficients
    ax = axes[0, k]
    ax.plot(lams, P[:, n_real:], color="0.7", lw=0.8)
    for j in range(n_real):
        ax.plot(lams, P[:, j], lw=2, label=numeric_cols[j])
    ax.set_title(f"{name}: all coefficients"); ax.axhline(0, color="k", lw=0.5)
    # bottom row: the 20 noise coefficients only, on their own scale
    ax = axes[1, k]
    ax.plot(lams, P[:, n_real:], color="0.4", lw=0.8)
    ax.set_title(f"{name}: noise coefficients only (zoom)"); ax.axhline(0, color="k", lw=0.5)
    ax.set_xscale("log"); ax.set_xlabel("lambda")
axes[0, 0].set_ylabel("coefficient"); axes[1, 0].set_ylabel("coefficient")
axes[0, 1].legend(fontsize=8)
plt.tight_layout(); plt.show()
```

<img width="1090" height="690" alt="download 1" src="https://github.com/user-attachments/assets/f18ff7e3-ba94-484b-86a1-ae77b144518f" />

The table below summarizes, for a few values of λ, how many real and noise features each method keeps, and the validation $R^2$.

```python
rows = []
for lam in [5, 15, 50, 150, 250]:
    for name, fit in [("Ridge", ridge_fit), ("Lasso", lasso_fit)]:
        w = fit(Z_train, y_train, lam)
        rows.append({"method": name, "lambda": lam,
                     "non zero, real (of 6)": int(np.sum(w[1:7] != 0)),
                     "non zero, noise (of 20)": int(np.sum(w[7:] != 0)),
                     "largest |noise coef|": np.abs(w[7:]).max(),
                     "validation R²": r2_score(y_val, add_ones(Z_val) @ w)})
display(pd.DataFrame(rows).set_index(["method", "lambda"]))
```

|  |  | non zero, real (of 6) | non zero, noise (of 20) | largest |noise coef| | validation R² |
| --- | --- | --- | --- | --- | --- |
| method | lambda |  |  |  |  |
| Ridge | 5 | 6 | 20 | 0.397 | 0.817 |
| Lasso | 5 | 6 | 18 | 0.404 | 0.82 |
| Ridge | 15 | 6 | 20 | 0.382 | 0.81 |
| Lasso | 15 | 5 | 16 | 0.371 | 0.824 |
| Ridge | 50 | 6 | 20 | 0.378 | 0.793 |
| Lasso | 50 | 4 | 9 | 0.304 | 0.824 |
| Ridge | 150 | 6 | 20 | 0.333 | 0.757 |
| Lasso | 150 | 4 | 4 | 0.0951 | 0.819 |
| Ridge | 250 | 6 | 20 | 0.293 | 0.724 |
| Lasso | 250 | 4 | 0 | 0 | 0.814 |

I will choose λ=50 for the best Validation ($R^2$ + Parsimony): λ=15 and λ=50 tie for the highest validation $R^2$ (0.824). Between the two, λ=50 is the better choice because it achieves that peak validation score with a much simpler, sparser model cutting the active noise features nearly in half (from 16 down to 9) and shrinking the largest noise coefficient from 0.371 to 0.304.

#### **Many features: choosing λ**

We build all the polynomial features of degree 3 (Polynomial features are a feature engineering technique where you create new input columns by raising your existing features to powers (squares, cubes, etc.) and multiplying them together (interaction terms)) of the six standardized features: 83 features for 234 training cars. Ordinary least squares (OLS) overfits:

```python
poly = PolynomialFeatures(3, include_bias=False).fit(X_train_s)
P_train, P_val, P_test = [poly.transform(X) for X in (X_train_s, X_val_s, X_test_s)]
pm, ps = P_train.mean(axis=0), P_train.std(axis=0)            # standardize with the training statistics
P_train, P_val, P_test = [(P - pm) / ps for P in (P_train, P_val, P_test)]
print("Number of features:", P_train.shape[1])

w = ridge_fit(P_train, y_train, 0)
print(f"OLS, degree 3: training R² = {r2_score(y_train, add_ones(P_train) @ w):.3f}, "
      f"validation R² = {r2_score(y_val, add_ones(P_val) @ w):.3f}")
print("OLS, degree 1 (Part A): validation R² = 0.817")
#output
Number of features: 83
OLS, degree 3: training R² = 0.933, validation R² = 0.677
OLS, degree 1 (Part A): validation R² = 0.817
```

Let’s fit the model for different $\lambda$ using training, validation and test sets. 

```python
from sklearn.linear_model import Ridge, Lasso

lambdas = [0.1, 1, 3, 10, 30, 100, 300]
N = len(P_train)
rows = []

best_val_r2 = -np.inf
best_config = None

for lam in lambdas:
    # Ridge: sklearn minimizes ||y - Xw||^2 + alpha * ||w||_2^2  --> alpha = lam
    # Lasso: sklearn minimizes (1 / 2N) * ||y - Xw||^2 + alpha * ||w||_1  --> alpha = lam / (2 * N)
    models = [
        ("Ridge", Ridge(alpha=lam)),
        ("Lasso", Lasso(alpha=lam / (2 * N), max_iter=50000))
    ]
    
    for name, model in models:
        model.fit(P_train, y_train)
        train_r2 = model.score(P_train, y_train)
        val_r2 = model.score(P_val, y_val)
        n_nonzero = int(np.sum(model.coef_ != 0))
        
        rows.append({
            "method": name,
            "lambda": lam,
            "training R²": train_r2,
            "validation R²": val_r2,
            "non-zero coefs (of 83)": n_nonzero
        })
        
        if val_r2 > best_val_r2:
            best_val_r2 = val_r2
            best_config = (name, lam, model)

# Display comparison table
df_results = pd.DataFrame(rows).set_index(["method", "lambda"])
display(df_results.style.format({"training R²": "{:.3f}", "validation R²": "{:.3f}"}))

# Select the best method and lambda, and evaluate on the test set
best_name, best_lam, best_model = best_config
test_r2 = best_model.score(P_test, y_test)

print(f"\nBest model on validation set: {best_name} with lambda = {best_lam} "
      f"(validation R² = {best_val_r2:.3f}, non-zero coefs = {int(np.sum(best_model.coef_ != 0))})")
print(f"Test R² (trained on P_train): {test_r2:.3f}")

# Optional: Refit on combined (train + validation) set before evaluating on test set
P_train_val = np.vstack([P_train, P_val])
y_train_val = np.concatenate([y_train, y_val])
if best_name == "Ridge":
    refit_model = Ridge(alpha=best_lam).fit(P_train_val, y_train_val)
else:
    refit_model = Lasso(alpha=best_lam / (2 * len(P_train_val)), max_iter=50000).fit(P_train_val, y_train_val)

print(f"Test R² (refitted on train + val): {refit_model.score(P_test, y_test):.3f}")
#output
```

|  |  | training R² | validation R² | non-zero coefs (of 83) |
| --- | --- | --- | --- | --- |
| method | lambda |  |  |  |
| Ridge | 0.100000 | 0.924 | 0.834 | 83 |
| Lasso | 0.100000 | 0.930 | 0.805 | 82 |
| Ridge | 1.000000 | 0.916 | 0.875 | 83 |
| Lasso | 1.000000 | 0.922 | 0.846 | 63 |
| Ridge | 3.000000 | 0.909 | 0.890 | 83 |
| Lasso | 3.000000 | 0.915 | 0.864 | 52 |
| Ridge | 10.000000 | 0.897 | 0.905 | 83 |
| Lasso | 10.000000 | 0.902 | 0.900 | 40 |
| Ridge | 30.000000 | 0.881 | 0.907 | 83 |
| Lasso | 30.000000 | 0.883 | 0.905 | 21 |
| Ridge | 100.000000 | 0.856 | 0.890 | 83 |
| Lasso | 100.000000 | 0.869 | 0.894 | 10 |
| Ridge | 300.000000 | 0.814 | 0.852 | 83 |
| Lasso | 300.000000 | 0.824 | 0.838 | 9 |

**Analysis of results**

**The Three Regimes Across** $\lambda$

- Under-regularized / Overfitting ($\lambda = 0.1$ to $3$): Both models fit the training data very tightly ($\text{training } R^2 \approx 0.91\text{–}0.93$) because they use almost all 83 polynomial terms to memorize noise. As a result, generalization suffers ($\text{validation } R^2$ starts low at $0.805\text{–}0.834$ and climbs as $\lambda$ increases).
- Optimal Sweet Spot ($\lambda = 10$ to $30$): Penalizing large weights eliminates overfitting without losing the true non-linear curve of the Auto MPG data. Training $R^2$ drops slightly to $\approx 0.88\text{–}0.90$, while validation $R^2$ peaks at $\lambda = 30$ ($0.907$ for Ridge, $0.905$ for Lasso).
- **Over-regularized / Underfitting (**$\lambda = 100$ **to** $300$**):** The penalty becomes too strong, shrinking even the useful weights toward zero. Both training $R^2$ ($0.814\text{–}0.824$) and validation $R^2$ ($0.838\text{–}0.852$) drop together as the model becomes too rigid.

## **Part C. Evaluating a classifier**

In this part, we move from regression to binary classification problem (assigning each item to one of two classes: $+1$ or $-1$). 

#### **How logistic regression works?**

- **Compute a confidence score** $s(x) \in [0, 1]$**:** For a given car $x$, the model outputs a probability-like score. Values near $1$ mean the model strongly believes $x$ belongs to class $+1$; values near $0$ mean class $-1$.
- **Apply a threshold** $t$**:** Compare $s(x)$ against a cutoff $t$ (default $t = 0.5$):
If $s(x) > t \implies$ predict $+1$
Otherwise $\implies$ predict $-1$

#### **How to evaluate a single threshold?**

**Confusion matrix**

|  | **Predicted +1** | **Predicted −1** |
| --- | --- | --- |
| **True** $+1$ | **TP** (True Positive caught it) | **FN** (False Negative missed it) |
| **True** $-1$ | **FP** (False Positive false alarm) | **TN** (True Negative correctly ignored) |

From these four counts, we derive four metrics:

Accurancy Acc which is fraction of all predictions that were right.: 

$Acc = \frac{TP + TN}{N}$

Precision measures reliability of positive flags; hurt by **FP:** 

$Pr = \frac{TP}{TP + FP}$

Recall measures completeness: 

$TPR = \frac{TP}{TP + FN}$

Out of all actual $-1$s, how many did I falsely flag as $+1$: 

$FPR = \frac{FP}{FP + TN}$

To judge the quality of the underlying scores $s(x)$ without locking into one arbitrary threshold $t$, we sweep $t$ across every possible value:

Instead of predicting exact MPG, we turn Auto MPG into a binary classification problem using a cutoff of $30\text{ mpg}$:

- $y = +1$ **(Positive):** Efficient car ($\text{MPG} \ge 30$)
- $y = -1$ **(Negative):** Inefficient car ($\text{MPG} < 30$)

```python
yc_train = np.where(y_train >= 30, 1, -1)
yc_val = np.where(y_val >= 30, 1, -1)
print(f"Efficient cars: {np.sum(yc_train == 1)} of {len(yc_train)} (training), "
      f"{np.sum(yc_val == 1)} of {len(yc_val)} (validation)")

clf_car = LogisticRegression().fit(X_train_s, yc_train)
s_val = clf_car.predict_proba(X_val_s)[:, 1]     # scores on the validation set, between 0 and 1
#output
Efficient cars: 60 of 234 (training), 20 of 79 (validation)
```

Let’s code the function `metrics`, which evaluates a classifier at one threshold: it predicts +1 when the score is above t, and returns the four counts of the confusion matrix, the precision, the recall and the FPR. Use NumPy, one line per quantity. 

**Numpy toolkit**

np.sum(condition): it counts how many elements in a boolean array are true. 

**Metrics**

```python
def metrics(y, s, t):
    '''y: true labels in {-1, +1}; s: scores between 0 and 1; t: threshold.
    Returns TP, FN, FP, TN, precision, recall, FPR.'''
    pred = np.where(s > t, 1, -1)
    TP = int(np.sum((y == 1) & (pred == 1)))
    FN = int(np.sum((y == 1) & (pred == -1)))
    FP = int(np.sum((y == -1) & (pred == 1)))
    TN = int(np.sum((y == -1) & (pred == -1)))
    precision = TP / (TP + FP) if (TP + FP) > 0 else 0.0
    recall = TP / (TP + FN) if (TP + FN) > 0 else 0.0
    fpr = FP / (FP + TN) if (FP + TN) > 0 else 0.0
    return TP, FN, FP, TN, precision, recall, fpr
```

Let’s check our function against scikit-learn, at the default threshold t=0.5. We ask scikit-learn to put the class +1 first, as in the table above: the first row of its confusion matrix is TP, FN and the second row FP, TN.

```python
TP, FN, FP, TN, precision, recall, fpr = metrics(yc_val, s_val, 0.5)
pred_val = np.where(s_val > 0.5, 1, -1)
print(f"Your function: TP = {TP}, FN = {FN}, FP = {FP}, TN = {TN}, "
      f"precision = {precision:.3f}, recall = {recall:.3f}, FPR = {fpr:.3f}")
print("scikit-learn confusion matrix:")
print(confusion_matrix(yc_val, pred_val, labels=[1, -1]))
assert np.allclose(confusion_matrix(yc_val, pred_val, labels=[1, -1]), [[TP, FN], [FP, TN]]), "the counts do not match"
assert np.isclose(precision, precision_score(yc_val, pred_val)) and np.isclose(recall, recall_score(yc_val, pred_val)), "precision or recall do not match"
print("Your function matches scikit-learn.")
#output
Your function: TP = 19, FN = 1, FP = 3, TN = 56, precision = 0.864, recall = 0.950, FPR = 0.051
scikit-learn confusion matrix:
[[19  1]
 [ 3 56]]
Your function matches scikit-learn.
```

Let’s compute function to evaluate the same scores at five thresholds.

```python
rows = []
for t in [0.1, 0.3, 0.5, 0.7, 0.9]:
    TP, FN, FP, TN, precision, recall, fpr = metrics(yc_val, s_val, t)
    rows.append({"threshold": t, "TP": TP, "FN": FN, "FP": FP, "TN": TN,
                 "precision": precision, "recall": recall, "FPR": fpr})
display(pd.DataFrame(rows).set_index("threshold"))
```

|  | TP | FN | FP | TN | precision | recall | FPR |
| --- | --- | --- | --- | --- | --- | --- | --- |
| threshold |  |  |  |  |  |  |  |
| 0.1 | 20 | 0 | 18 | 41 | 0.526 | 1 | 0.305 |
| 0.3 | 20 | 0 | 6 | 53 | 0.769 | 1 | 0.102 |
| 0.5 | 19 | 1 | 3 | 56 | 0.864 | 0.95 | 0.0508 |
| 0.7 | 16 | 4 | 1 | 58 | 0.941 | 0.8 | 0.0169 |
| 0.9 | 6 | 14 | 0 | 59 | 1 | 0.3 | 0 |

Notice that precision increases as threshold increases while recall decreases. As $t$ increases to $0.5$, $0.7$, and $0.9$, truly efficient cars with moderate confidence scores fail to exceed the higher cutoff and shift from **TP** to **FN**: $TP$ drops ($20 \to 19 \to 16 \to 6$) while $FN$ rises ($0 \to 1 \to 4 \to 14$). Since the denominator $TP + FN = 20$ is constant, recall drops from $1.0$ down to $\frac{6}{20} = 0.3$. A higher threshold eliminates borderline positive predictions, which filters out false alarms (**FP**) much faster than true positives (**TP**).
