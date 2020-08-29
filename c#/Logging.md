# Logging

Logging comes out of the box with ASP using `ILogger<>`.

Loggers specify a 'category', which identifies the source of the log. Conventionally, it's the class name of the `ILogger<>` instance, but it can be arbitrary.

Different levels of severity can be logged: Trace = 0, Debug = 1, Information = 2, Warning = 3, Error = 4, Critical = 5, and None = 6.

```c#
// razor page
public class IndexModel : PageModel
{
  public IndexModel(ILogger<IndexModel> logger) {...}

  public void OnGet()
  {
    logger.Log(LogLevel.Error, dbErrorMsg);
    logger.LogInformation("At index"); // dedicated methods
  }
}
```

As normal, configuring logging can be done everywhere: see <a href="./Configuration and Options.md#Configuration-sources">configuration sources</a>

## Filters

Filters specify the minimum severity of logs from an assembly.

```jsonc
// appsettings.json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```

## Scopes

To further collate logs, scopes define a group of logical operations. Along with the scope name, data associated with the scope will also be included in logs within the scope.

Note: only Console logs support scopes

```c#
using (_logger.BeginScope("using block message")) {...}
```

## Event IDs

Another way to group logs uses an `EventId`, which take an int and a name.

## Gotta go fast

`LoggerMessage` helps create cache-able delegates with strong types, which requires fewer allocations, and eliminates boxing to reduce computational overhead.

`Define(LogLevel, EventId, String)` returns an `Action` to log a message. The string is a format string, formatted with up to 6 additional arguments provided through generics.

```c#
public static class Loggers
{
  private static readonly Action<ILogger, Exception> noParams;
  private static readonly Action<ILogger, int, Exception> withParam;

  static Loggers()
  {
    noParams = LoggerMessage.Define(
      LogLevel.Information,
      0, // id
      "Gonna do something"
    );

    withParam = LoggerMessage.Define<int>(
      LogLevel.Warning,
      1,
      "User not found: {Id}"
    );
  }
}
```

`DefineScope(String)` returns a `Func<ILogger, IDisposable>` to create a scope with a format string. Up to 3 additional values can be passed.

```c#
public static class Loggers
{
  private static readonly Func<ILogger, int, IDisposable> aScope;

  static Loggers()
  {
    aScope = LoggerMessage.DefineScope<int>("Scope with an int: {i}");
  }
}
```

## Global exception handling

`UseExceptionHandler` middleware allows exceptions throughout the application to be captured, and automates page redirects.

```c#
app.useExceptionHandler("/Error"); // points to razor page
```

Where exceptions are thrown, information about the exception can be added manually to the `Data` dictionary property to improve error pages.

```c#
// injecting data into exception
var ex = new Exception();
ex.Data.Add("ApiRoute", $"GET {response.RequestUri}");
throw ex;

// razor page for /Error
public class ErrorModel : PageModel
{
  public string ApiRoute { get; set; }

  public void OnGet()
  {
    var exs = HttpContext.Features.Get<IExceptionPathFeature>();
    var ex = exceptionPathFeature?.Error;

    if (ex.Data.Contains("ApiRoute"))
    {
      ApiRoute = ex.Data["ApiRoute"]; // etc
    }
  }
}
```

## Custom exception handling

To implement a fully custom error page, creating your own middleware is the way to go. This examples is to serve detailed exception responses from an API, however the structure matches that of an error web.

Create a dedicated class to store the data relating to the exception:

```c#
public class ApiError
{
    public string Id { get; set; }
    public short Status { get; set; }
    public string Code { get; set; }
    public string Links { get; set; }
    public string Title { get; set; }
    public string Detail { get; set; }
}
```

Allowing the developer to configure additional functionality of the middleware uses an options class, accepted into the middleware as a parameter. The options class declares delegates, which the developer can implement to add functionality.

In this case, it enables the developer to further add details to the `ApiError` class when exceptions are raised:

```c#
public class ApiExceptionOptions
{
  public Action<HttpContext, Exception, ApiError> AddResponseDetails { get; set; }
}
```

To allow the middleware to be implemented, use extension methods:

