---
title: "Winning with Simple, even Linear, Models - PyData London 2018"
date: 2020-08-29
tags: [machine-learning, scikit-learn, linear-model]
categories:
    - pydata
    - 2018
    - london
header:
    teaser: "/images/2020-08-27-constrain-artificial-stupidity/header.png"
excerpt: "Simpler is better"
mathjax: "true"
---

The following is my takeaways from the PyData London 2018 talk given by [Vincent Warmerdam: Winning with Simple, even Linear, Models \| PyData London 2018](https://youtu.be/68ABAU_V8qI).

## Takeaway #1: Feature Engineering

Linear models can be applied to non-linear data by simple feature engineering tricks, namely adding interaction variables. Therefore, there is no reason to quickly conclude to non-linear complicated models such as Random forests or Support Vector Machines. Some benefits of simpler linear models:

1. Interpretable
2. More regularisation tricks
3. Easier to maintain in production and handover to data engineers that can continue monitoring it and fix it if and when issues are discovered

## Takeaway #2: Timeseries prediction

Linear models can be used to perform time series prediction. Typically time series data is non-linear in nature due to seasonality. However another feature engineering trick, namely Radial Basis functions (RBF).

**What are Radial functions?** They are a special class of functions with a unique property that their responses increase (or decrease) monotonically with distance from a central point. The center itself, the scale to measure the distance, and the precise shape of the radial function are all parameters of the model. A quite familiar example of RBFs is the Gaussian function [1]. Atypical Gaussian function looks like this:

$$h(x) = \exp{(-frac{(x-c)^{2}}{r^{2}})}$$

where for a scalar input $$x$$, the distance is centered around the mean $$c$$ and the scale is in units of the variance term $$r^{2}$$. The shape of the Gaussian function can be also elliptical in higher dimensions, e.g., for 2D vector input, we can have $$h(x, y)$$  such that the mean and variance are also vectors of same size as the input. See [here](https://en.wikipedia.org/wiki/Gaussian_function#Two-dimensional_Gaussian_function).



## Additional resources

1. [Gaussian function | Wikipedia](https://en.wikipedia.org/wiki/Gaussian_function)
2. [Introduction to
Radial Basis Function Networks](https://www.cc.gatech.edu/~isbell/tutorials/rbf-intro.pdf)
