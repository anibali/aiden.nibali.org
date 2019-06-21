+++
title = "Dealing with imbalanced classes"
description = """
  In the real world, it is not unusual to encounter datasets which are have
  imbalanced classes. This post discusses some strategies for dealing with
  such situations."""
date = "2019-06-20T15:14:22+10:00"
categories = ["deep learning"]
tags = []
comments = false
draft = false
showpagemeta = true
slug = ""
+++

Let's say that we have some classification dataset where items can be
categorised as being in one of $N$ classes: $C_1, C_2, \ldots, C_N$.
Each example in the dataset has a vector of features,
$\boldsymbol{x}$, and a class, $C_k$.
For many public datasets, the examples are balanced amongst classes such that
$\Pr(C_1) = \Pr(C_2) = \ldots = \Pr(C_N)$. Put another way, if you randomly
sample an item from the dataset there is an equal probability of it being in
any of the classes.

However, when it comes to solving practical problems, balanced datasets are
relatively rare. Instead, we have data that is generally imbalanced:
$\Pr(C_1) \ne \Pr(C_2) \ne \ldots \ne \Pr(C_N)$.

In this post we will consider classification models which are both
[probabilistic](https://en.wikipedia.org/wiki/Probabilistic_classification)
and [discriminative](https://en.wikipedia.org/wiki/Discriminative_model).
That is, after training on a classification dataset, the model is able to
estimate $\Pr(C_k|\boldsymbol{x})$---the probability of some input
$\boldsymbol{x}$ belonging to some class $C_k$.

One of the great things about having a _balanced_ dataset is that it plays
quite nicely with the usual algorithms and objective functions used to train
classification models (including neural networks). On the other hand,
_imbalanced_ datasets
can throw a proverbial spanner into the works by making trivial solutions
highly attractive to the learning algorithm. For example, if 99% of the examples
belong to a single class, 99% accuracy can be attained by simply predicting
that everything is in that class!


## Training on an imbalanced dataset

A simple technique for training on an imbalanced dataset is to draw samples
in accordance with the distribution of classes. So, whenever you need a training
example:

1. Select a class randomly by sampling from a uniform distribution.
2. Draw an example from that class.

We'll refer to the set of examples constructed according to this strategy
as the training dataset $A$. Our sampling strategy is specifically designed
to be balanced, so ${\Pr}_A(C_k) = \mathrm{constant}$.

An observation that we can make now is that our sampling strategy
**does not change the distribution of examples within each class** since we
aren't doing anything fancy when drawing examples from within a class. This
will become important later on.

After training on dataset $A$, we have a model for
${\Pr}_A(C_k|\boldsymbol{x})$. But we're not finished quite yet...


## Evaluating on an imbalanced dataset

Imagine for a moment that we trained a model which classifies children as
future millionaires or not. Clearly there is an imbalance between those two
classes, and the training data was sampled accordingly. However, there's a
problem when it comes to evaluation: for examples which contain no strong
indicators of future financial success, the model is outputting a 50%
chance of future millionairism! This is not altogether surprising,
since during training either class was equally likely. But even so, clearly a
random child is not so likely to be a millionare in practice!

What we want to do is incorporate into our predictions prior knowledge of the
likelihood of a random example being in a particular class (following our
previous example, the probability of a random child becoming a millionaire).

Let's call our evaluation dataset $B$. Our prior is ${\Pr}_B(C_k)$, and in
general ${\Pr}_B(C_1) \ne {\Pr}_B(C_2) \ne \ldots {\Pr}_B(C_N)$ (classes
are imbalanced). The adjusted version of the model which we are hoping to find
is ${\Pr}_B(C_k|\boldsymbol{x})$.

Using [Bayes' theorem](https://en.wikipedia.org/wiki/Bayes%27_theorem),

$$
\dfrac{{\Pr}_B(C_k|\boldsymbol{x})}
      {{\Pr}_A(C_k|\boldsymbol{x})}
= \dfrac{{\Pr}_B(\boldsymbol{x}|C_k) {\Pr}_B(C_k) {\Pr}_A(\boldsymbol{x})}
        {{\Pr}_A(\boldsymbol{x}|C_k) {\Pr}_A(C_k) {\Pr}_B(\boldsymbol{x})}
$$

With some minor rearranging for clarity,

$$
{\Pr}_B(C_k|\boldsymbol{x})
= {\Pr}_A(C_k|\boldsymbol{x})
  \cdot
  \dfrac{{\Pr}_B(C_k)}
        {{\Pr}_A(C_k)}
  \cdot
  \dfrac{{\Pr}_B(\boldsymbol{x}|C_k)}
        {{\Pr}_A(\boldsymbol{x}|C_k)}
  \cdot
  \dfrac{{\Pr}_A(\boldsymbol{x})}
        {{\Pr}_B(\boldsymbol{x})}
$$

Remember how we observed before that our sampling strategy did not change
the distribution of examples within each class? Well, we can write that formally
as ${\Pr}_B(\boldsymbol{x}|C_k) = {\Pr}_A(\boldsymbol{x}|C_k)$. That is,

$$
\dfrac{{\Pr}_B(\boldsymbol{x}|C_k)}
      {{\Pr}_A(\boldsymbol{x}|C_k)}
= 1
$$

So:

$$
{\Pr}_B(C_k|\boldsymbol{x})
= {\Pr}_A(C_k|\boldsymbol{x})
  \cdot
  \dfrac{{\Pr}_B(C_k)}
        {{\Pr}_A(C_k)}
  \cdot
  \dfrac{{\Pr}_A(\boldsymbol{x})}
        {{\Pr}_B(\boldsymbol{x})}
$$

Furthermore, ${\Pr}_A(\boldsymbol{x})$ and ${\Pr}_B(\boldsymbol{x})$ are
constant with respect to the class, $C_k$.

So:

$$
{\Pr}_B(C_k|\boldsymbol{x})
\propto
{\Pr}_A(C_k|\boldsymbol{x})
\dfrac{{\Pr}_B(C_k)}
      {{\Pr}_A(C_k)}
$$

Now we know that ${\Pr}_B(C_k|\boldsymbol{x})$ is a probability distribution and
therefore the probabilities of all classes must sum to one
($\sum_k{{\Pr}_B(C_k|\boldsymbol{x})}=1$). So, by normalising the probability
distribution we get:

$$
{\Pr}_B(C_k|\boldsymbol{x})
= \dfrac{
      {\Pr}_A(C_k|\boldsymbol{x})
      \dfrac{{\Pr}_B(C_k)}
            {{\Pr}_A(C_k)}
}{
      \sum_i{
            {\Pr}_A(C_i|\boldsymbol{x})
            \dfrac{{\Pr}_B(C_i)}
                  {{\Pr}_A(C_i)}
      }
}
$$

This formula works regardless of how examples from dataset $A$ are distributed
amongst classes. However, if we know that the training data was resampled such
that the classes are uniformly distributed, we can make one last simplification:

$$
{\Pr}_B(C_k|\boldsymbol{x})
= \dfrac{
      {\Pr}_A(C_k|\boldsymbol{x})
      {\Pr}_B(C_k)
}{
      \sum_i{
            {\Pr}_A(C_i|\boldsymbol{x})
            {\Pr}_B(C_i)
      }
}
$$

Bingo bango, we're done.

## An example

Let's run through a quick example based on the millionaire detector described
earlier on.

We'll say that 5% of people go on to become millionaires:

$${\Pr}_B(\mathrm{millionaire}) = 0.05$$

Now we run our classification model on a kid called Billy and get a 70%
probability of the child becoming a millionaire:

$${\Pr}_A(\mathrm{millionaire}|\mathrm{billy}) = 0.7$$

How good is that for the kid? Let's find out.

Recall that the classifier is assumed to have been trained on a balanced
dataset, so we calculate the adjusted probability (based on our prior) as
follows:

$$
{\Pr}_B(\mathrm{millionaire}|\mathrm{billy})
= \dfrac{
      0.7 \times 0.05
}{
      (0.7 \times 0.05) + (0.3 \times 0.95)
}
\approx 0.11
$$

So little Billy shouldn't get their hopes up too high, they only have an
~11% probability of becoming a millionaire. Too bad.


## An important caveat

It's assumed that the model's outputs are well-calibrated. If the predicted
probabilities aren't meaningful, the Bayesian evaluation-time strategy described
here probably won't work too well.

---

This article was motivated by [a Reddit post about common machine learning
interview questions](https://www.reddit.com/r/MachineLearning/comments/c1vxoc/d_17_interviews_4_phone_screens_13_onsite_5/).
