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

Query entities based on navigations:

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
