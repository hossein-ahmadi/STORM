# STORM
STORM (Fast and full-featured object database mapper for .NET)

## Benchmarks
STORM is fast in Insert, Update, Delete and also running queries. This benchmark is based on:

1. Insert 5000 records
2. Update 10000 records
3. Delete 1000 records
4. Query 50000 records

<i>Lazy load and proxy object creation features are not enabled for EF Core</i>
<br/>
<i>Lazy load enabled for STORM_Query and STORM_Query_NoTrack</i>
<br/
<i>Dapper is only compared in query results</i>

```
|                         Method |        Mean |     Error |    StdDev |      Gen 0 |     Gen 1 |     Gen 2 | Allocated |
|------------------------------- |------------:|----------:|----------:|-----------:|----------:|----------:|----------:|
|                   STORM_Insert |   760.02 ms | 14.016 ms | 23.028 ms |  8000.0000 | 3000.0000 | 1000.0000 |  29.23 MB |
|                  EFCore_Insert | 1,820.50 ms | 13.033 ms | 12.191 ms | 14000.0000 | 5000.0000 | 1000.0000 |  78.58 MB |
|                   STORM_Update |   921.50 ms | 17.335 ms | 39.482 ms | 13000.0000 | 5000.0000 | 2000.0000 |  62.23 MB |
|                  EFCore_Update | 2,548.29 ms | 18.371 ms | 16.286 ms | 35000.0000 | 9000.0000 | 2000.0000 | 151.82 MB |
|                    STORM_Query |   283.20 ms |  5.139 ms |  6.118 ms | 12000.0000 | 5000.0000 | 1000.0000 |     79 MB |
|                   EFCore_Query |   719.04 ms | 13.739 ms | 14.109 ms | 24000.0000 | 9000.0000 | 3000.0000 | 136.46 MB |
|            STORM_Query_NoTrack |    84.82 ms |  1.628 ms |  2.437 ms |  3000.0000 | 1000.0000 |         - |   20.5 MB |
|           EFCore_Query_NoTrack |   348.73 ms |  6.353 ms |  5.305 ms | 16000.0000 | 5000.0000 | 1000.0000 |  93.98 MB |
|  STORM_Query_NoTrack_NoProxies |    83.32 ms |  1.547 ms |  1.781 ms |  3000.0000 | 1000.0000 |         - |  19.35 MB |
| EFCore_Query_NoTrack_NoProxies |   107.17 ms |  2.104 ms |  3.685 ms |  5000.0000 | 2000.0000 |         - |  30.02 MB |
|                   Dapper_Query |    90.76 ms |  1.809 ms |  1.604 ms |  3000.0000 | 1000.0000 |         - |  21.98 MB |
|                   STORM_Delete |    78.57 ms |  2.687 ms |  7.881 ms |          - |         - |         - |   3.39 MB |
|                  EFCore_Delete |   118.12 ms |  2.320 ms |  4.357 ms |  2000.0000 | 1000.0000 |         - |  12.98 MB |
```

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

## Getting started
The following code demonstrates basic usage of STORM. The first step is creating entities:

```cs
public class User
{
	public int Id { get; set; }
	public string Username { get; set; }
	public string Password { get; set; }
	public DateTime RegisterDate { get; set; }

	// Many to one relation
	public Many<BlogPost> BlogPosts { get; set; }
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

## Create DbSession

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

## Fluent map
STORM supports auto mapping of entities, but you can use fluent api to customize mapping for entities:

```cs
public class BlogDbSession : DbSession
{
    public DbTable<User> Users { get; set; }
    public DbTable<Tag> Tags { get; set; }
    public DbTable<BlogPost> BlogPosts { get; set; }

