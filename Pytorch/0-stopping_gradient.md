# Stopping Gradient

As far as I know, there are 3 ways to stop gradient from propagating back to the input.

## Use `torch.no_grad()` Context Manager

Everything inside the `torch.no_grad()` block will not propagate gradient

```python
x= torch.randn(4, requires_grad=True)

with torch.no_grad():
    y= x*2
print(x.requires_grad, y.requires_grad)
# True, False

```

I often use it during testing.

## Directly Operate on the Underlying Data

You could directly work on the `tensor.data` and it will not propagate gradient.

```python
x = torch.randn(4, requires_grad=True)

y = x.data + 1
```

It's useful if you are implementing an optimizer since you will perform the parameters for training, something like this:

```python
net = MyNet()
loss = net(x, y)
loss.backward()
lr = 0.1

for param in net.parameters():
    param.data.sub_(f.grad.data * lr)
```

## Detach it From the Computation Graph

You could use the `tensor.detach()` function to detach the tensor from the computation graph. This function returns a different tensor but share the same data as the original one.

```python
x = torch.randn(4, requires_grad=True)
y = torch.detach()

print(id(x), id(y)) # they're different

print(id(x.data), id(y.data)) # but the underlying data is the same

```