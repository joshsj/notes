# Tag Helper Components

...automate the behaviour of tag helpers. They are registered as a dependency, and are automatically applied to `head` and `body` elements. This eliminates their usage in markup, so they can be used for global tag helper functionality.

```c#
public class AnalyticsTagHelperComponent : TagHelperComponent
{
  [HtmlAttributeName("tracking-code")]
  public string TrackingCode { get; set; }

  public AnalyticsTagHelperComponent(string trackingCode)
  {
    TrackingCode = trackingCode;
  }

  public override Task ProcessAsync(
    TagHelperContext context, TagHelperOutput output)
  {
    // only apply to <body>
    if (context.TagName.Equals("body",
     StringComparison.IgnoreOrdinalCase))
    {
      var script = "doSomething()";

      output.PostContent.AppendHtml(script);

      return Task.CompletedTask;
    }
  }
}
```
