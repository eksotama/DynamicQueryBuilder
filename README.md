# DynamicQueryBuilder

[![NuGet](https://img.shields.io/nuget/dt/DynamicQueryBuilder.svg)](https://www.nuget.org/packages/DynamicQueryBuilder/)
[![DynamicQueryBuilder](https://img.shields.io/nuget/v/DynamicQueryBuilder.svg)](https://www.nuget.org/packages/DynamicQueryBuilder/)
[![Travis](https://travis-ci.org/oplog/DynamicQueryBuilder.svg?branch=master)](https://travis-ci.org/oplog/DynamicQueryBuilder)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/oplog/DynamicQueryBuilder/blob/master/LICENSE)

# Usage

Dynamic Query Builder currently can `filter` and `sort` the applied `IQueryable<T>` instance with an extension behaviour.

DynamicQueryBuilder currently supports:

* .NET Standard 2.0

## Index

- <a href="#dynamicqueryoptions">DynamicQueryOptions Class</a> 
  * <a href="#filtering">Filtering</a> 
    + <a href="#supported-operations">Supported Operations</a> 
- <a href="#sorting">Sorting</a>
- <a href="#accessing-nested-objects">Accessing Nested Objects</a> 
- <a href="#pagination">Pagination</a>
- <a href="#dynamic-query-parsing">Dynamic Query Parsing</a>
  * <a href="#query-delivery-methods">Query Delivery Methods</a>
  * <a href="#http-parameters">HTTP Parameters</a>
  * <a href="#http-query-examples">HTTP Query Examples</a>
  * <a href="#web-action-examples">Web Action Examples</a>
  * <a href="#operation-shortcodes">Operation Shortcodes</a>
- <a href="#work-in-progress">Work In Progress</a>

## How It Works

To build a dynamic query you need to create an instance of `DynamicQueryOptions` class.

### DynamicQueryOptions

- <a href="#index">Index</a>

```csharp
    public sealed class DynamicQueryOptions
    {
        public List<Filter> Filters { get; set; } = new List<Filter>();

        public List<SortOption> SortOption { get; set; } = new List<SortOption>();
    }
```

which contains your `Filter` and `Sort` operation options.

## Filtering

- <a href="#index">Index</a>

Filters are `List<Filter>` instances in the `DynamicQueryOptions` class.

Since you have to pass the value of the property as `string` the property type and value matching will be made automatically by the library. This includes `null` values as well.

### Filter Class

```csharp
    public sealed class Filter
    {
        public string PropertyName { get; set; }

        public string Value { get; set; }

        public FilterOperation Operator { get; set; } = FilterOperation.Contains;
    }
```

### Supported Operations

- <a href="#index">Index</a>

Supported filter operations:

```csharp
    public enum FilterOperation
    {
        In,
        Equals,
        LessThan,
        Contains,
        NotEqual,
        EndsWith,
        StartsWith,
        GreaterThan,
        LessThanOrEqual,
        GreaterThanOrEqual
    }
```

Example `Filter` class usage:

```csharp
var filter = new Filter
{
    Value = "myvalue",
    Operator = FitlerOperation.Equals,
    PropertyName = "MyProperty"
};
```

## Sorting

- <a href="#index">Index</a>

Sorting is an option to `DynamicQueryOptions` class as a `SortOption` instance.

### SortOption class

```csharp
    public sealed class SortOption
    {
        public string PropertyName { get; set; }

        public SortingDirection SortingDirection { get; set; }
    }
```

Example `SortOption` class usage:

```csharp
var mySortOption = new SortOption
{
    SortingDirection = SortingDirection.Asc,
    PropertyName = "MyPropertyToSort"
}
```

TIP: DQB Supports multiple sort operations.


#### Accessing Nested Objects

- <a href="#index">Index</a>

DynamicQueryBuilder can access nested objects with `.` delimeter like default LINQ.

Example: 

```csharp
public class MyNestedClass 
{
    public int Age { get; set; }
}

public class MyClassToFilter
{
    public MyNestedClass MyNestedProperty { get; set; }
}
```

To Filter:

```csharp
Filter myNestedFilter = new Filter
{
    Value = "27",
    Operator = FilterOperation.Equals,
    PropertyName = "MyNestedProperty.Age"
};
```

Or to Sort: 

```csharp
var mySortOption = new SortOption
{
    SortingDirection = SortingDirection.Asc,
    PropertyName = "MyNestedProperty.Age"
}
```

## Pagination

- <a href="#index">Index</a>

Pagination can be done by specifying options to the `PaginationOptions` class member of `DynamicQueryOptions` class.

```csharp
    public sealed class PaginationOption
    {
        // defines how many results required to fetch.
        public int Count { get; set; }

        // defines how many results required to skip.
        public int Offset { get; set; }

        // defines if the DataSetCount should be set or not.
        [JsonIgnore]
        public bool AssignDataSetCount { get; set; }

        // shows how many results that are in the actual query without pagination.
        public int DataSetCount { get; internal set; }
    }
```

To paginate, simply use:

```csharp
var paginationOption = new PaginationOption
{
    Count = 10,
    Offset = 0,
    AssignDataSetCount = true
};
```

Tip: **after query execution**, if its required to access the total query result amount(whole set) you can access it via

```csharp
int totalDataSetCount = paginationOption.DataSetCount;
```

## Dynamic Query Parsing

- <a href="#index">Index</a>

Dynamic Query Builder can also parse the given `HTTP` request query parameters by its own conventions into a `DynamicQueryOptions` instance.

#### Query Delivery Methods

- <a href="#index">Index</a>

Dynamic Query Builder can retrieve your encoded/non-encoded queries via options below: 

##### Request QueryString

This can be configured on `QueryOptionsResolver` property of `DynamicQueryBuilderSettings` class by initializing `QueryStringResolver` class.

Below, there is an example of configuring DQB to retrieve queries from query string

```csharp
string parameterToResolveFrom = "myparamtoresolve";
Func<string, string> decodeFunction = (encodedQuery) => magicDecode(encodedQuery);

new DynamicQueryBuilderSettings
{
    // Other configurations
    QueryOptionsResolver = new QueryStringResolver(parameterToResolveFrom, decodeFunction)
}
```

Tip: you can leave `parameterToResolveFrom` null to resolve your queries directly from the raw querystring.

##### Request HTTP Header

This can be configured on `QueryOptionsResolver` property of `DynamicQueryBuilderSettings` class by initializing `HttpHeaderResolver` class.

Below, there is an example of configuring DQB to retrieve queries from HTTP Headers

```csharp
string httpHeaderName = "myhttpheadername";
Func<string, string> decodeFunction = (encodedQuery) => magicDecode(encodedQuery);

new DynamicQueryBuilderSettings
{
    // Other configurations
    QueryOptionsResolver = new HttpHeaderResolver(httpHeaderName, decodeFunction)
}
```

Tip: you can always leave `decodeFunction` null if your queries are not encoded.

#### Http Parameters

- <a href="#index">Index</a>

##### `o` Parameter

* Refers to the `FilterOperation` property of `Filter` class. 
* This parameter can be placed anywhere in the querystring.
* This parameter **should be forming a triplet with `p` and `v` parameters.**

##### `p` Parameter

* Refers to the `PropertyName` property of `Filter` class. 
* This parameter **should be placed after the `o` parameter.**.
* This parameter **should be forming a triplet with `o` and `v` parameters.**

##### `v` Parameter

* Refers to the `PropertyValue` property of `Filter` class. 
* This parameter **should be placed after the `p` parameter.**.
* This parameter **should be forming a triplet with `o` and `p` parameters.**

##### `s` Parameter

* Refers to the `SortOption` class. 
* This parameter can be placed anywhere in the querystring.
*  **If this parameter occurs more than once, sorting will be done in the given order.**

##### `offset` Parameter

* Refers to the `Offset` property of `PaginationOption` class. 
* This parameter can be placed anywhere in the querystring.
*  **If this parameter occurs more than once, the first occurence will be assigned.**

##### `count` Parameter

* Refers to the `Count` property of `PaginationOption` class. 
* This parameter can be placed anywhere in the querystring.
*  **If this parameter occurs more than once, the first occurence will be assigned.**

---

### HTTP Query Examples

- <a href="#index">Index</a>

###### Valid Example: `?o=Equals&p=myproperty&v=myvalue`

will be transformed into: 

```csharp
var filter = new Filter
{
    Operator = FilterOperation.Equals,
    PropertyName = "myproperty",
    Value = "myvalue"
};
```

or to apply multiple filters

###### Valid Example: `?o=Equals&p=myFirstFilterProperty&v=myFirstValue&o=Equals&p=mySecondProperty&v=mySecondValue`

will be transformed into: 

```csharp
var filterOne = new Filter
{
    Operator = FilterOperation.Equals,
    PropertyName = "myFirstFilterProperty",
    Value = "myFirstValue"
};

var filterTwo = new Filter
{
    Operator = FilterOperation.Equals,
    PropertyName = "mySecondProperty",
    Value = "mySecondValue"
};
```

###### Valid Example with ascending sort and pagination: `?o=Equals&p=myproperty&v=myvalue&s=myproperty,asc&offset=0&count=10`

will be transformed into: 

```csharp
var filter = new Filter
{
    Operator = FilterOperation.Equals,
    PropertyName = "myproperty",
    Value = "myvalue"
};

var sort = new SortOption
{
    PropertyName = "myproperty",
    SortingDirection = SortingDirection.Asc
};

var pagination = new PaginationOption
{
    Offset = 0,
    Count = 10
};
```

###### Valid Example with pagination: `?offset=0&count=10`

###### Valid Example with descending sort: `?o=Equals&p=myproperty&v=myvalue&s=myproperty,desc`

###### Valid Descending Sort Example without any filters: `?s=myproperty,desc`

Tip: if you do not provide any sorting direction, DynamicQueryBuilder will sort the data in `ascending` order.

###### Valid Example with ascending sort without stating the direction: `?o=Equals&p=myproperty&v=myvalue&s=myproperty`

## Operation ShortCodes

- <a href="#index">Index</a>

DQB has default operation short codes for shorter HTTP queries which are below;

```csharp
{ "eq", FilterOperation.Equals },
{ "lt", FilterOperation.LessThan },
{ "cts", FilterOperation.Contains },
{ "ne", FilterOperation.NotEqual },
{ "ew", FilterOperation.EndsWith },
{ "sw", FilterOperation.StartsWith },
{ "gt", FilterOperation.GreaterThan },
{ "ltoe", FilterOperation.LessThanOrEqual },
{ "gtoe", FilterOperation.GreaterThanOrEqual }
```

## Web Action Examples

- <a href="#index">Index</a>

#### DynamicQueryAttribute

`DynamicQueryAttribute` is the handler for parsing the querystring into `DynamicQueryOptions` class and has 3 optional parameters.

```csharp
DynamicQueryAttribute(
    // Declares the max page result count for the endpoint.
int maxCountSize = 100,
    // Declares the switch for inclusion of total data set count to `PaginationOptions` class.
bool includeDataSetCountToPagination = true,
    // Declares the behaviour when the requested page size exceeds the assigned maximum count.
PaginationBehaviour exceededPaginationCountBehaviour = PaginationBehaviour.GetMax,
    // Resolves the dynamic query string from the given query parameter value.
string resolveFromParameter = "")
```

### PaginationBehaviour enum

```csharp
    public enum PaginationBehaviour
    {
        // DynamicQueryBuilder will return maxCountSize of results if the `Count` property exceeds `maxCountSize`.
        GetMax,
        // DynamicQueryBuilder will throw MaximumResultSetExceededException if the `Count` property exceeds `maxCountSize`.
        Throw
    }
```

### Example with no pagination specified(default pagination options will be applied).

```csharp
[DynamicQuery]
[HttpGet("getMyDataSet")]
public IActionResult Get(DynamicQueryOptions filter)
{
    IEnumerable<MyObject> myDataSet = _myRepository.GetMyObjectList();
    return Ok(myDataSet.ApplyFilters(filter));
}
```

### Example with default pagination options for the endpoint specified.

```csharp
[HttpGet("getMyDataSet")]
[DynamicQuery(maxCountSize: 101, includeDataSetCountToPagination: true, exceededPaginationCountBehaviour: PaginationBehaviour.GetMax)]
public IActionResult Get(DynamicQueryOptions filter)
{
    IEnumerable<MyObject> myDataSet = _myRepository.GetMyObjectList();
    return Ok(myDataSet.ApplyFilters(filter));
}
```

### Example for OperationShortCodeCustomization
In your startup code, register `CustomOpCodes` type as a Singleton
```csharp
services.AddSingleton(new CustomOpCodes 
{
    { "fizz", FilterOperation.Equals },
    { "buzz", FilterOperation.Contains }
    // Rest of the operations
});
```

#### Warning: You should register every single filter operation even if you don't want to change them.
---

### Work In Progress

- <a href="#index">Index</a>
