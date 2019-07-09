---
title: 'IdentityServer4 Tutorial - Part 3: Store Refresh Token in Cosmos DB'
---
By default [refresh tokens are stored in memory](http://docs.identityserver.io/en/latest/topics/deployment.html?highlight=IPersistedGrantStore#operational-data){:target="_blank"}. In this tutorial we will add an `IPersistedGrantStore` implementation to store refresh tokens in Cosmos DB. Cosmos DB provides 5 APIs. We will use [SQL API with Version 3.0+ of the Azure Cosmos DB .NET SDK](https://docs.microsoft.com/en-nz/azure/cosmos-db/create-sql-api-dotnet-preview){:target="_blank"}. The work is based on [IdentityServer4 Tutorial - Part 2: Resource Owner Password Grant Type](https://dujushi.github.io/2019/07/03/IdentityServer4-Tutorial-Password-Grant-Type.html){:target="_blank"}. 

## Create an Azure Cosmos DB
Follow [this document](https://docs.microsoft.com/en-us/azure/cosmos-db/create-sql-api-dotnet-preview#create-an-azure-cosmos-account){:target="_blank"} to create an Azure Cosmos DB. When create your container, use '/id' as partition key.

## Nuget Update
Install nuget package [Microsoft.Azure.Cosmos](https://www.nuget.org/packages/Microsoft.Azure.Cosmos){:target="_blank"}. By the time the tutorial is written, the package is still under preview. You need to tick 'Include prerelease' to see the package in Package Manager.

## Define an Azure Cosmos Item
Refresh tokens are stored as a [PersistedGrant](https://github.com/IdentityServer/IdentityServer4/blob/master/src/Storage/src/Models/PersistedGrant.cs){:target="_blank"} object. But we can't save it directly to Azure Cosmos DB, because an Azure Cosmos Item requires [an id property](https://docs.microsoft.com/en-us/azure/cosmos-db/databases-containers-items#properties-of-an-item){:target="_blank"}. Let's add a `PersistedGrantItem` class as below.

{% highlight csharp %}
public class PersistedGrantItem
{
    [JsonProperty(PropertyName = "id")]
    public string Key;

    [JsonProperty(PropertyName = "type")]
    public string Type;

    [JsonProperty(PropertyName = "subject_id")]
    public string SubjectId;

    [JsonProperty(PropertyName = "client_id")]
    public string ClientId;

    [JsonProperty(PropertyName = "creation_time")]
    public DateTime CreationTime;

    [JsonProperty(PropertyName = "expiration")]
    public DateTime? Expiration;

    [JsonProperty(PropertyName = "data")]
    public string Data;
}
{% endhighlight %}

All the properties are the same as in `PersistedGrant`. But we map `Key` to `id` and change other property names to follow the naming convention.

Let's create a `Mapper` class to make conversion between the two easier.

{% highlight csharp %}
public static class Mapper
{
    public static PersistedGrantItem ConvertToPersistedGrantItem(PersistedGrant persistedGrant)
    {
        var persistedGrantItem = new PersistedGrantItem
        {
            Key = persistedGrant.Key,
            Type = persistedGrant.Type,
            SubjectId = persistedGrant.SubjectId,
            ClientId = persistedGrant.ClientId,
            CreationTime = persistedGrant.CreationTime,
            Expiration = persistedGrant.Expiration,
            Data = persistedGrant.Data
        };
        return persistedGrantItem;
    }

    public static PersistedGrant ConvertToPersistedGrant(PersistedGrantItem persistedGrantItem)
    {
        var persistedGrant = new PersistedGrant
        {
            Key = persistedGrantItem.Key,
            Type = persistedGrantItem.Type,
            SubjectId = persistedGrantItem.SubjectId,
            ClientId = persistedGrantItem.ClientId,
            CreationTime = persistedGrantItem.CreationTime,
            Expiration = persistedGrantItem.Expiration,
            Data = persistedGrantItem.Data
        };
        return persistedGrant;
    }
}
{% endhighlight %}

## CosmosDbPersistedGrantStore
Now we can add `CosmosDbPersistedGrantStore`.

{% highlight csharp %}
public class CosmosDbPersistedGrantStore : IPersistedGrantStore
{
    private readonly Container _container;

    public CosmosDbPersistedGrantStore(IConfiguration configuration, CosmosClient cosmosClient)
    {
        _container = cosmosClient.GetContainer(
            configuration["CosmosDB:DatabaseId"],
            configuration["CosmosDB:ContainerId"]);
    }

    public Task StoreAsync(PersistedGrant grant)
    {
        grant.Key = EncodePersistedGrantKey(grant.Key);
        var persistedGrantItem = Mapper.ConvertToPersistedGrantItem(grant);
        return _container.CreateItemAsync(persistedGrantItem);
    }

    public async Task<PersistedGrant> GetAsync(string key)
    {
        key = EncodePersistedGrantKey(key);

        var result = await _container.ReadItemAsync<PersistedGrantItem>(key, new PartitionKey(key));
        if (result.StatusCode == HttpStatusCode.NotFound)
        {
            return null;
        }

        var persistedGrantItem = (PersistedGrantItem)result;
        var persistedGrant = Mapper.ConvertToPersistedGrant(persistedGrantItem);
        return persistedGrant;
    }

    public Task<IEnumerable<PersistedGrant>> GetAllAsync(string subjectId)
    {
        throw new NotImplementedException();
    }

    public Task RemoveAsync(string key)
    {
        key = EncodePersistedGrantKey(key);
        return _container.DeleteItemAsync<PersistedGrantItem>(key, new PartitionKey(key));
    }

    public Task RemoveAllAsync(string subjectId, string clientId)
    {
        throw new NotImplementedException();
    }

    public Task RemoveAllAsync(string subjectId, string clientId, string type)
    {
        throw new NotImplementedException();
    }

    private static string EncodePersistedGrantKey(string key)
    {
        key = Base64UrlEncoder.Encode(key);
        return key;
    }
}
{% endhighlight %}

Add a `CosmosDB` section into appsettings.json.

{% highlight json %}
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*",
  "CosmosDB": {
    "AccountEndpoint": "your account endpoint",
    "AccountKey": "your account key",
    "DatabaseId": "your database id",
    "ContainerId": "your container id"
  } 
}
{% endhighlight %}

# Startup
Finally we need to tell IdentityServer to use `CosmosDbPersistedGrantStore` and add a singleton binding for `CosmosClient`. They are both configured through `Startup` class.

{% highlight csharp %}
public class Startup
{
    private readonly IConfiguration _configuration;

    public Startup(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    // This method gets called by the runtime. Use this method to add services to the container.
    // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
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
            .AddPersistedGrantStore<CosmosDbPersistedGrantStore>()
            .AddDeveloperSigningCredential();

        services.AddSingleton(s =>
        {
            var accountEndpoint = _configuration["CosmosDB:AccountEndpoint"];
            var accountKey = _configuration["CosmosDB:AccountKey"];
            var configurationBuilder = new CosmosClientBuilder(accountEndpoint, accountKey);
            return configurationBuilder.Build();
        });
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
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
}
{% endhighlight %}

# Refresh Token Lifetime
The lifetime of a refresh token is configured via [client setting](http://docs.identityserver.io/en/latest/topics/refresh_tokens.html?highlight=AbsoluteRefreshTokenLifetime#additional-client-settings){:target="_blank"} `AbsoluteRefreshTokenLifetime`. Normally we would need to create a task to delete expired refresh tokens. But Azure Cosmos DB has a [nice Time to live](https://docs.microsoft.com/en-us/azure/cosmos-db/how-to-time-to-live){:target="_blank"} feature. When it is configured, expired tokens will be deleted automatically.