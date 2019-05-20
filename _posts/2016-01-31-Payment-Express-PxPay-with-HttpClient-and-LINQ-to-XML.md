Payment Express is one of the most popular payment gateways based in New Zealand. PxPay provides handy APIs for Auth, Purchase, and Token Billing. All APIs use XML format for data exchange. This article shows you how I use HttpClient and LINQ to XML to consume those APIs.

## Payment Express PxPay
PxPay is currently at version 2.0. You can check out its Integration Guide from [this link](https://www.paymentexpress.com/Technical_Resources/Ecommerce_Hosted/PxPay_2_0){:target="_blank"}. It also provides sample C# code. You can even customize their hosted payment page with your own CSS from their admin site. 

## Web.config
{% highlight c# %}
<appSettings>
    <add key="PxPayUri" value="https://sec.paymentexpress.com/pxpay/pxaccess.aspx"/>
    <add key="PxPayUserId" value="[Your User Id]"/>
    <add key="PxPayKey" value="[Your Key]"/>
    <add key="PxPayUrlSuccess" value="https://www.dpsdemo.com/SandboxSuccess.aspx"/>
    <add key="PxPayUrlFail" value="https://www.dpsdemo.com/SandboxSuccess.aspx"/>
    <add key="PxPayDefaultCurrencyInput" value="NZD"/>
</appSettings>
{% endhighlight %}
## SendRequest
{% highlight c# %}
private async Task<XElement> SendRequest(string type, Dictionary<string, string> data)
{
    XElement responseXml = null;

    using (var client = new HttpClient())
    {
        data["PxPayUserId"] = UserId;
        data["PxPayKey"] = Key;

        var requestXml = new XElement(type, data.Select(d => new XElement(d.Key, d.Value)).ToArray());
        var requestContent = new StringContent(requestXml.ToString(), Encoding.UTF8, "application/xml");
        HttpResponseMessage response = await client.PostAsync(Uri, requestContent);
        if (!response.IsSuccessStatusCode)
        {
            Trace.TraceError($"Request failed. Response: {response}");
            throw new PxPayException("Request failed");
        }
        else
        {
            var responseContent = await response.Content.ReadAsStringAsync();
            responseXml = XElement.Parse(responseContent);
        }
    }

    return responseXml;
}
{% endhighlight %}
 We use SendRequest to interact with Payment Express Web Services. XElement is used to parse and generate xml string.
## GenerateRequest
{% highlight c# %}
public async Task<string> GenerateRequest(TxnType txnType, string txnId, decimal amountInput, 
    string merchantReference, bool enableAddBillCard, string dpsBillingId)
{
    var data = new Dictionary<string, string>
    {
        {"TxnType", txnType.ToString()},
        {"AmountInput", amountInput.ToString()},
        {"TxnId", txnId},
        {"DpsBillingId", dpsBillingId},
        {"EnableAddBillCard", enableAddBillCard ? "1" : "0"},
        {"MerchantReference", HttpUtility.HtmlEncode(merchantReference)},
        {"CurrencyInput", CurrencyInput},
        {"UrlSuccess", UrlSuccess}, 
        {"UrlFail", UrlFail}
    };

    var xml = await SendRequest("GenerateRequest", data);

    if (xml.Element("URI") == null)
    {
        Trace.TraceError($"GenerateRequest Error. Response: {xml}");
        throw new PxPayException("GenerateRequest Error", xml.Element("Reco").Value, xml.Element("ResponseText").Value);
    } else if (xml.Attribute("valid").Value != "1")
    {
        Trace.TraceError($"GenerateRequest Invalid. Response: {xml}");
        throw new PxPayException("GenerateRequest Invalid", "", xml.Element("URI").Value);
    }

    return xml.Element("URI").Value;
}
{% endhighlight %}
We use GenerateRequest to get the URL of DPS hosted payment page. Auth, Purchase, and Token Billing all use this method.

## Handy Wrappers
{% highlight c# %}
/*
 * for purchase and bill setup
 */
public async Task<string> Purchase(string txnId, decimal amountInput, string merchantReference = "", 
    bool enableAddBillCard = false)
{
    return await GenerateRequest(TxnType.Purchase, txnId, amountInput, merchantReference, false, "");
}

/*
 * for rebill
*/
public async Task<string> Purchase(string txnId, decimal amountInput, string dpsBillingId, 
    string merchantReference = "")
{
    return await GenerateRequest(TxnType.Purchase, txnId, amountInput, merchantReference, false, dpsBillingId);
}

public async Task<string> Auth(string txnId, decimal amountInput, string merchantReference = "", 
    bool enableAddBillCard = false)
{
    return await GenerateRequest(TxnType.Auth, txnId, amountInput, merchantReference, enableAddBillCard, "");
}

public async Task<string> BillSetup(string txnId, decimal amountInput, string merchantReference = "", bool withPurchase = false)
{
    if (withPurchase)
    {
        return await Purchase(txnId, amountInput, merchantReference, true);
    }
    else
    {
        return await Auth(txnId, amountInput, merchantReference, true);
    }
}

public async Task<string> Rebill(string txnId, decimal amountInput, string dpsBillingId, 
    string merchantReference = "")
{
    return await Purchase(txnId, amountInput, dpsBillingId, merchantReference);
}
{% endhighlight %}
 I write some wrappers around GenerateRequest so that I won't be annoyed by many parameters which might not be related with the transaction type I'm about to use. For example, if you need to issue a Purchase request, you can call Purchase method with your txnId and amountInput, or Auth with Auth method etc.
## ProcessResponse
{% highlight c# %}
public async Task<Response> ProcessResponse(string response)
{
    var data = new Dictionary<string, string>
    {
        {"Response", response}
    };

    var xml = await SendRequest("ProcessResponse", data);
    if (xml.Attribute("valid").Value != "1")
    {
        Trace.TraceError($"ProcessResponse Invalid. Response: {xml}");
        throw new PxPayException("ProcessResponse Invalid", xml.Element("Reco").Value, xml.Element("ResponseText").Value);
    }
    var serializer = new XmlSerializer(typeof(Response));
    return (Response)serializer.Deserialize(xml.CreateReader());
}
{% endhighlight %}
 After payment is made, customers will be redirected to your SuccessUrl or FailUrl with a response code. You can get transaction details with the code via ProcessResponse. We use XmlSerializer to deserialize xml result to a Response object. Instead of traversing an XML for the info, checking against an Response object is much more handy.

## Github
[https://github.com/dujushi/snippets/tree/master/PxPay](https://github.com/dujushi/snippets/tree/master/PxPay){:target="_blank"}

## Reference
1. [Payment Express Hosted - PxPay 2.0](https://www.paymentexpress.com/Technical_Resources/Ecommerce_Hosted/PxPay_2_0){:target="_blank"}
2. [LINQ To XML Tutorials with Examples](http://www.dotnetcurry.com/linq/564/linq-to-xml-tutorials-examples){:target="_blank"}
3. [How to send XML content with HttpClient.PostAsync](http://stackoverflow.com/questions/25352462/how-to-send-xml-content-with-httpclient-postasync){:target="_blank"}
4. [Mixing XmlSerializers with XElements and LINQ to XML](http://www.hanselman.com/blog/MixingXmlSerializersWithXElementsAndLINQToXML.aspx){:target="_blank"}
5. [Creating and Throwing Exceptions](https://msdn.microsoft.com/en-us/library/ms173163.aspx){:target="_blank"}
6. [How to use web.config when unit testing an asp .net application](http://stackoverflow.com/questions/516233/how-to-use-web-config-when-unit-testing-an-asp-net-application){:target="_blank"}
