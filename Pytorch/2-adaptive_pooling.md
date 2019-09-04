# Adaptive pooling
This operation maps arbitrary input to fix dimension output. Say you define a adaptive pooling
operation like this:
```python
pool = nn.AdaptiveAvgPool2d((2, 2))
```
then for any input tensor `x` of shape (n, c, h, w), the output will be (n, c, 2, 2).

## How does it work?
It's really simple. We loop through each output position (i, j). And for a given position (i, j),
we compute its corresponding subwindow inside the input tensor:
$$

top = floor(i * ih / oh) \\
bottom = ceil((i + 1) * ih / oh) \\
left = floor(i * iw / ow) \\
right = ceil((i + 1) * iw / ow)
$$

