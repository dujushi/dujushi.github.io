---
title: 'IdentityServer4 Tutorial - Part 2: Resource Owner Password Grant Type'
---
This tutorial will show you how to configure a client to use Resource Owner Password grant type. The work is based on [IdentityServer4 Tutorial - Part 1: Basic Setup](https://dujushi.github.io/2019/07/02/IdentityServer4-Tutorial-Basic-Setup.html){:target="_blank"}. 

## Define API Resources
The first thing is to define what API resources to protect. Modify `ConfigureServices` method in `Startup`:

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryClients(new Client[] { })
        .AddInMemoryApiResources(new[]
        {
            new ApiResource("api1")
        })
        .AddDeveloperSigningCredential();
}
{% endhighlight %}

Check out [the API Resource document](http://docs.identityserver.io/en/latest/reference/api_resource.html){:target="_blank"}.

## Define Clients
Now we can add clients. Modify `ConfigureServices` method in `Startup`:

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryClients(new[]
        {
            new Client
            {
                ClientId = "client_id1",
                RequireClientSecret = false,
                AllowedGrantTypes = { GrantType.ResourceOwnerPassword },
                AllowedScopes = { "api1" },
                AllowOfflineAccess = true
            }
        })
        .AddInMemoryApiResources(new[]
        {
            new ApiResource("api1")
        })
        .AddDeveloperSigningCredential();
}
{% endhighlight %}

The code defines a client without client secret. If you need a client secret, follow [the Secrets document](http://docs.identityserver.io/en/latest/topics/secrets.html){:target="_blank"}. `AllowOfflineAccess` is set to `true` which means a refresh token will be issued for every token request. By default, refresh tokens will be kept in memory. Later we will learn how to support other storages. 

## Add Resource Owner Password Validator
IdentityServer doesn't know your resource owners' credentials. You need to provide your own `IResourceOwnerPasswordValidator` implementation. The example below hard codes username and password. But you can inject your own user repository and do similar validation. 

{% highlight csharp %}
public class ResourceOwnerPasswordValidator : IResourceOwnerPasswordValidator
{
    public Task ValidateAsync(ResourceOwnerPasswordValidationContext context)
    {
        if (context.UserName == "username" && context.Password == "password")
        {
            context.Result = new GrantValidationResult(context.UserName, GrantType.ResourceOwnerPassword);
        }

        return Task.CompletedTask;
    }
}
{% endhighlight %}

Then we should tell IdentityServer to use our implementation.

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryClients(new[]
        {
            new Client
            {
                ClientId = "client_id1",
                RequireClientSecret = false,
                AllowedGrantTypes = { GrantType.ResourceOwnerPassword },
                AllowedScopes = { "api1" },
                AllowOfflineAccess = true
            }
        })
        .AddInMemoryApiResources(new[]
        {
            new ApiResource("api1")
        })
        .AddResourceOwnerValidator<ResourceOwnerPasswordValidator>()
        .AddDeveloperSigningCredential();
}
{% endhighlight %}

## Token Request
The [token request endpoint](http://docs.identityserver.io/en/latest/endpoints/token.html){:target="_blank"} is `/connect/token`. You can use the following cURL command to request a token. (PS: Change the port to match yours.) You can also [import cURL](https://learning.getpostman.com/docs/postman/collections/data_formats/#importing-curl){:target="_blank"} to Postman. Or learn more about [cURL](https://curl.haxx.se/docs/httpscripting.html){:target="_blank"} and play around with it.

{% highlight shell %}
curl -X POST https://localhost:44360/connect/token -d "client_id=client_id1&grant_type=password&username=username&password=password"
{% endhighlight %}

When you get a JWT, go to [https://jwt.io](https://jwt.io){:target="_blank"} to find out the details embeded.

## Custom Claims
You can add custom claims like this. 

{% highlight csharp %}
public class ResourceOwnerPasswordValidator : IResourceOwnerPasswordValidator
{
    public Task ValidateAsync(ResourceOwnerPasswordValidationContext context)
    {
        if (context.UserName == "username" && context.Password == "password")
        {
            var customClaims = new []
            {
                new Claim("custom_claim", "custom_value"), 
            };
            context.Result = new GrantValidationResult(context.UserName, GrantType.ResourceOwnerPassword, customClaims);
        }

        return Task.CompletedTask;
    }
}
{% endhighlight %}

But you need to tell IdentityServer to add them to your JWT.

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddIdentityServer()
        .AddInMemoryClients(new[]
        {
            new Client
            {
                ClientId = "client_id1",
                RequireClientSecret = false,
                AllowedGrantTypes = { GrantType.ResourceOwnerPassword },
                AllowedScopes = { "api1" },
                AllowOfflineAccess = true
            }
        })
        .AddInMemoryApiResources(new[]
        {
            new ApiResource("api1", new [] { "custom_claim" })
        })
        .AddResourceOwnerValidator<ResourceOwnerPasswordValidator>()
        .AddDeveloperSigningCredential();
}
{% endhighlight %}