I created a [tiny project](https://github.com/dujushi/BlobStorageZipArchive){:target="_blank"} which provides a Web Api endpoint to download files from Azure Blob Storage as zip archive. In this article, I will explain some fo the important pieces. 

## Add Zip Archive Entry From Cloud Blob Directory
{% highlight csharp %}
private static async Task AddZipArchiveEntryFromCloudBlobDirectory(CloudBlobDirectory cloudBlobDirectory, ZipArchive zipArchive)
{
    BlobContinuationToken blobContinuationToken = null;
    do
    {
        var blobResultSegment = await cloudBlobDirectory.ListBlobsSegmentedAsync(blobContinuationToken);
        blobContinuationToken = blobResultSegment.ContinuationToken;
        foreach (var listBlobItem in blobResultSegment.Results)
        {
            switch (listBlobItem)
            {
                case CloudBlobDirectory subCloudBlobDirectory:
                    await AddZipArchiveEntryFromCloudBlobDirectory(subCloudBlobDirectory, zipArchive);
                    break;
                case CloudBlockBlob cloudBlockBlob:
                {
                    var entry = zipArchive.CreateEntry(cloudBlockBlob.Name);
                    if (cloudBlockBlob.Properties.LastModified.HasValue)
                    {
                        entry.LastWriteTime = cloudBlockBlob.Properties.LastModified.Value;
                    }
                    using (var entryStream = entry.Open())
                    {
                        await cloudBlockBlob.DownloadToStreamAsync(entryStream);
                    }

                    break;
                }
            }
        }
    } while (blobContinuationToken != null);
}
{% endhighlight %}

This is a private function in `BlobStorageZipArchiveService`. It loops through each item in the directory and creates an entry in zip archive. If the item itself is a directory, the function will call itself. So this is a recursive function. 

## Generate Zip Archive Stream
{% highlight csharp %}
public async Task GenerateZipArchiveStream(Stream stream, string containerName, string relativeAddress)
{
    var cloudBlobContainer = _cloudBlobClient.GetContainerReference(containerName);
    // make sure container exists
    var exists = await cloudBlobContainer.ExistsAsync();
    if (!exists)
    {
        throw new BlobStorageZipArchiveException("cannot find container");
    }

    var cloudBlobDirectory = cloudBlobContainer.GetDirectoryReference(relativeAddress);
    // make sure there are blobs under the relative address
    var results = await cloudBlobDirectory.ListBlobsSegmentedAsync(null);
    if (!results.Results.Any())
    {
        throw new BlobStorageZipArchiveException("cannot find any file under the relative address");
    }

    // generate zip archive stream
    using (var zipArchive = new ZipArchive(stream, ZipArchiveMode.Create, leaveOpen: true))
    {
        await AddZipArchiveEntryFromCloudBlobDirectory(cloudBlobDirectory, zipArchive);
    }

    // reset stream position
    stream.Seek(0, SeekOrigin.Begin);
}
{% endhighlight %}
This function resides in `BlobStorageZipArchiveService` as well. We have to check if the container exists and also if the relative address contains any files before we create the zip archive. Otherwise, it will generate an empty archive file which is invalid. `leaveOpen` will keep the stream open, so we can read it later. The last statement reset stream position. If we forget to do that, controller will try to download from the end of the stream which has nothing and the API endpoint will freeze.

## Blob Storage Zip Archive Controller
{% highlight csharp %}
[HttpGet]
public async Task<ActionResult<IEnumerable<string>>> Get()
{
    var memoryStream = new MemoryStream();
    const string containerName = "containername";
    const string relativeAddress = "relativeAddress";
    await _blobStorageZipArchiveService.GenerateZipArchiveStream(memoryStream, containerName, relativeAddress);
    return File(memoryStream, "application/octet-stream", "fileDownloadName.zip");
}
{% endhighlight %}

This is the final endpoint. We cannot use `using` when instantiate `MemoryStream` instance, otherwise it will close the stream before `File` method tries to read it. We also leave `memoryStream` to `File` method to close. 