---
layout: post
title: View Generated Queries From EF Core
date: 2023-03-14 12:26 +0300
categories: [efcore]
tags: [csharp, dotnet, efcore, sql, performance] 
---
Debugging SQL queries can be a challenging task, particularly when trying to identify issues with an Entity Framework Core application. Fortunately, there are effective ways to view generated SQL queries from EF Core. These techniques can be beneficial for detecting and fixing issues. By applying these approaches, it becomes simpler to identify problems with our Entity Framework Core application. In this article, we'll explore some of the most common methods for viewing generated SQL queries.

## Debugging

The simplest way to view generated SQL queries is to use a debugger. We can set a breakpoint to the method that interests us and see what's inside `DebugView -> Query`


![Debug View](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5stzypvuvphko9yyzlth.png)



Alternatively, we can assign this value to a local variable with the `ToQueryString()` method of `IQueryable`.

```csharp
var sql = query.ToQueryString();
```

The generated query looks like this:

![Generated Query](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yta5xhd764l36x6kcquw.png)




## Logging

EF Core provides a built-in logging mechanism that can be used to log generated SQL queries to the console, a file, or any other logging provider supported by .NET. We can enable logging with these two ways:

### Using a logging provider

To use this approach, we need to add a logging provider and configure the logging options in our application's startup code. For example at our `Program.cs` file we can do the following :

```cs
var loggerFactory = LoggerFactory.Create(builder =>
{
    builder
        .AddFilter((category, level) =>
            category == DbLoggerCategory.Database.Command.Name
            && level == LogLevel.Information)
        .AddConsole();
});

builder.Services.AddDbContext<TodoDbContext>(opt => 
    opt.UseSqlServer(connectionString)
       .UseLoggerFactory(loggerFactory));
```

Here we use Microsoft's logging framework to create a logger factory that outputs log messages to the console with the `.AddConsole()` method and we add a filter that only logs messages with a `DbLoggerCategory.Database.Command.Name` category and a `LogLevel.Information` level.

### Using the LogTo method

Another approach is to use the `LogTo()` method of the `DbContextOptionsBuilder`. For example:

```cs
builder.Services.AddDbContext<TodoDbContext>(opt => 
opt.UseSqlServer(connectionString)
   .LogTo(Console.WriteLine,LogLevel.Information));
```

The `LogTo()` method takes an `Action` as a parameter, that specifies where log messages should be sent, and a `LogLevel` value that specifies the minimum log level at which messages should be sent.

In both approaches the results will show up in the console like this : 

```sql
info: Microsoft.EntityFrameworkCore.Database.Command[20101]  
Executed DbCommand (28ms) [Parameters=[@__p_0='?' (DbType = Int64)], CommandType='Text', CommandTimeout='30']  
SELECT TOP(1) [t].[Id], [t].[IsComplete], [t].[Name]  
FROM [TodoItems] AS [t]  
WHERE [t].[Id] = @__p_0
```

## LINQPad

LINQPad is a popular tool for writing and testing LINQ queries in .NET applications. It also includes a feature for visualizing and analyzing generated SQL queries from EF Core, making it a useful tool for debugging and optimizing database access.

To use LINQPad with EF Core, we follow these steps:

1) Click at the left top corner  on `Add connection`. 

![Add connection](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/icbqlontqbc9uwe8hnn1.png)

2) Select 'Entity Framework Core' on the Data Context prompt


![Data Context prompt](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ecbrqoa3wm5xr0gvkm75.png)



3) We add the necessary information to connect to our database and test the connection by clicking `Test`.


![connect to database](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zno13s40pxdrmfstjabi.png)


4) After pressing the `OK` button , the database gets populated. To visualize the generated SQL query for a LINQ query, we simply run the query and view the SQL output in LINQPad.

For example:

![view the SQL output in LINQPad](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pdxnd3rphazdex8jhie7.png)


## Conclusion

In conclusion, there are several ways to see generated SQL queries from EF Core. Debugging and Logging are simple and effective ways to see SQL queries during development, and LINQPad is a useful tool for quick and ad-hoc debugging and optimization of database access. By using one of these methods, we can improve the performance and efficiency of our EF Core applications and ensure that they are working as expected.