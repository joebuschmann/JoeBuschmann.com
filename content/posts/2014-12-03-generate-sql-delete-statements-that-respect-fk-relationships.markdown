---
layout: post
title: Generate SQL Delete Statements That Respect FK Relationships
date: '2014-12-03 21:00:00'
tags:
- c
- sql
---

Have you ever needed to delete a record from a database table only to be thwarted by one or more foreign key violations? Then when you try to delete records from the child tables you find a deep hierarchal relationship bound together by yet more foreign keys?

I recently ran into this issue with a SQL Server database. I needed to trim data from several large tables that had deep and wide relationship hierarchies enforced by foreign keys. To solve this problem, I wrote a short C# command line program that outputs a SQL script where records from child tables are deleted before their corresponding records in the parent tables.

For example, given the simple table relationships below.

![Order table diagram](http://media.joebuschmann.com/order_table_diagram.png)

When I compile the program to GenerateDeleteStatements.exe and run the command:

```
C:\>GenerateDeleteStatements.exe "DATABASE=order;SERVER=localhost;UID=sa;PWD=fakepwd" "order" "" results.sql
```

Then the following SQL script will be saved to results.sql. Notice how records from child tables are removed before parent tables per the foreign key relationships.

```
delete [order_item_history] from [order_item_history]
	join [order_item] on [order_item].[id] = [order_item_history].[order_item_id]
	join [order] on [order].[id] = [order_item].[order_id]

go

delete [order_item] from [order_item]
	join [order] on [order].[id] = [order_item].[order_id]

go

delete from [order] 

go
```

I can also provide an optional where clause to remove specific records.

```
C:\>GenerateDeleteStatements.exe "DATABASE=order;SERVER=localhost;UID=sa;PWD=fakepwd" "order" "where [order].date_created < '1-1-2005'" results.sql
```

```
delete [order_item_history] from [order_item_history]
	join [order_item] on [order_item].[id] = [order_item_history].[order_item_id]
	join [order] on [order].[id] = [order_item].[order_id]
where [order].date_created < '1-1-2005'

go

delete [order_item] from [order_item]
	join [order] on [order].[id] = [order_item].[order_id]
where [order].date_created < '1-1-2005'

go

delete from [order] where [order].date_created < '1-1-2005'

go
```

Hopefully someone else finds this useful. The full source is below.

<script src="https://gist.github.com/joebuschmann/7c6f98c133aa15bd6fb8.js"></script>
