# Pandas Cheat Sheet

## Equals

    our_value = 'some_value'
    df = df.query('column_name <= @our_value')
    
    #working with NaN
    df = df.query('column_name.isnull()', engine='python')
    df = df.query('column_name.notnull()', engine='python')

## Between

    min_value = 1
    max_value = 100
    df = df.query('@min_value <= column_name <= @max_value')

## Is In

    values = ['value_one', 'value_two']
    df = df[df['column_name'].isin(values)]

## Group by

    df = df.groupby('column_one')
    df = df.groupby(['column_one', 'column_two'])

## Reset index of grouped by column

    df = df.reset_index(level='grouped_by_column')

## Order by

    df = df.sort_values(by='column_one', ascending=False)
    df = df.sort_values(by=['column_one', 'column_two'], ascending=[False, True])

## Unique values in column

    df['column_name'].unique()

## Get column value of row

    row = game_ratings.query('column_to_find == "some_value"')
    column_value = row.iloc[0]['column_with_value']

## Aggregate

    df = df.agg({
        'column_one' : 'sum',
        'column_two' : 'max',
        'column_three' : 'count',
        'column_four' : { 'new_name_one': 'min', 'new_name_two': 'count' },
    })

Function | Description
|---|---|
count |	Number of non-null observations
sum	| Sum of values
mean |Mean of values
mad	Mean | absolute deviation
median |	Arithmetic median of values
min | Minimum
max | Maximum
mode | Mode
abs | Absolute Value
prod | Product of values
std	| Unbiased standard deviation
var	| Unbiased variance
sem	| Unbiased standard error of the mean
skew | Unbiased skewness (3rd moment)
kurt | Unbiased kurtosis (4th moment)
quantile | Sample quantile (value at %)
cumsum | Cumulative sum
cumprod | Cumulative product
cummax | Cumulative maximum
cummin | Cumulative minimum

## Rename column

    df = df.rename(columns={'old_name': 'new_name'})

## For each row

    for index, row in df.iterrows():
        print(row)

## Apply custom function to row

    df['calculated_column'] = df.apply(lambda row: 
        custom_function(
            row['column_one'], 
            row['column_two']
        ),
        axis = 1 #applies lambda to row
    )

## Read CSV

    df = pd.read_csv('./file_path/file_name.csv')

## Save as CSV

    df.to_csv(r'./file_path/file_name.csv', index=False)

## Read SQL

    sql = 'Select * From table_name'
    conn = pyodbc.connect(
        'Driver={ODBC Driver 17 for SQL Server};'
        'Server=host.docker.internal;'
        'Database=db_name;'
        'UID=db_username;'
        'PWD=db_password'
    )
    
    return pd.read_sql_query(sql, conn)