+++
title = "Neural network implementation tips and tricks"
description = """
  Implementing neural networks can be intimidating at the start, with a daunting
  number of choices to make with no real sense of which option might work best.
  By listing my (opinionated) defaults found from experience, I hope to provide
  you, dear reader, with a starting point from which you can train a successful
  neural network."""
date = "2016-11-06T11:15:01+11:00"
categories = ["deep learning"]
tags = ["torch"]
comments = false
draft = false
showpagemeta = true
slug = ""
+++

Implementing neural networks can be intimidating at the start, with a daunting
number of choices to make with no real sense of which option might work best.
By listing my (opinionated) defaults found from experience, I hope to provide
you, dear reader, with a starting point from which you can train a successful
neural network.

Snippets of code are shown for [Torch](http://torch.ch/), but most of these
ideas are applicable to any neural network framework.

Disclaimer: These guidelines come from my experience, and reflect what has
generally worked for me in the past. Consider using them as a starting point,
but don't be afraid to experiment!

### Data preprocessing

Do any data preprocessing you require ahead of time to reduce the amount
of work required during training. Yes, it will use disk space to do so but the
time saves are worth it. Decompression, cropping, and augmentation are all
things that you should consider doing prior to training.

### Training and data sampling

Organise your training into epochs, where an epoch is one full pass through the
data set. Make sure that your samples are shuffled - never group them by class
or anything like that. The shuffling can be done ahead of time and fixed, it
does not need to change each epoch. Report metrics such as loss and accuracy
at the completion of each epoch.

The number of epochs you should train for varies a lot with the network
architecture and data set. Usually 20-100 epochs are required before convergence
is reached, but you'll need to keep an eye on this.

Train using mini-batches so that parallelism-related optimisations can take
place.

### Loss function

Use what makes sense for your problem. Usually you will be selecting
cross-entropy loss for multi-class classification, binary cross-entropy loss
for binary classification, and mean squared error for regression.

Always have a sigmoid on your network output when using binary cross entropy.
In Torch, use
[CrossEntropyCriterion](https://github.com/torch/nn/blob/master/doc/criterion.md#nn.CrossEntropyCriterion)
for multi-class classification so that
you don't need to explicitly add a LogSoftMax layer to your network.

### Non-linearities

Ensure that each weighted layer (convolution, fully connected) is followed by
a non-linearity, except for the last one which will depend on your task.

Use ReLU (rectified linear unit) or something from that family
(leaky ReLU, ELU). Only use tanh or
sigmoid when you need to explicitly squash values into the (-1, 1) or (0, 1)
numeric range respectively. Apply non-linearities in place to save memory when
possible.

```lua
-- Create an in-place ReLU layer.
local relu = nn.ReLU(true)
```

### Batch normalisation

Batch normalisation[^batch-norm] makes training a lot easier (less dependent on
initial weights and faster to converge), and regularises your network
(reduces overfitting). I highly recommend using it. Place batch normalisation
layers before non-linearities.

Be aware that batch normalisation utilises statistics within mini-batches, so
you will need to use mini-batches of a reasonable size. I'd treat batch size 16
as a bare minimum, with 64-128 being preferable.

Ensure that you set your network to be in training mode or evaluation mode
appropriately when using batch normalisation, since it behaves differently
during inference.

```lua
nn.Sequential()
  :add(nn.SpatialConvolution(...))
  :add(nn.SpatialBatchNormalization(...))
  :add(nn.ReLU())
```

### Dropout

I've had mixed results with dropout[^dropout] personally. It might help if you
find your model overfitting, but I'd try to pin down the rest of the
architecture before considering dropout - batch normalisation already helps
a lot with regularisation.

Dropout can be applied on either side of a non-linearity, but ensure that it is
inserted after batch normalisation. Common dropout values are 0.2 (weak) and 0.5
(moderate).

As with batch normalisation, ensure that you set your network to be in training
mode or evaluation mode appropriately.

### Optimisation algorithm

Start with ADADELTA[^adadelta] when comparing architectures since you don't need to tune
any hyperparameters. There are already heaps of options when coming up with a neural network, and
optimiser hyperparamenters are just more balls to juggle. Once you have nailed down
the architecture, then think about whether you want to switch to something
with a learning rate to try and squeeze out an extra little bit of performance.

### Convolution layers

Use 3x3 convolutions, especially since CuDNN is optimised for these. Most state-of-the-art
competition-winning convolutional neural network architectures are based around
using 3x3 convolutions.

If the network is deep, use residual blocks with pre-activations as described in
the follow-up ResNet paper[^resnet-preact].

For classification, consider using 1x1 convolutions and global average pooling
instead of fully connected layers to go from spatial activations to class
predictions.

Pad with a border of 1 pixel when using 3x3 convolutions to keep the spatial
dimensions the same after the convolution operation.

Use max pooling or strided convolutions to downsample as you move through the
network. For most tasks you want big images with a low number of feature maps
near the input, with increasing maps and decreasing dimensions as you approach
the output.

### Use a CUDA-enabled GPU

Whether you like it or not, training and evaluating on an NVIDIA GPU with CUDA
is currently the fastest readily available option. If you can afford one,
I like the Titan X for its large amount of RAM (12 GiB). The
[first generation Titan X](https://en.wikipedia.org/wiki/GeForce_900_series)
works great, and should be available for a much lower price than the new
Pascal Titan X.

Provided that you are using CUDA, NVIDIA's CuDNN library can offer further
performance gains during training.

For the highest training speed possible, enable benchmarks and convert your
model into its CuDNN equivalent.

```lua
require('cudnn')
cudnn.benchmark = true
cudnn.convert(model, cudnn)
```

If you need deterministic output, do not use benchmarking and set the CuDNN
convolution layers to use deterministic convolution algorithms. This will
be slower, but you will have reproducable results.

```lua
-- Create a convolution layer
local conv = cudnn.SpatialConvolution(...)

-- Use deterministic algorithms
conv:setMode(
  'CUDNN_CONVOLUTION_FWD_ALGO_IMPLICIT_GEMM',
  'CUDNN_CONVOLUTION_BWD_DATA_ALGO_1',
  'CUDNN_CONVOLUTION_BWD_FILTER_ALGO_1')
```

## Example architecture

Here's a network architecture for image classification which ties together
a few of the ideas described above. It is loosely based on VGG[^vgg]. In
practice you would likely want to increase the depth of the network by adding
multiple Conv-BN-ReLU blocks at the same resolution.

```lua
-- The number of output classes
local n_classes = 10

local model = nn.Sequential()
  -- Let's say the input is 3 x 224 x 224. I'll keep track of the dimensions
  -- in the comments as we go through the network.
  :add(nn.SpatialConvolution(3,32, 3,3, 1,1, 1,1))
  :add(nn.SpatialBatchNormalization(32))
  :add(nn.ReLU(true))
  -- 32 x 224 x 224
  :add(nn.SpatialMaxPooling(2,2, 2,2))
  -- 32 x 112 x 112
  :add(nn.SpatialConvolution(32,64, 3,3, 1,1, 1,1))
  :add(nn.SpatialBatchNormalization(64))
  :add(nn.ReLU(true))
  -- 64 x 112 x 112
  :add(nn.SpatialMaxPooling(2,2, 2,2))
  -- 64 x 56 x 56
  :add(nn.SpatialConvolution(64,128, 3,3, 1,1, 1,1))
  :add(nn.SpatialBatchNormalization(128))
  :add(nn.ReLU(true))
  -- 128 x 56 x 56
  :add(nn.SpatialMaxPooling(2,2, 2,2))
  -- 128 x 28 x 28
  :add(nn.SpatialConvolution(128,256, 3,3, 1,1, 1,1))
  :add(nn.SpatialBatchNormalization(256))
  :add(nn.ReLU(true))
  -- 256 x 28 x 28
  :add(nn.SpatialMaxPooling(2,2, 2,2))
  -- 256 x 14 x 14
  :add(nn.SpatialConvolution(256,512, 3,3, 1,1, 1,1))
  :add(nn.SpatialBatchNormalization(512))
  :add(nn.ReLU(true))
  -- 512 x 14 x 14
  :add(nn.SpatialMaxPooling(2,2, 2,2))
  -- 512 x 7 x 7
  :add(nn.SpatialConvolution(512,512, 3,3, 1,1, 1,1))
  :add(nn.SpatialBatchNormalization(512))
  :add(nn.ReLU(true))
  -- 512 x 7 x 7
  :add(nn.SpatialConvolution(512,n_classes, 1,1, 1,1, 1,1))
  -- n_classes x 7 x 7
  :add(nn.SpatialAveragePooling(7,7, 1,1)) -- Apply global average pooling
  -- n_classes x 1 x 1
  :add(nn.View(n_classes))
  -- n_classes

model:cuda()

-- Use CuDNN and optimise for speed
cudnn.benchmark = true
cudnn.convert(model, cudnn)

-- This criterion applies the log softmax for us, then calculate loss using
-- negative log likelihood.
local criterion = nn.CrossEntropyCriterion()
criterion:cuda()
```

## References

[^batch-norm]:
    Batch Normalization: Accelerating Deep Network Training by Reducing Internal
    Covariate Shift.
    https://arxiv.org/abs/1502.03167

[^dropout]:
    Dropout: A Simple Way to Prevent Neural Networks from Overfitting.
    http://jmlr.org/papers/v15/srivastava14a.html

[^adadelta]:
    ADADELTA: An Adaptive Learning Rate Method.
    https://arxiv.org/abs/1212.5701

[^resnet-preact]:
    Identity Mappings in Deep Residual Networks.
    https://arxiv.org/abs/1603.05027

[^vgg]:
    Very Deep Convolutional Networks for Large-Scale Image Recognition.
    https://arxiv.org/abs/1409.1556