```c#
public static class ApiExceptionMiddlewareExtensions
{
  // without options configuration
  public static IApplicationBuilder UseApiExceptionHandler(this IApplicationBuilder builder)
  {
    var options = new ApiExceptionOptions();
    // add middleware
    return builder.UseMiddleware<ApiExceptionMiddleware>(options);
  }

  // with options configuration
  public static IApplicationBuilder UseApiExceptionHandler(
    this IApplicationBuilder builder,
    // action of options class
    Action<ApiExceptionOptions> optionsConfigurer
  ){
    var options = new ApiExceptionOptions();
    optionsConfigurer(options);

    return builder.UseMiddleware<ApiExceptionMiddleware>(options);
  }
}
```

And thus the middleware:

```c#
public class ApiExceptionMiddleware
{
  // next in chain of middlewares
  private readonly RequestDelegate _next;
  private readonly ILogger<ApiExceptionMiddleware> _logger;
  // options configured by developer
  private readonly ApiExceptionOptions _options;

  // injected dependencies
  public ApiExceptionMiddleware(ApiExceptionOptions options, RequestDelegate next,
        ILogger<ApiExceptionMiddleware> logger)
  {
    _next = next;
    _logger = logger;
    _options = options;
  }

  // called by application on request
  public async Task Invoke(HttpContext context /* other dependencies */)
  {
    // wrap the chain of middlewares until exception is throws
    try
    {
      await _next(context);
    }
    catch (Exception ex)
    {
      await HandleExceptionAsync(context, ex, _options);
    }
  }

  private Task HandleExceptionAsync(HttpContext context, Exception exception, ApiExceptionOptions opts)
  {
    // create new error object
    var error = new ApiError
    {
      Id = Guid.NewGuid().ToString(),
      Status = (short)HttpStatusCode.InternalServerError,
      Title = "Some kind of error occurred in the API.  Please use the id and contact our support team if the problem persists."
    };

    // use developers method if provided
    opts.AddResponseDetails?.Invoke(context, exception, error);


    var innerExMessage = GetInnermostExceptionMessage(exception);

    _logger.LogError(exception, "BADNESS!!! " + innerExMessage  + " -- {ErrorId}.", error.Id);

    // responding with JSON for the API user to process
    var result = JsonConvert.SerializeObject(error);
    context.Response.ContentType = "application/json";
    context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
    return context.Response.WriteAsync(result);
  }

  private string GetInnermostExceptionMessage(Exception exception)
    => (exception.InnerException == null)
        ? exception.Message
        : GetInnermostExceptionMessage(exception.InnerException);
}
```

## Additional scope information

By default, log entries don't include potentially import data:

- Machine name, especially in load-balanced environment where a single machine i sa bottleneck
- Environment, such as dev or test
- User Id, for current logged-in user
- User role
- Entry assembly/version, to distinguish centralised logs from multiple sources

Extra data can be provided using a custom dependency:

```c#
public interface IScopeInfo
{
  Dictionary<string, string> HostScopeInfo { get; }
}

public class ScopeInfo : IScopeInfo
{
  public Dictionary<string, string> HostScopeInfo { get; }

  public ScopeInfo() {
    HostScopeInfo = new Dictionary<string, string>
    {
      { "MachineName", Environment.MachineName },
      { "EntryPoint", Assembly.GetEntryAssembly().GetName().Name }
    }
  }
}

// usage in scope
public class SomeClass
{
  private readonly IScopeInfo scopeInfo;
  public SomeClass(IScopeInfo scopeInfo) {
    this.scopeInfo = scopeInfo;
  }

  public void SomeMethod() {
    using (_logger.BeginScope(scopeInfo.Environment)) {...}
  }
}
```

## Logging locations

Note: can also be referred to as 'providers', 'sinks', 'targets', cos why not

Files are the easiest choice, but aren't the best:

- Hard to query cos they're just text, so analysis is tricky
- Aggregating files from different servers/application/deployments is also tricky
- No native UI for files to browse them

Cloud-hosting services offer solutions, such as Azure's 'Application Insights' or 'App Service Diagnostics', integrated into ASP with `Microsoft.Extensions.Logging.AzureAppServices`

Packages like Serilog can connect to [many external services](https://github.com/serilog/serilog/wiki/Provided-Sinks#list-of-available-sinks) to store its sinks
