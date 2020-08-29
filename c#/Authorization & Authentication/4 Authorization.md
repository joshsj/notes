# Authorization

To enable authorization, add the middleware with `UseAuthentication`. Then authorization policies can be added to the service:

```c#
services.AddAuthorization(options => {
  options.AddPolicy(
    "IsAdmin",
    policy => policy.RequireRole("Admin"));

  options.AddPolicy(
    "CanAddConference",
     policy => policy.RequireClaim(
       "ConferencePermission", "Add"));
});
```

By default, roles are looked-up with a generated name. Since we identified the role claim with the key 'Role', this must be added to authentication options:

```c#
services.AddAuthentication(options => {
  options.TokenValidationParameters
    .RoleClaimType = "Role";
});
```

Once authorization is configured, the `[Authorize]` attribute can be added to controller/actions to require policies:

```c#
[Authorize(Policy = "Admin")]
public class AdminController : Controller {...}

public class ConferenceController : Controller
{
  [Authorize(Policy = "CanAddConference")]
  public IActionResult Add() {...}
}
```
