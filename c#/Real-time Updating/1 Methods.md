# Real-time Methods

## AJAX Polling

Traditionally, asynchronous requests from a web client (AJAX) would send requests to servers periodically (polling). This is enables sites to update live, useful for say food order status (like Dominoes). Polling servers is intensive on both ends, and logically servers should be responsible for serving new data when available.

## Long polling

Long polling solves this issue: on request, the server checks for a new update. If an update isn't available, it pauses and checks again, making the server responsible for polling the data instead. The client sends requests as usual, and just waits longer for responses:

```c#
public IActionResult GetOrderUpdate(int orderId)
{
  CheckResult result = orderChecker.GetUpdate(orderId);

  while (!result.Finished)
  {
    if (result.New) { return ObjectResult(result); }

    // pause before checking new result
    Thread.sleep(3000);

    result = orderChecker.GetUpdate(orderId);
  }
}
```

## Server Sent Events (SSE)

SSEs allow servers to send data to a client without an initial request.

**The good**

- No repetitious additional requests reduces overhead
- Easy to polyfill for older browsers
- Auto-reconnect

**The bad**

- Only text messages
- Most browsers have ~6 connection max.
- One-way connection

```c#
public async Task GetOrderUpdate(int orderId)
{
  Response.ContentType = "text/event-stream";
  CheckResult result;

  do
  {
    result = orderChecker.GetUpdate(orderId);

    if (result.New)
    {
      await HttpContext.Response.WriteAsync(result.Update);
      await.Response.Body.FlushAsync();
    }

    // pause before checking new result
    Thread.sleep(3000);
  } while (!result.Finished)

  Response.Body.Close();
}
```

```js
// client connection
listen = (id) => {
  var eventSource = new EventSource(`/Order/${id}`);
  eventSource.onmessage = (event) => {}; // update page
};
```

## Web Sockets

Web sockets are a standardised way to use a single TCP socket for two-way communication. The ~6 connection limit doesn't apply, and multi-data-type support.

The web socket handshake first sends a request to upgrade the connection:

```
# usual GET stuff

# upgrade the connection
Upgrade: websocket
Connection: Upgrade

# key, subprotocols, version, extensions
Sec-WebSocket-Key: kdfjbdjkfb
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: Extensions: deflate-stream
```

If the server supports web sockets, it responds with a `101` with a key for a key for the accept header in subsequent communications:

```
HTTP/1.1 101 Switching Protocols

# upgrade the connection
Upgrade: websocket
Connection: Upgrade

# key, subprotocols, version, extensions
Sec-WebSocket-Accept: kdfjbdjkfb
Sec-WebSocket-Protocol: chat
Sec-WebSocket-Key: Extensions: deflate-stream
```
