---
slug: auto-mapping
title: Auto Mapping
---

## Using ServiceStack's Built-in Auto-mapping

Although [we encourage keeping separate DTO models](http://stackoverflow.com/a/15369736/85785), you don't need to maintain your own manual mapping as you can use ServiceStack's built-in Auto Mapping support. It's quite comprehensive and resilient and does a good job in being able to co-erce one type into another, e.g. you can convert between different Enum types with the same name, between Enums and any value type and Strings, between properties and fields, POCOs and strings and many things in between - some of which can be seen in these [Auto Mapping tests](https://github.com/ServiceStack/ServiceStack.Text/blob/master/tests/ServiceStack.Text.Tests/AutoMappingTests.cs).

Here are some typical common use-cases you're likely to hit in your web service development travels:

Create a new DTO instance, populated with matching properties on viewModel:

```csharp
var dto = viewModel.ConvertTo<MyDto>();

var dto = new { Anon = "Object" }.ConvertTo<MyDto>();
```

Initialize DTO and populate it with matching properties on a view model:

```csharp
var dto = new MyDto { A = 1, B = 2 }.PopulateWith(viewModel);
```

Initialize DTO and populate it with **non-default** matching properties on a view model:

```csharp
var dto = new MyDto { A = 1, B = 2 }.PopulateWithNonDefaultValues(viewModel);
```

Initialize DTO and populate it with matching properties that are annotated with the **Attr** Attribute on a view model:

```csharp
var dto = new MyDto { A=1 }
    .PopulateFromPropertiesWithAttribute(viewModel, typeof(CopyAttribute));
```

There is also the inverse for mapping all properties that don't include a specific attribute:

```csharp
var safeUpdate = db.SingleById<MyTable>(id)
    .PopulateFromPropertiesWithoutAttribute(dto, typeof(ReadOnlyAttribute));
```

### Advanced mapping using custom extension methods

When mapping logic becomes more complicated we like to use extension methods to keep code DRY and maintain the mapping in one place that's easily consumable from within your application, e.g:

```csharp
public static class ConvertExtensions
{
    public static MyDto ToDto(this MyViewModel from)
    {
        var to = from.ConvertTo<MyDto>();
        to.Items = from.Items.ConvertAll(x => x.ToDto());
        to.CalculatedProperty = Calculate(from.Seed);
        return to;
    }
}
```

Which is now easily consumable with just:

```csharp
var dto = viewModel.ToDto();
```

### Populating Types from an Object Dictionary

The `ToObjectDictionary` and `FromObjectDictionary` extension methods are also useful in trying to convert loosely-typed data structures into a Typed POCO's, e.g:

```csharp
var dto = new User
{
    FirstName = "First",
    LastName = "Last",
    Car = new Car { Age = 10, Name = "ZCar" },
};

Dictionary<string,object> map = dtoUser.ToObjectDictionary();

map["LastName"] = "Updated";

User user = (User)map.FromObjectDictionary(typeof(User));
```


  [1]: http://martinfowler.com/eaaCatalog/dataTransferObject.html
  [2]: http://msdn.microsoft.com/en-us/library/ff649585.aspx
  [3]: http://www.palmmedia.de/Blog/2011/8/30/ioc-container-benchmark-performance-comparison
  [4]: /clients-overview
  [5]: http://ayende.com/blog/4769/code-review-guidelines-avoid-inheritance-for-properties
  [6]: https://github.com/AutoMapper/AutoMapper