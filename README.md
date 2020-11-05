# STORM
STORM (Fast and full-featured object database mapper for .NET). To know more about STORM, read our documentation:

* [Getting Started](https://github.com/hossein-ahmadi/STORM/wiki/Getting-Started)
* [Benchmarks](https://github.com/hossein-ahmadi/STORM/wiki/Benchmarks)
* [DbSession, DbTable and Entities](https://github.com/hossein-ahmadi/STORM/wiki/DbSession,-DbTable-and-Entities)
* [Navigations](https://github.com/hossein-ahmadi/STORM/wiki/Navigations)
* [Using enum](https://github.com/hossein-ahmadi/STORM/wiki/Enum-types)
* [Map with attributes](https://github.com/hossein-ahmadi/STORM/wiki/Map-with-attributes)
* [Map with fluent api](https://github.com/hossein-ahmadi/STORM/wiki/Map-with-fluent-api)
* [Create database from model](https://github.com/hossein-ahmadi/STORM/wiki/Create-database-from-model)
* [Add entities](https://github.com/hossein-ahmadi/STORM/wiki/Add-entities)
* [Update entities](https://github.com/hossein-ahmadi/STORM/wiki/Update-entities)
* [Delete entities](https://github.com/hossein-ahmadi/STORM/wiki/Delete-entities)
* Query data
 * [Query (Filter output)](https://github.com/hossein-ahmadi/STORM/wiki/Query--(Filter-output))
 * [Output projection](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Output-projection))
 * [Joins](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Joins))
 * [Advance features](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Advance-features))
 * [Performance tips](https://github.com/hossein-ahmadi/STORM/wiki/Query-(Performance-tips))
* [Concurrency check](https://github.com/hossein-ahmadi/STORM/wiki/Concurrency-check)
* [JSON Columns](https://github.com/hossein-ahmadi/STORM/wiki/JSON-Columns)
* [Indexes](https://github.com/hossein-ahmadi/STORM/wiki/Indexes)
* [Squences, Defaults and Check Constraints](https://github.com/hossein-ahmadi/STORM/wiki/Sequences,-Default-and-Check-constraints)
* [Shadow properties](https://github.com/hossein-ahmadi/STORM/wiki/Shadow-properties)
* [Entity validators](https://github.com/hossein-ahmadi/STORM/wiki/Entity-validators)
* [Session configuration](https://github.com/hossein-ahmadi/STORM/wiki/Session-configuration)

## Install
STORM is available in on Nuget. You can install it with following command:

```sh
Install-Package Tosinso.STORM
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


|                         Method |       Mean |    Error |   StdDev |     Median |      Gen 0 |      Gen 1 |     Gen 2 | Allocated |
|------------------------------- |-----------:|---------:|---------:|-----------:|-----------:|-----------:|----------:|----------:|
|                   STORM_Insert |   828.1 ms | 21.33 ms | 61.89 ms |   811.2 ms |  8000.0000 |  3000.0000 | 1000.0000 |  29.23 MB |
|                  EFCore_Insert | 1,854.1 ms | 25.60 ms | 23.94 ms | 1,858.2 ms | 14000.0000 |  5000.0000 | 1000.0000 |  78.64 MB |
|   STORM_Insert_WithNavigations | 1,622.1 ms | 32.02 ms | 34.26 ms | 1,627.2 ms | 58000.0000 | 13000.0000 | 1000.0000 | 185.28 MB |
|  EFCore_Insert_WithNavigations | 2,998.5 ms | 57.34 ms | 97.37 ms | 2,963.9 ms | 27000.0000 | 10000.0000 | 1000.0000 | 132.98 MB |
|                   STORM_Update |   970.0 ms | 19.37 ms | 44.90 ms |   962.8 ms | 13000.0000 |  5000.0000 | 2000.0000 |  62.24 MB |
|                  EFCore_Update | 2,626.8 ms | 50.55 ms | 54.08 ms | 2,610.4 ms | 35000.0000 |  9000.0000 | 2000.0000 | 151.83 MB |
|                    STORM_Query |   597.4 ms | 11.51 ms | 12.31 ms |   600.3 ms | 25000.0000 | 10000.0000 | 2000.0000 | 158.55 MB |
|                   EFCore_Query | 1,437.0 ms | 26.18 ms | 23.21 ms | 1,439.0 ms | 47000.0000 | 19000.0000 | 4000.0000 | 273.49 MB |
|            STORM_Query_NoTrack |   193.5 ms |  2.85 ms |  2.67 ms |   194.2 ms |  7000.0000 |  3000.0000 | 1000.0000 |  40.95 MB |
|           EFCore_Query_NoTrack |   682.0 ms | 12.95 ms | 18.57 ms |   680.8 ms | 34000.0000 | 13000.0000 | 3000.0000 | 187.93 MB |
|  STORM_Query_NoTrack_NoProxies |   184.5 ms |  2.56 ms |  2.27 ms |   184.9 ms |  7000.0000 |  3000.0000 | 1000.0000 |  38.66 MB |
| EFCore_Query_NoTrack_NoProxies |   236.3 ms |  4.70 ms |  7.85 ms |   239.8 ms | 11000.0000 |  4000.0000 | 1000.0000 |  60.02 MB |
|                   Dapper_Query |   212.9 ms |  2.65 ms |  2.48 ms |   212.6 ms |  7000.0000 |  2000.0000 | 1000.0000 |  43.97 MB |
|                   STORM_Delete |   119.8 ms |  5.62 ms | 16.49 ms |   120.4 ms |  1000.0000 |          - |         - |   5.54 MB |
|                  EFCore_Delete |   176.3 ms |  3.41 ms |  6.66 ms |   177.1 ms |  4000.0000 |  1000.0000 |         - |  21.89 MB |
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
```
