# Middleware

... is the series of components that form the application request pipeline. Reusable features which apply to many different applications are great candidates for middleware, but it can also be application-specific.

MVC provides many middlewares for routing, sessions, CORS, authentication, and caching.

The MVC pipeline is executed in order, meaning registering them requires some thought about their functionality, i.e., providing authentication before controllers.

## Results

**Generating a response** will exit the middleware chain and return to the client. Middleware for serving content, such as static Razor pages, would create this result with the ['Run'](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.runextensions?view=aspnetcore-3.1) method.

`Run(handler)` accepts a `RequestDelegate`, which accepts a `HTTPContext` parameter to access the request data:

```c#
// RequestDelegate with HTTPContext parameter
app.Run(async context => {
  await context.Response.WriteAsync("A response");
});
```

**Performing work** on the request will pass the modified request to the next middleware. Middlewares which authenticate clients, load configuration, or force HTTPS would crete this result with the ['Use'](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.useextensions.use?view=aspnetcore-3.1) method.

`Use(handler, next)` accepts a function accepting a `HTTPContext` as usual, and an additional `Func<Task>` pointing to the next middleware:

```c#
app.Use(async (context, next) => {
  /// context.Request... as per

  // before next middleware
  await next.Invoke();
  // after next middleware
};'
```

**Reroute requests** with `Map()`. Redirecting urls to sub-domains (or vice-versa) would result in reroutes.

`Map(path , config)` accepts a `String` to match the path, and an `Action<IApplicationBuilder>` to invoke if the patch matches:

```c#
app.Map("/about", About);

private static void About(IApplicationBuilder) {
  // app.Run, app.Use etc
}
```

## Custom middlewares

Normal class requiring no super or inherits. The `next` delegate can be injected through its constructor, and its called via `Invoke()`.

```c#

public class HelloMiddleware
{
  private RequestDelegate next;

  public HelloMiddleware(RequestDelegate next) {
    this.next = next;
  }

  public Task Invoke(HttpContext context) {
    // do stuff
    await next.Invoke(context);
  }
}
```

Methods like `UseStaticFiles` and `UseAuthorization` in ASP are extension methods. These make registration easier, but also allow configuration before the `UseMiddleware` method is called;

Shock, providing extension methods alongside custom middleware is good practise .
