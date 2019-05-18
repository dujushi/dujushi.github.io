This pagination feature with partial view was inspired by Adam Freeman's pagination html helper in his book `<Pro ASP.NET MVC 5>`. Instead of using html helper, I implemented it with a partial view so I could customize the content and style easily.<!-- more -->

## Pagination View Model
{% highlight c# %}
public class Pagination
{
    public int TotalItems { get; set; }
    public int ItemsPerPage { get; set; }
    public int CurrentPage { get; set; }
    public Func<int, string> PageUrl { get; set; }
    public int TotalPages => (int)Math.Ceiling((decimal)TotalItems / ItemsPerPage);
}
{% endhighlight %}
We use lamba expression PageUrl to generate page url dynamically.

## Partial View
{% highlight cshtml %}
@model Pagination.Models.Pagination

@{
    bool showPrevious = Model.CurrentPage > 1;
    bool showNext = Model.CurrentPage < Model.TotalPages;
    bool showAll = Model.TotalPages <= 11;
    int gapSize = 5;
    bool showFirstGap = Model.CurrentPage - 1 >= gapSize;
    bool showSecondGap = Model.TotalPages - Model.CurrentPage >= gapSize;
}

@if (Model.TotalPages > 1)
{
    <div class="paginate">
        <ul>
            @if (showPrevious)
            {
                <li>
                    <a href="@Model.PageUrl(Model.CurrentPage - 1)">Previous</a>
                </li>
            }

            @if (showAll)
            {
                for (int i = 1; i <= Model.TotalPages; i++)
                {
                    <li>
                        <a href="@Model.PageUrl(i)" class="@(Model.CurrentPage == i ? "active" : "")">@i</a>
                    </li>
                }
            }
            else
            {
                if (showFirstGap && showSecondGap)
                {
                    for (int i = 1; i <= 2; i++)
                    {
                        <li>
                            <a href="@Model.PageUrl(i)">@i</a>
                        </li>
                    }
                    <li>
                        <a href="#" class="gap">&hellip;</a>
                    </li>
                    for (int i = Model.CurrentPage - 1; i <= Model.CurrentPage + 1; i++)
                    {
                        <li>
                            <a href="@Model.PageUrl(i)" @(Model.CurrentPage == i ? "class=active" : "")>@i</a>
                        </li>
                    }
                    <li>
                        <a href="#" class="gap">&hellip;</a>
                    </li>
                    for (int i = Model.TotalPages - 1; i <= Model.TotalPages; i++)
                    {
                        <li>
                            <a href="@Model.PageUrl(i)">@i</a>
                        </li>
                    }
                }
                else if (!showFirstGap)
                {
                    for (int i = 1; i <= gapSize + 1; i++)
                    {
                        <li>
                            <a href="@Model.PageUrl(i)" @(Model.CurrentPage == i ? "class=active" : "")>@i</a>
                        </li>
                    }
                    <li>
                        <a href="#" class="gap">&hellip;</a>
                    </li>
                    for (int i = Model.TotalPages - 1; i <= Model.TotalPages; i++)
                    {
                        <li>
                            <a href="@Model.PageUrl(i)">@i</a>
                        </li>
                    }
                }
                else
                {
                    for (int i = 1; i <= 2; i++)
                    {
                        <li>
                            <a href="@Model.PageUrl(i)">@i</a>
                        </li>
                    }
                    <li>
                        <a href="#" class="gap">&hellip;</a>
                    </li>
                    for (int i = Model.TotalPages - gapSize; i <= Model.TotalPages; i++)
                    {
                        <li>
                            <a href="@Model.PageUrl(i)" @(Model.CurrentPage == i ? "class=active" : "")>@i</a>
                        </li>
                    }
                }
            }

            @if (showNext)
            {
                <li>
                    <a href="@Model.PageUrl(Model.CurrentPage + 1)">Next</a>
                </li>
            }
        </ul>
    </div>
}
{% endhighlight %}
If there are more than 11 page links, it will hide some page links to make it tidy.

## Style
{% highlight css %}
.paginate {
  text-align: center; 
}

.paginate ul {
    list-style: none;
    margin: 0;
    padding: 0; 
}

.paginate li {
    display: inline; 
}

.paginate a {
    text-decoration: none;
    margin: 1px 2px;
    padding: 5px 10px;
    border-radius: 3px;
    box-shadow: rgba(0, 0, 0, 0.2) 0 0 0 1px;
    color: #717171;
    background-color: #f5f5f5; 
}

.paginate .active {
    color: #f2f2f2;
    background-color: #676767;
    border-color: #505050; 
}

.paginate .gap {
    background: transparent;
    box-shadow: 0 0 0 0 transparent;
    cursor: context-menu; 
}
{% endhighlight %}
I borrowed some nice style from this repo: https://github.com/brajeshwar/paginate

## Controller
{% highlight c# %}
public ActionResult List(int page = 1)
{
    var model = db.Tasks.OrderByDescending(t => t.Id).AsQueryable();
    int totalItems = model.Count();
    int itemsPerPage = 10;
    model = model
        .Skip(itemsPerPage*(page - 1))
        .Take(itemsPerPage);

    ViewBag.Pagination = new Pagination
    {
        TotalItems = totalItems,
        ItemsPerPage = itemsPerPage,
        CurrentPage = page,
        PageUrl = x => Url.Action("List", new {page = x})
    };

    return View(model);
}
{% endhighlight %}

## Include Partial View
{% highlight c# %}
@Html.Partial("_Pagination", (Pagination)ViewBag.Pagination)
{% endhighlight %}
