# Identity Providers (IDP)

Identity providers store and manage users' digital identities, including usernames, passwords, and other secure information.

Implementing an IDP creates a single, unified location for authentication for distributed services, i.e., a website and mobile app can use the same IDP for authentication.

Information sent from the client includes:

- Client ID, to check the client is knows
- Client secret/certificate, to authenticate the client is who they say they are
- 'Scopes', the requested resources/access
- Response type, such as tokens
- Redirect URI, validated by the IDP

## OpenID Connect

OpenId Connect adds an identity layer on top of OAuth 2.0. In ASP, use OpenId Connect as the challenge scheme, then configure the connection:

```c#
services
  .AddAuthentication(options => {
    options.DefaultScheme =
      CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme =
      OpenIdDefaults.AuthenticationScheme;
  })
  .AddCookie()
  .AddOpenIdConnect(options => {
    // url of IDP server
    options.Authority = Configuration.Get<string>("openid:url");

    // client id (basically name of this application)
    options.ClientId = "ExampleApp";
    // client secret
    options.ClientSecret =  Configuration.Get<string>("openid:secret");

    options.CallbackPath = "/signin-confirmed";

    // scopes
    options.Scope.Add("some_access");
    options.Scope.Add("some_more_access");

    // save access token
    // (true if required by scope resources later)
    options.SaveTokens = true;

    // by default, identity token will not contain all claims
    // IDP will send an extra request to get user claims
    options.GetClaimsFromUserInfoEndpoint = true;

    // map additional properties on ApplicationUser
    options.ClaimActions.MapUniqueJsonKey("FirstName", "FirstName");
    options.ClaimActions.MapUniqueJsonKey("Role", "role");
    // etc

    // use code flow
    options.ResponseType = "code";
    options.ResponseMode = "form_post"; // GET is default
  });
```
