---
title: 'HttpClient PostAsync Sample'
---
[Make HTTP Requests](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-requests?view=aspnetcore-3.1){:target="_blank"} keeps getting easier with AspNetCore. But the document doesn't have a sample for `PostAsync` method, which is quite annoying. This article provides [a sample](https://github.com/dujushi/HttpClientSample){:target="_blank"} for reference.

### Client
{% highlight csharp %}
public class WeatherForecastClient : IWeatherForecastClient
{
    private static readonly JsonSerializerOptions JsonSerializerOptions = new JsonSerializerOptions
    {
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase
    };

    private readonly HttpClient _httpClient;

    public WeatherForecastClient(HttpClient httpClient)
    {
        _httpClient = httpClient ?? throw new ArgumentNullException(nameof(httpClient));
    }

    public async Task AddWeatherForecastAsync(WeatherForecast weatherForecast, CancellationToken cancellationToken = default)
    {
        var weatherForecastString = JsonSerializer.Serialize(weatherForecast, JsonSerializerOptions);
        var stringContent = new StringContent(weatherForecastString, Encoding.UTF8, MediaTypeNames.Application.Json);
        AddRequestHeader("Weather-Forecast-Header-Name", "WeatherForecastHeaderValue");
        await _httpClient.PostAsync("r/1fwlgic1", stringContent, cancellationToken);
    }

    private void AddRequestHeader(string name, string value)
    {
        _httpClient.DefaultRequestHeaders.Remove(name);
        _httpClient.DefaultRequestHeaders.Add(name, value);
    }
}
{% endhighlight %}

This sample uses `Typed clients`. I have created a [RequestBin](http://requestbin.net/r/1fwlgic1?inspect){:target="_blank"} to accept the post message. The tricky part is to generate a JSON string for the request. And if you need to pass a different header value for each individual request, remember to remove the header before add the new value. Otherwise, the new value will be appended to the previous value because the same `HttpClient` instance is shared between requests. You can use [Header Propagation](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-requests?view=aspnetcore-3.1#header-propagation-middleware) if the header is from the incoming request.

### Extension
{% highlight csharp %}
public static class WeatherForecastClientExtensions
{
    private static readonly Random Jitterer = new Random();
    private const int RetryCount = 5;

    public static IHttpClientBuilder AddWeatherForecastClient(this IServiceCollection services, Uri baseAddress)
    {
        return services
            .AddHttpClient<IWeatherForecastClient, WeatherForecastClient>(httpClient =>
            {
                httpClient.BaseAddress = baseAddress;
                httpClient.DefaultRequestHeaders.Add("Weather-Forecast-Api-Key-Name", "WeatherForecastApiKeyValue");
            })
            .AddTransientHttpErrorPolicy(p =>
                p.WaitAndRetryAsync(
                    RetryCount,
                    currentRetryNumber => TimeSpan.FromSeconds(Math.Pow(2, currentRetryNumber)) + TimeSpan.FromMilliseconds(Jitterer.Next(0, 100)),
                    (currentException, currentSleepDuration, currentRetryNumber, currentContext) => 
                    {
                        if (currentRetryNumber == RetryCount)
                        {
                            throw currentException.Exception;
                        }
#if DEBUG
                        Debug.WriteLine($"=== Attempt {currentRetryNumber} ===");
                        Debug.WriteLine(nameof(currentException) + ": " + currentException.Exception);
                        Debug.WriteLine(nameof(currentSleepDuration) + ": " + currentSleepDuration);
#endif
                    }));
    }
}
{% endhighlight %}

An interface `IWeatherForecastClient` can be bound to the client `WeatherForecastClient` at registration. To use the `Polly` policy, `Microsoft.Extensions.Http.Polly` nuget package should be installed.