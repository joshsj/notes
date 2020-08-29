# Action Methods Workflow

Action methods are public method on a controller which acts an an endpoint to handle a request. They are designed to handle specific actions taken by a user, and can return different types of action results.

When executed by a `ControllerActionInvoker`, model binding takes place using incoming request to data to populate any properties on the method. Action filters are then invoked, before and after the action is executed. It finally returns an result, deriving from `IActionResult`

Since the result type is an implementation of `IActionResult`, the result of a method can differ depending on the request, e.g., Accept headers specifying content types

## Model Binding

...is the process of mapping incoming request data to action method parameters.

Internally, there are three key components driving model binding:

- 'Model Binder Providers' determine which Model Binder to use for th request. `IModelBinderProvider` determines if this provider can supply a model binder for the request, taking in a `ModelBinderProviderContext` instance

- 'Model Binders' populate the data, implementing `IModelBinder`. This accepts a `ModelBindingContext` to help.

- 'Value Providers' supply models binders with values from the request
  - Value Providers supply key-value pair data from url parameters, form data, etc.
  - Input Formatters parse complex data like JSON and XML to provide data

Custom providers can be created, but are rarely used. An exception is handling old/unsupported data formats. They are implemented inside `AddControllersWithViews`:

```c#
services.AddControllersWithViews(options => {
  options.ModelBinderProviders.Insert(...);
});
```

## Action Filters

... run logic before and afters an action method executes.

`IActionFilter` provides two methods: `OnActionExecuting` and `OnActionExecuted`, both accepting an `ActionExecutedContext` which provides request data and information about its invocation. They can also jump the action method by setting an action result on the context object, similar to authorisation filters when routing.

One use case would be to enable an opt-in feature, like an experimental design, indicated with a custom HTTP header 'experimental-design':

```c#
public class ExperimentalDesignActionFilter
  : Attribute, IActionFilter
{
  // specified on attribute
  public string Controller { get; set; }
  public string Action { get; set; }

  public void OnActionExecuting(ActionExecutingContext context)
  {
    // check for header
    if (context.HttpContext.Request
      .Headers.Keys.Contains("experimental-design")) {
        context.Result = new RedirectToAction(Action, Controller, null);
    }
  }
}

public class HomeController : Controller
{
  // redirect to groovy new page need be
  [ExperimentalDesignActionFilter(
    Action = nameof(IndexExperimental)
    Controller = nameof(HomeController)
  )]
  public IActionResult Index() {

  }

  public IActionResult IndexExperimental() {

  }
}
```

## Action Results

...are components that render the result of a action method to the response object.

All action results implement `IActionResult`, which defines an intentionally vague method `ExecuteResultAsync`, accepting a, `ActionContext`. Returning a `Task`, it enables different implementations to generate different responses.

## Result Filters

`IResultFilter` implements two methods: `OnResultExecuting` and `OnResultExecuted`, both taking context objects.

## Razor View Engine

...is responsible for locating and rendering the views selected by an action method.

It relies on three key components:

- 'View Engine' locates the view to be rendered, implementing `IViewEngine`. It first uses `GetView`, accepting a file path to render a view. If the view isn't located, `FindView` is called, accepting an `ActionContext` to try and find a view using context info.

* 'View Engine Result' reports the results of the view engine search, using `ViewEngineResult`. It contains properties about the search results, and defines `Found` and `NotFound` methods.

- 'View' renders the data and markup, implementing `IView`, which renders a response via its `RenderAsync` method.
