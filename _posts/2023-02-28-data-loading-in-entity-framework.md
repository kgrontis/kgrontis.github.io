---
title: Data Loading in Entity Framework
date: 2023-02-28 15:33:12 +0300
categories: [CSHARP, DOTNET]
tags: [csharp, efcore, performance]     # TAG names should always be lowercase
---
# Data Loading in Entity Framework

Data retrieval is an important aspect of designing efficient applications in software development. It is common for applications to require data from a database in order to function. Data access is a critical performance factor that must be carefully considered in this regard. The three common methods of loading data from a database are lazy, eager, and explicit loading. In this article, we will look at the differences between these methods, as well as their benefits and drawbacks. We'll also look at how to use Entity Framework in C# to implement them.

### Lazy Loading

Lazy loading is a technique that allows us to defer the loading of related objects until they are accessed. In other words, we only load the required data when we need it, rather than loading everything at once. 

Let's consider an example to understand lazy loading better. Suppose we have two entities, `Author` and `Book`, and we want to retrieve all authors and their corresponding books. With lazy loading, we would first retrieve all authors from the database and then retrieve the books for each author only when needed. This approach is beneficial because it reduces the number of queries sent to the database and avoids loading unnecessary data.

We are going to see how to implement lazy loading in EF Core:

The first method is by installing a Proxy Package provided by Microsoft. All you have to do is install `Microsoft.EntityFrameworkCore.Proxies` package which will add all the required proxies needed to run Lazy Loading and enabling it with a call to `UseLazyLoadingProxies`. 

This can be achieved either in `OnConfiguring` method of your Context class:
```cs 
using Microsoft.EntityFrameworkCore;

namespace DataLoading;

internal class MyContext : DbContext
{
    protected override void OnConfiguring
       (DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseLazyLoadingProxies()
            .UseInMemoryDatabase(databaseName: "MyDb");
    }
}
```

or when using AddDbContext most likely in the `Program.cs`:
```cs
builder.Services.AddDbContext<MyContext>(
    b => b.UseLazyLoadingProxies()
          .UseInMemoryDatabase("MyDb"));
```

Any navigation property that can be overridden and is virtual and on a class that can be inherited from will then have lazy loading enabled by EF Core. The Author.Books and Book.Author navigation properties, for instance, will be lazy-loaded in the following entities.
```cs
public class Author
{
    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Book> Books { get; set; }
}

public class Book
{
    public int Id { get; set; }
    public int AuthorId { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public virtual Author Author { get; set; }
}
```

In the example above, we use the virtual keyword to mark the `Books` property as virtual, indicating that it should be lazily loaded. When we retrieve authors from the database, Entity Framework will generate SQL queries to retrieve only the authors, and not the books. When we access the `Book` property for a specific `Author`, Entity Framework generates another SQL query to retrieve the corresponding books.

The second method is by injecting the `ILazyLoader` service to an entity.
Lets look at an example:

```cs
public class Author
{
	private ICollection<Book> _books_;
	public Author()
	{
	}
	private Author(ILazyLoader lazyLoader)
	{ 
		LazyLoader = lazyLoader;
	}

	private ILazyLoader LazyLoader { get; set; }

    public int Id { get; set; }
    public string Name { get; set; }
    public virtual ICollection<Book> Books
    { 
	    get => LazyLoader.Load(this, ref _books); 
	    set => _books = value; 
	}
}

public class Book
{
	private Author _author;

	public Book()
	{
	}
	private Book(ILazyLoader lazyLoader)
	{
		LazyLoader = lazyLoader;
	} 
	private ILazyLoader LazyLoader { get; set; }
	
    public int Id { get; set; }
    public int AuthorId { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public virtual Author Author
    { 
	    get => LazyLoader.Load(this, ref _author);
	    set => _author = value;
	}
}
```

This technique enables entity instances generated with `new` to lazy-load once they are attached to a context and does not require entity types to inherit from, or navigation properties to be virtual.

Now , when you query for an `Author` entity , EF will generate an SQL query that looks like this:
```sql
SELECT [Id], [Name]
FROM [Authors]
WHERE [Id] = @id
```
This query only selects the `Id` and `Name` columns from the `Authors` table, as EF knows that it doesn't need to load the `Books` collection yet.

Then, when you access the `Books` property on the `Author` entity, EF will generate another SQL query to load the related `Book` entities:

```sql
SELECT [Id], [Name], [Price], [AuthorId] 
FROM [Books] 
WHERE [AuthorId] = @authorId
```

Similarly, when you query for a `Book` entity, EF will generate a SQL query that looks like this:

```sql
SELECT [Id], [Name], [Price], [AuthorId] 
FROM [Books] 
WHERE [Id] = @id
```

This query only selects the `Id`, `Name`, `Price`, and `AuthorId` columns from the `Books` table, as EF knows that it doesn't need to load the related `Author` entity yet.

Then, when you access the `Author` property on the `Book` entity, EF will generate another SQL query to load the related `Author` entity:

