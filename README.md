# STORM
STORM (Fast and full-featured object database mapper for .NET)

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
    
SQL Server provider is available on NuGet. Install it with following command:

```sh
Install-Package Tosinso.STORM.Providers.SqLServer
```

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
STORM supports auto mapping of entities, but uou can use fluent api to customize mapping for entities:

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
    LOWER([Users].[Id]) [Id],
    [Users].[Username] [Username],
    [Users].[Id] [BlogPosts@key1]
FROM
    Users [Users]

-- BlogPosts query
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

## STORM Features:
STORM Support many features that you can use:
```
POCO Entity map to database tables
One to Many Navigations
Many to One Navigations
Many to Many Navigations
Fluent and Attribute mapping
Auto map
Group property mapping
Support composite keys
Identity columns
Computed columns
Ignore properties
Check constraints
Default values (Use for default values and default in insert operations)
Row version and Concurrency check
Shadow Properties
Indexes
Sequences
JSON columns	
Change tracker
Batch operation on SaveChanges
Batch update and deletes on tables and many relations	
Interceptors
Create database from model
Support auto lazy loading for One to Many, Many to One and Many to Many navigations
ILazyLoad interface for custom lazy loading
Create object proxies	
Entity validators
Update from Models
Support async programming
Execute and Execute batch commands
Query features:
	Simple queries
	Complex queries
	Advance Projection			
		Project manies (loaded and lazy load)
	Support NoTrack and NoProxies for queries
	Second Level Cache
	ProjecTo => Create projection automatically by model
	ForXML and ForJSON (Only SQL Server)
	Load function
	Joins
		Left, Inner, Left, Right
	Intersect and Union
	Support Case,When
	Support advance grouping
	Predicate Functions (Between, PadIndex, Like, In)
	Other functions:
		RowNumberOver
		PartitionedRowNumberOver
		RankOver
		PartitionedRankOver
		NTile
		PartitionedNTile
		DenseRank
		PartitionedDenseRank
		Choose
		IIF
		STUFF,
		PatIndex
```

## Roadmap and features to implement:
The following features are planned to implement:

```
Column value map
Nullable Reference Types
Inheritance entity map:
	TPT, TPC, TPH
Subtables
	Dictionary<T,T>
	List<T>
	Array
Complex Types
Data encryption
Map CUD Functions to Stored Procedures
Stored Procedures
Views
User defined functions
Update database from model
Query Split function
Query Include function
Support datetime in aggergate functions
```
