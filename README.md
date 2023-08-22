# Toydiff

`toydiff` is a simple automatic differentiation library that I created to wrap
my head around how autodiff works.

It is built using only NumPy and tested against PyTorch.

The libray is very versatile, and can be used to create and train neural
networks (of course).

## Installation
Normal user:
```bash
git clone https://github.com/Xylambda/toydiff.git
pip install toydiff/.
```

Developer:
```bash
git clone https://github.com/Xylambda/toydiff.git
pip install -e toydiff/. -r toydiff/requirements-dev.txt
```

## Tests
To run test, you must install the library as a `developer`.

```bash
cd toydiff/
pytest -v tests/
```

## Differentiable operations
The use is almost the same as the one you would expect from PyTorch:

```python
>>> import toydiff as tdf

>>> # use `track_gradient=True` to allow backward to fill the gradients
>>> a = tdf.rand((3,3), track_gradient=True)
>>> b = tdf.rand((3,3), track_gradient=True)
>>> c = tdf.matmul(a, b)
>>> d = tdf.log(c)
>>> e = tdf.sum(d)
```

Variable `e` is a Tensor that allows to backpropagate:
```python
>>> e
Tensor(0.22356361, dtype=float32, backward_fn=<Sum(ReduceOp).Backward>)
>>> e.backward()  # can pass a gradient tensor too if needed
>>> a.gradient
Tensor([[0.7647713, 2.643686 , 3.2196524],
       [0.4147661, 1.494362 , 1.8254415],
       [0.5436615, 1.9049336, 2.524307 ]], dtype=float32, track_gradient=False)
```

The library tracks intermediate gradients by default, without needing to
perform extra steps to do so:

```python
>>> c.gradient
Tensor([[1.1955129 , 1.6154591 , 1.1438175 ],
       [0.7210128 , 0.91004455, 0.60389584],
       [0.83467734, 1.4432425 , 0.75835925]], dtype=float32, track_gradient=False)
```

## Neural networks (WIP)

The module `nn` contains all necessary functionality to create and optimize
basic neural networks:

```python
import numpy as np
import toydiff as tdf
from toydiff.nn.blocks import Linear
from toydiff.nn.optim import SGD
from toydiff.nn.functional import mse_loss

# generate data
X = np.arange(-1, 1, 0.01)
y = 2 * X + np.random.normal(size=len(X), scale=0.3)
X = X.reshape(-1,1)
y = y.reshape(-1,1)

# create model
model = Linear(1, 1, bias=False)

# wrap your data in Tensors with `track_gradient=True`
feat = tdf.Tensor(X, track_gradient=True)
labels = tdf.Tensor(y, track_gradient=True)

# pass model to optimizer
optimizer = SGD(model)

# build train loop
# we want to minimize the sum of squares
losses = []
n_epochs = 15_000
for i in range(n_epochs):
    # zero grads if you do not want to accumulate gradients
    optimizer.zero_grad()

    # forward pass, loss and backward pass
    out = model(feat)
    loss = mse_loss(out, labels)
    loss.backward()

    # use gradients to update parameters
    optimizer.step()
    
    losses.append(loss.value)
```
