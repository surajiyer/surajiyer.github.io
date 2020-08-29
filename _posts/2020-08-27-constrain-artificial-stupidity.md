---
title: "How to Constrain Artificial Stupidity - PyData London 2019"
date: 2020-08-27
tags: [machine-learning, scikit-learn, gaussian-mixture-modeling]
categories:
    - pydata
    - 2019
    - london
header:
    teaser: "/images/2020-08-27-constrain-artificial-stupidity/teaser.png"
excerpt: "Predict better by predicting less"
mathjax: "true"
---

This is my first blog post. The quality may not be optimal but I will get better at it one post at a time. The following is my takeaways from the PyData London 2019 talk given by [Vincent Warmerdam: How to Constrain Artificial Stupidity \| PyData London 2019](https://www.youtube.com/watch?v=Z8MEFI7ZJlA).

## Takeaway #1: Predict better by predicting less

ML algorithms are like lambda functions, input goes in and we get an output. We can, and more importantly should make ML "functions" more robust just like with any regular function by verifying the input. We can do this by checking if the input is an outlier in comparison to some decision boundary or distribution of data on which the model was trained. If the input is flagged as outlier, fall back to some default output instead of running the ML algorithm.

One method is to apply Gaussian mixture model (GMM) to the training data with the number of components set to the number of classes. The model can provide a probability score of a sample belonging to the learned distribution. Taking the 1<sup>th</sup> percentile of all predicted probabilities gives a decision boundary.

```python
n_classes = 3
out = GaussianMixture(n_classes).fit(X)
boundary = np.quantile(out.score_samples(X), .01)
```

For all data points within the boundary, we can be certain of model's ability to predict in that region. For anything outside, we can fall back to a default output. This method can work on neural networks also by applying it to the output of any hidden layers in the network. Additionally, if input is not an outlier, we pass it through the model, get a probability score and if it is not confident or less than specific threshold, then we also fall back to the same / another default output. This ensures that we do not consume erroneous predictions if the model is not confident.

## Takeaway #2: Model predictions are not actual probability

Model predicted probability scores are not the actual probability of the target label. They are only an approximate proxy. Keeping that in mind, we should always test for scenarios where our model might fail and account for those cases.

## Takeaway #3: Constrain features by measuring fairness

How fair is a prediction made a model can also be used to predict less? To check this, we need to first define groups in our dataset based on values of features used by our model corresponding to real-world groups that we do not want to discriminate against. For example, group by skin color, or age or gender or income etc. assuming these are features the model already uses and bootstrap sample from each group. Once we have defined the groups, we can calculate the expected difference in probability score between these groups made by our trained model to measure (un)fairness.

$$E[\hat{y}_{A}-\hat{y}_{B}] \approx 0\tag{1}\label{eq1}$$
<!-- <p style="text-align: center;"><img src="https://render.githubusercontent.com/render/math?math=E[\hat{y}_{A}-\hat{y}_{B}] \thickapprox 0"></p> -->

If what group you are in does not make much difference, the expected difference should be centered at zero which is what we want to achieve. This is *a* measure for fairness.

Now that we have a metric $$\eqref{eq1}$$, we can start dropping features which least lower accuracy but maximizes gain in fairness. This process will have to be repeated a few times even after removing the offending features because those features may be correlated with other features in the dataset.

The following is the MSE and fairness of an accurate model built on the Boston housing prices dataset which comes from the 70s and carries the biases in it from that time.

![Base: Highly accurate but unfair model]({{ site.url }}{{ site.baseurl }}/images/2020-08-27-constrain-artificial-stupidity/accurate_but_unfair_model.png)

After dropping two columns that introduce unfairness, the MSE is relatively the same but fairness proxy (orange) is moving closer to 0.

![After dropping two columns]({{ site.url }}{{ site.baseurl }}/images/2020-08-27-constrain-artificial-stupidity/after_dropping_two.png)

After dropping all unfair columns, the fairness proxy is nearly centered around 0, however we also see the MSE increase a lot. The unfortunate reality is that by this method, fairness comes at the cost of losing information but it has to be done to remove any real-world biases.

![After dropping all unfair columns]({{ site.url }}{{ site.baseurl }}/images/2020-08-27-constrain-artificial-stupidity/inaccurate_but_fair_model.png)

There is however a trick that can be applied in production to take the best of both the accurate as well as fair model. When we get an input to make a prediction for, we can predict using both models. Then we can check if the difference in prediction is greater than some set threshold, and consequently fall back to a default output that may suggest a human being to intervene.

There are two issues with measuring fairness however. Typically fairness is in the eyes of the beholder. Therefore this places a lot of responsibility on the Data scientist to be ethical and to model the data fairly and safely. The second issue is that, to measure fairness, we need to collect data which we used to group the dataset earlier like skin color or income in the first place. Therefore, **fairness can come at the cost of data privacy**. Whether or not this is acceptable must be a decision made by the end consumer of the prediction, i.e., whoever is affected by it. Simply not using these features in your model is not the same as being fair because as we mentioned earlier, correlated features that carry this bias can still exist. This also does not give accountability to companies for unfairness in their models.

## Takeaway #4: Consider models that constrain

Consider the following LP model equation:

$$\begin{equation}\begin{aligned}
\text{maximize  } & profit(x) &= \sum_{i=0}^{n} w_{i}x_{i} \\
\text{s.t.  } & \sum_{i}x_{i} &\le \text{budget} \\
\text{  } & risk(x) &\le \text{preference}
\end{aligned}\end{equation}\tag{2}\label{eq2}$$

Equation $$\eqref{eq2}$$ is an example of models that already exists and can add constraints to the equation, such as fairness constraints. Typically, regular ML models simply perform the maximization part without any constraints.

There are many constraints that can be added but might be numerically difficult to compute. But three of them as described in the video that are easy to implement are as follows:

1. Monotonicity
2. Fairness
3. Label guarantees

**Monotonicity** refers to the guarantee that certain learned relationships between a feature and the target must always be monotonically increasing or decreasing. This can be derived from our own domain knowledge. An example is that we know increased smoking must always predict worse health. Without such a constraint, there may be outliers in our data, which triggers the model to predict the opposite condition which should never be the case. Monotonicity can be applied using **scikit-learn**'s very own implementation of the `IsotonicRegression` model. Tensorflow provides similar ability to constrain representations using TF Lattices [2]. Similarly, monotonicity comes with XGBoost out of the box.

**Fairness** is what we have discuss earlier but is implemented by the author et al. in the scikit-lego [3] package.

**Label guarantees** refers to ensuring correct prediction in specific cases (samples in our dataset). This opens possibilities to personalizing model predictions.

## Takeaway #5: Bayesian models are more articulate

Consider libraries like PyMC3 [4] which allows specifying the data generating process in a simple syntax when we already have domain knowledge of the process to better model it instead of throwing it into any random model from sklearn. It may provide:

1. Improved explainability
2. More options to extrapolate from data with "What if" questions
3. Hypothesis testing

## Takeaway #6: Be Human!

Models are always bound to go wrong. It is important that we use our natural intelligence to reason about its behavior and fall back to safer scenarios before this happens.

## Acknowledgement

I want to thank the author **Vincent Warmerdam** for this resourceful talk. The goal of this article is to summarize my learnings from this talk and not to take away from the achievement of the author. I highly recommend watching the original talk and some of the additional resources linked below.

## Additional resources

1. [Vincent Warmerdam: Winning with Simple, even Linear, Models \| PyData London 2018](https://www.youtube.com/watch?v=68ABAU_V8qI)
2. [TF Lattice](https://youtu.be/ABBnNjbjv2Q)
3. [Fairness \| scikit-lego](https://scikit-lego.readthedocs.io/en/latest/fairness.html#Measuring-fairness-for-Regression)
4. [PyMC3](https://docs.pymc.io/)
