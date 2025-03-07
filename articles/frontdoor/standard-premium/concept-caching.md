---
title: 'Azure Front Door: Caching'
description: This article helps you understand the behavior of Azure Front Door Standard/Premium with routing rules that have enabled caching.
services: front-door
author: duongau
manager: KumudD
ms.service: frontdoor
ms.topic: conceptual
ms.date: 02/18/2021
ms.author: duau
---

# Caching with Azure Front Door Standard/Premium (Preview)

> [!Note]
> This documentation is for Azure Front Door Standard/Premium (Preview). Looking for information on Azure Front Door? View [here](../front-door-overview.md).

In this article, you'll learn how Front Door Standard/Premium (Preview) Routes and Rule set behaves when you have caching enabled. Azure Front Door is a modern Content Delivery Network (CDN) with dynamic site acceleration and load balancing.

> [!IMPORTANT]
> Azure Front Door Standard/Premium (Preview) is currently in public preview.
> This preview version is provided without a service level agreement, and it's not recommended for production workloads. Certain features might not be supported or might have constrained capabilities.
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Request methods

Only the GET request method can generate cached content in Azure Front Door. All other request methods are always proxied through the network.

## Delivery of large files

Front Door Standard/Premium (Preview) delivers large files without a cap on file size. Front Door uses a technique called object chunking. When a large file is requested, Front Door retrieves smaller pieces of the file from the origin. After receiving a full or byte-range file request, the Front Door environment requests the file from the originS in chunks of 8 MB.

After the chunk arrives at the Front Door environment, it's cached and immediately served to the user. Front Door then pre-fetches the next chunk in parallel. This pre-fetch ensures that the content stays one chunk ahead of the user, which reduces latency. This process continues until the entire file gets downloaded (if requested) or the client closes the connection.

For more information on the byte-range request, read [RFC 7233](https://www.rfc-editor.org/info/rfc7233).
Front Door caches any chunks as they're received so the entire file doesn't need to be cached on the Front Door cache. Ensuing requests for the file or byte ranges are served from the cache. If the chunks aren't all cached, pre-fetching is used to request chunks from the backend. This optimization relies on the origin's ability to support byte-range requests. If the origin doesn't support byte-range requests, this optimization isn't effective.

## File compression

Refer to [improve performance by compressing files](how-to-compression.md) in Azure Front Door.

## Query string behavior

With Front Door, you can control how files are cached for a web request that contains a query string. In a web request with a query string, the query string is that portion of the request that occurs after a question mark (?). A query string can contain one or more key-value pairs, in which the field name and its value are separated by an equals sign (=). Each key-value pair is separated by an ampersand (&). For example, `http://www.contoso.com/content.mov?field1=value1&field2=value2`. If there's more than one key-value pair in a query string of a request then their order doesn't matter.

* **Ignore query strings**: In this mode, Front Door passes the query strings from the requestor to the origin on the first request and caches the asset. All ensuing requests for the asset that are served from the Front Door environment ignore the query strings until the cached asset expires.

* **Cache every unique URL**: In this mode, each request with a unique URL, including the query string, is treated as a unique asset with its own cache. For example, the response from the origin for a request for `www.example.ashx?q=test1` is cached at the Front Door environment and returned for ensuing caches with the same query string. A request for `www.example.ashx?q=test2` is cached as a separate asset with its own time-to-live setting.
* You can also use Rule Set to specify **cache key query string** behavior, to include, or exclude specified parameters when cache key gets generated. For example, the default cache key is: /foo/image/asset.html, and the sample request is `https://contoso.com//foo/image/asset.html?language=EN&userid=100&sessionid=200`. There's a rule set rule to exclude query string 'userid'. Then the query string cache-key would be `/foo/image/asset.html?language=EN&sessionid=200`.

## Cache purge

Refer to cache purge.

## Cache expiration
The following order of headers is used to determine how long an item will be stored in our cache:</br>
1. Cache-Control: s-maxage=\<seconds>
2. Cache-Control: max-age=\<seconds>
3. Expires: \<http-date>

Cache-Control response headers that indicate that the response won't be cached such as Cache-Control: private, Cache-Control: no-cache, and Cache-Control: no-store are honored.  If no Cache-Control is present, the default behavior is that Front Door will cache the resource for X amount of time. Where X gets randomly picked between 1 to 3 days.

> [!NOTE]
> Cache expiration can't be greater than **366 days**.
> 

## Request headers

The following request headers won't be forwarded to an origin when using caching.
* Content-Length
* Transfer-Encoding

## Cache behavior and duration

Cache behavior and duration can be configured in both the Front Door designer routing rule and in Rules Engine. Rules Engine caching configuration will always override the Front Door designer routing rule configuration.

* When *caching* is **disabled**, Front Door doesn’t cache the response contents, irrespective of origin response directives.

* When *caching* is **enabled**, the cache behavior is different for different values of *Use cache default duration*.
    * When *Use cache default duration* is set to **Yes**, Front Door will always honor origin response header directive. If the origin directive is missing, Front Door will cache contents anywhere from 1 to 3 days.
    * When *Use cache default duration* is set to **No**, Front Door will always override with the *cache duration* (required fields), meaning that it will cache the contents for the cache duration ignoring the values from origin response directives. 

> [!NOTE]
> * The *cache duration* set in the Front Door designer routing rule is the **minimum cache duration**. This override won't work if the cache control header from the origin has a greater TTL than the override value.
> * Azure Front Door makes no guarantees about the amount of time that the content is stored in the cache. Cached content may be removed from the edge cache before the content expiration if the content is not frequently used. Front Door might be able to serve data from the cache even if the cached data has expired. This behavior can help your site to remain partially available when your origins are offline.
>

## Next steps

* Learn more about [Rule Set Match Conditions](concept-rule-set-match-conditions.md)
* Learn more about [Rule Set Actions](../front-door-rules-engine-actions.md)
