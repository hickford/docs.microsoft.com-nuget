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

### Sample output

```
No cursor found. Defaulting to 11/2/2017 9:41:28 PM.
Fetched catalog index https://api.nuget.org/v3/catalog0/index.json.
Fetched catalog page https://api.nuget.org/v3/catalog0/page2935.json.
Processing 69 catalog leaves.
11/2/2017 9:32:35 PM: DotVVM.Compiler.Light 1.1.7 (type is nuget:PackageDetails)
11/2/2017 9:32:35 PM: Momentum.Pm.Api 5.12.181-beta (type is nuget:PackageDetails)
11/2/2017 9:32:44 PM: Momentum.Pm.PortalApi 5.12.181-beta (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Genesys.Extensions.Standard 3.17.11.40 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Genesys.Extensions.Core 3.17.11.40 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.FileStores.Serialization.Bond 1.0.4 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.FileStores.AmazonS3 1.0.4 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.DocumentStores.DocumentDb 1.0.6 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.FileStores.BlobStorage 1.0.5 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.FileStores.Serialization.Protobuf 1.0.4 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.FileStores.GoogleDrive 1.0.4 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.FileStores.Serialization.Yaml 1.0.4 (type is nuget:PackageDetails)
11/2/2017 9:35:14 PM: Halforbit.DataStores.DocumentStores.PostgresMarten 1.0.4 (type is nuget:PackageDetails)
11/2/2017 9:35:24 PM: DotVVM.Templates 1.1.7 (type is nuget:PackageDetails)
11/2/2017 9:35:35 PM: Genesys.Extensions.Standard 3.17.11.40 (type is nuget:PackageDetails)
11/2/2017 9:35:35 PM: Genesys.Extensions.Core 3.17.11.40 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer2StableMetadataUnlisted.171102.213624.9073204 1.0.0+metadata (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer2DueToSemVer2Dep.171102.213624.9073204 1.0.0-beta (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: Halforbit.DataStores.FileStores.BlobStorage 1.0.6 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer1StableUnlisted.171102.213624.6416541 1.0.0 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer2PrerelUnlisted.171102.213624.9073204 1.0.0-alpha.1 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer2Prerel.171102.213624.9073204 1.0.0-alpha.1 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: DotVVM.Tracing.ApplicationInsights 1.1.7 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: Maestro 2.0.2 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: Lykke.MatchingEngineConnector 1.0.20-beta (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer1Stable.171102.213624.6416541 1.0.0 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer2PrerelRelisted.171102.213624.9073204 1.0.0-alpha.1 (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: E2E.SemVer2StableMetadata.171102.213624.9073204 1.0.0+metadata (type is nuget:PackageDetails)
11/2/2017 9:38:01 PM: Lykke.MatchingEngineConnector.Abstractions 1.0.20-beta (type is nuget:PackageDetails)
11/2/2017 9:38:12 PM: Maestro 2.0.1 (type is nuget:PackageDetails)
11/2/2017 9:38:12 PM: E2E.SemVer1StableUnlisted.171102.213624.6416541 1.0.0 (type is nuget:PackageDetails)
11/2/2017 9:38:12 PM: E2E.SemVer2PrerelUnlisted.171102.213624.9073204 1.0.0-alpha.1 (type is nuget:PackageDetails)
11/2/2017 9:38:12 PM: E2E.SemVer2StableMetadataUnlisted.171102.213624.9073204 1.0.0+metadata (type is nuget:PackageDetails)
11/2/2017 9:38:12 PM: E2E.SemVer2PrerelRelisted.171102.213624.9073204 1.0.0-alpha.1 (type is nuget:PackageDetails)
11/2/2017 9:40:34 PM: Lykke.MatchingEngineConnector.Abstractions 1.0.17 (type is nuget:PackageDetails)
11/2/2017 9:40:34 PM: Lykke.MatchingEngineConnector 1.0.17 (type is nuget:PackageDetails)
11/2/2017 9:42:59 PM: Halforbit.DataStores 1.0.6 (type is nuget:PackageDetails)
11/2/2017 9:42:59 PM: DotVVM.Tracing.ApplicationInsights.Owin 1.1.7 (type is nuget:PackageDetails)
11/2/2017 9:43:09 PM: Clancey.SimpleAuth 1.0.44 (type is nuget:PackageDetails)
11/2/2017 9:45:35 PM: DotVVM.Tracing.ApplicationInsights.AspNetCore 1.1.7 (type is nuget:PackageDetails)
11/2/2017 9:45:45 PM: DotVVM.Tracing.MiniProfiler.AspNetCore 1.1.7 (type is nuget:PackageDetails)
11/2/2017 9:48:05 PM: DotVVM.Tracing.MiniProfiler.Owin 1.1.7 (type is nuget:PackageDetails)
11/2/2017 9:50:30 PM: salt 1.0.1 (type is nuget:PackageDetails)
11/2/2017 9:50:30 PM: Lykke.Service.Kyc.Abstractions 1.0.30 (type is nuget:PackageDetails)
11/2/2017 9:50:30 PM: Lykke.Service.Kyc.Client 1.0.30 (type is nuget:PackageDetails)
11/2/2017 9:59:44 PM: KRPC.Client 0.4.1 (type is nuget:PackageDetails)
11/2/2017 9:59:53 PM: E2E.SemVer2PrerelRelisted.171102.213624.9073204 1.0.0-alpha.1 (type is nuget:PackageDetails)
11/2/2017 10:02:18 PM: ICCorp.Infraestructura.Repositorio.RabbitMQ 1.0.0.6 (type is nuget:PackageDetails)
11/2/2017 10:04:38 PM: Halforbit.DataStores.FileStores.Serialization.Bond 1.0.5 (type is nuget:PackageDetails)
11/2/2017 10:04:38 PM: Halforbit.DataStores.FileStores.GoogleDrive 1.0.5 (type is nuget:PackageDetails)
11/2/2017 10:04:38 PM: Halforbit.DataStores.FileStores.AmazonS3 1.0.5 (type is nuget:PackageDetails)
11/2/2017 10:04:38 PM: Halforbit.DataStores.FileStores.Serialization.Protobuf 1.0.5 (type is nuget:PackageDetails)
11/2/2017 10:04:38 PM: Halforbit.DataStores.FileStores.BlobStorage 1.0.7 (type is nuget:PackageDetails)
11/2/2017 10:04:47 PM: Halforbit.DataStores.FileStores.Serialization.Yaml 1.0.5 (type is nuget:PackageDetails)
11/2/2017 10:04:57 PM: Halforbit.DataStores.DocumentStores.DocumentDb 1.0.7 (type is nuget:PackageDetails)
11/2/2017 10:05:07 PM: Halforbit.DataStores.DocumentStores.PostgresMarten 1.0.5 (type is nuget:PackageDetails)
11/2/2017 10:09:49 PM: WebEssentials.AspNetCore.ServiceWorker 1.0.7 (type is nuget:PackageDetails)
11/2/2017 10:09:49 PM: Griddly 2.0.0-alpha2 (type is nuget:PackageDetails)
11/2/2017 10:09:49 PM: Griddly.Core 2.0.0-alpha2 (type is nuget:PackageDetails)
11/2/2017 10:16:31 PM: TIKSN-Framework 3.0.0-alpha.12 (type is nuget:PackageDetails)
11/2/2017 10:18:55 PM: Algorithmia.Client 0.0.3 (type is nuget:PackageDetails)
11/2/2017 10:21:26 PM: SecureStrConvertor.VARUN_RUSIYA 1.0.0.5 (type is nuget:PackageDetails)
11/2/2017 10:23:54 PM: Cake.GitPackager 0.1.2 (type is nuget:PackageDetails)
11/2/2017 10:23:54 PM: UtilPack.NuGet 2.0.0 (type is nuget:PackageDetails)
11/2/2017 10:23:54 PM: UtilPack.NuGet.AssemblyLoading 2.0.0 (type is nuget:PackageDetails)
11/2/2017 10:26:26 PM: UtilPack.NuGet.Deployment 2.0.0 (type is nuget:PackageDetails)
11/2/2017 10:26:26 PM: UtilPack.NuGet.Common.MSBuild 2.0.0 (type is nuget:PackageDetails)
11/2/2017 10:26:36 PM: InstaClient 1.0.2 (type is nuget:PackageDetails)
11/2/2017 10:26:36 PM: SecureStrConvertor.VARUN_RUSIYA 1.0.0.5 (type is nuget:PackageDetails)
Writing cursor value: 11/2/2017 10:26:36 PM.
```