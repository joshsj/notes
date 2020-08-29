# Vue Router (VR)

- Passed as objects to Vue instance
- Paths are named

```js
new Router({
  routes: [
    {
      path: "/",

      // named path
      name: "home",

      // vue component used for route
      component: Home,

      // aliases
      alias: [ "/home", "/root" ]
    },

    // redirects
    { path: "/index", redirect: { name: "home" }}

    {
      // url parameters
      path: "/user/:id",
      name: "user-page",
      // enable props to receive ID in component
      props: true,
    }
  ]
});
```

## Navigation links

- Uses `router-link` to navigate between locations

```html
<nav>
  <router-link
    :to="{name: "user-page" params: { id: '123' }}">
  Profile
  </router-link>
</nav>
```

## History mode

- Default is 'hash-mode', using `/#/` at start of URL to
- Emulates page IDs prevent page reload
- Changing to 'history' mode uses browser state API, only in HTML5 thus IE10+

```js
new Router({ mode: "history" }); // no more #
```

## Route Guards (Lifecycle Hooks)

- Same as Vue's lifecycle hooks, but apply to routing.
- Declared as `Router` object members

Most hooks accept 3 parameters:

- `routeFrom`, the current route
- `routeTo`, the route being navigated to
- `next`, a callback method to resolve the guard
  - `next()` moves to next hook, or confirm navigation if none are left
  - `next(false)` aborts the navigation
  - `next(some location)` navigates to a location
  - `next(error)` takes an instance of `Error` and is passed to callbacks

**In-component Guards**

- `beforeRouteEnter`
  - Called before the component is confirmed, i.e., not navigated away
  - No `this` access as the component isn't created
- `beforeRouteUpdate`
  - Called when the same component is used between routes
  - Useful for dynamic routing like `/events/:id`, as the same view is used
  - Access to `this`
- `beforeRouteLeave`
  - Called when the route rendering the component changes
  - Access to `this`

**Global Guards**

- `beforeEach`, called before navigating to a component
- `afterEach`
  - Called after the route renders the component
  - No `next` parameter as routing is already confirmed

Hooks are called in the following order:

1. Global `beforeEach`
2. Per-component guards
3. Global `afterEach`
4. Vue component hooks such as 'created`

```js
beforeRouteLeave(routeTo, routeFrom, next) {
  // exit confirmation dialogue
  const answer =
    window.confirm("Are you sure you want to leave?");

  if (answer) {
    next(); // go to requested route
  }
  else {
    next(false); // don't navigate
  }
}
```

## Errors

### 404

- For missing paths, implemented with a catch-all as the last route
- For missing dynamic routes, like `/event/:id`, components can redirect

```js
{
  path: "/404",
  name: "404",
  component: NotFound,
},
{
  // catch all
  path: "*",
  // redirect to 404 page
  redirect: { name: "404" },
}
```
