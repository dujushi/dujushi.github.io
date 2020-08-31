While [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/){:target="_blank"} has a [connection resiliency](https://docs.microsoft.com/en-us/ef/core/miscellaneous/connection-resiliency){:target="_blank"} feature to retry failed database commands, [Dapper](https://github.com/StackExchange/Dapper){:target="_blank"} doesn't. [Ben Hyrman](https://hyr.mn/){:target="_blank"} wrote a nice article [SQL Server Retries with Dapper and Polly](https://hyr.mn/dapper-and-polly/){:target="_blank"} to show us how they use [Polly](https://github.com/App-vNext/Polly.Extensions.Http){:target="_blank"} to support this. By following the article, I wrote a tiny [sample project](https://github.com/dujushi/Garage.Polly.Extensions.Dapper){:target="_blank"} for my future reference. 

### SqlServerTransientExceptionDetector
`SqlServerTransientExceptionDetector` is a static class to check if the exceptions are from transient errors. The main exception it checks is `SqlException`. Microsoft maintains [two versions](https://devblogs.microsoft.com/azure-sql/microsoft-data-sqlclient-2-0-0-is-now-available/){:target="_blank"} of `SqlException` currently: `Microsoft.Data.SqlClient` and `System.Data.SqlClient`. This sample project uses `Microsoft.Data.SqlClient`.

### Retry Policy
```
private const int RetryCount = 4;
private static readonly Random Random = new Random();
private static readonly AsyncRetryPolicy RetryPolicy = Policy
    .Handle<SqlException>(SqlServerTransientExceptionDetector.ShouldRetryOn)
    .Or<TimeoutException>()
    .OrInner<Win32Exception>(SqlServerTransientExceptionDetector.ShouldRetryOn)
    .WaitAndRetryAsync(
        RetryCount,
        currentRetryNumber => TimeSpan.FromSeconds(Math.Pow(1.5, currentRetryNumber - 1)) + TimeSpan.FromMilliseconds(Random.Next(0, 100)),
        (currentException, currentSleepDuration, currentRetryNumber, currentContext) =>
        {
#if DEBUG
            Debug.WriteLine($"=== Attempt {currentRetryNumber} ===");
            Debug.WriteLine(nameof(currentException) + ": " + currentException);
            Debug.WriteLine(nameof(currentContext) + ": " + currentContext);
            Debug.WriteLine(nameof(currentSleepDuration) + ": " + currentSleepDuration);
#endif
        });
```
The retry policy [adds some randomness to the exponential backoff in case of high concurrency](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly#add-a-jitter-strategy-to-the-retry-policy){:target="_blank"}.

### Garage.Polly.Extensions.Dapper
The nuget package is pushed to [nuget.org](https://www.nuget.org/packages/Garage.Polly.Extensions.Dapper/){:target="_blank"} with an Azure Pipeline wich you can find in the repository.


### Sample project
I have also created a sample project to test the nuget package: `Garage.Polly.Extensions.Dapper.Sample`. There is a Terraform script to provision an Azure Sql Database for testing. A Fluent Migrator project is added to set up the test tables. 

