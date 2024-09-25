### Serilog Tutorial for .NET 8

**Serilog** is a popular logging library for .NET, offering structured logging to various sinks such as files, console, and databases. This tutorial will guide you through setting up and using Serilog in a .NET 8 application.

### 1. **Setting up Serilog in .NET 8**

#### Step 1: Install Serilog NuGet Packages
In your .NET 8 project, install the required Serilog packages using the NuGet Package Manager or via the command line:

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
```

You can also add other sinks as needed, such as `Serilog.Sinks.Seq` for centralized logging or `Serilog.Sinks.MongoDB` for logging to MongoDB.

#### Step 2: Modify `Program.cs` for Serilog Integration

Edit your `Program.cs` to configure and use Serilog. Here’s an example of setting up Serilog to log to both the console and a file:

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()    // Log to the console
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)  // Log to a file with daily rolling
    .CreateLogger();

// Replace default .NET logging with Serilog
builder.Host.UseSerilog();

var app = builder.Build();

app.MapGet("/", () =>
{
    Log.Information("Handling request for '/'");
    return "Hello, World!";
});

app.Run();
```

In this example:
- We create a `LoggerConfiguration` with `WriteTo.Console()` and `WriteTo.File()` to log to both the console and a file.
- The `builder.Host.UseSerilog()` replaces the default logging provider with Serilog.

#### Step 3: Configure Logging Levels

Serilog provides different levels of logging (e.g., Debug, Information, Warning, Error, Fatal). You can configure the minimum level of logging for your application by modifying the `LoggerConfiguration`:

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()  // Set minimum log level to Debug
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();
```

### 2. **Enriching Logs with Additional Information**

You can enrich your logs with extra information, such as the machine name or thread ID, using `Enrich`:

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.WithMachineName()   // Include machine name in the logs
    .Enrich.WithThreadId()      // Include thread ID in the logs
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();
```

### 3. **Logging in Controllers**

Once Serilog is configured, you can use it in your controllers by injecting an `ILogger`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Serilog;

[ApiController]
[Route("[controller]")]
public class MyController : ControllerBase
{
    private readonly ILogger<MyController> _logger;

    public MyController(ILogger<MyController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Get()
    {
        _logger.LogInformation("Handling GET request.");
        return Ok("Data from API");
    }
}
```

Here, the logger is injected into the controller, and you can use `_logger.LogInformation()` to log messages.

### 4. **Handling Exceptions**

You can log exceptions using Serilog by passing the exception object to the logging method:

```csharp
try
{
    // Some code that might throw an exception
}
catch (Exception ex)
{
    Log.Error(ex, "An error occurred while executing some code");
}
```

This will log the exception along with a custom message, making it easier to diagnose issues.

### 5. **Using Configuration Files**

Instead of hardcoding the configuration in `Program.cs`, you can move the configuration to `appsettings.json`:

```json
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console"
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/log-.txt",
          "rollingInterval": "Day"
        }
      }
    ]
  }
}
```

Then, in `Program.cs`, use the `ReadFrom.Configuration` method to load the configuration:

```csharp
var builder = WebApplication.CreateBuilder(args);

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)  // Load Serilog configuration from appsettings.json
    .CreateLogger();

builder.Host.UseSerilog();

var app = builder.Build();
app.Run();
```

### 6. **Structured Logging**

One of the powerful features of Serilog is structured logging. You can log structured data using placeholders:

```csharp
Log.Information("User {Username} logged in at {LoginTime}", "john_doe", DateTime.Now);
```

This creates structured log entries with the values for `Username` and `LoginTime`.

### 7. **Sinks and Log Storage**

You can log to various sinks depending on your requirements. Here are some popular Serilog sinks:
- **File:** Logs messages to a text file.
- **Console:** Logs messages to the console.
- **Seq:** Logs messages to a Seq server for centralized log management.
- **Elasticsearch:** Logs to an Elasticsearch instance.
- **Database:** Logs to databases like SQL Server or MongoDB.

To install additional sinks, use NuGet packages, like so:

```bash
dotnet add package Serilog.Sinks.Seq
```

And configure them in `Program.cs` or `appsettings.json`.

### 8. **Rolling Log Files**

You can roll log files by size or date. In the `File` sink example, we used `RollingInterval.Day` to create a new log file every day. You can also roll logs by file size:

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.File("logs/log.txt", fileSizeLimitBytes: 10485760, rollOnFileSizeLimit: true)  // Roll at 10MB
    .CreateLogger();
```

### 9. **Conclusion**

This guide provides a comprehensive overview of integrating Serilog into a .NET 8 project. From installation and configuration to advanced features like structured logging and rolling log files, Serilog offers flexible and powerful logging for .NET applications.
