## Introduction

Recently I needed to populate several SQL tables from an API of questionable quality. As I was inserting over a million records over several tables, a speedy solution was essential. Since the code base was in .net core, I chose to use Boris Djurdjevic's excellent tool [EFCore.BulkExtensions]('https://github.com/borisdj/EFCore.BulkExtensions'). As past data sometimes changes without warning, I used the `BulkInsertOrUpdateOrDelete` option. This allowed me to update over 800k rows with 40+ columns in less than a minute. 

## The problem

Unfortunately the API sometimes provided duplicate rows (every value in every column was the same) causing the tool to throw the following exception:

```
The MERGE statement attempted to UPDATE or DELETE the same row more than once. This happens when a target row matches more than one source row. A MERGE statement cannot UPDATE/DELETE the same row of the target table multiple times. Refine the ON clause to ensure a target row matches at most one source row, or use the GROUP BY clause to group the source rows.'
```

Because of this, using `BulkInsertOrUpdateOrDelete` was no longer an option. Instead I decided to clear the previous data, use `BulkInsert` and then deleted the duplicate rows afterwards. 

Given that I had 12 tables to update with 40+ columns, I didn't want to write each `group by` by hand. So instead, I created a dynamic query that will work on any table.

## SQL

```SQL
CREATE PROCEDURE dbo.RemoveDuplicatesFromTable
    @table varchar(255)
AS SET NOCOUNT ON;
	declare @columns varchar(max)
	select @columns = COALESCE(@columns+',' ,'') + Name
		FROM sys.columns 
		WHERE object_id = OBJECT_ID(@table)

	declare @sql nvarchar(max) = ''
	+ 'WITH CTE(' + @columns + ', DuplicateCount)'
		+ 'AS (SELECT ' + @columns + ', ROW_NUMBER() OVER(PARTITION BY ' + @columns + ' Order by (SELECT NULL)) AS DuplicateCount' + ' FROM ' + @table + ')'
	+ 'DELETE FROM CTE WHERE DuplicateCount > 1;'
	exec sp_executesql @sql
GO  
```