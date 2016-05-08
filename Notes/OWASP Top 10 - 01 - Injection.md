# OWASP Top 10 #1 - Injection #

## What is it? ##

An injection attack is where a program, database or interpreter is tricked into executing code or performing functionality which was not intended to be run.

Though SQL injection is probably the most commonly known type of injection attack, it is a vulnerability that can affect many technologies;

- SQL
- XPath
- LDAP
- Applications / Operating Systems

SQL injection is when a query or command parameter which should contain a simple variable value actually contains additional syntax which is inadvertently executed by the database.

## How does it happen? ##

For many web applications a URL such as http://www.TheFooSite.com/MyDatabaseTable?Id=1 would translated into a query such as:

```
Select * From MyDatabaseTable Where Id = 1
```

The integrity of the value of the id has come from the user and as such can contain potentially any malicious payload.

For example, we can change the sql query to return all data within the table:

```
http://www.TheFooSite.com/MyDatabaseTable?Id=1 or 1=1
```

Alternatively we could make the database throw an error which could potentially propagate to the user and show our technology stack, this in turn would invite any potential hacker to start a more in depth probe:

```
http://www.TheFooSite.com/MyDatabaseTable?Id=x
```

Additionally we could append on queries which query the schema tables such sysobjects and could start to expose our database structure to the user. This could then be used to append queries which exposes the potentially sensitive data contained within these tables such as credit card numbers and identity numbers:

```
http://www.TheFooSite.com/MyDatabaseTable?Id=SELECT name FROM master..sysobjects WHERE xtype = 'U';
```

###Untrusted data sources###

Any information which comes from the user, third party or external entity should be considered as untrusted and therefore potentially dangerous. 

The sources of untrusted data are numerous:

- From the user
 - In the URL via a query string or route parameter
 - Posted via a form
- From the browser
 - In cookies
 - In the request headers
 - Web services calls
 - Database
 - Files such as XML, XSD, CSV

In fact anything which does not come directly from our running source code should be treated as potentially malicious. That includes data and services from within the bounds of our company as well as data entered directly from our users, even if they are our colleagues.

## How can I protect myself from this? ##

Security is all about addressing the security concern at multiple levels; this way if one level is breached there are more levels which can catch the breach.

###The principle of least privilege###

The principle of least privilege is about giving any entity that requires permissions to perform a job, the minimum level of permissions it requires to perform that job.

Why grant access for the user account which is used by a web application to query an entire database, delete data, modify schema or even grant administrator level access? By granting the minimum level of permissions you dramatically reduced the level of damage that a breach can perform.

Don't  give the account db_owner access; this can read, update, delete, truncate and drop any table as well as execute any stored procedures and modify the schema.
 
A user account should be given access to:
 - Read from only the individual table(s) required
 - Write to only the individual table(s) required
 - Execute only the required stored procedures required

Any account accessed by a web application should probably never be given access to modify any schema, read or write to any schema or system tables and should probably not even be given access to physically delete data from any tables. 

If you need to delete data, consider marking records as deleted by a status field and archiving off the records by a system task running in the background.

Where different levels of access is required between types of user account groups or roles, consider having multiple accounts. For example a public account used by public user groups and an admin account which is used by back office staff. 

**Please note:** just because the user role is 'admin' it does not mean they should be given anything but the bare minimum permissions within the database.

### In line SQL parametrisation ###

Most issues from SQL injection are from parameters being attached to the query or command via string concatenation. Take the following example which concatenates a parameter taken from the user via a URL query string parameter.

```
var id = Request.QueryString["Id"];
var sqlCommand = "SELECT * FROM MyTable WHERE id = " + id;
```

The data within the parameter is run as is within the database allowing the data within the parameter to break out of its parameter scope and into a query scope.

For example  "1 or 1=1" will be searched upon as if it has two parts of the predicate.

By parametrising the query, the data contained within the parameter will stay within the scope of the parameter; here we will try and search for an id which is equal to "1 or 1=1" which at some level will either retrieve no data or throw an error when trying to convert between data types.

The following shows how we can parametrise an line query: 
```
var id = Request.QueryString["id"];

var sqlCommand = "SELECT * FROM Foo WHERE Id = @id";

using (var connection = new SqlConnection(connString))
{
    using (var command = new SqlCommand(sqlString, connection))
    {
        command.Parameters.Add("@d", SqlDbType.VarChar).Value = id;
        command.Connection.Open();
        var data = command.ExecuteReader();
        }
      }
```

We now have an error thrown within by the database as it cannot convert the string "1 or 1=1" into an integer; the data type of the id field.

### Stored procedure Parametrisation ###

The problem with in line parametrised queries is that you have to remember to do them. By using stored procedures you are enforced to perform this step.

```
var id = Request.QueryString["id"];

      var sqlString = "GetFoo";
      using (var conn = new SqlConnection(connString))
      {
        using (var command = new SqlCommand(sqlString, conn))
        {
          command.CommandType = CommandType.StoredProcedure;
          command.Parameters.Add("@id", SqlDbType.VarChar).Value = id;
          command.Connection.Open();
          var data = command.ExecuteReader();
        }
      }
```

This will cause a similar issue that our bad string cannot be converted to an integer.

### White listing untrusted data ###

Where possible, all untrusted data should always be validated against a white list of known safe values.

A white list states a set of rules which defines the data. Any incoming data must be validated against these rules to be considered safe. Failure of the validation should interpret the data as an expected attack.

There are multiple ways of performing white list:

- Type conversion
 - Integer, date, GUID
- Regular expression
 - Email address, phone number, name
- Enumerated list of known good values
 - Titles, countries and colours for example
 
The following shows how we can use the try parse function to white list our id parameter by trying to convert it to an integer.

```
int idString;
if(!int.TryParse(idString, out id))
{
 throw new Exception("The id is not a valid integer.");
}
```

**Note:** any errors should be handled, where required audited and then the user should be displayed a human readable message which gives nothing away about our implemented technology stack or in-depth details of the error. 

Now that we have the parameter as  integer we can add our parameter into the command as an integer:

```
command.Parameters.Add("@id", SqlDbType.Int).Value = id;
```
Any suspect attacks now should be caught way before the database which is a good place to be.

### ORM's ###

ORM's should automatically parametrise any SQL.

The only thing to note is that when using ORM's it is often required to drop to native SQL for performance reasons. When doing this care should be made to ensure all untrusted data is parametrised using the steps above.

##Injection through stored procedures##

Care should be taken when using stored procedures that they are not themselves concatenating parameters before calling exec.

```
declare @query varchar(max)
set @query = 'select * from dbo.MyTable where MyField like ''%'+ @parameter + '%''
EXEC(@query)
```

The stored procedure sp_executesql allows parametrising sql statements which are held as string.

```
declare @query varchar(max)
set @query = 'select * from dbo.Foo where MyField like ''%'+ @myFieldParameter + '%''

EXEC sp_executesql @query, N'@search varchar(50)', @myFieldParameter=@parameter
```

## Automated SQL injection tools ##

There are a number of tools which can automate the hacking process, allowing people with next to no technical experience to perform automated attacks or to test the strength of web applications.

http://www.hackingarticles.in/best-of-hacking/best-sql-injection-tools/

