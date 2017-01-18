+++
title = "Mode collapse in GANs"
description = """
  How to address mode collapse, a commonly encountered failure case for GANs
  where the generator learns to produce samples with extremely low variety."""
date = "2017-01-18T14:19:00+11:00"
categories = ["deep learning"]
tags = ["torch", "gan"]
comments = false
draft = false
showpagemeta = true
slug = ""
+++

## What is mode collapse?

Most interesting real-world data distributions are highly complex and
_multimodal_. That is to say, the probability distribution which describes
the data has multiple "peaks" where different sub-groups of samples are
concentrated.

For example, let's say you have a dataset containing a mixture of summer day
temperature readings from Alice Springs in central Australia (typically very
hot) and the South Pole in Antarctica (typically very cold). The distribution of
the data is bimodal - there are peaks at the average temperatures of the two
places with a gap inbetween. The graph below illustrates
this more clearly.

![Bimodal temperature data](/img/bimodal_temperatures.svg)

Now let's say you want to train a GAN which produces plausible temperature
values. Intuitively speaking, we would expect the generator to learn to produce
hot and cold temperatures with roughly equal probability.
However, a commonly encountered issue is that a mode collapse will occur,
resulting in the generator only outputting samples from a single mode (eg
only cold temperatures). To understand why, consider the
following scenario:

1. The generator learns that it can fool the discriminator into
   thinking that it is outputting realistic temperatures by producing values
   close to Antarctic temperatures.
2. The discriminator counters by learning that all Australian temperatures are
   real (not produced by the generator), and essentially guesses whether
   Antarctic temperatures are real or fake since they are indistinguishable.
3. The generator exploits the discriminator by switching modes to produce values
   close to Australian temperatures instead, abandoning the Antarctic mode.
4. The discriminator now assumes that all Australian temperatures are fake and
   Antarctic temperatures are real.
5. Return to step 1.

This game of cat-and-mouse repeats ad nauseum, with the generator never
being directly incentivised to cover both modes. In such a scenario the
generator will exhibit very poor diversity amongst generated samples, which
limits the usefulness of the learnt GAN.

In reality, the severity of mode collapse varies from complete collapse
(all generated samples are virtually identical) to partial collapse (most of the
samples share some common properties). Unfortunately, mode collapse can
be triggered in a seemingly random fashion, making it very difficult to play
around with GAN architectures.

## Addressing mode collapse

Mode collapse is a well-recognised problem, and researchers have made a few
attempts at addressing it. I have identified 4 broad approaches to tackling
mode collapse, which are described below.

### Directly encourage diversity

It is impossible to determine output diversity by considering individual
samples in isolation. This leads to a very logical next step of using
batches of samples to directly assess diversity. Minibatch discrimination and
feature mapping [^imprgan] are two techniques which fall into this category.

Minibatch discrimination gives the discriminator the power of comparing
samples across a batch to help determine whether the batch is real or fake.

Feature matching modifies the generator cost function to factor in the
diversity of generated batches. It does this by matching statistics of
discriminator features for fake batches to those of real batches. I had
some success combining feature matching with the traditional GAN generator
loss function to form a hybrid objective.

### Anticipate counterplay

One way to prevent the cat-and-mouse game of hopping between modes is to
peek into the future and anticipate counterplay when updating parameters. This
approach should be familiar to those who know a bit about game theory
(eg [minimax](https://en.wikipedia.org/wiki/Minimax)). Intuitively, this
prevents players of the GAN game from making moves which are easily countered.

Unrolled GANs [^unrolled] take this kind of approach by allowing the generator
to "unroll" updates of the discriminator in a fully differentiable way. Now
instead of the generator learning to fool the current discriminator, it
learns to maximally fool the discriminator _after it has a chance to respond_,
thus taking counterplay into account. Downsides of this approach are increased
training time (each generator update has to simulate multiple discriminator
updates) and a more complicated gradient calculation (backprop through an
optimiser update step can be difficult).

### Use experience replay

Hopping back and forth between modes can be minimised by showing old fake
samples to the discriminator every so often. This prevents the discriminator
from becoming too exploitable, but only for modes that have already been
explored by the generator in the past.

A similar kind of effect can be achieved by occasionally substituting in an old
discriminator/generator for a few iterations.

### Use multiple GANs

Rather than fight mode collapse we could simply accept that the GAN will cover
only a subset of the modes in the dataset, and train multiple
GANs for different modes. When combined, these GANs cover all of the modes.
AdaGAN [^adagan] takes this approach. The major downside here is that training
multiple GANs takes a lot of time. Furthermore, using a combination of GANs is
generally more unwieldy than working with just one.

## How about tweaking the objective?

In an earlier post [I took a look at the GAN objective function](../2016-12-21-gan-objective)
and mentioned that different f-divergences have different mode-seeking vs
mode-covering behaviour. It is my current belief that this has minimal (if any)
impact on the mode collapse problem described here. No matter what the objective
function is, if it only considers individual samples (without looking forward
or backward) then the generator is not directly incentivised to produce diverse
examples.

Section 3.2.5 of "NIPS 2016 Tutorial: Generative Adversarial Networks"
[^nips2016gan] covers this point in more detail.

## References

[^imprgan]:
    Improved Techniques for Training GANs.
    https://arxiv.org/abs/1606.03498

[^unrolled]:
    Unrolled Generative Adversarial Networks.
    https://arxiv.org/abs/1611.02163

[^adagan]:
    AdaGAN: Boosting Generative Models.
    https://arxiv.org/abs/1701.02386v1

[^nips2016gan]:
    NIPS 2016 Tutorial: Generative Adversarial Networks.
    https://arxiv.org/abs/1701.00160v3
