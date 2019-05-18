We often use an Enum in C# to define all options for a field, such as a category Enum:
{% highlight c# %}
public enum CategoryEnum
{
    Book,
    Food,
    Tourism
}
{% endhighlight %}
And we need a dropdown list for this category field. How would you implement this?<!-- more -->

## Basic
First, we get all values of CategoryEnum:
{% highlight c# %}
Enum.GetValues(typeof(CategoryEnum))
{% endhighlight %}
Then, we generate all select items for these values:
{% highlight c# %}
Enum.GetValues(typeof(CategoryEnum)).Cast<CategoryEnum>().Select(t => new SelectListItem { Text = t.ToString(), Value = ((int)t).ToString() })
{% endhighlight %}
Here we use `Cast<CategoryEnum>()` to cast all the values so we can apply Select method on them.
Finally, we have our dropdown list:
{% highlight c# %}
@Html.DropDownListFor(m => m.Category, Enum.GetValues(typeof(CategoryEnum)).Cast<CategoryEnum>().Select(t => new SelectListItem { Text = t.ToString(), Value = ((int)t).ToString() }), "Select one")
{% endhighlight %}

## Custom Text
We can add descriptions to CategoryEnum and use them as the text value of each select list item.
{% highlight c# %}
public enum CategoryEnum
{
    [Description("Fancy Book")]
    Book,
    [Description("Delicious Food")]
    Food,
    [Description("The World Is Big")]
    Tourism
}
{% endhighlight %}
We then get the description with Attribute.GetCustomAttribute method.
`{% highlight c# %}
((DescriptionAttribute)Attribute.GetCustomAttribute((t.GetType()).GetField(t.ToString()), typeof(DescriptionAttribute))).Description
{% endhighlight %}
The updated code for the dropdown list would be:
{% highlight c# %}
@Html.DropDownListFor(m => m.Category, Enum.GetValues(typeof(CategoryEnum)).Cast<CategoryEnum>().Select(t => new SelectListItem { Text = ((DescriptionAttribute)Attribute.GetCustomAttribute((t.GetType()).GetField(t.ToString()), typeof(DescriptionAttribute))).Description, Value = ((int)t).ToString() }), "Select one")
{% endhighlight %}

## Option Group
Sometimes we only want to display a subset of the Enum. We can define a static class for this so it can be shared everywhere.
{% highlight c# %}
public static class CategoryEnumGroups
{
    public static CategoryEnum[] OutdoorGroup = { CategoryEnum.Food, CategoryEnum.Tourism };
}
{% endhighlight %}
Then we can filter them with Where method.
{% highlight c# %}
Enum.GetValues(typeof(CategoryEnum)).Cast<CategoryEnum>().Where(t => CategoryEnumGroups.OutdoorGroup.Contains(t))
{% endhighlight %}
Final dropdown list code:
{% highlight c# %}
@Html.DropDownListFor(m => m.Category, Enum.GetValues(typeof(CategoryEnum)).Cast<CategoryEnum>().Where(t => CategoryEnumGroups.OutdoorGroup.Contains(t)).Select(t => new SelectListItem { Text = ((DescriptionAttribute)Attribute.GetCustomAttribute((t.GetType()).GetField(t.ToString()), typeof(DescriptionAttribute))).Description, Value = ((int)t).ToString() }), "Select one")
{% endhighlight %}