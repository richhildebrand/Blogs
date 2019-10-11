## Introduction

With `matplotlib` and `LaTex` processing you are able to create text blocks containing multiple font sizes. This works well with labels, ticks, legends, and annotions.

## The full code

```python
import numpy as np
import matplotlib.pyplot as plt

plt.rc('text', usetex=True) #Added LaTeX processing

x = np.arange(1,5,0.5)
plt.figure(1)
plt.plot(x,x,label=r'\Large Curve \Huge 1')
plt.plot(x,2*x,label=r'\tiny Curve \small 2')
plt.legend(loc = 0)
plt.show();
```

![matplotlib_mutiple_font_sizes](https://richardhildebrand.files.wordpress.com/2019/10/matplotlib_label_different_font_size.png)


## A word of caution

The one downside is that the multiple fontsizes cannot currently be processed by a PDF backend.