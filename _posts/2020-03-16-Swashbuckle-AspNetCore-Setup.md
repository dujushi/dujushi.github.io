---
title: 'Swashbuckle ASP.NET Core Setup'
---
[Swagger](https://swagger.io/){:target="_blank"} is a great tool for documenting APIs. This article will show you how to set it up with ASP.NET Core. 

### Getting Started
[Swashbuckle.AspNetCore](hps://gittthub.com/domaindrivendev/Swashbuckle.AspNetCore){:target="_blank"} is a nuget package, which includes some Swagger tools for documenting APIs built on ASP.NET Core. Install the latest version to your API project.

Setup the dependency injection:
{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
    });
}
{% endhighlight %}

Use the middlewares:
{% highlight csharp %}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    app.UseSwagger();

    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
    });
    // ...
}
{% endhighlight %}

Swagger UI is now ready to use.

### XML Comments
[XML Comments](https://docs.microsoft.com/en-us/dotnet/csharp/codedoc){:target="_blank"} are a special kind of comment to document the program. Swagger can be configured to include the documentation.
{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

        var filePath = Path.Combine(AppContext.BaseDirectory, "SwaggerSample.xml");
        c.IncludeXmlComments(filePath);
    });
}
{% endhighlight %}
Make sure the project file is configured to generate the XML file.

### Bearer Security Definition
Most APIs use bearer authentication header to protect the endpoints. We can configure Swagger to ask for a bearer authentication header. [Refer to this blog](https://thecodebuzz.com/jwt-authorization-token-swagger-open-api-asp-net-core-3-0/){:target="_blank"}.

{% highlight csharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

        c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
        {
            Name = "Authorization",
            Type = SecuritySchemeType.ApiKey,
            Scheme = "Bearer",
            BearerFormat = "JWT",
            In = ParameterLocation.Header,
            Description = "JWT Authorization header using the Bearer scheme."
        });

        c.AddSecurityRequirement(new OpenApiSecurityRequirement
        {
            {
                new OpenApiSecurityScheme
                {
                    Reference = new OpenApiReference
                    {
                        Type = ReferenceType.SecurityScheme,
                        Id = "Bearer"
                    }
                }, 
                new string[] {}
            }
        });

        var filePath = Path.Combine(AppContext.BaseDirectory, "SwaggerSample.xml");
        c.IncludeXmlComments(filePath);
    });
}
{% endhighlight %}

### Unauthorized Response
For protected endpoints, we can use `Operation Filter` to add a `401` status code. 

{% highlight csharp %}
/// <summary>
/// This operation filter adds unauthorized response.
/// </summary>
public class UnauthorizedResponseOperationFilter : IOperationFilter
{
    /// <inheritdoc />
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        if (context.MethodInfo.DeclaringType == null)
        {
            return;
        }

        var authorizeAttributes = context.MethodInfo.DeclaringType.GetCustomAttributes(true)
            .Union(context.MethodInfo.GetCustomAttributes(true))
            .OfType<AuthorizeAttribute>();

        if (authorizeAttributes.Any())
            operation.Responses.Add(((int)HttpStatusCode.Unauthorized).ToString(), new OpenApiResponse { Description = HttpStatusCode.Unauthorized.ToString() });
    }
}
{% endhighlight %}

Then add it to the configuration:
{% highlight csharp %}
c.OperationFilter<UnauthorizedResponseOperationFilter>();
{% endhighlight %}

Similarly we can add [403](https://github.com/dujushi/SwaggerSample/blob/master/SwaggerSample/Swagger/OperationFilters/ForbiddenResponseOperationFilter.cs){:target="_blank"} and [501](https://github.com/dujushi/SwaggerSample/blob/master/SwaggerSample/Swagger/OperationFilters/InternalServerErrorResponseOperationFilter.cs){:target="_blank"} status codes.