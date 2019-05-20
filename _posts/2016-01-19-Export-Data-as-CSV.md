CSV is a common format for data exchange. It stands for "Comma Separated Values". Values need to be double-quoted if they contain comma, double quotes, leading or trailing spaces. The format is actually quite simple. We can generate one easily with StringBuidler.

## Product Model
{% highlight c# %}
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string PhotoUrl { get; set; }
    public string ThumbnailUrl { get; set; }
}
{% endhighlight %}
## Fake Product Data
{% highlight c# %}
private List<Product> getProducts()
{
    return Enumerable.Range(0, 1000).Select(x => new Product {Id = x, Name = $"Name {x}"}).ToList();
}
{% endhighlight %}
## CSV with StringBuilder
{% highlight c# %}
public FileResult Csv()
{
    var data = new StringBuilder();
    data.AppendLine("Id,Name");
    foreach (var product in getProducts())
        data.AppendLine($"{product.Id},{product.Name}");
    var bytes = Encoding.UTF8.GetBytes(data.ToString());
    return File(bytes, "text/csv", "Products.csv");
}
{% endhighlight %}
We can also generate a CSV file with FileHelpers library.
## CSV with FileHelpers
{% highlight c# %}
[DelimitedRecord(",")]
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string PhotoUrl { get; set; }
    public string ThumbnailUrl { get; set; }
}
    
public FileResult FileHelperCsv()
{
    var engine = new FileHelperEngine<Product> {HeaderText = "Id,Name"};
    var data = engine.WriteString(getProducts());
    var bytes = Encoding.UTF8.GetBytes(data);
    return File(bytes, "text/csv", "Products.csv");
}
{% endhighlight %}
FileHelpers is handy. We can install FileHelpers through Nuget Package Console. But when data gets complicated we may need to create a specific data model for it. For exaple, we may want to exclude PhotoUrl and ThumbnailUrl from the result. We can add FieldHidden annotations to hide them. But the annotation only supports fields, not properties. 

## Github
[https://github.com/dujushi/snippets/tree/master/CSV](https://github.com/dujushi/snippets/tree/master/CSV){:target="_blank"}

## References
1. [Comma-separated values](https://en.wikipedia.org/wiki/Comma-separated_values){:target="_blank"}
2. [FileHelpers](http://www.filehelpers.net/){:target="_blank"}
