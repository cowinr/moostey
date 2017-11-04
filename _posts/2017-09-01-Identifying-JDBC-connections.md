---
layout: default
title: "Identifying JDBC Connections in an Oracle Database"
date: 2017-09-01
---

By default a JDBC connection identifies itself in the database as "JDBC Thin Client". For a large set of deployments, across dev, test, preprod, prod etc. it can be tricky finding where a malfunctioning connection is configured.

In Oracle, selecting from the `V$SESSION` table gives useful information about what is connecting to a database.
 
With Tomcat connection pools you can use `connectionProperties` to set the `V$SESSION.PROGRAM` column to the server instance name for example:
 
```xml    
<Resource name="my_connection"
      username="username"
      ...
      connectionProperties="v$session.program=tomcat_server_name"
      validationQuery="SELECT 1 FROM DUAL"
/>
```
 
Et voila, no more "JDBC Thin Client"...

```sql
SELECT SCHEMANAME, STATUS, OSUSER, PROGRAM FROM V$SESSION
```

| SCHEMANAME | STATUS | OSUSER | PROGRAM |
|----|
| TEST | INACTIVE | username | tomcat_server_name |
