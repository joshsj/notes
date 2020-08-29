# Action Method Selection

## Route Constraints

...ensure the content of an URL properly matches a route. Default constraints include `alpha`, `bool`, `guid`, `regex`, etc.

Creating a route constraint uses the `IRouteConstraint` interface, which implements the `Match` method, returning a bool to indicate whether a value matches custom criteria.

Custom route constraints can be added globally using conventional routing:

```c#
app.UseEndpoints(endpoints =>
{
  endpoints.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}",
    constraints: new { id = new IRouteConstraint });
  );
});
```

They can also be added to the `RouteOptions` services and applied inline onto attributes:

```c#
services.Configure<RouteOptions>(options =>
  options.ConstraintMap.Add("custom", typeof(CustomConstraint));

// on an action
[Route("api/id:customConstraint")]
public IActionResult Index() {...}
```

## Action Constraints

... help determine which action method is the best candidate to handle a request. For example, let two actions with the same name only differ by their parameters; action selection uses additional data from the request to decide the appropriate method.

ASP uses `IActionConstraint` which offers the `Accept` method to return a `bool` indicating whether the action method is a valid candidate. It also offers an `Order` property to indicate the priority of the action.

One use case is to indicate mobile-specific actions:

```c#
public class MobileSelector : Attribute, IActionConstraint
{
  public int Order => 1;

  // checks user agent header for mobile platforms
  public bool Accept(ActionConstraintContext context)
    => context.RouteContext.HttpContext.Request
         .Headers["user-agent"].ToString().Contains("android");
}
```

```c#
[MobileSelector]
public IActionResult MobileIndex() {...}
```
