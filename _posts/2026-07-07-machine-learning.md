---
title: "Introduction to Machine Learning"
date: 2026-07-07 15:06:00 +0200
categories: [Machine Learning]
tags: [ML, Deep Learning, Decision Trees, Random Forest, Pandas, Scikit-Learn]
math: true
---
# Machine Learning

The foundation of AI comes from Machine learning and Deep Learning. 

Deep Learning $\rightarrow$ Machine Learning $\rightarrow$ AI

A deep understanding of ML (Machine Learning) is required to become a good AI engineer. This blog will explore my first steps in ML, sharing my understanding of this subject in the simplest way as possible. 

## How models work? [1]

The principle of models is based on the idea that by using previous data, it is possible to predict the future. The first step is capturing patterns from the data. This step is called fitting or training the model. One of the simplest models is the decision tree, which is easy enough to get started with, but complex enough to be a basic building block for some of the best models in data science.

### Decision Tree [1]

<img width="454" height="281" alt="image" src="https://github.com/user-attachments/assets/1061aea8-0c51-4a96-9b3f-cdf7095799cc" />

The primary problem of this model is that it is not complex enough to precisely predict the future price of the house. Capturing more factors leads to a tree with more splits: a "deeper" tree. Picking the path corresponding to a specific house's characteristics will lead directly to the predicted price of the house at the bottom of the tree (the leaf). An important point I learned is that the predicted price is driven by the data itself; the more accurate the data is, the more correct the predicted price will be.

### Basic Data exploration [1]

#### Pandas

Using pandas to get familiar with data. Pandas is a Python library that lets you explore and manipulate data.

```python
import pandas as pd
```

The most important part of the pandas library is the DataFrame, which is simply a table in Python like a table in SQL.

#### How to read a CSV file using pandas?

A CSV file is the most commonly used file in AI, ML, etc., because it is completely simple.

```markdown
name, age, city
Gabriel, 22, Sophia-Antipolis
Cristiano, 41, Madrid
#it's a table where
#columns are separeted by comma and rows by line
```

A CSV file is universal, all languages can decode it, and the file is light.

```python
import pandas as pd
df = pd.read_csv("data.csv")
```

The file allows very simple data manipulation and data exportation.

```python
df.describe()
```

Output: 

| name | age | city |
| --- | --- | --- |
| Gabriel | 22 | Sophia-Antipolis |
| Cristiano | 41 | Madrid |

```python
df.columns
#output: 
Index(['name', 'age', 'city'],
      dtype='object')
```

If the df has some missing values, you can use:

```python
df = df.dropna(axis=0)
# dropna drops missing values (na: "not available"), it's one of the solution but not the only one
# axis=0: drop rows
# axis=1: drop columns
```

The two principal ways to select a subset of data: dot notation and selecting with a column list. The dot notation is used to select the predicted column, which is called the predicted target $y$.

```python
y = df.name #for example
```

Columns inputting in the model are called features, $X$ by convention.

```python
df_features = ['name', 'age']
X = df[df_features]
X.head() #it's used to shows the 5 first rows of the table, in our example case, it will show the entire table
```

**Scikit-learn** library is used to train, manipulate, and create ML models.

```python
from sklearn.tree import DecisionTreeRegressor
# Define model. 
df_model = DecisionTreeRegressor(random_state=1)
#random_state=1 is set to fix the random seed. It will make test reproductible, 
#thus it means that the result will be the same every time the code will run. 
#Then, it will possible to compare different models in the same dataset. 
#A random seed is a number that determines how a computer generates "random" values. 
#If you start the algorithm with the same seed, you get the same sequence of “random” numbers.

# Fit model
df_model.fit(X, y)
#It’s the step where the algorithm adjusts itself to minimize error and learns to:
#find the best parameter, train the model on data, learn patterns
```

#### How to build a model? [1]

They are few steps to follow:

1. Define: What type of model will it be?
2. Fit: Capture patterns from provided data.
3. Predict
4. Evaluate: Determine how accurate the model's predictions are.

To make predictions based on pattern data, our example is not predictable, but imagine instead of name, etc., we have the characteristics of cars depending on their km, model, etc. Based on fewer data, we can predict, for example, the price of cars with:

```python
print("Making predictions for the following 5 cars:")
print(X.head())
print("The predictions are")
print(cars_model.predict(X.head()))
```

