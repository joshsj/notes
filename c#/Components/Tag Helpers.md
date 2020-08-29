# Tag Helpers

**Note:** the code examples have <a href="./View Components.md">view component</a> counterparts.

Tag helpers extend the functionality of HTML elements (tags) by either extending a default tag, like `asp-for` for `\<input>, or creating a new one.

They derive from`TagHelper`, which defines a `Process` method to implement the helper's functionality. Properties store data relevant to the tag, provided when declared in a view.

Using the `@addTagHelper` directive in '\_ViewImports.cshtml' will include them globally in Razor pages.

Conventionally, tag helper classes are postfixed with 'TagHelper'. In code, class names are automatically converted into kebab-case. Their location isn't important, but a 'TagHelper' folder is the norm.

```c#
// optional
public class SpeakerCardTagHelper : TagHelper
{
  public Speaker Speaker { get; set; }

  public override void Process(TagHelperContext context, TagHelperOutput output)
  {
    string content = @$"<h1>{Speaker.Name}";

    // bootstrap configuration on class attribute
    output.Attributes.SetAttribute("class", "col-xs-12, col-sm-6");
    output.TagName = "div"; // final tag after render
    output.content.SetHtmlContent(content);
  }
}
```

```html
<speaker-card speaker="@new Speaker{Name = John}"> </speaker-card>
```

## Naming

The `[HtmlTargetElement]` attribute on the class name can target a different tag if required. This is also how tag helpers are extend existing HTML elements.

To change the attribute names, use `[HtmlAttributeName(name)]`

## Cherry-picking tag helpers

Usually, tag helpers are included with a wildcard `*`, which searches all classes for TagHelpers. Specifying namespaces allows for better control.

For a class library providing tag helpers in a 'TagHelpers' folder, only the tag helpers can be imported with:

```
@addTagHelpers, ClassLib.TagHelpers*, ClassLib
```

## Accessing ViewContext

Use the `[ViewContext]` attribute, however also use `[HtmlAttributeNotBound]` to prevent access to the property in Razor pages, thus to avoid leaking internal data.

## Parent-child components

Tag helpers can restrict their child tags using `[RestrictChildren(child-tag-name)]` on the parent class. Parent tags can also be restricted using `[HtmlTargetElement(ParentTag = parent-name)]`

When using parent and child tag helpers, the `TagHelperContext` parameter can share data between them.

The following example automates nav items:

```c#
[RestrictChildren("active-page")]
public class NavTagHelper : TagHelper
{
  public string ActivePage { get; set; }

  public override void Process(
    TagHelperContext context, TagHelperOutput output)
  {
    // add the active class to items for children
    context.Items["ActivePage"] = ActivePage;

    // do some html to set up nav...
  }
}

[HtmlTargetElement("nav-item")]
public class NavItemTagHelper : TagHelper
{
  public string Text { get; set; }
  public string Href { get; set; }

  public override void Invoke(
    TagHelperContext context, TagHelperOutput output)
  {
    // get active page
    var activePage = context.Items["ActivePage"] as string;

    // check if active
    var activeClass = activePage == Title ? "active" : "";

    // add content with active indicator
    output.Content.SetHtmlContent($@"
      <a class='nav-link {activeClass}'
        data-toggle='pill' href={Href}>
        {Text}
      </a>
   ");

   output.Tag = "li";
  }
}
```

## Targeting element attributes

The `[HtmlTargetElement]` can target attributes on any element.

## Closing tag style

The `TagStructure` property on `[HtmlTargetElement]` has three options: `TagStructure.NormalOrSelfClosing` and `.Unspecified` allow normal or self-closing tags (shocker), while `.WithoutTagEnd` prevents self-closing tags.

## Handy built-in tag helpers

`<img>` <br/>
`asp-append-version` appends a randomly-generated query parameter to image URLs, which changes when the image is changed to prevent browsers from loading cached images.

`<Cache>` <br/>
Caches the content of the element. Without configuration, the data is cached for 20 minutes by default, however it can be configured a lot. The most common attribute is `expires-after`, receiving a duration to wait until refreshing the content.
