---
# required metadata 

title: Query for all packages published to nuget.org | Microsoft Docs
author:
- joelverhagen
- kraigb
ms.author:
- joelverhagen
- kraigb
manager: skofman
ms.date: 11/2/2017
ms.topic: get-started-article
ms.prod: nuget
ms.technology: null
ms.assetid: 5d017cd4-3d75-4341-ba90-3c57be093b7d

# optional metadata

description: Using the NuGet API, you can query for all packages published to nuget.org and stay up-to-date over time.
keywords: NuGet API enumerate all packages, NuGet API replicate packages, latest packages published to nuget.org
ms.reviewer:
- karann
- unniravindranathan

---

# Query for all packages published to nuget.org

One common query pattern on the legacy OData V2 API was enumerating all packages published to nuget.org, ordered by when
the package was published. Scenarios requiring this kind of query against nuget.org vary widely. Some common scenarios
include:

- Replicating nuget.org entirely
- Detecting when packages have new versions released
- Tracking how broadly a new feature is adopted on nuget.org

The legacy way of doing this typically depended on sorting the OData package entity by a timestamp and paging across
the massive result set using `skip` and `top` (page size) parameters. Unfortunately, this approach has some drawbacks
such as:

- Possibility of missing packages, since the queries are being made on data that is actively being changed
- Performance issues, since the queries are not optimized since they don't fit into a mainline client scenario
- Use of deprecated and undocumented API, meaning the support of such queries in the future is not guaranteed

For this reason, the following guide can be followed to address the aforementioned scenarios in a more reliable and
future-proof way.

## Overview

At the center of this guide is resource in the [NuGet API](../../api/overview.md) called the **catalog**. The catalog
is an append-only API that allows the caller to see a full history of packages added to nuget.org

This guide is intended to be a high-level walkthrough but if you are interested in the fine-grain details of the
catalog, see its [API reference document](../../api/catalog-resource.md).

The following steps can be implemented in a programming language of your choice. If you want a full running sample,
take a look at the [.NET sample](#net-sample-code) mentioned below.

## .NET sample code

Since the catalog is simply a set of JSON documents, it can be interacted with using any programming language that has
an HTTP client and JSON deserializer.

For improved understanding and convenient, we have made a small sample available written in C# to demonstrate how to
use read from the catalog. The solution file requires Visual Studio 2017. The project depends on the
[NuGet.Protocol 4.4.0](https://www.nuget.org/packages/NuGet.Protocol/4.4.0) (for resolving the service index) and
[Newtonsoft.Json 9.0.1](https://www.nuget.org/packages/Newtonsoft.Json/9.0.1) (for JSON deserialization).

Simply clone the [NuGet/Samples](https://github.com/NuGet/Samples) repository on GitHub, restore, build, and run the
project file under the `CatalogReaderExample` directory. 

```
git clone https://github.com/NuGet/Samples.git
```

The main logic of the code is visible in the
[`Program.cs`](https://github.com/NuGet/Samples/blob/master/CatalogReaderExample/CatalogReaderExample/Program.cs)
file.
