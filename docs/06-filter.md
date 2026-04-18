[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Filter

Http request dan response filter.

## Request Filter
- ResourceInfo wajib ada karena akan digunakan untuk mengecek request dan dikaitkan dengan [RequestInfo Collector](./02-advice.md#requestinfo-collector)
- RoutingContext wajib ada karena akan digunakan untuk response.

``` java
@Provider
@ApplicationScoped
class RequestFilter extends net.ideahut.quarkus.web.WebRequestFilter implements ContainerRequestFilter {
    RequestFilter(
        WebInterceptor webInterceptor
    ) {
        setAutoDetect(true) // auto detect interceptor dari konteks aplikasi
        .setInterceptors(Arrays.asList(webInterceptor)); // manual menambahkan interceptor
    }

    // ResourceInfo wajib ada
    @Context
    private ResourceInfo resourceInfo;
    
    // ResourceInfo wajib ada
    @Context
    private RoutingContext routingContext;

    @Override
    protected ResourceInfo resourceInfo() {
        return resourceInfo;
    }

    @Override
    protected RoutingContext routingContext() {
        return routingContext;
    }
    
}
```

## Response Filter

```java
@Provider
@ApplicationScoped
class ResponseFilter extends net.ideahut.quarkus.web.WebResponseFilter implements ContainerResponseFilter {
    ResponseFilter() {
        .setDataMapper(null)
        .setPathMatcher(null)
        .setSkipClasses(null)
        .setSkipPaths(null);
    }
}
```

- `setDataMapper`: [DataMapper](./04-mapper.md) bean.
- `setPathMatcher`: Custom PathMatcher.
- `setSkipClasses`: Class response entity yang akan langsung dikirim ke klien, tidak disusun lagi mengikuti standar framework.
- `setSkipPaths`: Request Path yang responsnya akan langsung dikirim ke klien, tidak disusun lagi mengikuti standar framework.

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
