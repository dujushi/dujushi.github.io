It is quit often we want to post a list of data to an action to save them. Ideally ASP.NET MVC Model Binding should be able to deal with this scenario. It should bind them into a list of data entities. And yes! Model Binding does offer us this convenicency. You can check out references below for more details. In this post, I will show how I use this feature with jQuery Ajax Post.

## Post Action
{% highlight c# %}
[HttpPost]
public ActionResult AddChores(ICollection<Chore> chores)
{
    if (chores.Any())
    {
        _db.Chores.AddRange(chores);
        _db.SaveChanges();
    }
    
    return Json(new { Success = true });
}
{% endhighlight %}
We declare a collection of chore data entities.

## Ajax Post
{% highlight js %}
$(".ux-submit-new-chores").click(function(e) {
    e.preventDefault();

    var chores = [];
    $(".ux-new-chore").each(function() {
        var chore = {
            name: $(this).find('input[name="name"]').val(),
            status: $(this).find('input[name="status"]').prop("checked")
        };
        chores.push(chore);
    });

    $.ajax({
        context: this,
        type: "POST",
        url: "/Home/AddChores/",
        data: JSON.stringify({ chores }),
        contentType: "application/json; charset=utf-8",
        dataType: "json",
        success: function (data) {
            if (data.Success) {
                location.href = location.href;
            }
        }
    });
});
{% endhighlight %}
We loop through all new chores and generate an array of them. And we post the data in JSON format to the action method.

## Github
[https://github.com/dujushi/snippets/tree/master/CollectionModelBinding](https://github.com/dujushi/snippets/tree/master/CollectionModelBinding){:target="_blank"}

## References
1. [ASP.NET Wire Format for Model Binding to Arrays, Lists, Collections, Dictionaries](http://www.hanselman.com/blog/ASPNETWireFormatForModelBindingToArraysListsCollectionsDictionaries.aspx){:target="_blank"}
2. [ASP.Net MVC 3 - JSON Model binding to array](http://stackoverflow.com/questions/5284613/asp-net-mvc-3-json-model-binding-to-array){:target="_blank"}
