---
title: 'Serilog Setup With Raygun'
---
[Raygun](https://raygun.com/){:target="_blank"} is a very popular monitoring tool. [Serilog.Sinks.Raygun](https://github.com/serilog/serilog-sinks-raygun){:target="_blank"} is a Serilog sink that writes events to Raygun. This tutorial will demo how to set it up for an AspNetCore project.

Firstly, follow [Serilog Setup for AspNetCore](https://dujushi.github.io/2020/01/28/Serilog-Setup-For-AspNetCore.html){:target="_blank"} to set up Serilog for the project.

Then, install nuget package `Serilog.Sinks.Raygun`.

Now you can add the [configuration](https://github.com/dujushi/SerilogRaygunSetup/blob/master/SerilogRaygunSetup/appsettings.json){:target="_blank"}. 

{% highlight json %}
{
    "Name": "Raygun",
    "Args": {
        "applicationKey": "RaygunAPIKey",
        "tags": [ "ProjectName" ]
    }
}
{% endhighlight %}

Replace `RaygunAPIKey` and `ProjectName` with your own value. The sink has a few other arguments. You can refer to [its document](https://github.com/serilog/serilog-sinks-raygun){:target="_blank"} to configure them.

There are a few points deserves your notice: 
1. Raygun uses exception message as the error title. If you log any messages without an exception. The messages will be grouped under `Unknown error`. 
2. The `restrictedToMinimumLevel` argument defaults to Error. You may set up `MinimumLevel` to `Debug`. But it still only logs errors and fatals. You'd better not to change the valules, because Raygun only likes exceptions.


