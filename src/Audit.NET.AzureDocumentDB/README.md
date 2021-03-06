# Audit.NET.AzureDocumentDB
**Azure Document DB provider for [Audit.NET library](https://github.com/thepirat000/Audit.NET)**. (An extensible framework to audit executing operations in .NET)

Store the audit events in an Azure Document DB collection, in JSON format.

## Install

**NuGet Package** 
To install the package run the following command on the Package Manager Console:

```
PM> Install-Package Audit.NET.AzureDocumentDB
```

Or for the [Strong-Named](https://www.nuget.org/packages/Audit.NET.AzureDocumentDB.StrongName/) version:
```
PM> Install-Package Audit.NET.AzureDocumentDB.StrongName
```

[![NuGet Status](https://img.shields.io/nuget/v/Audit.NET.AzureDocumentDB.svg?style=flat)](https://www.nuget.org/packages/Audit.NET.AzureDocumentDB/)

## Usage
Please see the [Audit.NET Readme](https://github.com/thepirat000/Audit.NET#usage)

## Configuration
Set the static `Audit.Core.Configuration.DataProvider` property to set the Document DB data provider, or use the `UseAzureDocumentDB` method on the fluent configuration. This should be done before any `AuditScope` creation, i.e. during application startup.

For example:
```c#
Audit.Core.Configuration.DataProvider = new Audit.AzureDocumentDB.Providers.AzureDbDataProvider()
{
    ConnectionString = "https://mycompany.documents.azure.com:443/",
    AuthKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==",
    Database = "Audit",
    Collection = "Event"
};
```

Or by using the [fluent configuration API](https://github.com/thepirat000/Audit.NET#configuration-fluent-api):
```c#
Audit.Core.Configuration.Setup()
    .UseAzureDocumentDB(config => config
        .ConnectionString("https://mycompany.documents.azure.com:443/")
        .AuthKey("xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==")
        .Database("Audit")
        .Collection("Event"));
```

### Provider options

Mandatory:
- **ConnectionString**: The Azure Document DB Connection String.
- **AuthKey**: The Auth Key to use.
- **Database**: The audit database name.
- **Collection**: The events collection name.

## Query events

The Azure Document DB data provider includes support for querying the events collection.

Use the `QueryEvents()` method on `AzureDbDataProvider` class to run LINQ queries against the audit events.


For example, to get the top 10 most time-consuming events for a specific machine:
```c#
IQueryable<AuditEvent> query = azureDbDataProvider.QueryEvents()
	.Where(ev => ev.Environment.MachineName == "HP")
	.OrderByDescending(ev => ev.Duration)
	.Take(10);
```

Also you can use the `EnumerateEvents()` method to run SQL-like queries. For example the previous query can be written as:

```c#
IEnumerable<AuditEvent> events = azureDbDataProvider.EnumerateEvents(
       @"SELECT TOP 10 * 
         FROM c 
         WHERE c.Environment.MachineName = 'HP' 
         ORDER BY c.Duration DESC");
```

This [post](https://docs.microsoft.com/en-us/azure/documentdb/documentdb-sql-query) contains information about the SQL query syntax supported by Azure Document DB.
