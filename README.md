# STORM
STORM (Fast and full-featured object database mapper for .NET Core 3.1). To know more about STORM, read our documentation:

* [Release notes](https://github.com/hossein-ahmadi/STORM/wiki/Release-notes)
* [Getting Started](https://github.com/hossein-ahmadi/STORM/wiki/Getting-Started)
* [Benchmarks](https://github.com/hossein-ahmadi/STORM/wiki/Benchmarks)
* [DbSession, DbTable and Entities](https://github.com/hossein-ahmadi/STORM/wiki/DbSession,-DbTable-and-Entities)
* [Microsoft ASP.NET Core integration](https://github.com/hossein-ahmadi/STORM/wiki/Microsoft-ASP.NET-Core-integration)
* [Navigations](https://github.com/hossein-ahmadi/STORM/wiki/Navigations)
* [Using enum](https://github.com/hossein-ahmadi/STORM/wiki/Enum-types)
* [Map with attributes](https://github.com/hossein-ahmadi/STORM/wiki/Map-with-attributes)
* [Map with fluent api](https://github.com/hossein-ahmadi/STORM/wiki/Map-with-fluent-api)
* [Create database from model](https://github.com/hossein-ahmadi/STORM/wiki/Create-database-from-model)
* [Add entities](https://github.com/hossein-ahmadi/STORM/wiki/Add-entities)
* [Update entities](https://github.com/hossein-ahmadi/STORM/wiki/Update-entities)
* [Delete entities](https://github.com/hossein-ahmadi/STORM/wiki/Delete-entities)
* [Introuction to query data](https://github.com/hossein-ahmadi/STORM/wiki/Query--(Filter-output))
* [Output projection](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Output-projection))
* [Joins](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Joins))
* [Advance query features](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Advance-features))
* [Loading navigations](https://github.com/hossein-ahmadi/STORM/wiki/Loading-navigations)
* [Performance tips](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Performance-tips))
* [Optimistic concurrency](https://github.com/hossein-ahmadi/STORM/wiki/Concurrency-check)
* [JSON Columns](https://github.com/hossein-ahmadi/STORM/wiki/JSON-Columns)
* [Indexes](https://github.com/hossein-ahmadi/STORM/wiki/Indexes)
* [Squences, Defaults and Check Constraints](https://github.com/hossein-ahmadi/STORM/wiki/Sequences,-Default-and-Check-constraints)
* [Shadow properties](https://github.com/hossein-ahmadi/STORM/wiki/Shadow-properties)
* [Entity validators](https://github.com/hossein-ahmadi/STORM/wiki/Entity-validators)
* [Dependency Injection](https://github.com/hossein-ahmadi/STORM/wiki/Dependency-Injection)
* [Session configuration](https://github.com/hossein-ahmadi/STORM/wiki/Session-configuration)

## Install
STORM is available in on Nuget. To use SQL Server provider install following package:

```sh
Install-Package Tosinso.STORM.SqlServer
```

## Supported providers
STORM Support following db providers:

    1. SQL Server

Following providers is under development:

    1. Oracle
    2. MySQL
    3. SQLite

## Benchmarks
STORM is fast in all CRUD operations. The following benchmark compared STORM with EF Core and Dapper. Benchmark is based on:

1. Inserting 5000 users
1. Inserting 25000 records (1000 users, 5 blog post for each user)
2. Updating 10000 users
3. Deleting 1500 blogposts
4. Query 100000 users

<i>Dapper is only compared in query results</i>

```
BenchmarkDotNet=v0.12.1, OS=Windows 10.0.19041.572 (2004/?/20H1)
Intel Core i5-6500 CPU 3.20GHz (Skylake), 1 CPU, 4 logical and 4 physical cores
.NET Core SDK=3.1.402
  [Host]     : .NET Core 3.1.8 (CoreCLR 4.700.20.41105, CoreFX 4.700.20.41903), X64 RyuJIT
  DefaultJob : .NET Core 3.1.8 (CoreCLR 4.700.20.41105, CoreFX 4.700.20.41903), X64 RyuJIT


|                         Method |        Mean |     Error |    StdDev |      Gen 0 |      Gen 1 |     Gen 2 | Allocated |
|------------------------------- |------------:|----------:|----------:|-----------:|-----------:|----------:|----------:|
|                   STORM_Insert |   754.82 ms | 17.510 ms | 49.958 ms |  8000.0000 |  3000.0000 | 1000.0000 |  29.27 MB |
|                  EFCore_Insert | 1,787.91 ms | 35.452 ms | 39.404 ms | 14000.0000 |  5000.0000 | 1000.0000 |  78.57 MB |
|   STORM_Insert_WithNavigations | 1,599.69 ms | 31.776 ms | 77.947 ms | 57000.0000 | 13000.0000 | 1000.0000 | 181.99 MB |
|  EFCore_Insert_WithNavigations | 2,721.27 ms | 17.618 ms | 14.712 ms | 25000.0000 |  8000.0000 | 1000.0000 | 127.34 MB |
|                   STORM_Update |   929.25 ms | 19.046 ms | 55.257 ms | 12000.0000 |  4000.0000 | 2000.0000 |  64.87 MB |
|                  EFCore_Update | 2,494.05 ms | 22.519 ms | 17.582 ms | 34000.0000 | 10000.0000 | 2000.0000 | 152.97 MB |
|                    STORM_Query |   552.79 ms | 10.589 ms | 12.195 ms | 26000.0000 | 10000.0000 | 3000.0000 | 154.94 MB |
|                   EFCore_Query | 1,456.24 ms | 26.596 ms | 24.878 ms | 47000.0000 | 19000.0000 | 4000.0000 | 273.65 MB |
|            STORM_Query_NoTrack |   183.06 ms |  3.618 ms |  6.524 ms |  7000.0000 |  3000.0000 | 1000.0000 |  38.88 MB |
|           EFCore_Query_NoTrack |   693.58 ms | 13.681 ms | 15.207 ms | 34000.0000 | 13000.0000 | 3000.0000 | 188.08 MB |
|  STORM_Query_NoTrack_NoProxies |   181.34 ms |  1.858 ms |  1.551 ms |  7000.0000 |  3000.0000 | 1000.0000 |  38.88 MB |
| EFCore_Query_NoTrack_NoProxies |   224.82 ms |  4.347 ms |  5.498 ms | 11000.0000 |  4000.0000 | 1000.0000 |  60.17 MB |
|                   Dapper_Query |   204.98 ms |  3.520 ms |  3.293 ms |  7000.0000 |  2000.0000 | 1000.0000 |  44.12 MB |
|                   STORM_Delete |    99.01 ms |  3.852 ms | 10.990 ms |  1000.0000 |          - |         - |   5.05 MB |
|                  EFCore_Delete |   160.41 ms |  2.661 ms |  2.489 ms |  4000.0000 |  1000.0000 |         - |  19.99 MB |
```

## Getting started
The following sections demonstrates basic usage of STORM.

### Creating POCO entity classes

```cs
public class User
{
	public int Id { get; set; }
	public string Username { get; set; }
	public string Password { get; set; }
	public DateTime RegisterDate { get; set; }

	// Many to one relation
	public virtual Many<BlogPost> BlogPosts { get; set; }
}

public class Tag
{
	public int Id { get; set; }
	public string Title { get; set; }

	// Many to many relation
	public virtual Many<BlogPost> BlogPosts { get; set; }
}

public class BlogPost
{
	public int Id { get; set; }
	// One to many relations
	public virtual User User { get; set; }
	public int UserId { get; set; }
	public virtual User PublishedByUser { get; set; }
	public int? PublishedByUserId { get; set; }
	public string Title { get; set; }
	public string Text { get; set; }

	// Many to many relations
	public virtual Many<Tag> Tags { get; set; }
}
```

### Create DbSession

```cs
public class BlogDbSession : DbSession
{
    public DbTable<User> Users { get; set; }
    public DbTable<Tag> Tags { get; set; }
    public DbTable<BlogPost> BlogPosts { get; set; }

    protected override void OnSetup(DbSessionConfiguration config)
    {
        config.UseProvider<STORM.Providers.SqlServer.SqlServerDbProvider>("Data Source=.;Initial Catalog=BlogDb;Integrated Security=true;MultipleActiveResultSets=True");
        base.OnSetup(config);
    }
}
```

### Create database from model

```cs
using var session = new BlogDbSession();
session.Database.CreateIfNotExists();
```

### Add entities

```cs
using var session = new BlogDbSession();
session.Users.Add(new User()
{
    Username = "tosinso",
    Password = "123",
    RegisterDate = DateTime.UtcNow,
    BlogPosts = new Many<BlogPost>()
    {
        new BlogPost()
        {
            Title = "blog post title",
            Text = "blog post text",
            Tags = new Many<Tag>()
            {
                new Tag(){Title = "tag1"}
            }
        }
    }
});
session.SaveChanges();
```

### Update entities

```cs
using var session = new BlogDbSession();
var user = session.Users.First();
user.Password = "new password";
session.SaveChanges();
```

### Delete entities

```cs
using var session = new BlogDbSession();
var blogpost = session.BlogPosts.First();
blogpost.Tags.DeleteAll();
session.SaveChanges();
session.BlogPosts.Delete(blogpost);
session.SaveChanges();
```

### Query data

Retreive all records:
```cs
using var session = new BlogDbSession();
var users = session.Users.ToList();
```

Or filter with linq expressions:

```cs
using var session = new BlogDbSession();
var users = session.Users.Where(u=>u.Username == "tosinso").ToList();

var users2 = from p in session.Users
    where p.Username == "tosinso"
    select p;
```
