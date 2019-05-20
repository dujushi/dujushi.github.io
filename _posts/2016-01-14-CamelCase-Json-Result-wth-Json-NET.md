  .NET has PascalCase naming convention for properties while Javascript has camelCase. Is there an easy way to convert the Json result? The answer is Json.NET. This article will show you how to create a custom result with Json.NET.

* Install Newtonsoft.Json Nuget Package
* Custom Action Result
{% highlight c# %}
public class JsonCamelCaseResult : ActionResult
{
    public Encoding ContentEncoding { get; set; }

    public string ContentType { get; set; }

    public object Data { get; set; }

    public override void ExecuteResult(ControllerContext context)
    {
        if (context == null)
        {
            throw new ArgumentNullException(nameof(context));
        }

        HttpResponseBase response = context.HttpContext.Response;
        response.ContentType = !string.IsNullOrEmpty(ContentType) ? ContentType : "application/json";
        if (ContentEncoding != null)
        {
            response.ContentEncoding = ContentEncoding;
        }
        if (Data != null)
        {
            var jsonSerializerSettings = new JsonSerializerSettings
            {
                ContractResolver = new CamelCasePropertyNamesContractResolver()
            };

            response.Write(JsonConvert.SerializeObject(Data, jsonSerializerSettings));
        }
    }
}
{% endhighlight %}
 We use CamelCasePropertyNamesContractResolver to convert PascalCase into camelCase.

* Controller Extension Method
{% highlight c# %}
public static class JsonCamelCaseHelper
{
    public static JsonCamelCaseResult JsonCamelCase(this Controller controller, object data)
    {
        return new JsonCamelCaseResult
        {
            Data = data
        };
    }
}
{% endhighlight %}
 This extension method can save us some typing. We can use this.JsonCamelCase([data]) in any controllers. Or you can put this method into your base controller if you have one.
 
* Usage
{% highlight c# %}
public ActionResult Index()
{
    var product = new Product
    {
        FancyName = "iPad Pro",
        SalesPrice = 12.10m
    };
    return this.JsonCamelCase(product);
}
{% endhighlight %}
 'FancyName' will be changed to 'fancyName' in the result.

## Gist
[https://gist.github.com/dujushi/890bf6b4455a58a7256e](https://gist.github.com/dujushi/890bf6b4455a58a7256e){:target="_blank"}

## References
1. [Serialize .NET objects as camelCase JSON](http://www.matskarlsson.se/blog/serialize-net-objects-as-camelcase-json){:target="_blank"}
2. [ASP.NET MVC and Json.NET](http://james.newtonking.com/archive/2008/10/16/asp-net-mvc-and-json-net){:target="_blank"}
3. [Setting the Default JSON Serializer in ASP.NET MVC](http://stackoverflow.com/questions/14591750/setting-the-default-json-serializer-in-asp-net-mvc){:target="_blank"}
