---
title: ASP.NET Core 中的请求和响应操作
author: jkotalik
description: 了解如何在 ASP.NET Core 中读取请求正文和写入响应正文。
monikerRange: '>= aspnetcore-3.0'
ms.author: jukotali
ms.custom: mvc
ms.date: 08/29/2019
uid: fundamentals/middleware/request-response
ms.openlocfilehash: 5e531c0ce0ed48097054fd81ddc3655a66cc7c5f
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2019
ms.locfileid: "71081671"
---
# <a name="request-and-response-operations-in-aspnet-core"></a>ASP.NET Core 中的请求和响应操作

作者：[Justin Kotalik](https://github.com/jkotalik)

本文介绍如何读取请求正文和写入响应正文。 写入中间件时，可能需要这些操作的代码。 除写入中间件外，通常不需要自定义代码，因为操作由 MVC 和 Razor Pages 处理。

请求正文和响应正文有两个抽象元素：<xref:System.IO.Stream> 和 <xref:System.IO.Pipelines.Pipe>。 对于请求读取，[HttpRequest.Body](xref:Microsoft.AspNetCore.Http.HttpRequest.Body) 是 <xref:System.IO.Stream>，`HttpRequest.BodyReader` 是 <xref:System.IO.Pipelines.PipeReader>。 对于响应写入，[HttpResponse.Body](xref:Microsoft.AspNetCore.Http.HttpResponse.Body) 是 <xref:System.IO.Stream>，`HttpResponse.BodyWriter` 是 <xref:System.IO.Pipelines.PipeWriter>。

建议使用管道替代流。 在一些简单操作中，使用流会比较简单，但管道具有性能优势，并且在大多数场景中更易于使用。 ASP.NET Core 开始在内部使用管道替代流。 示例包括：

* `FormReader`
* `TextReader`
* `TextWriter`
* `HttpResponse.WriteAsync`

流不会从框架中删除。 流将继续在 .NET 中使用，并且许多流类型不具有等效的管道，如 `FileStreams` 和 `ResponseCompression`。

## <a name="stream-examples"></a>流示例

假设目标是要创建一个中间件，它将整个请求正文作为一个字符串列表读取，并在新行上进行拆分。 一个简单的流实现可能如下例所示：

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringsFromStream)]

此代码有效，但存在一些问题：

* 在追加到 `StringBuilder` 之前，示例创建另一个字符串 (`encodedString`)，该字符串会立即被丢弃。 此过程发生在流中的所有字节上，因此结果是为整个请求正文大小分配额外的内存。
* 该示例在新行上进行拆分之前读取整个字符串。 检查字节数组中的新行会更有效。

下面是修复上面其中一些问题的示例：

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringsFromStreamMoreEfficient)]

上面的此示例：

* 不在 `StringBuilder` 中缓冲整个请求正文（除非没有任何换行符）。
* 不在字符串上调用 `Split`。

然而，仍然存在一些问题：

* 如果换行符稀疏，则大部分请求正文都在字符串中进行缓冲。
* 该代码会继续创建字符串 (`remainingString`) 并将它们添加到字符串缓冲区，这将导致额外的分配。

这些问题是可修复的，但代码逐渐变得复杂，几乎没有什么改进。 管道提供了一种以最低的代码复杂性解决这些问题的方法。

## <a name="pipelines"></a>管道

下面的示例显示了如何使用 `PipeReader` 处理相同的场景：

[!code-csharp[](request-response/samples/3.x/RequestResponseSample/Startup.cs?name=GetListOfStringFromPipe)]

此示例解决了流实现中的许多问题：

* 不需要字符串缓冲区，因为 `PipeReader` 会处理未使用的字节。
* 编码后的字符串将直接添加到返回的字符串列表。
* 除了字符串使用的内存之外，无需再为创建的字符串分配内存（`ToArray()` 调用除外）。

## <a name="adapters"></a>适配器

`Body` 和 `BodyReader/BodyWriter` 属性均可用于 `HttpRequest` 和 `HttpResponse`。 将 `Body` 设置为另一个流时，一组新的适配器会自动使每种类型彼此适应。 如果将 `HttpRequest.Body` 设置为新流，则 `HttpRequest.BodyReader` 将自动设置为包装 `HttpRequest.Body` 的新 `PipeReader`。

## <a name="startasync"></a>StartAsync

`HttpResponse.StartAsync` 用于指示标头不可修改并运行 `OnStarting` 回调。 使用 Kestrel 作为服务器时，在使用 `PipeReader` 之前调用 `StartAsync` 可确保 `GetMemory` 返回的内存属于 Kestrel 的内部 <xref:System.IO.Pipelines.Pipe> 而不是外部缓冲区。

## <a name="additional-resources"></a>其他资源

* [System.IO.Pipelines 简介](https://devblogs.microsoft.com/dotnet/system-io-pipelines-high-performance-io-in-net/)
* <xref:fundamentals/middleware/write>
