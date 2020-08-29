# Hosted Services Basics

Hosted services are basically background tasks, which provide asynchronous functionality to exist outside the MVC pipeline. This enables polling for data from en external service, responding to external messages, or performing data-intensive work.

ASP provides `BackgroundService`, which inherits from `IHostedService`. Essentially, it hides boilerplate code behind virtual start and stop methods, leaving an `abstract ExecuteAsync` method to implement the background work.

```c#
public class WeatherCacheService : BackgroundService
{
  private readonly IWeatherApiClient api;
  private readonly IDistributedCache cache;

  public WeatherCacheService(IWeatherApiClient api,
    IDistributedCache cache)
  {
    this.api = api;
    this.cache = cache;
  }

  public override async Task ExecuteAsync(CancellationToken ct)
  {
    while (ct.IsCancellationRequested)
    {
      var response = await api.GetForecast(ct);

      if (response is Forecast forecast)
      {
        // unique key for cache entry
        var cacheKey = $"CurrentWeather:{DateTime.UtcNow:yyyy_MM_dd}";

        await cache.SetAsync(cacheKey, forecast, 1000); // 1s cache
      }

      // pause
      await Task.Delay(TimeSpan.FromSeconds(1), ct);
    }
  }
}
```

Like dependencies, hosted services must also be registered:

```c#
services.AddHostedService<WeatherCacheService>();
```

## Thread channels

`System.Thread.Channels` implement a thread-safe queue to channel data between 'producers' and 'consumers'. In this example, uploaded files are store temporarily on-disk (producer), and a processing service consumers the files to process the results.

```c#
public class FileProcessingChannel
{
  private const int MaxMessagesInChannel = 100;

  private readonly Channel<string> channel;

  public FileProcessingChannel()
  {
    var options = new BoundedChannelOptions
    {
      Capacity = MaxMessagesInChannel,
      SingleWriter = false,
      SingleReader = true
    }

    channel = Channel.CreatedBounded(options);
  }

  public async Task<bool> AddAsync(
    string fileName,
    CancellationToken ct)
  {
    while (await channel.Writer.WaitToWriteAsync(ct) &&
      !ct.IsCancellationRequested)
    {
      // store the filename to be read in the channel
      if (channel.Writer.TryWrite(fileName))
      {
        return true;
      }
    }

    return false;
  }
}
```

In POST, the temporary file can be pushed to the queue channel:

```c#
public async Task<IActionResult> Upload(
  IFormFile file, CancellationToken ct)
{
  // check has file and has data
  if (file is object and file.Length > 0)
  {
    var fileName = Path.GetTempFileName();

    using (var stream = new FileStream(fileName, FileMode.Create, FileAccess.Write))
    {
      // copy file contents to disk
      await file.CopyToAsync(stream, ct);
    }

    // add 3s timeout for subsequent operations
    var cts = CancellationTokenSource.CreateLinkedTokenSource(ct);
    cts.CancelAfter(TimeSpan.FromSeconds(3));

    try {
      var fileWritten = await channel.AddFileNameAsync(fileName, cts.Token);

      if (fileWritten)
      {
        return RedirectToAction("UploadComplete");
      }
    }
    catch (OperationCancelledException) when (cts.IsCancellationRequested)
    {
      System.IO.File.Delete(fileName); // cleanup file
    }
  }
}
```

A hosted service process the temporary files from the channel. It uses scopes from `IServiceProvider` to access a result processor instance for each file:

```c#
public class FileProcessingService : BackgroundService
{
  private readonly FileProcessingService fileChannel;
  private readonly IServiceProvider serviceProvider;

  public FileProcessingService(
    FileProcessingService fileChannel,
    IServiceProvider serviceProvider
  )
  {
    this.fileChannel = fileChannel;
    this.serviceProvider = serviceProvider;
  }

  protected override async Task ExecuteAsync(CancellationToken ct)
  {
    // each file asynchronously
    await foreach (var fileName in fileChannel.ReadAllAsync())
    {
      // get scoped service, disposed after iteration
      await using var scope = serviceProvider.CreateScope();
      // get new result processor
      await var processor = scope.ServiceProvider
        .GetRequiredService<IResultProcessor>();

      try {
        await using stream = File.OpenRead(file);
        await processor.ProcessAsync(stream)
      }
      finally {
        File.Delete(fileName); // dispose of temp file
      }
    }
  }
}

```
