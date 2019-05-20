Using a large number of values in an IN clause is not recommended. It is time consuming and may result in an error. Out of curiosities I ran two simple tests against this.

## With Entity Framework
{% highlight c# %}
public void InClauseMaxValuesWithEntityFramework()
{
    using (var db = new EfDbContext())
    {
        for (var i = 10000; i < 20000; i += 1000)
        {
            var log = "";
            db.Database.Log = s => log += s;
            var idArr = Enumerable.Range(0, i).Select(x => Guid.NewGuid()).ToArray();
            var count = db.Categories.Count(c => idArr.Contains(c.Id));
            Console.WriteLine($"Guid count: {i}, sql length: {log.Length}, result: {count}");
        }
    }
}
{% endhighlight %}
I set db.Database.Log to log query into local log variable. The queries were really slow with more than 10,000 of values. I got the following result.
{% highlight c# %}
Test Name:  InClauseMaxValuesWithEntityFramework
Test Outcome:   Failed
Result Message: 
Test method UnitTest.Demo.InClauseMaxValuesWithEntityFramework threw exception: 
System.Data.Entity.Core.EntityCommandExecutionException: An error occurred while executing the command definition. See the inner exception for details. ---> System.Data.SqlClient.SqlException: Timeout expired.  The timeout period elapsed prior to completion of the operation or the server is not responding. ---> System.ComponentModel.Win32Exception: The wait operation timed out
Result StandardOutput:  
Guid count: 10000, sql length: 662035, result: 0
Guid count: 11000, sql length: 726388, result: 0
Guid count: 12000, sql length: 792389, result: 0
Guid count: 13000, sql length: 858389, result: 0
Guid count: 14000, sql length: 924389, result: 0
Guid count: 15000, sql length: 990389, result: 0
{% endhighlight %}
It threw a timeout exception when it reached 16,000 values.

## With Raw Query
{% highlight c# %}
public void InClauseMaxValuesWithRawQuery()
{
    using (var db = new EfDbContext())
    {
        for (var i = 10000; i < 20000; i += 1000)
        {
            var idArr = Enumerable.Range(0, i).Select(x => Guid.NewGuid()).ToArray();
            var idStr = string.Join("','", idArr);
            var sql = $"select count(*) from categories where id in ('{idStr}')";
            var count = db.Database.SqlQuery<int>(sql).Single();
            Console.WriteLine($"Guid count: {i}, sql length: {sql.Length}, result: {count}");
        }
    }
}
{% endhighlight %}
We can use db.Database.SqlQuery to run raw query. Result:
{% highlight c# %}
Test Name:  InClauseMaxValuesWithRawQuery
Test Outcome:   Failed
Result Message: 
Test method UnitTest.Demo.InClauseMaxValuesWithRawQuery threw exception: 
System.Data.SqlClient.SqlException: Timeout expired.  The timeout period elapsed prior to completion of the operation or the server is not responding. ---> System.ComponentModel.Win32Exception: The wait operation timed out
Result StandardOutput:  
Guid count: 10000, sql length: 390045, result: 0
Guid count: 11000, sql length: 429045, result: 0
Guid count: 12000, sql length: 468045, result: 0
Guid count: 13000, sql length: 507045, result: 0
Guid count: 14000, sql length: 546045, result: 0
Guid count: 15000, sql length: 585045, result: 0
Guid count: 16000, sql length: 624045, result: 0
{% endhighlight %}
It threw a timeout exception with 17,000 values. 

If we have this extremely large number of values, we should store them into a table first and then query against it.

## Github
[https://github.com/dujushi/snippets/tree/master/InClauseLargeNumberOfValues](https://github.com/dujushi/snippets/tree/master/InClauseLargeNumberOfValues){:target="_blank"}

## Reference
1. [IN (Transact-SQL)](https://technet.microsoft.com/en-us/library/ms177682.aspx){:target="_blank"}
2. [Raw SQL Queries](https://msdn.microsoft.com/en-nz/data/jj592907.aspx){:target="_blank"}
3. [Logging and Intercepting Database Operations (EF6 Onwards)](https://msdn.microsoft.com/en-us/data/dn469464.aspx){:target="_blank"}
