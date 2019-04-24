[TOC]

# Multi-Class Classification vs Multi-Label Classification

- **Multi-Class Classification**: each sample belongs to **one** of $C$ classes. The label could be represented as an index integer $c$ or a $C$-dimension vector $[0, 0, 1, 0]$ with only one non-zero element.
- **Multi-Label** Classification: each sample could belong to **more than one** of $C$ classes. The label is usually represented as a one-hot vector $[1, 0, 1, 0]$.



# Implementation in PyTorch

1. `nn.BCELoss()`
   $$
   \begin{align}
   l(p, y) &= -\frac{1}{C}\sum_n^C\left[ y_n \cdot \log p_n + (1 - y_n) \cdot \log (1 - p_n) \right]\\
   p &\in [0, 1]\\
   y & \in \{0, 1\}
   \end{align}
   $$
   Usage:

   - Support both multi-class and multi-label classification.
   - **The input must be probability between [0, 1].** It's usually done by squashing the logit tensor with sigmoid function.

2. `nn.MultiLabelSoftMarginLoss()`
   $$
   \begin{align}
   \sigma(z) &= \frac{1}{1 + e^{-z}}\\
   l(x, y) &= - \frac{1}{C}\sum_i y_i\cdot\log(\sigma(x_i)) + (1-y_i) \cdot \log(1 - \sigma(x_i))\\
   \end{align}
   $$
   Usage:

   - Support both multi-class and multi-label classification.
   - **The input must be logits (scores) before activation** since the loss function handles the probability transformation itself.

> In summary, these two implementations are identical if we're using sigmoid to output probability in `nn.BCELoss()`, see example below:
>
> ```python
> import torch
> import torch.nn as nn
> 
> torch.manual_seed(2)
> 
> bce_loss = nn.BCELoss()
> mlsm_loss = nn.MultiLabelSoftMarginLoss()
> 
> s = torch.randn(1, 4)
> label = torch.tensor([[0, 1, 0, 1]], dtype=torch.float)
> print(bce_loss(torch.sigmoid(s), label))
> print(mlsm_loss(s, label))
> >>> tensor(0.9332)
> >>> tensor(0.9332)
> ```
>
> 