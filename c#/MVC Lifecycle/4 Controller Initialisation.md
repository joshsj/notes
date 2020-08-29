# Controller Initialisation

The abstract `ResourceInvoker` provides a framework to control the invocation of resources (shock). It receives contextual data about the request from the endpoint request delegate. Following is a simplified version of its process:

1. Authorisation filters and...
2. Resource filters are executed to determine if a controller needs to be created

3) A `ControllerFactory` consumes the contextual data to produce a controller instance

4. The controller executes the relevant **action workflow**. After producing a result, control returns to the `ResourceInvoker`, which performs...

5) Result filters
6) Result execution, and again
7) Result filters

`ControllerActionInvoker` derives from this class, which performs a <a href="5 Action Method Workflow.md">controllers' action workflow</a> (see step 3): It's responsible for:

1. Model binding
2. Action filters
3. Action execution
4. Action filters

## Custom filters

Filters allow logic to be injected during the lifecycle. Generally, filters are implemented using provided interfaces, such as `IAuthorizationFilter` and `IExceptionFilter`.

```jsonc
// appsettings.json
{
  "outage": true
}
```

```c#
public class OutageAuthFilter : Attribute, IAuthorizationFilter
{
  private IConfiguration config;

  public OutageAuthFilter(IConfiguration config) {
    this.config = config;
  }

  // provided by resource invoker
  public void OnAuthorization(AuthorizationFilterContext context) {
    var outage = config.GetValue<bool>("outage");

    if (outage) {
      // provide outage page
      context.Result = new ViewResult() { ViewName = "Outage" };
    }
  }
}
```

```c#
// [OutageAuthFilter] normal usage without injection
[TypeFilter(typeof(OutageAuthFilter))] // allows injection
public class HomeController : Controller {...}
```
