---
layout: post
title: Linear Regression
date: 2020-04-07
tags: [machine-learning]
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

Our linear regression function is defined as $$h_\theta(x)$$ where 

<h4 style="text-align: center;">$$h_\theta(x) = \theta_0 + \theta_1x$$</h4>

$$\theta_0$$ is the y-intercept, and $$\theta_1$$ is the slope of the line. As a side note, another name for our linear regression function is "hypothesis." Another side note - when you see anything to the i'th power in this tutorial, the notation is not to be interpreted as an exponent; it's referring to the current iteration.

The cost function is defined as 

<h4 style="text-align: center;">$$J(\theta_0, \theta_1) = \frac{1}{2m} \sum_{i=0}^{m-1} (h_\theta(x^i) - y^i)^2$$</h4>

where $$m = $$ the number of data points (which is 5 in our ongoing example). We want to find values of theta that will minimize the cost function. Before calculating $$J(\theta_0, \theta_1$$) for our red line, we need to visualize and compute the values for $$h_\theta(x^i$$). Our values for $$\theta_0$$ and $$\theta_1$$ are $$-50$$ and $$77$$, respectively. Don't worry about how we got these values; in the next part of this tutorial we'll see how these values are found using gradient descent. We'll now look at the values located on the red line that are on the same vertical line as our data points. Each black point located on the red line in the plot below will be a value of $$h_\theta(x^i)$$ when we compute the cost function. The black double-sided arrows in the plot below are the lengths that we're summing the squares of when we compute the cost function.

<figure>
<img src="/linear-regression/image4.png" alt="Sum of Squares">
<figcaption>Fig 4. Sum of Squares</figcaption>
</figure>

These will be the values for $$h_\theta(x^i)$$ when we compute the cost function: 

<h4 style="text-align: center;">$$h_\theta(x^0) = -50 + 77(1250) = 96,200$$</h4>
<h4 style="text-align: center;">$$h_\theta(x^1) = -50 + 77(1300) = 100,050$$</h4>
<h4 style="text-align: center;">$$h_\theta(x^2) = -50 + 77(1500) = 115,450$$</h4>
<h4 style="text-align: center;">$$h_\theta(x^3) = -50 + 77(1750) = 134,700$$</h4>
<h4 style="text-align: center;">$$h_\theta(x^4) = -50 + 77(2000) = 153,950$$</h4>

Plugging these values into the cost function we get: 

<h4 style="text-align: center;">$$J(-50, 77) = \frac{1}{10} * $$</h4>
<h4 style="text-align: center;">$$((96,200 - 90,000)^2 + $$</h4>
<h4 style="text-align: center;">$$(100,050 - 95,000)^2 + $$</h4>
<h4 style="text-align: center;">$$(115,450 - 100,000)^2 + $$</h4>
<h4 style="text-align: center;">$$(134,700 - 110,000)^2 + $$</h4>
<h4 style="text-align: center;">$$(153,950 - 150,000)^2)$$</h4>

which simplifies to

<h4 style="text-align: center;">$$J(-50, 77) = \frac{1}{10} * $$</h4>
<h4 style="text-align: center;">$$((6,200)^2 + $$</h4>
<h4 style="text-align: center;">$$(5,050)^2 + $$</h4>
<h4 style="text-align: center;">$$(15,450)^2 + $$</h4>
<h4 style="text-align: center;">$$(24,700)^2 + $$</h4>
<h4 style="text-align: center;">$$(3,950)^2)$$</h4>

and for our final answer we get

<h4 style="text-align: center;">$$J(-50, 77) = 92,833,750$$</h4>

If we were to repeat the same process for the green line, our final answer would be

<h4 style="text-align: center;">$$J(-2204.03, 71.28) = 30,264,510.62$$</h4>

$$30,264,510.62$$ is the absolute lowest value we can get from the cost function. In other words, any combination of $$\theta_0$$ and $$\theta_1$$ values that we plug into the equation will not give us a value less than $$30,264,510.62$$. Now let's take a look at how we find our $$\theta_0$$ and $$\theta_1$$ values using gradient descent.

## Gradient Descent

The general algorithm for gradient descent is defined as 

<h4 style="text-align: center;">$$\theta_j := \theta_j - \alpha \frac{\partial}{\partial\theta_j} J(\theta_0, \theta_1)$$</h4>

Note that the "$$:=$$" operator is the assignment operator, and $$\alpha$$ is a constant called the learning rate. The learning rate controls how large each step of gradient descent will be; if it's too big gradient descent will fail to converge, and if it's too small gradient descent will be inefficient. Also note that $$\frac{\partial}{\partial\theta_j} J(\theta_0, \theta_1)$$ represents taking the partial derivative of the cost function. I will not go over the calculus involved in taking a partial derivative in this tutorial, but there are many <a href="https://www.youtube.com/results?search_query=partial+derivatives" class="in-text-link">videos</a> on the subject if you're interested.

In this equation, $$j = 0,1$$ represents the feature index number, and this algorithm will be repeated until convergence.

Basically what happens here is the values of $$\theta_0$$ and $$\theta_1$$ keep getting updated until the cost function is minimized. This algorithm can be used for many machine learning applications, but in this tutorial we'll look at how it works with linear regression.

After taking the partial derivative of $$\theta_0$$ and $$\theta_1$$, the gradient descent algorithm for linear regression is defined as 

<h4 style="text-align: center;">$$\theta_0 := \theta_0 - \alpha \frac{1}{m} \sum_{i=0}^{m-1} (h_\theta(x^i) - y^i)$$</h4>

<h4 style="text-align: center;">$$\theta_1 := \theta_1 - \alpha \frac{1}{m} \sum_{i=0}^{m-1} (h_\theta(x^i) - y^i) * x^i$$</h4>

This algorithm will also be repeated until convergence. We can choose any $$\theta_0$$ and $$\theta_1$$ to start with, and after many iterations we'll arrive at a minimum.

## Summary

We start by picking any two numbers for $$\theta_0$$ and $$\theta_1$$. You can just assign both variables the value $$0$$. You then perform gradient descent to find values of $$\theta_0$$ and $$\theta_1$$ that minimize the cost function. After finding the values you'll have a linear regression line that best fits the data. For our example above, our best fitting linear regression line was defined as 

<h4 style="text-align: center;">$$h_\theta(x) = -2204.03 + 71.28(x)$$</h4>

You can now plug any potential value in for $$x$$ (size) to find the estimated price. If we want to know the estimated price for a house that's $$3000$$ square feet we just plug $$3000$$ in for $$x$$

<h4 style="text-align: center;">$$h_\theta(3000) = -2204.03 + 71.28(3000) = 211,635.97$$</h4>

## Conclusion

Many high level languages provide libraries that have functions to do the math needed for linear regression (and many other algorithms). Still, I believe it's important to know what's going on inside of the functions. Who wants to plug numbers into a function and have absolutely no idea how the output is generated? It helps to have a basic understanding of the math involved, and I hope that this tutorial was able to provide you with that.

Check out the <a href="" class="in-text-link">next tutorial</a> on Logistic Regression.