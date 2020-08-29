# Application Model

...represents the discovered controllers, actions, and other components of an MVC application. The default patterns in MVC are created by this class, e.g., 'views/{controller}/{action}.cshtml', but the entire discovery process can be changed using the application model.

## Application Model Providers

...influence the discovery and construction of the application model. `IApplicationModelProvider` contains methods to write application discovery logic. It also has an `Order` property.

**Caution** &ensp; It's real hard so it's easy to look past edge case

## Model Convention

...add additional customisations on top of the constructed application model. Since they don't replace the model, this is a simpler implementation.

MVC offers lots of interfaces for model conventions, with `IControllerModelConvention` being most common. It exposes an `Apply` method which accepts a `ControllerModel` parameter to work with.

There are two methods to apply a model convention, either through attributes for per-controller use, or globally using MVC configuration at start-up.
