# Matplotlib Cheat Sheet

## Figure

### Set figure size
```python
plt.figure(figsize=(10, 13))
```
### Set figure title
```python
fig.suptitle('Your figure title', fontsize=24)
```

## Subplots

### Make multiple subplots
```python
import matplotlib.gridspec as gridspec
gs = gridspec.GridSpec(1, 3)

plt.figure()
ax = plt.subplot(gs[0, 0])
```

### Adjust spacing between subplots
```python
plt.subplots_adjust(wspace=0, hspace=0)
```
### Set subplot title
```python
ax.set_title('your subplot title', fontsize=20)
```

### Set subplot border color

```python
# set all borders and ticks to the same color
def draw_axis_spine(ax, color, width):
    plt.setp(ax.spines.values(), color=color, linewidth=width)
    plt.setp([ax.get_xticklines(), ax.get_yticklines()], color=color)

# set borders to specific color
ax.spines['top'].set_color('#cbcbcb')
ax.spines['right'].set_color('#cbcbcb')
ax.spines['bottom'].set_color('#cbcbcb')
ax.spines['left'].set_color('#cbcbcb')

# hide part of the border
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
```

### Add and style grid lines

```python
# use zorder to have the bars appear under the plots
# zorder must also be added to plots
ax.grid(zorder=0, color='#cbcbcb', linestyle='-', linewidth=0.25)
ax.yaxis.grid()
ax.xaxis.grid()


def draw_n_even_lines(ax, n, color, size):
    lower_bound, upper_bound = ax.get_xlim()
    full_range = abs(lower_bound) + abs(upper_bound)
    tick_spacing = full_range / float(n+1)
    
    value = lower_bound
    ticks = [value]
    while value < upper_bound:
        value = value + tick_spacing
        ticks.append(value)
        
    ax.xaxis.grid(zorder=0, color=color, linestyle='-', linewidth=size)
    ax.get_xaxis().set_ticks(ticks)
    
# Example using draw_n_even_lines
draw_n_even_lines(ax, 4, '#cbcbcb', 0.25)
```



## Labels

### Hide a label
```python
ax.get_xaxis().set_visible(False)
ax.get_yaxis().set_visible(False)
```

### Set Label Text

```python	
ax.set_xlabel('x label text', fontsize=20)
ax.set_ylabel('y label text', fontsize=20)
```

### Show a label on a barplot

```python
def x_format(value):
    return round(value, 1)


def draw_label_on_bar(ax, df, x_column, sort_column, sort_direction, x_format_function, offset):
    temp = df.sort_values(by=sort_column, ascending=sort_direction)

    count = 0
    for index, row in temp.iterrows():
        x = row[x_column]
        x_position = x - offset
        x_value = x_format_function(x)

        ax.text(x_position, count, str(x_value), color='blue', fontweight='bold')
        count = count + 1

# Example using draw_label_on_bar
draw_label_on_bar(ax, pt, 'value_column', 'sort_column', False, x_format, 0)
```

## Ticks

### Hide ticks

```python
ax.xaxis.set_ticks_position('none') # hide only ticks, not ticks and tick labels
ax.get_xaxis().set_ticks([]) # hides ticks and tick labels
ax.get_xaxis().set_visible(False) # also hides axis label

def hide_ticks_and_tick_labels(ax):
    for tick in ax.xaxis.get_major_ticks():
        tick.tick1line.set_visible(False)
        tick.label1.set_visible(False)
```

