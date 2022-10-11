---
title: NumPy vectorize 
categories:
 - Scientific computing
tags:
 - Python
 - NumPy
---

# Usage of np.vectorize

[NumPy reference](https://numpy.org/doc/stable/reference/generated/numpy.vectorize.html) 

Using np.vectorize() has much higher perfomance compare with pandas.DataFrame.apply()

```python
import pandas as pd
import numpy as np

x = np.random.randint(300, 500, 100000)
y = np.random.randint(300, 500, 100000)
```

```python
df = pd.DataFrame({'x':x, 'y':y})
%timeit df.apply(lambda row: row['x'] + row['y'] - 360 if row['x'] + row['y'] > 337.5 else row['x'] + row['y'], axis = 1)

>>> 1.52 s ± 8.22 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```

```python
vfunc = np.vectorize(lambda x,y: x+y-360 if x+y>337.5 else x+y)
%timeit vfunc(x, y)

>>> 26.3 ms ± 575 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)
```
