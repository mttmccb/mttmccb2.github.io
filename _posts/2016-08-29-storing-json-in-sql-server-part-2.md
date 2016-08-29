---
title: "Storing JSON in SQL Server Part 2"
excerpt: "A bit more on this..."
tags: [sql server 2016]
---

I've given the [/storing-json-in-sql-server/](previous post) a little more thought, which helps to solve the problem of storing in the ubiquitous JSON format on SQL Server.

It doesn't go as far as having a special JSON format, and for backward compatibility it uses NVARCHAR but you can set constraints on that column to ensure it's valid JSON.

You can also store related information a little easier, like phone numbers or emails, rather than using a separate table and then joining that later, increasing complexity and potentially reducing performance. On top of that you can add check on those properties to make sure they're valid.

Finally you can compress these columns (using gzip).

All these together make this a compelling addition, particularly if you were considering a NoSQL database, a hybrid approach might work well too.