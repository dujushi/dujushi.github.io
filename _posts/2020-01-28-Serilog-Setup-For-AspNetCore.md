---
title: 'Serilog Setup for AspNetCore'
---
[Serilog](https://serilog.net/){:target="_blank"} is one of the most popular logging libraries. This tutorial will show you how to set it up for an AspNetCore project.

## Serilog.AspNetCore
Install Nuget Package [Serilog.AspNetCore](https://github.com/serilog/serilog-aspnetcore){:target="_blank"}. It is the recommended nuget package to configure Serilog for AspNetCore projects.

## Add JSON Configuration
{% highlight json %}
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.File" ],
    "MinimumLevel": {
      "Default": "Debug",
      "Override": {
        "System": "Information",
        "Microsoft": "Information",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "File",
        "Args": {
          "path": "C:\\Logs\\serilog-setup.txt",
          "rollingInterval": "Day",
          "outputTemplate": "[{Timestamp:HH:mm:ss.fff} {Level:u3}] {SourceContext}: {Message:lj}{NewLine}{Exception}"
        }
      }
    ]
  }
}
{% endhighlight %}

This sample configuration asks Serilog to use file logging. To learn more about Serilog configuration, please read [this doc](https://github.com/serilog/serilog/wiki/Configuration-Basics){:target="_blank"}. For json configuration syntax, please refer to [Serilog.Settings.Configuration](https://github.com/serilog/serilog-settings-configuration){:target="_blank"}. You can find a more advanced sample [here](https://github.com/serilog/serilog-settings-configuration/blob/dev/sample/Sample/appsettings.json){:target="_blank"}.

## Load JSON Configuration
{% highlight csharp %}
var environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");

var configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .AddJsonFile($"appsettings.{environment}.json", true)
    .Build();

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(configuration)
    .CreateLogger();
{% endhighlight %}

Refer to [Program.cs](https://github.com/dujushi/SerilogSetup/blob/master/SerilogSetup/Program.cs){:target="_blank"}.

## Log HostBuilder Exceptions
{% highlight csharp %}
try
{
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
{% endhighlight %}

Refer to [Program.cs](https://github.com/dujushi/SerilogSetup/blob/master/SerilogSetup/Program.cs){:target="_blank"}.

## Use Serilog Middleware
{% highlight csharp %}
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        })
        .UseSerilog();
{% endhighlight %}

Refer to [Program.cs](https://github.com/dujushi/SerilogSetup/blob/master/SerilogSetup/Program.cs){:target="_blank"}.

## Do Not Use Serilog Request Logging Middleware
{% highlight csharp %}
app.UseSerilogRequestLogging();
{% endhighlight %}

This is suggested in the document. Although it offers some benefits, it also causes the unhandled exceptions being [logged twice](https://github.com/serilog/serilog-aspnetcore/blob/19d871c78f697951704767d3d603708fc6039e9f/src/Serilog.AspNetCore/AspNetCore/RequestLoggingMiddleware.cs#L64){:target="_blank"}. Until it is fixed, I don't suggest using it.