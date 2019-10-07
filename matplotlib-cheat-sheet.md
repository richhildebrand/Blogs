# Matplotlib Cheat Sheet

## Set figure size

```python
    plt.figure(figsize=(10, 13))
```

## Subplots

```python
    import matplotlib.gridspec as gridspec
    gs = gridspec.GridSpec(1, 3)

    plt.figure()
    ax = plt.subplot(gs[0, 0])
```

## Labels

```python
    ax.get_yaxis().set_visible(False)
```    

To show a label on a barplot

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