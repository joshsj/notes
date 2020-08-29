# View Components

**Note:** the code examples have <a href="./Tag Helpers.md">tag helper</a> counterparts.

View components create HTML components used Razor pages. The HTML content and logic are separate; inheriting from `ViewComponent`, the `Invoke` method handles the components' logic and returns an `IViewComponentResult` with the components HTML, usually stored in a Razor model page.

View components are postfixed with `ViewComponent`, and are usually located in a 'ViewComponents' folder.

Data is attained through parameters to the `Invoke` method, and passed to the Razor page by including an `@model` directive.

Full syntax for including a view component is nasty, but thankfully there is a shorthand.

```c#
public class SpeakerCardViewComponent : ViewComponent
{
  public IViewComponentResult Invoke(Speaker speaker)
    => View(speaker);
}
```

```
@* view component markup *@
@model Speaker

<h1>@Model.Name</h1>
```

```
@* nasty *@
@await Component.InvokeAsync("SpeakerCardViewComponent", new Speaker{ Name = John });

@* better *@
<vc:speaker-card
  speaker="@new Speaker{ Name = "John" }">
</vc:speaker-card>
```