    protected override void OnSetup(DbSessionConfiguration config)
    {
        config.UseProvider<STORM.Providers.SqlServer.SqlServerDbProvider>("Data Source=.;Initial Catalog=BlogDb;Integrated Security=true;MultipleActiveResultSets=True");
        config.Entity<User>()
            .Property(p => p.Username).HasMaxLength(100).IsUnique();
        config.Entity<User>()
            .Many(u => u.BlogPosts)
            .ToRequiredOne()
            .HasForeignKey(p => p.UserId);

        config.Entity<BlogPost>()
            .Many(p => p.Tags)
            .ToMany(t => t.BlogPosts)
            .LeftKeys("BlogPostId")
            .RightKeys("TagId")
            .ToTable("BlogPostsTags");
        base.OnSetup(config);
    }
}
```

## Create database from model

```cs
using var session = new BlogDbSession();
session.Database.CreateIfNotExists();
```

## Insert entities

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

## Update entities

STORM use change tracker to detect changed values of entities. Update data is simple as:

```cs
using var session = new BlogDbSession();
var user = session.Users.First();
user.Password = "new password";
session.SaveChanges();
```

## Delete entities

The following code demonstrates how to delete entities from database:

```cs
using var session = new BlogDbSession();
var blogpost = session.BlogPosts.First();
blogpost.Tags.DeleteAll();
session.SaveChanges();
session.BlogPosts.Delete(blogpost);
session.SaveChanges();
```

## Query data
STORM Support simple queries to advance queries. to select all entities:

```cs
using var session = new BlogDbSession();
var users = session.Users.ToList();
```

To filter entities:

```cs
using var session = new BlogDbSession();
var users = session.Users.Where(u=>u.Username == "tosinso").ToList();
```

### Query entities based on navigations:

```cs
using var session = new BlogDbSession();
var blogPosts = session.BlogPosts.Where(p => p.User.Username == "ali");
var users = session.Users.Where(u => u.BlogPosts.Any(p => p.Tags.Any(t => t.Title.Contains("tag1"))));
```

The following query is generated for blogPosts query:

```sql
SELECT
    [BlogPosts].*
FROM
    BlogPosts [BlogPosts]
LEFT JOIN [dbo].[Users] [BlogPosts.User] ON [BlogPosts.User].[Id] = [BlogPosts].[UserId]
WHERE
    ([BlogPosts.User].[Username] = @param1)
```

and following query is generated for users query:

```sql
SELECT
    [Users].*
FROM
    Users [Users]
WHERE
    EXISTS (SELECT
        1
     FROM
        [dbo].[BlogPosts] [Users.BlogPosts]
     WHERE
        [Users.BlogPosts].[UserId] = [Users].[Id]
     AND
        EXISTS (SELECT
            1
         FROM
            [dbo].[Tags] [Users.BlogPosts.BlogPostsTags.Tags]
         INNER JOIN BlogPostsTags [Users.BlogPosts.BlogPostsTags] ON [Users.BlogPosts.BlogPostsTags].[TagId] = [Users.BlogPosts.BlogPostsTags.Tags].[Id]
         WHERE
            [Users.BlogPosts.BlogPostsTags].[BlogPostId] = [Users.BlogPosts].[Id]
         AND
            ([Users.BlogPosts.BlogPostsTags.Tags].[Title] LIKE '%'+@param1+'%')
         )
     )
```

### Projecting output
The simplest way to projecting output result is using select method:

```cs
var result = session.Users.Select(u => new
{
    u.Id,
    Username = u.Username.ToLower(),
    BlogPosts = u.BlogPosts.Select(p => new
    {
        p.Id,
        p.Title,
        p.Text,
        Tags = p.Tags
    }).ToList()
});
```

The following queries run for getting result, 1 query for users, 1 query for blogposts and Tags is lazy loaded:

```sql
-- Main query
SELECT
    [Users].[Id] [Id],
    LOWER([Users].[Username]) [Username],
    [Users].[Id] [BlogPosts@key1]
FROM
    Users [Users]

----------
BlogPosts
(SELECT
    [Users.BlogPosts].[Id] [Id],
    [Users.BlogPosts].[Title] [Title],
    [Users.BlogPosts].[Text] [Text],
    [Users.BlogPosts].[Id] [Tags@key1]
FROM
    [dbo].[BlogPosts] [Users.BlogPosts]
WHERE 
    [Users.BlogPosts].[UserId] in (@key1_where)
)
```

### Disable tracking and proxy creation
To make query faster, tracking and proxy creation can be disabled:

```cs
var result = session.Users.NoTrack().Select(u => new
{
    u.Id,
    Username = u.Username.ToLower(),
    BlogPosts = u.BlogPosts.NoProxies().Select(p => new
    {
        p.Id,
        p.Title,
        p.Text,
        Tags = p.Tags
    }).ToList()
});
```
NoTrack disable tracking object by change tracker and proxy object will create, to disable proxy creation and tracking, use NoProxies method

### Use second level cache:
STORM supports second-level cache, to enable cache, use FromCache method:

```cs
var result = session.Users.FromCache().NoTrack().Select(u => new
{
    u.Id,
    Username = u.Username.ToLower(),
    BlogPosts = u.BlogPosts.NoProxies().Select(p => new
    {
        p.Id,
        p.Title,
        p.Text,
        Tags = p.Tags
    }).ToList()
});
```
