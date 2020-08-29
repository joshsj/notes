# Configuration and Options

Following [Using Configuration and Options in .NET Core and ASP.NET Core Apps](https://app.pluralsight.com/library/courses/dotnet-core-aspnet-core-configuration-options/table-of-contents)

`Microsoft.Extensions.Configuration`:

- Provides initial settings of the application
- Supports changing behaviour without recompilation
- Defined in one or many sources
- Accessed at runtime to control application behaviour

Data is accessed using key-value pairs, and the hierarchy of configuration data is identified using colons: `Section1:Key1`

Simple access to keys and values uses normal getters:

```c#
public class WeatherForcaster : IWeatherForecaster
{
  private readonly IConfiguration config;
  // injected as per
  public WeatherForcaster(IConfiguration config)
  {
    this.config = config;
  }

  public void SomeMethod()
  {
    var sb = config.GetValue<bool>("SomeBool");

    // key hierarchy
    var ab = config.GetValue<string>("NestedBool:AnotherBool");

    // section access
    var faveDrinks = config.GetSection("Favourites:Drinks");
    var faveHotDrink = faveDrinks.GetValue<string>("Hot");
  }
}
```

Storing connection strings under the `ConnectionStrings` key is a common pattern; `GetConnectionString(cs)` method automatically checks for a string under the `ConnectionStrings` key.

However, using getters is repetitive, and uses fragile naming for key access - it's better practise to use models to access configuration data.

## Configuration sources

`appsettings.json` is the default configuration file used by .Net. For an ASP application, some defaults are generated:

```jsonc
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },

  "AllowedHosts": "*",

  "SomeBool": true,
  "NestedBool": {
    "AnotherBool": false
  },

  "Favourites": {
    "Fruit": "Banana",
    "Drink": {
      "Hot": "Tea",
      "Cold": "Fanta"
    }
  }
}
```

Additional `appsettings` files can overwrite the global file for specific use-cases, e.g., `appsettings.Development.json` or `appsettings.Staging.json`

Environment variables can overwrite values further. When using the command line, double underscores `__` represent the hierarchy spacer:

```shell
$Favourites__Food=Banana
```

`launchSettings.json` in Visual Studio also uses `__`:

```jsonc
{
  "profiles": {
    "ISS Express": {
      "environmentVariables": {
        "Favourites__Food": "Banana"
      }
    }
  }
}
```

Command-line arguments to the program, using `:`

```shell
$ dotnet run --Favourites:Food Banana
```

## Models and Binding

Instead of getters, binding config data is much better: it prevents incorrect key usage and repetitive code.

```c#
// models for config data
public class Drinks
{
  public string Hot { get; set; }
  public string Cold { get; set; }
}

}
public class Favourites
{
  public string Fruit{ get;set; }
  public Drinks Drinks { get; set; }
}

// bind to instance of class
var faves = new Favourites();
config.Bind("Favourites", faves);
```

## Securing configuration data

Storing secure data, such as passwords or keys, in files is bad. Dotnet provides `user-secrets` to store secure data at development-time instead:

```shell
$ dotnet user-secrets set "Favourites:Food" "Banana"
```

For production, Azure Key Vault can be implemented with the `Microsoft.Azure.Services.AppAuthentication` and `Microsoft.Extensions.Configuration.AzureKeyVault` packages.

# Options

Although `IConfiguration` can be injected, the Options pattern is better using the `IOptions<>` interface, which registers configurations as singleton services.

```c#
public class FavouritesConfiguration
{
  public string Fruit{ get;set; }
  public Drinks Drink { get; set; }
}

// configured using getter
services.Configure<FavouritesConfiguration>(
  Configuration.GetSection("Favourites")
);

// injected
public class FavouritesController
{
  public FavouritesController(IOptions<FavouritesConfiguration> faveConfig) {...}
}
```

## Live updates

Alternatively, the `IOptionsSnapshot<>` interface registers configs as scoped services, allowing live changes to configurations. Inherently this can introduce scope issues when attempting to store scoped dependencies, so `IOptionsMonitor<>` is the better solution.

`IOptionsMonitor<>` offers a `CurrentValue` property to access the current value in the configuration file, avoiding scope risks.

## Named options

Snapshot and monitor can store multiple section within an instance:

```jsonc
{
  "apis": {
    "weather": {
      "url": "weather.com/api",
      "auth": ...
    },

    "traffic": {
      "url": "traffic.com/api",
      "auth": ...
    }
  }
}
```

```c#
services.Configure<ApisConfiguration>(
  "Weather",
  Configuration.GetSection("apis:weather")
);
services.Configure<ApisConfiguration>(
  "Traffic",
  Configuration.GetSection("apis:traffic")
);
```

## Validation

The extension method `ValidateDataAnnotations()` can validate configuration files against attributes like `[Required]`.

**Note:** validation is triggered on the first access to the object, not on creation, so be smart when using this.
