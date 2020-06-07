---
title: 'Sync Cosmos DB Collection to Blob Storage with Azure Functions Cosmos DB Trigger'
---
[Azure Cosmos DB change feed](https://docs.microsoft.com/en-us/azure/cosmos-db/change-feed){:target="_blank"} allows us to listen to all inserts and updates to Cosmos DB collections. [Azure Cosmos DB trigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-trigger?tabs=csharp){:target="_blank"} is the easiest way to utilize the feature. This article demonstrates how to sync Cosmos DB collection to Blob Storage with change feed.

### Azure Cosmos DB Emulator
Follow [Azure Cosmos DB Emulator](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator){:target="_blank"} document to install the emulator. So we can develop Cosmos DB applications locally.

### Azure Storage Emulator
[Azure Storage Emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator?toc=/azure/storage/blobs/toc.json){:target="_blank"} comes with Visual Studio 2019. Follow [this document](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator?toc=/azure/storage/blobs/toc.json#start-and-initialize-the-storage-emulator){:target="_blank"} to start the emulator. 

### Azure Storage Explorer
We can use [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/){:target="_blank"} to view contents in Azure Storage Emulator.

### Getting Started with Azure Functions
If you are new to Azure Functions, follow [Quickstart: Create your first function in Azure using Visual Studio](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-your-first-function-visual-studio){:target="_blank"} to create your first function. Instead of using HTTP trigger, let's use Cosmos DB trigger when create the project.

### Blob Storage Client Dependency Injection
To manipulate blob storage, we need to inject the blob storage client to the function. [Use dependency injection in .NET Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection){:target="_blank"} has a clear explaination on how to do it. The following code shows how to configure the blob container client. `BlobContainerClient` is from [Azure.Storage.Blobs](Azure.Storage.Blobs){:target="_blank"} nuget package.

{% highlight csharp %}
builder.Services.AddSingleton(x =>
{
    var configuration = x.GetService<IConfiguration>();
    var container = new BlobContainerClient(configuration["PaymentBlobStorageConnectionString"], configuration["PaymentBlobStorageContainerName"]);
    container.CreateIfNotExists();
    return container;
});
{% endhighlight %}

### Function
The following implementation first extracts payment model from the document object and then use the injected blob container client to upload payment data. Check out [AzureCosmosDBChangeFeedSample](https://github.com/dujushi/AzureCosmosDBChangeFeedSample){:target="_blank"} for the complete source code.

{% highlight csharp %}
public class ChangeFeedSample
{
    private readonly BlobContainerClient _blobContainerClient;

    public ChangeFeedSample(BlobContainerClient blobContainerClient)
    {
        _blobContainerClient = blobContainerClient ?? throw new ArgumentNullException(nameof(blobContainerClient));
    }

    [FunctionName(nameof(ChangeFeedSample))]
    public async Task Run(
        [CosmosDBTrigger(
            "ChangeFeedSample",
            "Payment",
            ConnectionStringSetting = "CosmosDBConnectionString",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]
        IReadOnlyList<Document> documents,
        ILogger log)
    {
        if (documents != null && documents.Count > 0)
        {
            log.LogInformation("Documents modified: " + documents.Count);
            foreach (var document in documents)
            {
                var jsonSerializerOptions = new JsonSerializerOptions
                {
                    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
                };
                var payment = JsonSerializer.Deserialize<Payment>(document.ToString(), jsonSerializerOptions);
                var paymentString = JsonSerializer.Serialize(payment, jsonSerializerOptions);
                var memoryStream = new MemoryStream(Encoding.UTF8.GetBytes(paymentString));
                await _blobContainerClient.UploadBlobAsync(document.Id, memoryStream);
            }
        }
    }
}
{% endhighlight %}