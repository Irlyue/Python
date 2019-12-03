# Batch Normalization

On pytorch implementation of [batch normalization](https://pytorch.org/docs/stable/_modules/torch/nn/modules/batchnorm.html).

There are two type of variables in the BatchNorm layer:
- parameters: including the $\gamma$ and $\beta$, they'll get updated during training unless the
attribute `requires_grad` of the parameters is set to false.
- buffer: including the running mean and running variance, if `track_running_stats` is set to false, these variables will be `None`

parameters
- **training**: if training, parameters will gets updated
- **track_running_stats**: if False, does not track running mean and variance.



**Takeaways**

1. There is a new buffer variable `num_batches_tracked` added (since I don't know when: ). It is added to tracked the number of batches to support cumulative moving average (simply the average) beyond exponential moving average. It's achieved by dynamically changing the decay factor `momentum`.

2. There is one line of code that looks obscure at first (at least to me)

   ```python
   return F.batch_norm(
       input, self.running_mean, self.running_var, self.weight, self.bias,
       self.training or not self.track_running_stats,  # passed to `training` parameter
       exponential_average_factor, self.eps)
   ```

   Why `self.training or not self.track_running_stats`? If `training=False` and `track_running_stats=False` too, then this expression will be evaluated as `True`. That means if we create a BN layer with `track_running_stats` set to false and call `bn.eval()` at the beginning, suddenly we are in the training mode.

   To see why, we need to know one limitation of BN: it cannot handle batch with only one sample during training. Otherwise it will overflow due to zero division. So inside `F.batch_norm`, there's a checking section of code that will raise error if batch with one sample is passed in.

   ```python
       if training:
           size = input.size()
           # XXX: JIT script does not support the reduce from functools, and mul op is a
           # builtin, which cannot be used as a value to a func yet, so rewrite this size
           # check to a simple equivalent for loop
           #
           # TODO: make use of reduce like below when JIT is ready with the missing features:
           # from operator import mul
           # from functools import reduce
           #
           #   if reduce(mul, size[2:], size[0]) == 1
           size_prods = size[0]
           for i in range(len(size) - 2):
               size_prods *= size[i + 2]
           if size_prods == 1:
               raise ValueError('Expected more than 1 value per channel when training, got input size {}'.format(size))
   ```

   To answer the above question, when `training=False` and `track_running_stats=False` (that means we are in evaluation mode and not tracking any statistics during training), we need to use the batch statistics as running mean and variance during evaluation. It also implies that we can't use batch with single sample during evaluation if `track_running_stats` is set to `False`.