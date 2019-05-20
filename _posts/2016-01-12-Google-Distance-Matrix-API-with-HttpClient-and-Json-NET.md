Google Map is excellent. So is Google Map API. This article will create a simple class with HttpClient and Json.NET to utilise Google Distance Matrix API.

* Add System.Net.Http reference for HttpClient and install Newtonsoft.Json Nuget Package
* Add API Url and API Key settings into Web.config

{% highlight c# %}
<appSettings>
  <add key="GoogleDistanceMatrixApiUrl" value="https://maps.googleapis.com/maps/api/distancematrix/json"/>
  <add key="GoogleDistanceMatrixApiKey" value="YOUR API KEY"/>
</appSettings>
{% endhighlight %}

* GoogleDistanceMatrixApi Constructor
{% highlight c# %}
public GoogleDistanceMatrixApi(string[] originAddresses, string[] destinationAddresses)
{
    OriginAddresses = originAddresses;
    DestinationAddresses = destinationAddresses;

    var appSettings = ConfigurationManager.AppSettings;

    if (string.IsNullOrEmpty(appSettings["GoogleDistanceMatrixApiUrl"]))
    {
        throw new Exception("GoogleDistanceMatrixApiUrl is not set in AppSettings.");
    }
    Url = appSettings["GoogleDistanceMatrixApiUrl"];

    if (string.IsNullOrEmpty(appSettings["GoogleDistanceMatrixApiKey"]))
    {
        throw new Exception("GoogleDistanceMatrixApiKey is not set in AppSettings.");
    }
    Key = appSettings["GoogleDistanceMatrixApiKey"];
}
{% endhighlight %}

  The constructor receives origin addresses and destination addresses from the user and it gets App Url and Key settings from AppSettings. If the Url and Key settings are missing, it will throw an exception.
* Generate request url
{% highlight c# %}
private string GetRequestUrl()
{
    OriginAddresses = OriginAddresses.Select(HttpUtility.UrlEncode).ToArray();
    var origins = string.Join("|", OriginAddresses);
    DestinationAddresses = DestinationAddresses.Select(HttpUtility.UrlEncode).ToArray();
    var destinations = string.Join("|", DestinationAddresses);
    return $"{Url}?origins={origins}&destinations={destinations}&key={Key}";
}
{% endhighlight %}

  All addresses need to be url encoded.
* Response Ojbect
{% highlight c# %}
public class Response
{
    public string Status { get; set; }

    [JsonProperty(PropertyName = "origin_addresses")]
    public string [] OriginAddresses { get; set; }

    [JsonProperty(PropertyName = "destination_addresses")]
    public string [] DestinationAddresses { get; set; }

    public Row [] Rows { get; set; }

    public class Data
    {
        public int Value { get; set; }
        public string Text { get; set; }
    }

    public class Element
    {
        public string Status { get; set; }
        public Data Duration { get; set; }
        public Data Distance { get; set; }
    }

    public class Row
    {
        public Element[] Elements { get; set; }
    }
}
{% endhighlight %}

  This Response class maps API response. Json.NET can parse Json into a C# class. It is handy for us to analyse the result later on. We use JsonProperty attribute here to map Json properties to class properties if they are different.
* Request with HttpClient
{% highlight c# %}
public async Task<Response> GetResponse()
{
    using (var client = new HttpClient())
    {
        var uri = new Uri(GetRequestUrl());

        HttpResponseMessage response = await client.GetAsync(uri);
        if (!response.IsSuccessStatusCode)
        {
            throw new Exception("GoogleDistanceMatrixApi failed with status code: " + response.StatusCode);
        }
        else
        {
            var content = await response.Content.ReadAsStringAsync();
            return JsonConvert.DeserializeObject<Response>(content);
        }
    }
}
{% endhighlight %}

  HttpClient's built in async feature makes program efficient. JsonConvert parses Json response into a Response class we defined.
* Usage
{% highlight c# %}
public async Task<ActionResult> Index()
{
    GoogleDistanceMatrixApi api = new GoogleDistanceMatrixApi(new [] { "Auckland Airport" } , new [] { "Corner Princes Street and Waterloo Quadrant" });
    var response = await api.GetResponse();
    return Json(response, JsonRequestBehavior.AllowGet);
}
{% endhighlight %}

## Gist
[https://gist.github.com/dujushi/86a6555f0df73bb8ced4](https://gist.github.com/dujushi/86a6555f0df73bb8ced4){:target="_blank"}

## References
1. [The Google Maps Distance Matrix API](https://developers.google.com/maps/documentation/distance-matrix/intro){:target="_blank"}
2. [Json.NET](http://www.newtonsoft.com/json){:target="_blank"}
3. [Walkthrough: Accessing the Web by Using Async and Await (C# and Visual Basic)](https://msdn.microsoft.com/en-us/library/hh300224.aspx#BKMK_CompleteCodeExamples){:target="_blank"}
4. [ConfigurationManager.AppSettings Property](https://msdn.microsoft.com/en-us/library/system.configuration.configurationmanager.appsettings(v=vs.110).aspx){:target="_blank"}
