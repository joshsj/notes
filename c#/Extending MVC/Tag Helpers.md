# Tag Helpers

Limit reuse of code by providing custom tags for existing HTML elements, or custom components.

They derive from `TagHelper`, which defines a `Process` method to implement the helper's functionality. Properties store data relevant to the tag, provided when declared in a view. The `[HtmlTargetElement]` attribute declares the html element it targets, custom or not.

They also fully support dependency injection 🤔

## Importing

The `@addTagHelper` Razor directive accepts a name and assembly. The name can be a wildcard `*` to include all tag helpers from the assembly.

```c#
@addTagHelper "BannerTagHelper", "MyHelpers"
@addTagHelper "*", "MyHelpers"
```

## Simple custom control

```c#
namespace MyHelpers {
  [HtmlTargetElement("banner")]
  public class BannerTagHelper : TagHelper
  {
    // tag data
    public string Title { get; set; }
    public string Subtitle { get; set; }

    public override void Process(
      TagHelperContext context,
      TagHelperOutput output
    )
    {
      output.Content.SetHtmlContent(
        $@"
        <h1>{Title}</h1>
        <h3>{Subtitle}</h3>"
      ); // etc
    }
  }
}
```

```html
<banner title="Welcome"></banner>
```

## Extending a HTML element

For a nav component, this helper automatically sets its navigation link to active.

```c#
[HtmlTargetElement("a", Attributes="active-url")]
public class ActiveTagHelper : TagHelper
{
  public string ActiveUrl { get; set; }
  private readonly IHttpContextAccessor httpService;

  public ActiveUrl(IHttpContextAccessor httpService) {
    this.httpService = httpService;
  }

  public override void Process(
    TagHelperContext context,
    TagHelperOutput output
  )
  {
    // add active class if on link page
    if (httpService.HttpContext
    .Request.Path.ToString().Contains(ActiveUrl)) {
      var classAttrs = output.Attributes["class"]?.Value;

      output.Attributes
        .SetAttribute("class", "active " + classAttrs);
    }
  }
}
```

```html
<a class="nav-item" active-url="about" asp-action="about" ...
```
