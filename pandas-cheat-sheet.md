# Pandas Cheat Sheet

## Equals

    our_value = 'some_value'
    df = df.query('column_name <= @our_value')

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

## Order by

    df = df.sort_values(by='column_one', ascending=False)
    df = df.sort_values(by=['column_one', 'column_two'], ascending=[False, True])

## Get column value of row

    row = game_ratings.query('column_to_find == "some_value"')
    column_value = row.iloc[0]['column_with_value']

## Aggregate

    df = df.agg({
        'column_one' : 'sum',
        'column_two' : 'max',
        'column_three' : 'count'
    })

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

    pd = pd.read_csv('./file_path/file_name.csv')

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