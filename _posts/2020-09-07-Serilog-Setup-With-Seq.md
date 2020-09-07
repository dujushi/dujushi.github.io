[Seq](https://datalust.co/seq){:target="_blank"} is built for modern structured logging with [message templates](https://messagetemplates.org/){:target="_blank"}. It allows us to search and filter logs with SQL-style queries. If you don't have paid subscription, you can run Seq on your local machine with Docker. You can use [Serilog.Sinks.Seq](https://github.com/serilog/serilog-sinks-seq){:target="_blank"} to send logs to Seq with Serilog. This tutorial will show you how to set it up for your Asp.Net Core application.

### Getting started with Docker
If you are new to [Docker](https://docs.docker.com/){:target="_blank"}, follow the [quick start document](https://docs.docker.com/get-started/){:target="_blank"} to set it up on your machine. Then you can follow [this article](https://docs.datalust.co/docs/getting-started-with-docker){:target="_blank"} to set up Seq as a local Docker instance. Alternatively, if you know Terraform, you can use this [Terraform script](https://github.com/dujushi/Garage.SerilogSeqSetup/tree/master/Terraform){:target="_blank"} to provision a Docker instance. 

If everything goes well, you can visit Seq UI with [http://localhost:5341](http://localhost:5341){:target="_blank"}.

### Set up Asp.Net Core 
First install the following packages: [Serilog.AspNetCore](https://github.com/serilog/serilog-aspnetcore){:target="_blank"}, [Serilog.Sinks.Seq](https://github.com/serilog/serilog-sinks-seq){:target="_blank"}, and [Serilog.Exceptions](https://github.com/RehanSaeed/Serilog.Exceptions){:target="_blank"}. 

Then update `Program.cs` to use `Serilog`:
```
public class Program
{
    public static void Main(string[] args)
    {
        var environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");

        var configuration = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json")
            .AddJsonFile($"appsettings.{environment}.json", true)
            .Build();

        Log.Logger = new LoggerConfiguration()
            .ReadFrom.Configuration(configuration)
            .CreateLogger();

        try
        {
            Log.Information("Starting web host");
            CreateHostBuilder(args).Build().Run();
        }
        catch (Exception e)
        {
            Log.Fatal(e, "Host terminated unexpectedly");
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .UseSerilog()
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```
It reads Serilog configuration from app settings.

Lastly, add Serilog app settings:
```
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.Seq" ],
    "MinimumLevel": {
      "Default": "Debug",
      "Override": {
        "Microsoft": "Information"
      }
    },
    "WriteTo": [
      {
        "Name": "Seq",
        "Args": {
          "serverUrl": "http://localhost:5341"
        }
      }
    ],
    "Enrich": [
      "WithExceptionDetails"
    ]
  }
}
```

### Message Templates
[Message Templates](https://messagetemplates.org/){:target="_blank"} is a specification for capturing and rendering structured logs. Instead of logging a text message, it logs the message template with properties. It then allows you to search against the properties. 

For example, you can log an exception in Asp.Net Core, like this:
```
var exception = new Exception("Engine error");
exception.Data["ErrorNo"] = 1;
_logger.LogError(exception, "Car crashed {Name}", "Apple");
```

The raw log in Seq looks like:
```
{
    "@t": "2020-09-07T21:16:34.7050951+12:00",
    "@mt": "Car crashed {Name}",
    "@m": "Car crashed Apple",
    "@i": "9bb3256d",
    "@l": "Error",
    "@x": "System.Exception: Engine error",
    "Name": "Apple",
    "SourceContext": "Garage.SerilogSeqSetup.Controllers.LogsController",
    "ActionId": "5a4daf50-a42f-426c-823b-38e65ee9fcaf",
    "ActionName": "Garage.SerilogSeqSetup.Controllers.LogsController.Get (Garage.SerilogSeqSetup)",
    "RequestId": "80000050-0007-ff00-b63f-84710c7967bb",
    "RequestPath": "/logs",
    "SpanId": "|246beaa6-4555748038f06ade.",
    "TraceId": "246beaa6-4555748038f06ade",
    "ParentId": "",
    "ExceptionDetail": {
        "Type": "System.Exception",
        "Data": {
            "ErrorNo": 1
        },
        "HResult": -2146233088,
        "Message": "Engine error",
        "Source": null
    }
}
```
You can easily find all logs related to the same name or exception type.

### Demo
See [Garage.SerilogSeqSetup](https://github.com/dujushi/Garage.SerilogSeqSetup){:target="_blank"}.