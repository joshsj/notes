# Dependency Injection (DI)

Following [Dependency Injection in ASP.NET Core](https://app.pluralsight.com/library/courses/aspdotnet-core-dependency-injection/table-of-contents)

There are two main components used for DI in ASP.NET MVC:

- `IServiceCollection`, used to register services
- `IServiceProvider`, built by `IServiceCollection` to provide services

## Lifetimes

**Transient** dependencies are instantiated every time the dependency is resolved

- Doesn't need to be thread-safe, as each instance is only referenced once
- Quite inefficient

**Singleton** dependencies are instantiated once, and shared for the lifetime of the application

- Must be thread-safe
- Suited to stateless applications

**Scoped** dependencies are used throughout the lifetime of a request, meaning middleware through to the controller access the same instance

## Options pattern

Used to access configuration data via DI, using `IOptions` generic interface.

1. Create a model for the configuration options
2. Add the options to `appsettings.json`
3. Register the class as a dependency, binding the class to the section in `appsettings.json`
4. Require an `IOptions<>` instance as a dependency

```jsonc
// appsettings.json
{
  "MyOptions": {
    "AString": "My Value",
    "AnInt": 23
  }
}
```

```c#
// options model
public class MyOptions {
  public string AString { get; set; }
  public int AnInt { get; set; }
}

// register as dependency
services.Configure<MyOptions>(options =>
  // key name in appsettings.json
  Configuration.GetSection("MyOptions").Bind(options));

// standard DI pattern
public class MyController : Controller
{
  public MyController(IOptions<MyOptions> _options) {...}
}
```

## Multiple implementations

Some use-cases may require a dependency to be registered multiple times, e.g., a rule interface

```c#
services.AddSingleton<IRule, RequiredRule>();
services.AddSingleton<IRule, MaxLengthRule>();
services.AddSingleton<IRule, IsAlphaRule>();
```

Dependencies registered multiple times are provided with an `IEnumerable`

```c#
public SomeController(IEnumerable<IRule> rules) {...}
```

When using multiple registrations, using `TryAddEnumerable()` is best, taking a `ServiceDescriptor` instance:

```c#
services.TryAddEnumerable(
  // using ctor
  new ServiceDescriptor(
    typeof(IRule), typeof(RequiredRule),
    ServiceLifetime.Singleton
  )
);

services.TryAddEnumerable(
  // using helper method
  ServiceDescriptor.Singleton<IRule, MaxLengthRule>()
);

services.TryAddEnumerable(
  ServiceDescriptor.Singleton<IRule, IsAlphaRule>()
);

// can also be combined into a list
```

## Implementation factories

Allow the creation of dependencies to be controlled manually, using Add methods on `IServiceProvider` which accept a function. This function has a parameter of `IServiceProvider` allowing access to the service provider, and thus previously registered services.

One use is to implement dependencies which use a builder:

```c#
services.AddScoped<IRule>(sp => {
  // access registered service
  var builder  = sp.GetService(IRuleBuilder);

  // example builder stuff
  builder.WithSeverity(Severity.High);

  return builder.Build();
})
```

Another use is to wrap a dependency:

```c#
// register notifier services
services.AddSingleton<EmailNotifier>();
services.AddSingleton<SMSNotifier>();

// wrap interface with multiple services
services.AddSingleton<INotifier>(sp =>
  // multiple notifications via single class
  new MultiNotifier(new[]
    {
      // provide registered services
      sp.GetRequiredService<EmailNotifier>();
      sp.GetRequiredService<SMSNotifier>();
    }
  )
);
```

## Registering an implementation to multiple interfaces

Using an implementation factory, a single implementation can be provided for multiple interfaces:

```c#
// register the implementation
services.TryAddSingleton<GreetingService>();

// register the implementation with relevant interfaces
services.TryAddSingleton<IHomeGreetingService>(
  sp => sp.GetRequiredService<GreetingService>()
);

services.TryAddSingleton<IGeneralGreetingService>(
  sp => sp.GetRequiredService<GreetingService>()
);
```

## Using dependencies in controller actions

Where a dependency is only used in a/certain actions, they can be injected into method parameters with the `[FromServices]` attribute

## Using dependencies in middleware

Middleware have a lifetime of the whole application, meaning they are constructed once. Therefore only singleton services should be injected via its constructor to avoid misbehaviour.

`Invoke` and `InvokeAsync` methods are called on middleware each time a request is made, meaning all service lifetimes are safe to use.

```c#
public class SomeMiddleware {
  // empty constructor
  public SomeMiddleware() {}

  // passed in from services
  public Invoke(IDataRepository dep) {...}
}
```

## Using dependencies in Razor pages

Razor pages provide the `@inject` directive to access services.

## Using Scopes

Scopes are used to access services outside the usual flow.

```c#
public static void Main(string[] args)
{
  var webHost = CreateWebHostBuilder(args).Build();

  using (var scope = webHost.Services.CreateScope())
  {
    // access just like the ConfigureServices method
    var sp = scope.ServiceProvider;
  }
}
```

## Beyond the built-in container - Scrutor

[Scrutor](https://github.com/khellang/Scrutor) is a tool to enhance DI in ASP by providing assembly scanning and decoration extensions.

Assembly scanning automates service registration:

```c#
services.Scan(scan => scan
  // assembly to be scanned for classes
  .FromAssemblyOf<IRule>
    // can be implemented as IRule
    .AddClasses(c => c.AssignableTo<IRule>())
      .AsImplementedInterfaces()
      .WithScopedLifetime()+
);
```

The decorator pattern allows functionality of services to be 'wrapped'. By wrapping services, decorated classes retain their single purpose, and the decorating class is responsible for any additional functionality.

One useful implementation is to add caching to an existing service which preserves the role of the base class `WeatherForecaster`, and isolates the caching functionality to `CachedWeatherForecaster`:

```c#
public class WeatherForecaster : IWeatherForecaster
{
  public WeatherResult IWeatherForecaster.GetForecast()
  {
    // idk it's raining
  }
}

public class CachedWeatherForecaster : IWeatherForecaster
{
  // used by decorator to wrap non-cache implementation
  public WeatherResult IWeatherForecaster.GetForecast()
  {
    // made up code
    if (cached) {
      return cache.Get<WeatherResult>();
    }

    // use non-cache implementation
    cache.set<WeatherResult>();
  }
}
```
