Recently I tried to create a simple Owin middleware with params parameters. 

My first version looked like this:
{% highlight c# %}
public class TestMiddleware
{
    private readonly RequestDelegate _next;

    public TestMiddleware(RequestDelegate next, params string[] currencies)
    {
        _next = next;
    }

    public Task Invoke(HttpContext httpContext)
    {

        return _next(httpContext);
    }
}

public static class TestMiddlewareExtensions
{
    public static IApplicationBuilder UseMiddlewareClassTemplate(this IApplicationBuilder builder, params string[] currencies)
    {
        return builder.UseMiddleware<TestMiddleware>(currencies);
    }
}
{% endhighlight %}

You can compile the code, because there is no syntax error. But eventually you will get runtime error similar to this:

> A suitable constructor for type 'test.TestMiddleware' could not be located. Ensure the type is concrete and services are registered for all parameters of a public constructor.

To fix this issue, you can introduce an OwinMiddlewareOptions parameter:
{% highlight c# %}
    public class TestMiddleware
    {
        private readonly RequestDelegate _next;

        public TestMiddleware(RequestDelegate next, TestMiddlewareOptions options)
        {
            _next = next;
        }

        public Task Invoke(HttpContext httpContext)
        {

            return _next(httpContext);
        }
    }

    public static class TestMiddlewareExtensions
    {
        public static IApplicationBuilder UseMiddlewareClassTemplate(this IApplicationBuilder builder, params string[] currencies)
        {
            var options = new TestMiddlewareOptions 
            {
                currencies = currencies
            };
            return builder.UseMiddleware<TestMiddleware>(options);
        }
    }

    public class TestMiddlewareOptions
    {
        public string[] currencies;
    } 
{% endhighlight %}

`UseMiddleware` method is not smart enough to create an instance of Owin middleware with params parameters. `Use` method in .NET Framework has similar issue. 