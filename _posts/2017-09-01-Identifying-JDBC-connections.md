By defauklt a JDBC connection identifies itself in the database as "JDBC Thin Client".
For a large set of deployments, across dev, test, preprod, prod etc. it can be tricky finding where
a malfunctioning connection is configured.

In Oracle, selecting from the `V$SESSION` table gives useful information about what is connecting to a database.
 
With Tomcat connection pools you can set the `V$SESSION.PROGRAM` column to the server instance name for example:
 
```xml    
<Resource name="my_connection"
      username="username"
      ...
      connectionProperties="v$session.program=tomcat_server_name"
      validationQuery="SELECT 1 FROM DUAL"
/>
```
 
Et voila, no more "JDBC Thin Client"...
 
