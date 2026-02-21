# 🚀 Production-Level Redis Implementation with .NET 8 Web API

------------------------------------------------------------------------

# 📌 What is Redis?

Redis (Remote Dictionary Server) is an in-memory data store used as:

-   Distributed Cache
-   Database
-   Message Broker (Pub/Sub)
-   Session Store
-   Rate Limiter
-   Distributed Lock Provider

Because Redis stores data in memory, it provides extremely low-latency
responses compared to traditional databases.

------------------------------------------------------------------------

# ❓ Why Use Redis in Production?

In high-traffic APIs:

-   Database becomes a bottleneck
-   Repeated reads increase latency
-   Scaling DB vertically is expensive

Redis solves this by:

-   Reducing database load
-   Improving response time
-   Supporting distributed systems
-   Enabling horizontal scalability

------------------------------------------------------------------------

# ✅ Benefits

  -----------------------------------------------------------------------
  Benefit                        Explanation
  ------------------------------ ----------------------------------------
  ⚡ High Performance            In-memory storage makes it extremely
                                 fast

  📉 Reduced DB Load             Minimizes frequent database calls

  🔄 Distributed Cache           Works across multiple API instances

  🧠 Supports Multiple Data      Strings, Lists, Sets, Hashes, Sorted
  Types                          Sets

  ⏱ Expiration Support           Auto-expire cached data

  📦 Scalable                    Can be scaled horizontally
  -----------------------------------------------------------------------

# 🏗 Recommended Architecture (Cache-Aside Pattern)

Client → API → Redis → SQL Database

Flow:

1.  API checks Redis
2.  If cache hit → return data
3.  If cache miss → fetch from DB
4.  Store in Redis with expiry
5.  Return response

This is called **Cache-Aside Pattern** (recommended for microservices).

------------------------------------------------------------------------

# 🛠 Step-by-Step Implementation (.NET 8)

------------------------------------------------------------------------

## 1️⃣ Install Required Packages

``` bash
dotnet add package StackExchange.Redis
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

------------------------------------------------------------------------

## 2️⃣ appsettings.json

``` json
{
  "Redis": {
    "ConnectionString": "localhost:6379",
    "InstanceName": "MyApp:"
  }
}
```

------------------------------------------------------------------------

## 3️⃣ Register Redis (Program.cs)

``` csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration["Redis:ConnectionString"];
    options.InstanceName = builder.Configuration["Redis:InstanceName"];
});
```

------------------------------------------------------------------------

## 4️⃣ Create Generic Cache Service (Production Ready)

``` csharp
using System.Text.Json;
using Microsoft.Extensions.Caching.Distributed;

public interface ICacheService
{
    Task<T?> GetAsync<T>(string key);
    Task SetAsync<T>(string key, T value, TimeSpan expiry);
    Task RemoveAsync(string key);
}

public class CacheService : ICacheService
{
    private readonly IDistributedCache _cache;

    public CacheService(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task<T?> GetAsync<T>(string key)
    {
        var cachedData = await _cache.GetStringAsync(key);

        if (cachedData == null)
            return default;

        return JsonSerializer.Deserialize<T>(cachedData);
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan expiry)
    {
        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = expiry
        };

        var serializedData = JsonSerializer.Serialize(value);

        await _cache.SetStringAsync(key, serializedData, options);
    }

    public async Task RemoveAsync(string key)
    {
        await _cache.RemoveAsync(key);
    }
}
```

Register:

``` csharp
builder.Services.AddScoped<ICacheService, CacheService>();
```

------------------------------------------------------------------------

## 5️⃣ Use in Controller

``` csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ICacheService _cacheService;

    public ProductsController(ICacheService cacheService)
    {
        _cacheService = cacheService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> Get(int id)
    {
        string cacheKey = $"product:{id}";

        var cachedProduct = await _cacheService.GetAsync<string>(cacheKey);

        if (cachedProduct != null)
            return Ok(cachedProduct);

        // Simulated DB call
        var product = $"Product from DB with Id: {id}";

        await _cacheService.SetAsync(cacheKey, product, TimeSpan.FromMinutes(10));

        return Ok(product);
    }
}
```

------------------------------------------------------------------------

# 🐳 Running Redis with Docker (Recommended)

``` bash
docker run -d -p 6379:6379 --name redis redis
```

------------------------------------------------------------------------

# 🔥 Production Best Practices

✅ Use Redis in distributed cache mode\
✅ Always set expiration\
✅ Use JSON serialization for complex objects\
✅ Implement cache invalidation on update/delete\
✅ Monitor memory usage\
✅ Use Redis Cluster in high-scale systems\
✅ Secure with password & firewall rules

------------------------------------------------------------------------

## ⏳ Cache Expiration Strategies

-   Absolute Expiration (e.g., 5 minutes)
-   Sliding Expiration
-   Manual Invalidation
-   Cache Aside Pattern (Recommended)

------------------------------------------------------------------------

## 🎯 When Should You Use Redis?

Use Redis when:

-   You have high read-heavy APIs
-   Data does not change frequently
-   You want to improve response time
-   You need distributed caching
-   You want session storage in microservices

Avoid using Redis when:

-   Data changes on every request
-   Strong transactional consistency is required

------------------------------------------------------------------------

# 🚀 Advanced Use Cases

-   Rate Limiting Middleware
-   Distributed Locking
-   Background Jobs
-   SignalR Backplane
-   Session Storage
-   Pub/Sub Eventing

------------------------------------------------------------------------

# 📊 Performance Impact

Without Redis: - Every request hits DB - Higher latency - Higher
infrastructure cost

With Redis: - First request hits DB - Next requests served from memory -
5x--20x performance improvement (based on workload)

------------------------------------------------------------------------

# 🧠 When NOT to Use Redis

❌ Highly volatile data\
❌ Strong transactional consistency required\
❌ Write-heavy systems without read optimization need

------------------------------------------------------------------------

# 🏁 Conclusion

Redis is a powerful in-memory caching solution that significantly
improves the performance and scalability of .NET APIs.

If you're building high-performance, scalable microservices or
enterprise APIs, Redis is highly recommended.

------------------------------------------------------------------------

👨‍💻 Author\
Chandrashekhar Singh\
Senior .NET + Angular Developer
