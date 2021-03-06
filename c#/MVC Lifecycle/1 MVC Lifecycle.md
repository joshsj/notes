# MVC Lifecycle Overview

**1** Request

**2** Middleware, the building blocks of the HTTP pipeline

- Ordered chain of components to handle requests
- Able to serve static files, handle exceptions, authorise users, etc

**3** Routing

- Middleware like everything before it
- ASP's Endpoint Routing middleware determines the controller and action to handle a request and executes

**4** Controller initialisation

- Invoked by endpoint router
- ControllerFactory sets up the controller
- Controller Action Invoker orchestrates the action call

**5** Action method execution

- Model binding
- Action filters
- Action execution
- Action filters
- Action result

**6** Action result execution

MVC separates the creation of results from execution. It includes:

- Result filters
- Invoke action result

Responses differ depending on the result type, e.g., a `ViewResult` goes to the view engine
