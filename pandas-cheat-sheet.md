# Pandas Cheat Sheet

## Equals

```python
    our_value = 'some_value'
    df = df.query('column_name <= @our_value')
    
    our_column = 'column_name' 
    df.query(f'{our_column} == @our_value')
    
    #working with NaN
    df = df.query('column_name.isnull()', engine='python')
    df = df.query('column_name.notnull()', engine='python')
```

## Between

```python
    min_value = 1
    max_value = 100
    df = df.query('@min_value <= column_name <= @max_value')
```

## Is In

```python
    values = ['value_one', 'value_two']
    df = df[df['column_name'].isin(values)]
```

## Group by

```python
    df = df.groupby('column_one')
    df = df.groupby(['column_one', 'column_two'])
```

## Reset index of grouped by column

```python
    df = df.reset_index(level='grouped_by_column')

    #after multiple column aggregates
    df.columns = df.columns.droplevel(0)
    df = df.reset_index(drop=0)
```

## Order by

```python
    df = df.sort_values(by='column_one', ascending=False)
    df = df.sort_values(by=['column_one', 'column_two'], ascending=[False, True])
```

## Unique values in column

```python
    df['column_name'].unique()
```

## Replace value in column

```python
    df.loc[df['column'] == 'old_value', 'column'] = 'new_value'
```

## Get column value of row

```python
    row = df.query('column_to_find == "some_value"')
    column_value = row.iloc[0]['column_with_value']
```

## Aggregate

```python
    df = df.agg({
        'column_one' : 'sum',
        'column_two' : 'max',
        'column_three' : 'count',
        'column_four' : { 'new_name_one': 'min', 'new_name_two': 'count' },
    })

    #renaming columns
    df = df.agg(
        new_column_name=('old_column_name', 'sum')
    )
```

Full list of aggregate commands

* **count** - Number of non-null observations
* **sum** - Sum of values
* **mean** - Mean of values
* **mad** -	Mean absolute deviation
* **median** - Arithmetic median of values
* **min** - Minimum
* **max** - Maximum
* **mode** - Mode
* **abs** - Absolute Value
* **prod** - Product of values
* **std** - Unbiased standard deviation
* **var** - Unbiased variance
* **sem** - Unbiased standard error of the mean
* **skew** - Unbiased skewness (3rd moment)
* **kurt** - Unbiased kurtosis (4th moment)
* **quantile** - Sample quantile (value at %)
* **cumsum** - Cumulative sum
* **cumprod** - Cumulative product
* **cummax** - Cumulative maximum
* **cummin** - Cumulative minimum

## Shift (lead / lag)

```python
    data['next_column'] = data['column'].shift(1)

    # with group by
    data['previous_column'] = data.groupby('group_by_column')['column'].shift(-1)
```

## Rename column

```python
    df = df.rename(columns={'old_name': 'new_name'})
```

## For each row

```python
    for index, row in df.iterrows():
        print(row)
```

## Apply custom function to row

```python
    df['calculated_column'] = df.apply(lambda row: 
        custom_function(
            row['column_one'], 
            row['column_two']
        ),
        axis = 1 #applies lambda to row
    )
```

## Create a pivot table

```python
    pt = pd.pivot_table(df, values='values_column', index=['index_column'], columns=['column_columns'])
    pt.reset_index(inplace=True)
```

## Read CSV

```python
    df = pd.read_csv('./file_path/file_name.csv')
```

## Save as CSV

```python
    df.to_csv('./file_path/file_name.csv', index=False)
```

## Read SQL

```python
    sql = 'Select * From table_name'
    conn = pyodbc.connect(
        'Driver={ODBC Driver 17 for SQL Server};'
        'Server=host.docker.internal;'
        'Database=db_name;'
        'UID=db_username;'
        'PWD=db_password'
    )
    
    return pd.read_sql_query(sql, conn)
```