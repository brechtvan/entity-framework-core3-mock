# EntityFrameworkCore3Mock

Easy Mock wrapper for mocking EntityFrameworkCore 3 (EFCore3) DbContext and DbSet in your unit-tests. Integrates with Moq or NSubstitute.

## Get it on NuGet

### Moq integration

    PM> Install-Package EntityFrameworkCore3Mock.Moq

### NSubstitute integration

    PM> Install-Package EntityFrameworkCore3Mock.NSubstitute

## Supports

* In-memory storage of test data
* Querying of in-memory test data (synchronous or asynchronous)
* Tracking of updates, inserts and deletes of in-memory test data
* Emulation of `SaveChanges` and `SaveChangesAsync` (only saves tracked changes to the mocked in-memory DbSet when one of these methods are called)
* Auto-increment identity columns, annotated by the `[Key, DatabaseGenerated(DatabaseGeneratedOption.Identity)]` attribute
* Primary key on multiple columns, annotated by the `[Key, Column(Order = X)]` attributes

## TODO

* Throwing a `DbUpdateException` when inserting 2 or more entities with the same primary key while calling `SaveChanges` / `SaveChangesAsync` (emulating EF behavior)
* Throwing a `DbUpdateConcurrencyException` when removing a model that no longer exists (emulating EF behavior)

For the Moq version, you can use all known [Moq](https://github.com/Moq/moq4/wiki/Quickstart) features, since both `DbSetMock` and `DbContextMock` inherit from `Mock<DbSet>` and `Mock<DbContext>` respectively.

## Example usage

    public class User
    {
        [Key, Column(Order = 0)]
        public Guid Id { get; set; }

        public string FullName { get; set; }
    }

    public class TestDbContext : DbContext
    {
        public TestDbContext(DbContextOptions<TestDbContext> options)
            : base(options)
        {
        }

        public virtual DbSet<User> Users { get; set; }
    }

    public class MyTests
    {
        [Fact]
        public void DbSetTest()
        {
            var initialEntities = new[]
                {
                    new User { Id = Guid.NewGuid(), FullName = "Eric Cartoon" },
                    new User { Id = Guid.NewGuid(), FullName = "Billy Jewel" },
                };
            
            var dbContextMock = new DbContextMock<TestDbContext>(DummyOptions);
            var usersDbSetMock = dbContextMock.CreateDbSetMock(x => x.Users, initialEntities);
        
            // Pass dbContextMock.Object to the class/method you want to test
        
            // Query dbContextMock.Object.Users to see if certain users were added or removed
            // or use Mock Verify functionality to verify if certain methods were called: usersDbSetMock.Verify(x => x.Add(...), Times.Once);
        }
    }

    public DbContextOptions<TestDbContext> DummyOptions { get; } = new DbContextOptionsBuilder<TestDbContext>().Options;
