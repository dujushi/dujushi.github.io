To limite page size, we only want to list a limited number of items at first. And then click Show More button to render more items. We can use partial view to reuse our code. In this example we will implement a simple task list. 

* Create show task action
{% highlight c# %}
public ActionResult ShowTask(bool showAll = false)
{
    int pageSize = 3;
    bool showMoreButton = false;
    var model = db.Tasks.AsQueryable();
    if (!showAll)
    {
        showMoreButton = model.Count() > pageSize;
        model = model.Take(pageSize);
    }
    ViewBag.ShowMoreButton = showMoreButton;
    return PartialView("_ShowTask", model);
}
{% endhighlight %}

* Create PartialView _ShowTask.cshtml
{% highlight html %}
@model IEnumerable<HtmlAction.Models.Task>

<div id="task_container">
    @foreach (var item in Model)
    {
        <div data-id="@item.Id" class="task_item">
            <span>
            @item.Content
        </span>
            <a href="#" class="switch@(item.Status ? " complete" : "")" title="Complete">&#10004</a>
        </div>
    }

    @if (ViewBag.ShowMoreButton)
    {
        <a href="#" id="show_more">Show More</a>
    }
</div>
{% endhighlight %}

* In the list page, we can use Html.Action to show some items.
{% highlight c# %}
@Html.Action("ShowTask")
{% endhighlight %}

* And then we implement an AJAX function for 'Show More' button.
{% highlight javascript %}
$("#show_more").click(function() {
    $.ajax({
        type: "GET",
        url: "/Task/ShowTask",
        data: { "showAll": true},
        success: function (data) {
            $("#task_container").html(data);
        },
        error: function (request, status, err) {
            alert("System error, try again later.");
        }
    });
});
{% endhighlight %}
* We may want to implement an AJAX function to switch the status.
{% highlight javascript %}
$(document).on("click", ".switch", function () {
    var id = $(this).closest(".task_item").data("id");
    var className = "complete";
    var status = $(this).hasClass(className);
    $.ajax({
        context: this,
        type: "POST",
        url: "/Task/StatusSwitch",
        data: { "id": id, "status": status },
        dataType: "json",
        success: function (data) {
            if (data.Success == true) {
                if (status) {
                    $(this).removeClass(className);
                } else {
                    $(this).addClass(className);
                }
            } else {
                alert("System error, try again later.");
            }
        },
        error: function (request, status, err) {
            alert("System error, try again later.");
        }
    });
});
{% endhighlight %}

There are some tricks in this function:
1. we use `$(document).on("click", ".switch", function () {})` instead of `$(".switch").click(function() {})`, so that all AJAX renderred switches can invoke the AJAX function as well.
2. We add `context: this` to change context to the switch button we click on. So we can change the style of the switch button inside the function.