---
layout: post
title: Row-Level Security in SQL Server
date: '2017-03-01 14:18:28'
tags:
- sql
- sql-server
---

Up until version 2016, Microsoft's SQL Server didn't support row-level security (RLS) - a feature that's been around in Oracle and IBM's offerings for a while. If you ran SQL Server and needed RLS, you had to build your own scheme which is exactly what I had to do for a recent project. I needed RLS for a new data warehouse I was building using SQL Server 2014. The requirements called for storing sensitive financial and sales data from multiple users in the same tables. I needed a scheme where a user could connect and extract just their data without seeing information for any others. Without native RLS, it is doable but takes some work.

In this post, I'll cover the approach I settled on and then discuss my plan for migrating to native RLS in SQL Server 2016.

#### Table Structure

My first step was to add a column to each table named **UserId** to identify the owner of the row. Its value could be used to filter out rows belonging to other users. For example, the **baseAccount** table below represents a typical setup.

```
CREATE TABLE [dbo].[baseAccount](  
    [UserId] [dbo].[UserId] NOT NULL,
    [Id] [varchar](18) NOT NULL,
    [Name] [nvarchar](100) NOT NULL,
    [CreatedDate] [datetime] NOT NULL CONSTRAINT [DF_baseAccount_CreatedDate]  DEFAULT (getdate()),
 CONSTRAINT [PK_Account] PRIMARY KEY CLUSTERED
(
    [UserId] ASC,
    [Id] ASC
))
```

I left out most of the columns, but this gives you a general idea of how each table was structured. I added the prefix **base** to distinguish each table from its corresponding view. I also created a simple user-defined data type for the UserId column to enforce consistency.

```
CREATE TYPE [dbo].[UserId] FROM [nvarchar](10) NOT NULL
GO
```

#### Views

With the tables in place, the next task was to create a view for each table. This is were the UserId column comes into play. It is used to filter data by the current user and ensure the user can see only their data. The where condition makes an exception for database owners who have access to all data.

```
CREATE VIEW [dbo].[Account]
AS
SELECT * FROM dbo.baseAccount
WHERE USER_NAME() = 'dbo' OR dbo.baseAccount.UserId = USER_NAME()
```

#### Restricting Access

Next I restricted database access by creating a common role for all of the users in the data warehouse. This new role was granted SELECT rights to the views but not their corresponding tables. This forced all data access to go through the views where the RLS filtering took place.

```
CREATE ROLE [useraccess];

-- Allow access to the view but not the table
GRANT SELECT ON [Account] TO [useraccess];
```

With the new role in place, it was time to create the logins and users. Each user had a dedicated SQL Server login. For example, if John Smith had the Windows login **somedomain\jsmith**, then his login was mapped to a user in the database named **jsmith**.

```
-- Create a new login
CREATE LOGIN [somedomain\jsmith] FROM WINDOWS;

-- Map a user to the login
CREATE USER [jsmith] FOR LOGIN [somedomain\jsmith] WITH DEFAULT_SCHEMA=[dbo];

-- Add him to useraccess role to access the views
ALTER ROLE [useraccess] ADD MEMBER [jsmith];
```

At this point I had a simple row-level security scheme. When John Smith queries the database with his Windows account, he will only see data for his user. All other rows will be filtered out by the views.

#### Native RLS in SQL Server 2016

This was a lot of work for something that should be native to any mainstream DBMS. SQL Server 2016 fixes this problem with the introduction of security policies for row-level security. Fortunately, the RLS setup I've described so far can be changed fairly easily to use security policies. The logins, users, and roles remain the same, but the views are replaced with a security policy per table. Each security policy uses a function to determine row-level access.

###### Views and Tables

The views are no longer needed and can be dropped. The **base** prefix is removed from each table name, and SELECT permission is granted for the **useraccess** role.

```
DROP VIEW dbo.[Account];

EXEC sp_rename 'dbo.baseAccount', 'Account';

GRANT SELECT ON [Account] TO [useraccess];
```

###### Filter Predicate

SQL Server 2016 uses a function to determine access to a row. In this case, the function checks the value for the UserId column against the current user name. Oddly, instead of returning a boolean value, it needs to return a table with a single row with a value of 1 for a match and nothing otherwise.

```
CREATE FUNCTION dbo.UserAccess
( @userId AS [nvarchar](10) )
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS AccessRight
    WHERE USER_NAME() = 'dbo' OR USER_NAME() = @userId
```

###### Security Policy

Finally, a security policy is applied per table to implement RLS.

```
CREATE SECURITY POLICY AccountSecurityPolicy
ADD FILTER PREDICATE dbo.UserAccess(UserId) ON dbo.Account
WITH (STATE = ON)
```

<hr />

So that's a example of RLS implemented with and without security policies. I have yet to use SQL Server 2016 in production, but I'm looking forward to the simplicity that native RLS will bring.