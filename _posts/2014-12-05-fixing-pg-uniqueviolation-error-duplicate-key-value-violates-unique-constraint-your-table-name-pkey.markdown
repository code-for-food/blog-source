---
layout: post
title: "Fixing: PG::UniqueViolation: ERROR: Duplicate Key Value Violates Unique Constraint 'Your_table_name_pkey'"
date: 2014-12-05 10:04:09 +0700
source_name: fixing-pg-uniqueviolation-error-duplicate-key-value-violates-unique-constraint
source_site: http://blog.thekindof.me/blog/2013/12/05/fixing-pg-uniqueviolation-error-duplicate-key-value-violates-unique-constraint/
comments: true
categories: [database, rails]
author: chicken-dp 
---

##### Issue

Postgresql maintain a internal value which is the max value of the id column of a given table. And uses this to generate values for primary keys when you insert new records. Sometimes this internally stored value can get messed up. And that can cause the error above when you are trying to insert a new record to the table.

##### How to fix ????

You need to basically correct the above said value.

Access to your database and run :

```
	pg_dump -U user_name database_name -f database_backup_name.sql

```

Change your rails app and run :

```
	cd /your/rails/app/root/
	rails db production
```

Update your_table_id_sql

```
	SELECT setval('your_table_id_seq', (SELECT MAX(id) FROM your_table));
```

replace your_table with whatever the table name you are using. And the code above assumes your primary key column name is id.

Good lucks to you :) 
