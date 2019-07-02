IdentityServer4 is one of the most popular OpenID Connect and OAuth 2.0 framework for ASP.NET Core. In this tutorial, it will show you how easy it is to build an authentication server with the library.

## Create AuthenticationServer Project
Create an empty ASP.NET Core web application with Visual Studio. Uninstall nuget package `Microsoft.AspNetCore.Razor.Design`. We don't need Razor in this project. Install nuget package `IdentityServer4`. 

## Add Dependency Injection
Modify `ConfigureServices` method in `Startup`:

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer();
}
{% endhighlight %}

You can refer to [this document](http://docs.identityserver.io/en/latest/topics/startup.html){:target="_blank"} to learn more. You can also have a look at the [source code](https://github.com/IdentityServer/IdentityServer4/blob/master/src/IdentityServer4/src/Configuration/DependencyInjection/IdentityServerServiceCollectionExtensions.cs){:target="_blank"}

## Add IdentityServer to OWIN Pipeline
Modify `Configure` method in `Startup`:

{% highlight csharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseIdentityServer();

    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.Run(async (context) =>
    {
        await context.Response.WriteAsync("Hello World!");
    });
}
{% endhighlight %}

You can take a look at the [source code](https://github.com/IdentityServer/IdentityServer4/blob/master/src/IdentityServer4/src/Configuration/IdentityServerApplicationBuilderExtensions.cs){:target="_blank"} to learn more.

## Configure Clients and Resources
The program will thrown an `InvalidOperationException` if you run it now. That's because clients and resources haven't been configured yet. In IdentityServer, clients are applications that request tokens. Resources are APIs or identity information.

Modify `ConfigureServices` method in `Startup`:

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryClients(new Client[] {})
        .AddInMemoryApiResources(new ApiResource[]{});
}
{% endhighlight %}

Currently both lists are empty. We will add some later. But you can now run the application. The OpenID Connect discovery document endpoint is availabe at `/.well-known/openid-configuration`. You can learn more about the discovery document from [here](https://auth0.com/docs/protocols/oidc/openid-connect-discovery){:target="_blank"} and [here](https://openid.net/specs/openid-connect-discovery-1_0.html){:target="_blank"}.

## Add Signing Credential
Currently `jwks_uri` is missing in the discovery document. IdentityServer uses a SSL certificate to generate signatures in JWTs. Applications which accept the tokens need a public certificate to validate the tokens. `jwks_uri` will provide the public certificate.

Modify `ConfigureServices` method in `Startup`:

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryClients(new Client[] {})
        .AddInMemoryApiResources(new ApiResource[]{})
        .AddDeveloperSigningCredential();
}
{% endhighlight %}

Visit the discovery document endpoint again, `jwks_uri` should appear. `AddDeveloperSigningCredential` is for development only. We will change it to load SSL certificates from Azure KeyVault.