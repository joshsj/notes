# SignalR

...provides a wrapper for real-time web technologies.

Internally, it supports long polling, SSEs and web sockets. Where browsers don't support the latest technologies, it falls-back to the latest technology available. If desired, fall-backs can be disabled.

SignalR uses Remote Procedure Call (RPC) to exchange data, which allows clients and servers to call functions across the connection.

**Hubs** are a component of SignalR; server-side classes that exchange messages with the client. This includes a (de)serialisation protocol for parameters, JSON by default. It also supports 'MessagePack', a binary format more compact than JSON, and inherently faster to process.

## ASP Implementation

Implementing SignalR uses the `Hub` class; methods in this class will be invokable from the client.

```c#
public class OrderHub : Hub
{
  private readonly OrderChecker orderChecker;

  public OrderHub(OrderChecker orderChecker)
  {
    this.orderChecker = orderChecker;
  }

  public async Task GetOrderUpdate(int orderId)
  {
    CheckResult result;

    do
    {
      result = orderChecker.GetUpdate(orderId);

      if (result.New)
      {
        await Clients.Caller.SendAsync(
          "OrderUpdate", result.Update);
      }

      // pause before checking new result
      Thread.sleep(3000);
    } while (!result.Finished)

    await Clients.Caller.SendAsync("Finished");
  }
}
```

It is configured as a dependency and requires hubs to be routed:

```c#
// dependency
services.AddSignalR();

// routing configuration
app.UseEndpoints(endpoints =>
{
  endpoints.MapHub<OrderHub>("/order-hub");

  endpoints.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
});
```

The client side can then expect messages from the server:

```js
const status = document.getElementById("status");

connection = new signalR.HubConnectionBuilder().withUrl("/order-hub").build();

connection.on("OrderUpdate", (update) => {
  status.innerHTML = `Current status: ${update}`;
});
```
