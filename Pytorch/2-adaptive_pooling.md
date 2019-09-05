# Adaptive pooling
This operation maps arbitrary input to fix dimension output. Say you define a adaptive pooling
operation like this:

```python
pool = nn.AdaptiveAvgPool2d((2, 2))
```
then for any input tensor `x` of shape (n, c, h, w), the output will be (n, c, 2, 2).

## How does it work?
It's really simple. We loop through each output position (i, j). And for a given position (i, j), we compute its corresponding sub-window inside the input tensor:
$$
\begin{align}
top &= floor(i * ih / oh) \\
bottom &= ceil((i + 1) * ih / oh) \\
left &= floor(i * iw / ow) \\
right &= ceil((i + 1) * iw / ow)
\end{align}
$$

The `floor` and `ceil` pair make sure there are at least one element inside the sub-window defined by $(top, bottom, left, right)$.

## Examples

Say your input tensor x is:

```python
[[0, 0, 1, 1],
 [0, 0, 1, 1],   -->   [[0, 1],
 [2, 2, 3, 3],          [2, 3]].
 [2, 2, 3, 3]]
```

Let's see a special case:

```python
[[0, 0, 1],
 [0, 0, 1]]
```

Say we want an output of (2, 2) tensor. Then the corresponding sub-windows are 

```python
[[0]]    [[0, 1]]             [[0,       0.5],
                       --> 
[[0]]    [[0, 1]]              [0,       0.5]]
```

## Implementation

Here is a possible implementation of the adaptive pooling operation (only consider 2d inputs). You could refer to the pytorch source code [here](https://github.com/pytorch/pytorch/blob/65b00aa5972e23b2a70aa60dec5125671a3d7153/aten/src/ATen/native/AdaptiveAveragePooling.cpp) for more details.

Put the following source code inside a script file, say `my_adaptive_avg_pool.py`, and test this implementation against the standard pytorch outputs with `python -m unittest my_adaptive_avg_pool.py`. You should see all the test cases pass.

```python
import math
import torch
import unittest
import torch.nn.functional as F


def my_adpative_avg_pool(x, out_size):
    """
    :param x: torch.tensor(dtype=torch.float, shape=(h, w)), input tensor
    :param out_size: tuple, given the output dimension (oh, ow)
    :return torch.tensor(shape=(oh, ow))
    """
    ih, iw = x.shape
    oh, ow = out_size
    out = torch.zeros(oh, ow, dtype=x.dtype)

    for i in range(oh):
        for j in range(ow):
            top = math.floor(i * ih / oh)
            bottom = math.ceil((i+1)*ih / oh)
            left = math.floor(j * iw / ow)
            right = math.ceil((j+1)*iw / ow)
            out[i, j] = x[top:bottom, left:right].mean()
    return out


class MyTestCase(unittest.TestCase):

    def _util_func(self, in_size, out_size):
        # torch.manual_seed(2)
        x = torch.randint(2, size=[1, 1, *in_size]).float()
        out = my_adpative_avg_pool(x.view(*in_size), out_size)
        torch_out = F.adaptive_avg_pool2d(x, out_size).view(*out_size)
        err = torch.abs(torch_out - out).max().item()
        self.assertTrue(err < 1e-7)

    def test_in8x8_out2x2(self):
        self._util_func((8, 8), (2, 2))

    def test_in8x8_out3x3(self):
        self._util_func((8, 8), (3, 3))

    def test_in8x8_out5x3(self):
        self._util_func((8, 8), (5, 3))

    def test_in8x9_out5x3(self):
        self._util_func((8, 9), (5, 3))

    def test_in2x2_out4x4(self):
        self._util_func((2, 2), (4, 4))
```