```sql
SELECT [Id], [Name] 
FROM [Authors] 
WHERE [Id] = @authorId
```

This query selects the `Id` and `Name` columns from the `Authors` table, which is all that's needed to populate the `Author` property on the `Book` entity.

### Eager Loading

The opposite of lazy loading is eager loading. When we retrieve data from the database using eager loading, we load the primary object as well as all the related objects. When we are certain that we will want all the relevant data and wish to reduce the number of database queries, this strategy is advantageous.

To further understand eager loading, let's look at an example. Let's say we have the same two entities, `Author` and `Book`, and we want to use eager loading to get a list of all authors and their books. 
Here is an example of how to implement eager loading using Entity Framework in C#:

```cs
// To load authors with eager loading
using (var context = new MyContext())
{
    var authors = context.Authors.Include(author => author.Books).ToList();
    foreach (var author in authors)
    {
        Console.WriteLine(authors.Name);
        foreach (var book in author.Books)
        {
            Console.WriteLine(book.Name);
        }
    }
}   

```


The Include method is used in the example above to express the desire to eagerly load the `Books`property for each `Author`. The `Authors` and `Books` tables are joined together by Entity Framework to provide a single SQL query that returns all the necessary information.

```sql
SELECT [a].[Id], [a].[Name], [b].[Id], [b].[Name], [b].[Price], [b].[AuthorId]
FROM [Authors] AS [a]
LEFT JOIN [Books] AS [b] ON [a].[Id] = [b].[AuthorId]
WHERE [a].[Id] = @authorId
```

It is worth mentioning that you can include multiple levels of related data using the `ThenInclude` method. For example:
```cs
using (var context = new MyContext())
{
    var authors = context.Authors
	    .Include(author => author.Books)
	    .ThenInclude(book => book.CoverImage)
	    .Include(author => author.ContactDetails)
	    .ToList();
} 
```

Additionally, we can configure to always include a navigation in a model, by using the `AutoInclude` method on the modelBuilder.

```cs
modelBuilder.Entity<Author>().Navigation(author => author.Books).AutoInclude();
```

Use the `IgnoreAutoIncludes` method in your query if you don't want to load associated data through a navigation that is defined at the model level to be automatically included. The user-configured auto-include navigations will all cease loading if this approach is used. Using the query below will retrieve all authors from the database, but Books won't load even if it is set to `AutoInclude`.

```cs
using (var context = new MyContext()) 
{ 
	var authors = context.Authors.IgnoreAutoIncludes().ToList(); 
}
```

We have to be extra cautious when we use eager loading on collection navigation, because it might cause serious performance issues.

### Explicit Loading

After the main object has been loaded, explicit loading allows you to load related entities as needed. By limiting the amount of data that needs to be imported from the database, it offers a mechanism to selectively load relevant entities only when they are required, which can enhance efficiency.

In explicit loading, you use the `Load` method on a `CollectionEntry` or `ReferenceEntry` object to explicitly load the related entities. A `CollectionEntry` object represents a collection navigation property on an entity, while a `ReferenceEntry` object represents a reference navigation property on an entity.

When you don't want to suffer the overhead of loading associated entities each time the primary entity is loaded, explicit loading can be helpful. For instance, you can use explicit loading to load only the related entities you require at the moment if you have a lot of entities but only occasionally need to access related entities.

Lazy loading must be enabled for explicit loading to function because it depends on it. Calling `Load` on a `CollectionEntry` or `ReferenceEntry` object has no effect if lazy loading is not enabled.

Lets see an example and the SQL query generated by EF Core:

```cs
var author = dbContext.Authors.Find(authorId);
dbContext.Entry(author).Collection(a => a.Books).Load();
```

Assuming that lazy loading is enabled, the initial query to retrieve the `Author` entity would look something like this:

```sql
SELECT [a].[Id], [a].[Name]
FROM [Authors] AS [a]
WHERE [a].[Id] = @authorId
```

When `Load` is called on the `CollectionEntry` object for the `Books` collection, Entity Framework generates a separate query to retrieve the related `Book` entities:

```sql
SELECT [b].[Id], [b].[Name], [b].[Price], [b].[AuthorId]
FROM [Books] AS [b]
WHERE [b].[AuthorId] = @authorId
```

### Conclusion

In conclusion, lazy loading automatically loads related entities when they are visited for the first time, making it a practical and simple method. If too many related entities are loaded at once, however, it may cause performance issues. In contrast, eager loading loads related entities in addition to the main entity in a single query, which might be more effective than lazy loading in circumstances when you know you will require the associated entities in advance. It can, however, lead to queries that are more complex and may take longer to perform. In circumstances when you only need to access related entities for a subset of entities, explicit loading is a selective technique that loads related entities as needed after the primary entity has been loaded. On the other hand, it could be less straightforward to use and takes more code to implement. The optimal strategy ultimately depends on the particular needs of your application and the trade-offs between simplicity, performance, and complexity.







