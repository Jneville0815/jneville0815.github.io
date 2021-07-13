---
layout: post
title: Linear Regression
date: 2020-04-07
tags: [machine learning]
---

Linear regression is one of the most well-known machine learning algorithms. Most people interested in machine learning will study this algorithm first. In this tutorial we're going to take a look at the math involved in the algorithm while also learning some basic machine learning terminology.

I'm writing this tutorial after completing the linear regression portion of <a href="https://www.coursera.org/learn/machine-learning" class="in-text-link">Andrew Ng's Machine Learning Course</a>

## Unsupervised Learning

Machine learning algorithms are classified as either unsupervised learning algorithms, supervised learning algorithms or reinforcement learning algorithms. Linear regression is an unsupervised learning algorithm which means that the output of the model is known in advance. An example of a dataset that can be used to create an unsupervised learning algorithm is a dataset that contains a list of houses with the size of each house and the price of each house. We know in advance that given the size of a house we want to predict the price. Supervised learning algorithms and reinforcement learning algorithms will be explained in a later tutorial. Below we have an example housing dataset with 5 entries:

<figure>
<img src="/linear-regression/image1.png" alt="House Size/Price">
<figcaption>Fig 1. House Size/Price</figcaption>
</figure>

## Regression Problem

Unsupervised learning algorithms are broken down into regression problems and classification problems. Regression problems have continuous valued output, and classification problems have discrete valued output. Examples of continuous valued output are prices and temperatures, and examples of discrete valued output are true/false values and anything else that can be split up into individual categories.

## Visualizing Our Data

To get a better understanding of the dataset we're working with it helps to visualize the data. Note that sometimes you will hear input values called independent variables, and the output value called a dependent variable. For this example we only have one independent variable. If you want to have several independent variables there are a few modifications that need to take place, but we will not be covering multivariate linear regression in this tutorial. Here we plot the price on the y-axis and the size on the x-axis.

<figure>
<img src="/linear-regression/image2.png" alt="Scatter Plot">
<figcaption>Fig 2. Scatter Plot</figcaption>
</figure>

Our goal with linear regression is to find a regression line that best fits our data. In the next plot you can see a green regression line and a red regression line; the green line fits our data best because the cost function is minimized when using the green line.

<figure>
<img src="/linear-regression/image3.png" alt="Regression Lines">
<figcaption>Fig 3. Regression Lines</figcaption>
</figure>

## The Cost Function

