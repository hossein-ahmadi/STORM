# STORM
STORM (Fast and full-featured object database mapper for .NET Core). To know more about STORM, read our documentation:

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
STORM is fast in all CRUD operations. The following benchmark compared STORM with EF Core 5.0 and Dapper. Benchmark is based on:

1. Inserting 5000 users
1. Inserting 11000 records (1000 users, 10 blog post for each user)
2. Updating 10000 users
3. Deleting 500 blogposts
4. Query 100000 users

<i>Dapper is only compared in query results</i>

STORM Batch Size: 250 o/s
```
BenchmarkDotNet=v0.12.1, OS=Windows 10.0.19042
Intel Core i5-3570K CPU 3.40GHz (Ivy Bridge), 1 CPU, 4 logical and 4 physical cores
.NET Core SDK=5.0.100
  [Host]     : .NET Core 5.0.0 (CoreCLR 5.0.20.51904, CoreFX 5.0.20.51904), X64 RyuJIT
  DefaultJob : .NET Core 5.0.0 (CoreCLR 5.0.20.51904, CoreFX 5.0.20.51904), X64 RyuJIT


|                         Method |         Mean |      Error |     StdDev |       Median |      Gen 0 |      Gen 1 |     Gen 2 |    Allocated |
|------------------------------- |-------------:|-----------:|-----------:|-------------:|-----------:|-----------:|----------:|-------------:|
|                   STORM_Insert |   430.200 ms |  8.0645 ms |  7.5435 ms |   431.320 ms |  6000.0000 |  1000.0000 |         - |  25288.41 KB |
|                  EFCore_Insert |   521.599 ms |  7.8862 ms |  6.9909 ms |   519.500 ms | 10000.0000 |  3000.0000 |         - |   63779.6 KB |
|   STORM_Insert_WithNavigations | 1,516.083 ms | 29.6653 ms | 48.7409 ms | 1,511.916 ms | 26000.0000 |  5000.0000 | 2000.0000 | 100116.13 KB |
|  EFCore_Insert_WithNavigations | 1,566.309 ms | 10.3883 ms |  9.2090 ms | 1,564.177 ms | 38000.0000 | 12000.0000 | 1000.0000 | 198046.23 KB |
|                   STORM_Update |   476.239 ms |  1.7810 ms |  1.6659 ms |   475.719 ms | 39000.0000 |  5000.0000 | 2000.0000 | 132500.56 KB |
|                  EFCore_Update |   627.717 ms |  6.3968 ms |  5.9836 ms |   627.095 ms | 20000.0000 |  6000.0000 | 1000.0000 |  107419.8 KB |
|                    STORM_Query |   489.170 ms |  7.4803 ms |  6.9971 ms |   487.325 ms | 27000.0000 | 11000.0000 | 3000.0000 | 165710.16 KB |
|                   EFCore_Query | 1,684.000 ms | 16.2304 ms | 15.1819 ms | 1,690.256 ms | 49000.0000 | 19000.0000 | 3000.0000 |  308055.7 KB |
|                     STORM_Load |     1.118 ms |  0.0091 ms |  0.0085 ms |     1.118 ms |    39.0625 |          - |         - |    119.49 KB |
|                    EFCore_Load |     1.567 ms |  0.0657 ms |  0.1842 ms |     1.547 ms |          - |          - |         - |     54.13 KB |
|         STORM_QueryFilter_Load |    16.769 ms |  0.3332 ms |  0.8777 ms |    17.233 ms |   734.3750 |   343.7500 |  140.6250 |   4301.26 KB |
|        EFCore_QueryFilter_Load |    84.211 ms |  1.5433 ms |  1.5158 ms |    84.059 ms |  3000.0000 |  1000.0000 |         - |  22378.43 KB |
|            STORM_Query_NoTrack |   159.485 ms |  3.1340 ms |  3.0780 ms |   158.750 ms |  6666.6667 |  2666.6667 |  666.6667 |   39823.1 KB |
|           EFCore_Query_NoTrack |   769.094 ms |  7.0530 ms |  6.2523 ms |   770.517 ms | 13000.0000 |  4000.0000 | 1000.0000 |  78818.15 KB |
|            STORM_Query_Complex |   475.644 ms |  6.1008 ms |  5.4082 ms |   476.715 ms | 21000.0000 |  5000.0000 | 1000.0000 | 135362.98 KB |
|           EFCore_Query_Complex |   765.239 ms |  7.2627 ms |  6.0647 ms |   765.633 ms | 13000.0000 |  5000.0000 | 1000.0000 |  78817.43 KB |
|  STORM_Query_NoTrack_NoProxies |   154.443 ms |  2.8828 ms |  2.6965 ms |   154.745 ms |  6750.0000 |  2750.0000 | 1000.0000 |  39823.94 KB |
| EFCore_Query_NoTrack_NoProxies |   195.383 ms |  3.9056 ms |  6.0805 ms |   195.606 ms | 11000.0000 |  4000.0000 | 1000.0000 |  61620.88 KB |
|                   Dapper_Query |   173.045 ms |  3.4304 ms |  4.0836 ms |   172.505 ms |  7000.0000 |  2000.0000 | 1000.0000 |  45179.28 KB |
|                   STORM_Delete |    41.716 ms |  2.9138 ms |  8.5913 ms |    41.535 ms |   312.5000 |   125.0000 |         - |   1959.22 KB |
|                  EFCore_Delete |    48.903 ms |  0.8881 ms |  0.8307 ms |    48.689 ms |          - |          - |         - |   4982.94 KB |
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
