---
(Part 1: Supervised Machine Learning and Notebooks)

Required Reading: 🕮 Hulten, Geoff. “[Building Intelligent Systems: A Guide to Machine Learning Engineering](https://cmu.primo.exlibrisgroup.com/permalink/01CMU_INST/6lpsnm/alma991019649190004436).” (2018), Chapters 16–18, 20.

Suggested complementary reading: 🕮 Géron, Aurélien. ”[Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow](https://cmu.primo.exlibrisgroup.com/permalink/01CMU_INST/6lpsnm/alma991019662775504436)”, 2nd Edition (2019), Ch 1.
---

# Learning goals

* Understand how machine learning learns models from labeled data (basic mental model)
* Explain the steps of a typical machine learning pipeline and their responsibilities and challenges
* Understand the role of hyper-parameters
* Appropriately use vocabulary for machine learning concepts
* Apply steps of a machine-learning pipeline to build a simple model from static labeled data
* Evaluate a machine-learned classifier using cross-validation
* Explain the benefits and drawbacks of notebooks
* Demonstrate effective use of computational notebooks

# Machine Learning

> A computer program is said to learn from experience E with respect to some task T and some performance measure P, if its performance on T, as measured by P, improves with experience E. -- [Tom Mitchell, 1997](https://cmu.primo.exlibrisgroup.com/permalink/01CMU_INST/1feg4j8/alma991003098569704436)

## Defining Machine Learning (simplified)

learn a function (called model)

$f(x_1, x_2, x_3, ..., x_n) \rightarrow y$

by observing data

**Examples:**
* Detecting cancer in an image
* Transcribing an audio file
* Detecting spam
* Predicting recidivism
* Detect suspicious activity in a credit card

Typically used when writing that function manually is hard because the problem is hard or complex.

## Running Example: House Price Analysis

Given data about a house and its neighborhood, what is the likely sales price for this house?

$f(size, rooms, tax, neighborhood, ...) \rightarrow price$

## Supervised Machine Learning

Given a training dataset containing instances $(x_{1,i}, ..., x_{n,i}, y_i)$ 

which consists of features $x_1, ..., x_n$ and a corresponding outcome label $y$, 

learn a function $f(x_1, ..., x_n) \rightarrow y$ 

that "fits" the given training set and "generalizes" to other data.

## Training Data for House Price Analysis

Collect data from past sales

| Crime Rate | %Large Lots | %Industrial | Near River | # Rooms | ... | Price |
| - | - | - | - | - | - | - |
| 0.006 | 18 | 2.3|  0 | 6 | ... | 240.000 |
| 0.027 | 0  | 7.0 | 0 | 6 | ... | 216.000 |
| 0.027 | 0  | 7.0 | 0 | 7 | ... | 347.000 |
| 0.032 | 0  | 2.1 | 0 | 6 | ... | 334.000 |
| 0.069 | 0  | 2.1 | 0 | 7 | ... | 362.000 |

## Common Datastructures: Dataframes

* Tabular data, 2 dimensional
* Named columns
* Heterogeneous data: different types per column

```py 
>>> d = {'x1': [1, 2], 
         'x2': ["overcast", "sunny"], 
         'y': [True, False]}
>>> df = pd.DataFrame(data=d)
>>> df
   x1        x2      y
0   1  overcast   True
1   2     sunny  False
>>> df.dtypes
x1     int64
x2    object
y       bool
dtype: object
```

# Learning with Decision Trees

```mermaid
graph TD;
  size[Size in m^2]
  bedr[#Bedrooms]
  size -->|<=160| Age;
  size -->|>160| bedr;
  Age -->|<25| price1(($268k))
  Age -->|>=25| price2(($298k))
  bedr -->|<=3| price3(($321k))
  bedr -->|>3| price4(($411k))
```

Note: We are using decision trees as an example of a simple
and easy to understand learning algorithm. It is worth to
understand at least one learning approach in some detail, to 
get an intutive sense of the functioning and limitations of 
machine learning. Also this example will illustrate the role of
hyperparameters and how they relate to overfitting/underfitting.

## Decision Trees

| Outlook | Temperature | Humidity | Windy | Play |
| - | - | - | - | - |
| overcast | hot  |  high   |  false |    yes |
| overcast | hot  |  high   |  false |    no |
| overcast | hot  |  high   |  false |    yes |
| overcast | cool |  normal |  true |       yes |
| overcast | mild |  high   |  true|     yes |
| overcast | hot  |  normal |  false |   yes |
| rainy    | mild |  high   |  false | yes |
| rainy    | cool |  normal |  false | yes |
| rainy    | cool |  normal |  true |  no |
| rainy    | mild |  normal |  false | yes |
| rainy    | mild |  high   |  true |no |
| sunny    | hot  |  high   |  false |  no |
| sunny    | hot  |  high   |  true | no |
| sunny    | mild |  high   |false | no |
| sunny    | cool |  normal |false |   yes |
| sunny    | mild |  normal |  true |   yes |


f(Outlook, Temperature, Humidity, Windy) = 

```mermaid
graph TD;
  Outlook -->|Sunny| Windy;
  Outlook -->|Overcast| Yes((Yes));
  Outlook -->|Rainy| Humidity;
  Windy -->|true| No((No));
  Windy -->|false| No2((No));
  Humidity -->|high| No3((No));
  Humidity -->|Normal| Yes2((Yes));
```

## Building Decision Trees

* Identify all possible decisions
* Select the decision that best splits the dataset into distinct outcomes (typically via entropy or similar measure)
* Repeatedly further split subsets, until stopping criteria reached

## Overfitting with Decision Trees

```
f(Outlook, Temperature, Humidity, Windy) = 
  IF Humidity ∈ [high] 
    IF Outlook ∈ [overcast,rainy]
      IF Outlook ∈ [overcast] 
        IF Temperature ∈ [hot,cool] 
          true (0.667)
          true (1.000)
        IF Windy ∈ [FALSE] 
          true (1.000)
          false (1.000)
      false (1.000)
    IF Windy ∈ [FALSE] 
      true (1.000)
      IF Temperature ∈ [hot,cool] 
        IF Outlook ∈ [overcast] 
          true (1.000)
          false (1.000)
        true (1.000)
```

The tree perfectly fits the data, except on overcast, hot and humid days without wind, where there is not enough data to distinguish 3 outcomes.

Not obvious that this tree will generalize well.

## Underfitting with Decision Trees

```
f(Outlook, Temperature, Humidity, Windy) = 
  IF Humidity ∈ [high] 
    false (0.556)
    true (0.857)
```

If the model can only learn a single decision, it picks the
best fit, but does not have enough freedom to make good
predictions.

## Overfitting/Underfitting

**Overfitting:** Model learned exactly for the input data, but does not generalize to unseen data (e.g., exact memorization)

**Underfitting:** Model makes very general observations but poorly fits to data (e.g., brightness in picture)

Typically adjust degrees of freedom during model learning to balance between overfitting and underfitting: can better learn the training data with more freedom (more complex models); but with too much freedom, will memorize details of the training data rather than generalizing

## On Terminology

* The decisions in a model are called *model parameter* of the model 
(constants in the resulting function, weights, coefficients), their values are usually learned from the data
* The parameters to the learning algorithm that are not the
data are called *model hyperparameters*
* Degrees of freedom ~ number of model parameters

```
// max_depth and min_support are hyperparameters
def learn_decision_tree(data, max_depth, min_support): Model = 
  ...

// A, B, C are model parameters of model f
def f(outlook, temperature, humidity, windy) =
   if A==outlook
      return B*temperature + C*windy > 10
```

## Learning for House Price Analysis

| Crime Rate | %Large Lots | %Industrial | Near River | # Rooms | ... | Price |
| - | - | - | - | - | - | - |
| 0.006 | 18 | 2.3|  0 | 6 | ... | 240.000 |
| 0.027 | 0  | 7.0 | 0 | 6 | ... | 216.000 |
| 0.027 | 0  | 7.0 | 0 | 7 | ... | 347.000 |
| 0.032 | 0  | 2.1 | 0 | 6 | ... | 334.000 |
| 0.069 | 0  | 2.1 | 0 | 7 | ... | 362.000 |

## Improvements

* Averaging across multiple trees (ensemble methods, including Boosting and Random forests) to avoid overfitting
  * building different trees on different subsets of the training data or basing decisions on different subsets of features
+ Different decision selection criteria and heuristics, Gini impurity, information gain, statistical tests, etc
+ Better handling of numeric data
+ Extensions for graphs

## Summary of Learning with Decision Trees

* Learning function fitting the data
* Generalizes to different decisions (categorical and numeric data) and different outcomes (classification and regression)
* Customizable **hyperparameter** (here: max tree height, min support, ...) to control learning process
* Many decisions influence qualities: accuracy/overfitting, learning cost, model size, ...
* Resulting models easy to understand and interpret (up to a size), mirroing human decision making procedures
* Scales fairly well to large datasets

## On Specifications

No specification given for `f(outlook, temperature, humidity, windy)` 

Learning `f` from data!

We do not expect perfect predictions; no possible model could always predict all training data correctly:（同样的数据点，但是label不同）

| Outlook | Temperature | Humidity | Windy | Play |
| - | - | - | - | - |
| overcast | hot  |  high   |  false |    yes |
| overcast | hot  |  high   |  false |    no |
| overcast | hot  |  high   |  false |    yes |
| overcast | cool |  normal |  true |       yes |

We are looking for models that *generalize well*

# Evaluating Models (Supervised Learning)

## Basic Approach

Given labeled data, how well can the function predict the outcome labels?

Basic measure accuracy:

$\textit{accuracy} = \frac{\textit{correct predictions}}{\textit{all predictions}}$

```scala
def accuracy(model, xs, ys):
  count = length(xs)
  countCorrect = 0
  for i in 1..count:
    predicted = model(xs[i])
    if predicted == ys[i]:
      countCorrect += 1
  return countCorrect / count
```


----
## Separate Training and Validation Data

Always test for generalization on *unseen* validation data

Accuracy on training data (or similar measure) used during learning
to find model parameters

```scala
train_xs, train_ys, valid_xs, valid_ys = split(all_xs, all_ys)
model = learn(train_xs, train_ys)

accuracy_train = accuracy(model, train_xs, train_ys)
accuracy_valid = accuracy(model, valid_xs, valid_ys)
```

$\textit{accuracy\_train} >> \textit{accuracy\_valid}$ = sign of overfitting

## Running Example: House Price Analysis

Given data about a house and its neighborhood, what is the likely sales price for this house?

$f(size, rooms, tax, neighborhood, ...) \rightarrow price$

## Detecting Overfitting

Change hyperparameter to detect training accuracy (blue)/validation accuracy (red) at different degrees of freedom

![Overfitting example](overfitting-error.png)


Notes: Overfitting is recognizable when performance of the evaluation set decreases.

Demo: Show how trees at different depth first improve accuracy on both sets and at some point reduce validation accuracy with small improvements in training accuracy

## Crossvalidation

* Motivation
  * Evaluate accuracy on different training and validation splits
  * Evaluate with small amounts of validation data
* Method: Repeated partitioning of data into train and validation data, train and evaluate model on each partition, average results
* Many split strategies, including 
  * leave-one-out: evaluate on each datapoint using all other data for training
  * k-fold: $k$ equal-sized partitions, evaluate on each training on others
  * repeated random sub-sampling (Monte Carlo)

![Visualization of K-Fold Crossvalidation](kfold.gif)

## Separate Training, Validation and Test Data

Often a model is "tuned" manually or automatically on a validation set (hyperparameter optimization)

In this case, we can overfit on the validation set, separate test set is needed for final evaluation


```scala
train_xs, train_ys, valid_xs, valid_ys, test_xs, test_ys = 
            split(all_xs, all_ys)

best_model = null
best_model_accuracy = 0
for (hyperparameters in candidate_hyperparameters) 
  candidate_model = learn(train_xs, train_ys, hyperparameter)
  model_accuracy = accuracy(model, valid_xs, valid_ys)  
  if (model_accuracy > best_model_accuracy) 
    best_model = candidate_model
    best_model_accuracy = model_accuracy

accuracy_test = accuracy(model, test_xs, test_ys)
```

## Academic Escalation: Overfitting on Benchmarks

[![Overfitting Benchmarks](overfitting-benchmarks.png)](overfitting-benchmarks.png)

Note: If many researchers publish best results on the same benchmark, collectively they perform "hyperparameter optimization" on the test set

# Machine Learning Pipeline

![Pipeline](pipeline.png)

Graphic: Amershi, Saleema, Andrew Begel, Christian Bird, Robert DeLine, Harald Gall, Ece Kamar, Nachiappan Nagappan, Besmira Nushi, and Thomas Zimmermann. "[Software engineering for machine learning: A case study](https://www.microsoft.com/en-us/research/uploads/prod/2019/03/amershi-icse-2019_Software_Engineering_for_Machine_Learning.pdf)." In 2019 IEEE/ACM 41st International Conference on Software Engineering: Software Engineering in Practice (ICSE-SEIP), pp. 291-300. IEEE, 2019.

Notes: An average data scientist spends most of their time collecting and cleaning data.

## Pipeline Steps

* Data collection: identify training data, often many sources
* Data cleaning: remove wrong data, outliers, merge data from multiple sources
* Data labeling: identify labels ($Y$) on training data, possibly croudsourced or (semi-)automated
* Feature engineering: convert raw data into a form suitable for learning, identifying features, encoding, normalizing
* Model training: build the model, tune hyperparameters
* Model evaluation: determine fitness for purpose
* Data science education focuses on feature engineering and model training
* Data science practitioners spend substantial time collecting and cleaning data
* Requirements, deployment, and monitoring rarely a focus in data science education

## Common Challenges

* Insufficient Quantity of Training Data
* Nonrepresentative Training Data
* Poor Data Quality
* Irrelevant Features
* Overfitting, Underfitting
* Data + Model Match

Suggested complementary reading: 🕮 Géron, Aurélien. ”[Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow](https://cmu.primo.exlibrisgroup.com/permalink/01CMU_INST/6lpsnm/alma991019662775504436)”, 2nd Edition (2019), Ch 1.

# Iteration and Exploration with Computational Notebooks

## Data Science is Iterative and Exploratory

![Data Science Workflow](data-science-process.jpg)

(Source: Guo. "[Data Science Workflow: Overview and Challenges](https://cacm.acm.org/blogs/blog-cacm/169199-data-science-workflow-overview-and-challenges/fulltext)." Blog@CACM, Oct 2013)

## Data Science is Iterative and Exploratory

[![Data Science Lifecycle](https://docs.microsoft.com/en-us/azure/machine-learning/team-data-science-process/media/overview/tdsp-lifecycle2.png)](https://docs.microsoft.com/en-us/azure/machine-learning/team-data-science-process/media/overview/tdsp-lifecycle2.png)

(Microsoft Azure Team, "[What is the Team Data Science Process?](https://docs.microsoft.com/en-us/azure/machine-learning/team-data-science-process/overview)" Microsoft Documentation, Jan 2020)

## Similar to Spiral Process Model or Agile?

并不相似，比如Agile需要不断的迭代，每一次迭代都需要和客户交流。但是build model不一样，每一次迭代只需要靠自己的经验和智慧，把accuracy做到更好更好。

![Spiral Model](spiral_model.svg)
![Scrum Process](scrum.svg)

Note: There is similarity in that there is an iterative process, but the idea is different and the process model seems mostly orthogonal to iteration in data science. The spiral model prioritizes risk, especially when it is not clear whether a model is feasible. One can do similar things in model development, seeing whether it is feasible with data at hand at all and build an early prototype, but it is not clear that an initial okay model can be improved incrementally into a great one later. Agile can work with vague and changing requirements, but that again seems to be a rather orthogonal concern. Requirements on the product are not so much unclear or changing (the goal is often clear), but it's not clear whether and how a model can solve it.

## Data Science is Iterative and Exploratory

[![Experimental results showing incremental accuracy improvement](accuracy-improvements.png)](accuracy-improvements.png)


Source: Patel, Kayur, James Fogarty, James A. Landay, and Beverly Harrison. "[Investigating statistical machine learning as a tool for software development](http://www.kayur.org/papers/chi2008.pdf)." In Proc. CHI, 2008.

Notes:
This figure shows the result from a controlled experiment in which participants had 2 sessions of 2h each to build a model. Whenever the participants evaluated a model in the process, the accuracy is recorded. These plots show the accuracy improvements over time, showing how data scientists make incremental improvements through frequent iteration.

## Data Science is Iterative and Exploratory
* Science mindset: start with rough goal, no clear specification, unclear whether possible
* Heuristics and experience to guide the process
* Try and error, refine iteratively, hypothesis testing
* Go back to data collection and cleaning if needed, revise goals

## Computational Notebooks

* Origins in "literal programming", interleaving text and code, treating programs as literature (Knuth'84)
* First notebook in Wolfram Mathematica 1.0 in 1988
* Document with text and code cells, showing execution results under cells
* Code of cells is executed, per cell, in a kernel
* Many notebook implementations and supported languages, Python + Jupyter currently most popular

![Notebook example](notebook-example.png)

Notes:
* See also https://en.wikipedia.org/wiki/Literate_programming
* Demo with public notebook, e.g., https://colab.research.google.com/notebooks/mlcc/intro_to_pandas.ipynb

## Notebooks Support Iteration and Exploration

* Quick feedback, similar to REPL
* Visual feedback including figures and tables
* Incremental computation: reexecuting individual cells
* Quick and easy: copy paste, no abstraction needed
* Easy to share: document includes text, code, and results

## Brief Discussion: Notebook Limitations and Drawbacks?

不方便调试，大工程需要抽象，很难转变成大工程。

只能用于解释型语言，编译型语言必须一次性编译完所有的源码。（当然其实scala也可以）

都是全局状态。

没有抽象，封装的概念。

总的来说就是一个冗长的超大型程序。

没有人去测试notebook里面的代码。

如果要写long time maintainable code，就不能使用notebook了，但是notebook确实支持explore。

# Summary

* Key concepts in machine learning: dataframes, model, train/validation/test data
* A simple machine-learning algorithm: Decision trees
* Overfitting, underfitting, hyperparameter tuning
* Basic model accuracy measures and crossvalidation
* Steps of a machine learning pipeline
* Introduction to working with computational notebooks, typical iterative workflow, benefits and limitations of notebooks