---
layout: post
title: A Shared Development Database is Bad...Very Bad
date: '2015-02-17 23:52:12'
tags:
- best-practices-2
---

![Where are your DB create scripts?](http://media.joebuschmann.com/where_are_your_db_create_scripts.png)

The software development group at my office has a shared database that developers connect to for their daily work. In the last few months, the number of developers has doubled, and the practice of using the shared database has not scaled well. It has caused frustration as members of one team make schema changes before another team can consume them. The resulting breakages has led to lost productivity. To make matters worse, the same database serves as a reference for the daily migration process. There's always a chance that a junior developer (or anyone really) could make a mistake and kill migrations.

That gave me an idea for a post on why a shared database is bad. Really bad. Terribly bad. But a Google search showed others coming to the same conclusion, so instead of reiterating their points, I've included the links below.

__In short, instead of using a shared database, put your schema and any seed data into source control.__

[The unnecessary evil of the shared development database](http://www.troyhunt.com/2011/02/unnecessary-evil-of-shared-development.html)

[Top 4 Reasons Why a Shared Development Database is Evil](http://www.benday.com/2011/01/25/top-4-reasons-why-a-shared-development-database-is-evil/)

[Three Rules for Database Work](http://odetocode.com/blogs/scott/archive/2008/01/30/three-rules-for-database-work.aspx)