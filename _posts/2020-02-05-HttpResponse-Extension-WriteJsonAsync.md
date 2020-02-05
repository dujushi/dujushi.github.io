---
title: 'HttResponse extension: WriteJsonAsync'
---
This is a brief extension method for `HttpReponse` to write any object in JSON format.

{% highlight csharp %}
public static async Task WriteJsonAsync(this HttpResponse httpResponse, object obj, CancellationToken cancellationToken = default)
{
    httpResponse.ContentType = MediaTypeNames.Application.Json;
    var json = JsonConvert.SerializeObject(obj);
    await httpResponse.WriteAsync(json, cancellationToken);
}
{% endhighlight %}