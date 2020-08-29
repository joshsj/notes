# Identity Framework (IF)

## Set-up

```c#
// enhanced db context with authentication tables
services.AddDbContext<ApplicationDbContext>(options =>
  options.UseSqlServer(
    Configuration.GetConnectionString("DefaultConnection")));

// sign in class
services
  .AddDefaultIdentity<IdentityUser>(options =>
    options.SignIn.RequireConfirmedAccount = true)
  // use entity framework to store auth information
  .AddEntityFrameworkStores<ApplicationDbContext>();
```

IF also provides `_LoginPartial.cshtml`, which shows account information on the navbar e.g., when signed-out, it shows 'Register' and 'Sign-in` buttons; or when logged-in, it shows the user's email and a 'Logout' button.

## Data store

### DbContext

Identity Framework provides `IdentityDbContext<>`, which adds the `DbSet`s required for authentication. The non-generic class provides a default of `<IdentityUser, IdentityRole, string>`, with `string` identifying the type for primary keys. `IdentityUser` and `IdentityRole` are also non-generic implementations, accepting the same primary key type defaulted to `string`.

Additional `DbSet`s can be added like normal.

```c#
public class ApplicationDbContext : IdentityDbContext
{
  public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) {}

  public DbSet<Product> Products { get; set; }
}
```

Changing the table names is done by overriding `OnModelCreating()`.

### User data

To add data to users, override `IdentityClass`, conventionally named `ApplicationUser`. Data attributed with `[PersonalData]` will be deleted when the user account is deleted.

```c#
public class ApplicationUser : IdentityUser
{
  [PersonalData]
  public string FirstName { get; set; }
  [PersonalData]
  public DateTime Dob { get; set; }
}
```

To use this data with `IdentityDbContext`, update the inheritance with the user data class (and perform a migration). The dependency identifying the default identity also needs updating:

```c#
public class ApplicationDbContext
  : IdentityDbContext<ApplicationUser>
{
  public ApplicationDbContext(
    DbContextOptions<ApplicationDbContext> options)
    : base(options) {}
```

```c#
services.AddDefaultIdentity<ApplicationUser>(options =>...);
```

## Account pages

IF provides any pages related to authentication, e.g., `Account/Login`, `Account/ResetPassword`, `Account/ConfirmEmail`, etc. They are stored within the project dependency by default, but they can be modified. In Visual Studio:

1. Right-click project → Add → New Scaffolded Item
2. Identity
3. Select the pages to add locally

## Extending claims

Internally, claims are created for an identity using `IUserClaimsPrincipalFactory<>`. Implementing this interface and overriding the `GenerateClaims` method allows for additional claims. As normal, there are classes provided in IF with default behaviour, so we only specify the custom `ApplicationUser` here:

```c#
public class ApplicationUserClaimsPrincipleFactory : UserClaimsPrincipalFactory
{
  public ApplicationUserClaimsPrincipleFactory(
    UserManager<ApplicationUser> userManager,
    IOptions<IdentityOptions> options
  ) : base(userManager, options) {}

  protected override async Task<ClaimsIdentity>
    GenerateClaimsAsync(ApplicationUser user)
  {
    // get standard claims
    var identity = await base.GenerateClaimsAsync(user);

    // additional claims
    identity.AddClaim(new Claim("FirstName", user.FirstName));
    identity.AddClaim(new Claim("Dob", user.Dob));

    return identity;
  }
}
```

The new factory must also be registered as a dependency:

```c#
services.AddScoped(
  IUserClaimsPrincipleFactory<ApplicationUser>
  ApplicationUserClaimsPrincipleFactory
);
```

## Roles

Roles are disabled in IF by default, so registering the identity needs to be changed. Other defaults, such as the UI, also need to be added:

```c#
services.AddIdentity<ApplicationUser, IdentityRole>(options => ...)
  .AddDefaultUI()
  .AddDefaultTokenProviders(); // used for email verification
```

The claims factory must also be updated:

```c#
public class ApplicationUserClaimsPrincipleFactory : UserClaimsPrincipalFactory
{
  public ApplicationUserClaimsPrincipleFactory(
    UserManager<ApplicationUser> userManager,
    RoleManager<IdentityRole> roleManager,
    IOptions<IdentityOptions> options
  ) : base(userManager, roleManager, options) {}
```

Roles with now be included in the user identity's claims.

## Email verification

IF provides `IEmailSender` dependency, only as an interface to send emails. Behind an implementation, an email client is required. Working with an email client requires additional data, at minimum a sender email from user secrets:

```c#
public class EmailSender : IEmailSender
{
  private readonly string senderEmail;

  public EmailSender(IConfiguration config) {
    senderEmail = config.Get<string>("email:sender");
    // other stuff
  }

  public task SendEmailAsync(
    string recipientEmail, string subject, string message
  )
  {
    var client = new ExampleEmailClient();
    var message = new ExampleMessage
    {
      From = senderEmail,
      To = recipientEmail,
      Subject = subject,
      Message = message
    };

    await client.sendAsync(message);
  }
}
```

```c#
services.AddTransient<IEmailSender, EmailSender>();
```

After the dependency is configured, email verification should work out of the box, provided tokens are enabled for the user identity.
