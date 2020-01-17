---
layout: post
title: A Poor Man's Queue
date: '2019-01-10 14:29:33'
tags:
- tsql
- sql
- sql-server
---

Any sufficiently large enterprise software application is going to need a queue at some point. A queue is a good way to introduce an asynchronous process and decouple two parts of a system. For example, a user could upload a large file to a web application for processing, and instead of making the user wait for the work to complete, the application could queue up the work and return immediately. Later, when the results are ready, the user could be notified.

The best way to do this is to introduce a message broker like [RabbitMQ](https://www.rabbitmq.com/). With RabbitMQ, not only do you get simple queues with a single subscriber, but you can set up exchanges with multiple subscribers. But taking the leap with an enterprise message broker may be a step too far. Sometimes all you need is a simple queue without the overhead of introducing new middleware.

You can do this with a SQL Server table acting as a queue. It's not sexy, but it gets the job done.

## A Table as a Queue

Implementing a queue backed by a table seems simple enough but can be surpisingly difficult in practice. The solution is strightforward if there is only one process dequeuing work. As soon as concurrency is introduced with multiple processes, excessive blocking and deadlocks will cause problems if the queries are not structured correctly.

The following table structure could be used for a simple message queue.

```
CREATE TABLE [Queue]
(
	[Id] INT IDENTITY(1,1) NOT NULL,
	[Status] TINYINT NOT NULL, /* 1 - READY, 2 - IN PROGRESS */
	[Message] VARBINARY(MAX)
)
```

Notice the `Status` column with values of 1 (READY) and 2 (IN PROGRESS). When a message is dequeued, the status is set to 2 while it is being processed. Eventually message handling completes, and the record is deleted.

The stored procedure to enqueue a message is straightforward.

```
CREATE PROCEDURE EnqueueMessage
	@message VARBINARY(MAX)
AS
BEGIN
	INSERT INTO [Queue] ([Status], [Message])
	VALUES (1, @message)
END
```

The proc for dequeuing a message is where things get interesting. You might be tempted to write a procedure that does the following.

1. Select the next row.
2. Update the selected row's status to 2.
3. Return the Id and Message values.

Implementing these steps as three separate DML statements introduces a race condition between steps one and two. Two transactions could select the same row at the same time (step 1) before either one updates it. Next, one transaction updates the row (step 2) while the other is blocked. Finally, the second transaction performs the same update and both return the same message to different consumers (step 3). This approach won't work in highly concurrent applications.

## A Better Way to Dequeue

A better approach would combine all three steps into one DML statement. You can do this with the `OUTPUT` clause, a CTE, and the `ROWLOCK` and `READPAST` hints.

```
CREATE PROCEDURE DequeueMessage
AS
BEGIN
	;WITH NextMessage AS
	(
		SELECT TOP 1 *
		FROM [Queue] WITH (ROWLOCK READPAST)
		WHERE [Status] = 1
		ORDER BY Id
	)
	UPDATE NextMessage
	SET [Status] = 2
	OUTPUT
		inserted.[Id],
		inserted.[Message]
END
```

Let's break this down starting with the `OUTPUT` clause. Introduced in SQL Server 2005, the `OUTPUT` clause combines an `UPDATE` and `SELECT` into a single operation. We can use it to combine steps two and three into one query. Next, the CTE combines the `SELECT` statement, which identifies the next message, and the `UPDATE` statement into a single operation. The CTE acquires an update lock which precludes any other update locks on the same record. No other transaction can grab that record while the update is in progress. Finally, the `ROWLOCK` and `READPAST` hints are crucial for preventing blocking from multiple consumers.

With this approach, three queries are combined into one efficient non-blocking query. Also, note the `TOP` and `ORDER BY` clauses to ensure one message is returned in the right order. Interestingly, this query could be updated to sort in decending order for a stack implementation. Check out [Using Tables as Queues](http://rusanu.com/2010/03/26/using-tables-as-queues/) for a more in depth discussion of how these pieces all come together.

Once a consumer has handled the message, it can invoke the following stored procedure to remove its corresponding record from the table.

```
CREATE PROCEDURE CompleteMessage
	@id INT
AS
BEGIN
	DELETE FROM [Queue] WHERE [Status] = 2 AND [Id] = @id
END
```

That completes the implementation. I've used this successfully in production systems with no issues. Again, I recommend a more robust broker like RabbitMQ, but a table as a queue will work in a pinch.