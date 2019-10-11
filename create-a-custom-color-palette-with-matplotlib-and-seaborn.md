## Introduction

Often when creating graphs in python I have found the need to use a custom color pattern. Fortunately, `matplotlib` makes this fairly easy to accomplish!

## The full code

```python
import numpy as np; np.random.seed(0)
import seaborn as sns; sns.set()

import matplotlib.pyplot as plt
from matplotlib.colors import ListedColormap, LinearSegmentedColormap


def create_color(r, g, b):
return [r/256, g/256, b/256]

def get_custom_color_palette():
    return LinearSegmentedColormap.from_list("", [
        create_color(227, 101, 33), create_color(246, 145, 53), create_color(251, 168, 74),
        create_color(218, 212, 200),
        create_color(141, 193, 223), create_color(114, 167, 208), create_color(43, 92, 138)
    ])

low, high = -5, 15
data = np.random.uniform(low, high, (10, 15))

cmap = get_custom_color_palette()
sns.heatmap(data, cmap=cmap, center=0);
```

![matplotlib_custom_color_palette](https://richardhildebrand.files.wordpress.com/2019/09/matplotlib_custom_color_palette.png)


## A few additional notes

If you would prefer, you may use hash values instead of RGB values.

```python
def get_custom_color_palette_hash():
    return LinearSegmentedColormap.from_list("", [
        '#ff6521', '#f69135', '#fba84a',
        '#dad4c8',
        '#8dc1df', '#72a7d0', '#2b5c8a'
    ])
```

It may also not be immediatly obvious, but I have used `center=0` to place our blue colors above `0` and our orange colors below `0`. This will not make sense for all data an may be excluded.