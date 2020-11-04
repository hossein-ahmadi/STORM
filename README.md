# STORM
STORM (Fast and full-featured object database mapper for .NET)

## Benchmarks
STORM is fast in Insert, Update, Delete and also running queries. This benchmark is based on:

1. Insert 5000 records
2. Update 10000 records
3. Delete 1000 records
4. Query 50000 records

Dapper is only compared in query results

```
|                        Method |        Mean |     Error |    StdDev |      Median |      Gen 0 |     Gen 1 |     Gen 2 | Allocated |
|------------------------------ |------------:|----------:|----------:|------------:|-----------:|----------:|----------:|----------:|
|                  STORM_Insert |   763.81 ms | 15.181 ms | 43.799 ms |   748.46 ms |  8000.0000 | 3000.0000 | 1000.0000 |  29.21 MB |
|                     EF_Insert | 1,871.75 ms | 37.277 ms | 55.794 ms | 1,865.52 ms | 14000.0000 | 5000.0000 | 1000.0000 |  77.83 MB |
|                  STORM_Update | 1,002.40 ms | 30.901 ms | 86.649 ms |   993.37 ms | 13000.0000 | 4000.0000 | 2000.0000 |  62.16 MB |
|                     EF_Update | 2,468.83 ms | 21.770 ms | 19.299 ms | 2,464.31 ms | 32000.0000 | 8000.0000 | 2000.0000 | 138.98 MB |
|                   STORM_Query |   282.88 ms |  4.963 ms |  4.144 ms |   283.83 ms | 12000.0000 | 5000.0000 | 1000.0000 |  78.62 MB |
|                      EF_Query |   311.64 ms |  4.637 ms |  4.337 ms |   310.97 ms | 11000.0000 | 3000.0000 | 1000.0000 |   72.5 MB |
|           STORM_Query_NoTrack |    85.25 ms |  1.641 ms |  1.685 ms |    85.42 ms |  3000.0000 | 1000.0000 |         - |  20.12 MB |
| STORM_Query_NoTrack_NoProxies |    82.17 ms |  1.632 ms |  1.676 ms |    82.54 ms |  3000.0000 | 1000.0000 |         - |  19.35 MB |
|              EF_Query_NoTrack |   108.21 ms |  1.459 ms |  1.293 ms |   108.16 ms |  5000.0000 | 1000.0000 |         - |  30.03 MB |
|                  Dapper_Query |    89.74 ms |  1.790 ms |  1.675 ms |    90.30 ms |  3000.0000 | 1000.0000 |         - |  21.98 MB |
|                  STORM_Delete |    78.97 ms |  2.726 ms |  8.038 ms |    77.95 ms |          - |         - |         - |   3.39 MB |
|                     EF_Delete |   110.68 ms |  2.180 ms |  4.647 ms |   109.83 ms |  2000.0000 | 1000.0000 |         - |  11.42 MB |
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