### Model validation [1]

Model validation is used to measure the quality of a model. The relevant measure of model quality is predictive accuracy. The model quality must be converted in an understandable way. One of the metrics used to summarize model quality is MAE: mean absolute error.

#### MAE

With the MAE metric, we take the absolute value of each error. This converts each error to a positive number. We then take the average of those absolute errors. This is our measure of model quality.

```python
from sklearn.metrics import mean_absolute_error
predicted_car_prices = car_model.predict(X)
mean_absolute_error(y, predited_car_prices)
#output: in-sample score 
```

#### The problem with “In Sample” scores

The measure can be called an "in-sample" score. A possible error to avoid here is using a single sample for both building the model and evaluating it. Imagine that, in the large real estate market, black rims are unrelated to car price. However, in the sample of your data, all cars with dark rims were very expensive. The model’s job is to find patterns that predict car prices. Then, it will see this pattern and always predict high prices for cars with dark rims. The model will look very accurate during training but would be very inaccurate in practice.

One solution is using the train_test_split function.

```python
from sklearn.model_selection import train_test_split

#the function randomly selects which rows go into training and which go into validation.
# For both features and target the split is based on a random number generator. Supplying a numeric value to
# the random_state argument guarantees we get the same split every time we run this script.
train_X, val_X, train_y, val_y = train_test_split(X, y, random_state = 0)
# Define model
cars_model = DecisionTreeRegressor(random_state=0)
# Fit model
cars_model.fit(train_X, train_y)

# get predicted prices on validation data
val_predictions = cars_model.predict(val_X)
print(mean_absolute_error(val_y, val_predictions))
```

### **Underfitting and Overfitting [1]**

Underfitting happens when the model is too simple to capture the structure of the data. It can be caused by a model too simple, too much regularization, too few features, or too little training time.

Overfitting happens when the model learns too much, including noise and irrelevant details. It can be caused by a model too complex, too many parameters, too much training, not enough data, no regularization, or data leakage.

One of the key functions in the sklearn library is to determine the tree depth (list/max leaf). The max provides a very sensible way to control overfitting and underfitting. It’s possible to use a utility function to help compare MAE scores from different values for max_leaf_nodes:

```python
from sklearn.metrics import mean_absolute_error
from sklearn.tree import DecisionTreeRegressor

def get_mae(max_leaf_nodes, train_X, val_X, train_y, val_y):
    model = DecisionTreeRegressor(max_leaf_nodes=max_leaf_nodes, random_state=0)
    model.fit(train_X, train_y)
    preds_val = model.predict(val_X)
    mae = mean_absolute_error(val_y, preds_val)
    return(mae)
```

We can use a for-loop to compare the accuracy of models built with different values for max_leaf_nodes.

```python
# compare MAE with differing values of max_leaf_nodes
for max_leaf_nodes in [5, 50, 500, 5000]:
    my_mae = get_mae(max_leaf_nodes, train_X, val_X, train_y, val_y)
    print("Max leaf nodes: %d  \t\t Mean Absolute Error:  %d" %(max_leaf_nodes, my_mae))
```

The best model is the one with the lowest MAE, because it measures how far predictions are from the true values, on average. We use validation data, which isn't used in model training, to measure a candidate model's accuracy. Validation data is the part of the dataset used to evaluate and tune the model during training.

### Random Forest [1]

It looks very difficult to play with underfitting and overfitting parameters. The random forest uses many trees, and it makes a prediction by averaging the predictions of each component tree. It generally has much better predictive accuracy than a single decision tree and works well with default parameters.

```python
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error

forest_model = RandomForestRegressor(random_state=1)
forest_model.fit(train_X, train_y)
cars_preds = forest_model.predict(val_X)
print(mean_absolute_error(val_y, cars_preds))
```

## Sources

1. [https://www.youtube.com/watch?v=MFSFcPsMsuE](https://www.youtube.com/watch?v=MFSFcPsMsuE)
2. https://youtube.com/playlist?list=PLoROMvodv4rMiGQp3WXShtMGgzqpfVfbU&si=ii32GeBVI4rXx6rP
3. [https://www.kaggle.com/learn](https://www.kaggle.com/learn)
4. [https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeRegressor.html](https://scikit-learn.org/stable/modules/generated/sklearn.tree.DecisionTreeRegressor.html)
