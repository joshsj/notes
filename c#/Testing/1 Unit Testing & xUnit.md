# Unit Testing

## Terminology

**Unit** &nbsp;
A unit within a system is situational; maybe a single class, or three with frequent interaction. Unit tests are low-level, which enables highly-focused tests, e.g., values returns from methods. Thus they are quick to execute and plentiful.

**System under test (SUT)** &nbsp;
The class/framework/whatever being tested. Variables under test are conventionally named 'sut'.

**Assert**
Evaluate and verify the outcome of a test, either pass or fail. This is based on its result, final state, or occurrence of events during execution.

# XUnit

Tests are annotated with `[Fact]`, which are automatically run by the test runner. The `Assert` contains many static methods to test values, nulls, collections, object, exceptions, and raised events.

## Categorising

XUnit uses traits to categorise tests, taking a category name and value. They can be used to indicate priority, with a level as the trait value, or a bug, with an ID. Test runners can run tests with specified traits.

```c#
[Fact]
[Trait("Priority", "1")]
public void TestSomething() {...}

[Fact]
[Trait("Priority", "2")]
public void TestSomethingElse() {...}
```

**Note** [Xunit Categories](https://github.com/brendanconnolly/Xunit.Categories) provides better annotations for categorising tests

## Skipping

The `Skip` attribute on `[Fact]` takes a reason why the test is skipped. This allows deprecated and legacy tests to remain in the suite without running.

```c#
[Fact(Skip = "Don't need to test this")]
public void DoMoreTesting() {...}
```

## Custom output

In ASP, XUnit provides an `ITestOutputHelper` service to manually log additional information, injected as a dependency.

```c#
public class Tests
{
  private readonly ITestOutputHelper output;

  public Tests(ITestOutputHelper outputHelper)
    => output = outputHelper;

  public void AnTest()
  {
    output.WriteLine("Doing an test");
    Assert...;
  }
}
```

## Sharing test systems

It is common for unit test classes to share set-up and clean-up code (often called 'test context'). XUnit implements this using fixtures.

When tests in a class use the same object with the same set-up, store an instance of a fixture:

```c#
public class DatabaseFixture : IDisposable
{
  public SqlConnection Db { get; private set; }

  public DatabaseFixture()
  {
    Db = new SqlConnection("MyConnectionString");
  }

  public void Dispose()
  {
    // clean up
  }
}
```

```c#
public class DatabaseTests : IClassFixture<DatabaseFixture>
{
  DatabaseFixture fixture;

  public MyDatabaseTests(DatabaseFixture fixture)
  {
    this.fixture = fixture;
  }

  // tests
}
```

Additionally, fixtures can be shared across test classes using collection fixtures. A collection class defines the fixture to share, and classes use the `[Collection]` attribute to use the fixture:

```c#
[CollectionDefinition("DatabaseCollection")]
public class DatabaseCollection
  : ICollectionFixture<DatabaseFixture> {}

[Collection("DatabaseCollection")]
public class DatabaseTests1
{
  private DatabaseFixture db;

  public DatabaseTests1(DatabaseFixture db)
  {
    this.db = db;
  }
}
```

## Data-driven Tests

Repeated test logic can be isolated to a function annotated with `[Theory]`. Function arguments accept test data, and `[InlineData(data)]` supplies data for each test:

```c#
[Theory]
[InlineData(100, 0)]
[InlineData(99, 1)]
[InlineData(0, 100)]
public void TakeDamage(int damage, int expectedHealth)
{
  sut.TakeDamage(damage);

  Assert.Equal(expectedHealth, sut.Health);
}
```

Storing test data separately uses `[MemberData]`, specifying a class and member. Naturally, this could be stored in a file:

```c#
public class PlayerTestData
{
  public static IEnumerable<object[]> TakeDamage
  {
    get
    {
      yield return new object[] { 100, 0 };
      yield return new object[] { 99, 1 };
      yield return new object[] { 0, 100 };
    }
  }
}
```

```c#
[Theory]
[MemberData(
  nameof(PlayerTestData.TakeDamage),
  MemberType = typeof(PlayerTestData)
)]
public void TakeDamage(int damage, int expectedHealth)
{
  sut.TakeDamage(damage);

  Assert.Equal(expectedHealth, sut.Health);
}
```
