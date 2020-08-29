# Results

## Views

The `ViewEngine` is responsible fo location and rendering a view.
By default, ASP uses the Razor view engine, and Razor pages to server HTML-based content.

The Razor view search process searches directories based on path templates, e.g., 'views/{controller}/{action}.cshtml` or 'shared/{action}.cshtml'

Internally, MVC's view engine has three main components:

- View Engine, which initiates View search workflow
- View Engine Result, which reports the search results and the located view
- View, which renders a template or formatted response

### Extending search locations

`IViewLocationExpander` provides two methods: `ExpandViewLocations` is used to customise the view search paths; and `PopulateViews`, which supplies additional context information.

Enabling theme-ing without changing controllers or actions can be achieved by expanding the view search paths.

```c#
public class ThemeExpander : IViewLocationExpander
{
  public ExpandViewLocations(
    ViewLocationExpanderContext context,
     IEnumerable<String> viewLocations)
  {
    var activeTheme = "light"; // would be in config
    var newLocations = viewLocations.ToList();

    for (int i = 0; i < viewLocations.Count(); ++i)
    {
      newLocations.Insert(i, viewLocations.At(i)
        .Replace("/Views/", $"/Views/Themes/{activeTheme}/"));
    }

    return newLocations;
  }
}
```

## Custom action results

Action results are responsible for writing output to the HTTP response object, implemented with the `IActionResult` interface.

```c#
public class CsvResult : IActionResult
{
  private IEnumerable data;
  private string fileName;

  public CsvResult(IEnumerable data, string fileName) {
    this.data = data;
    this.fileName = fileName;
  }

  public Task ExecuteResultAsync(ActionContext context)
  {
    // do some parsing

    // add content headers
    context.HttpContent.Response.headers["content-disposition"]
      = $"attachment; filename ={fileName}.csv";

    // write the csv data to the response
    return context.HttpContent.Response.Body.WriteAsync(...);
  }
}
```
