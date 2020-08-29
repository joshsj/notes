# Routing

## Routing pipeline

**1** Request

**2** Static files serve their content

**3 Endpoint routing middleware** determines which Endpoint should handle the request. Metadata on the endpoints is analysed to determine which is best.

**4** Intermediate middleware that require an endpoint to function are executed, such as authorization

**5 Endpoint middleware** executes the selected Endpoint to generate a response

## Deduction methods

**Conventional** routing (i.e., convention-based) used a application-wise pattern to match URLs to controllers and actions. They are registered during start-up and consumed by routing middleware.

Defaults can be provided with `=actionName`, as well as optional parameters.

```c#
app.UseEndpoints(endpoints => {
  endpoints.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"
  );
});
```

**Attribute** routing is implemented with attributes directly on controllers and actions.

```c#
[Route("api")]
public class ApiController : ControllerBase
{
  [Route("")]
  public IActionResult GetAll() {...}

  [Route("{id}")]
  public IActionResult GetById() {...}
}
```

**Note** &ensp; Attribute routes are evaluated first, in parallel, then conventional routes are evaluated in order.

## Endpoints up-close

Endpoints are classes containing a request delegate and other metadata used to generate a response.

ASP has multiple types of endpoints, but a general design of [RouteEndpoint](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.routeendpoint) is as follows:

```c#
public class RouteEndpoint : Endpoint // subclass
{
  public string DisplayName { get; set; } // easier to reference

  public string Pattern { get; set; } // stored url pattern

  // misc data for determining request
  public EndpointMetadata Metadata { get; set; }

  // to generate a response
  public RequestDelegate RequestDelegate { get; set; }
}
```

During start-up, all controller action methods are converted to endpoint objects, with the request delegate wrapping the execution of the controller action. The route templates are also stored.
