# CPM coock book

## Naming

#### Avoid misleading and cryptic names

👎 Bad

```csharp
var ds = new DeliverySite(); // not clear
var point = new ConnectionPoint(); // not clear
var deliverySiteForOverviewPanel = new DeliverySiteViewModel(); // hard to read
var newCp = new ConnectionPoint(); // confusing CP = connection point = connection process
```

In general avoid of making shortcuts like`delSite` it is little bit shorter than `deliverySite` but in some cases it can be confusing so it is better to not build this habit. 

There is one acceptable exception `DeliverySiteVm` **Viewmodel** is often shorten just to **Vm** this shortcut makes mapping methods less noisy, and in our project it is so common that nobody will mix it with Virtual machine. But always use **camelCase** styling so uppercase **V** and lower case **m**.

#### Don't add unnecessary context

👎 Bad

```csharp
public class DeliverySite 
{
    public string DeliverySiteId { get; set; }
    public string DeliverySiteType { get; set; }
    public Address DeliverySiteAddress { get; set; }
    
    public DeliverySite GetDeliverySite(string deliverySiteId)
    { }
}
```

Adding entity name inside variable and function add unnecessary noise and make code longer. 

👍 Good

```csharp
public class DeliverySite 
{
    public string Id { get; set; }
    public string Type { get; set; }
    public Address Address { get; set; }
    
    public DeliverySite Get(string id)
    { }
}
```

## Conditions

### Brackets

Don't use two line conditions without brackets. Without clearly defined borders brackets border is hard to read where the nested code start and end. This is especially hard in wrongly formatted code. 

And it is also a big source of stupid mistakes when you add another line to condition and don't realize that it will not be evaluated in your condition, the compiler doesn't give you any warning.

**👎 Bad**

```csharp
if (deliverySite.IsCleared())
    return new DeliverySite();
```

**👍 Good**

```csharp
if (deliverySite.IsCleared())
{
    return new DeliverySite();
}
```

### Complicated conditions

Avoid of complicated conditions based of external parameters \(metadata, configurations\) or multiple constants or variables. Try to avoid of conditions which have more than 3 conditions or conditions which cannot fit into one line.

It is better to export decision logic to independent unit \(e.g. helper classes, utils\) or split condition to smaller pieces.

**👎 Bad**

```csharp
if (deliverySites?.Contains(deliverySite.Id) && deliverySite.Type == Int32.Parse(metadataClient.GetMetadata("validDeliverySiteType"))
{
    return deliverySite;
}
```

👍 **Good**

```csharp
if (productionDeliverySiteHelper.IsValid(deliverySite))
{
    return deliverySite;
}
```

## Functions & classes

### 

## Comments

### Regions 

**❗ Never** use regions in any cases. Concept of regions is inconsistent with OOP practices, regions brings another level of structuring code and nesting apart of classes and methods. 

Developers often use regions to hide long and complicated code for better orientation, but it is strongly against principles of clean code and OOP. If code is too complicated to understand you should thing more about meaning of you code, try to found independent entities which can be changed to independent reusable classes with their own properties and methods. Exclude all complicated conditions to separate methods to make it more readable and understandable. Avoid of long DTO to DTO mapping in controllers and methods which contains actual logic.

Using regions not make your code cleaner or better readable it just hide bad and ugly code under the carpet which is not even a part of any programming paradigm.

### Commenting old code

❗ Do not push your code to Git with commented code blocks and don't promise that you will do it later.

> LeBlanc's Law:   **Later == Never**

Commented blocks makes code hard to read for other people and does not bring any added value, nobody will read your old commented code. And don't worry you will never lost your old code it will be in Git history forever.

👎 **Bad**

```csharp
// if (deliverySite.IsCleared())
// {
//     return new DeliverySite();
// }
```

### Commenting complex business

✔ At the first place we should always try to make code self explanatory, use naming so clear that everyone will understand meaning of code without any comments. But sometimes under your code be some non obvious business or something really domain specific. 

In those cases it is really good to add one line comment for your colleagues or your future self. 

👍 Good

```csharp
public string GridCode { get; set; } // Grid owner code in CAB
```

### Doc strings 

✔ It is good to use doc string comments above non-trivial methods \(trivial methods == Add, Set, Update, Delete, IsAdded, ...\). If you have filled doc strings Intelli-sense will help you what method do without checking implementation.

👍 Good

```csharp
/// <summary>
/// Get changes between delivery sites. Good for resolving changes for WFE.
/// </summary>
/// <param name="original">Original Delivery site</param>
/// <param name="updated">New Delivery site</param>
public DeliverySite GetDiff(DeliverySite original, DeliverySite updated)
{
    // code ...
}
```



