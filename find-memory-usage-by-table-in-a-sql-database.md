## The Problem

Sometimes, when working with new clients they won't understand why their database is so large. Often this is caused by excessive logging, storing documents, images, or videos. Running this script helps put the problem into quantifiable terms so that we can begin making more informed decisions.

## Credits

Every six months or so, I find myself searching for this query by marc-s. Because of this, I wanted to preseve it somewhere I can easily find it. Full credit for this solution goes to [marc-s](http://stackoverflow.com/users/13302/marc-s).

You can find the origional [stackoverflow article here](http://stackoverflow.com/questions/7892334/get-size-of-all-tables-in-database).

## The Code

[code language="sql"]

    SELECT 
        t.NAME AS TableName,
        s.Name AS SchemaName,
        p.rows AS RowCounts,
        SUM(a.total_pages) * 8 AS TotalSpaceKB, 
        SUM(a.used_pages) * 8 AS UsedSpaceKB, 
        (SUM(a.total_pages) - SUM(a.used_pages)) * 8 AS UnusedSpaceKB
    FROM 
        sys.tables t
    INNER JOIN      
        sys.indexes i ON t.OBJECT_ID = i.object_id
    INNER JOIN 
        sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
    INNER JOIN 
        sys.allocation_units a ON p.partition_id = a.container_id
    LEFT OUTER JOIN 
        sys.schemas s ON t.schema_id = s.schema_id
    WHERE 
        t.NAME NOT LIKE 'dt%' 
        AND t.is_ms_shipped = 0
        AND i.OBJECT_ID > 255 
    GROUP BY 
        t.Name, s.Name, p.Rows
    ORDER BY 
        t.Name
	

[/code]