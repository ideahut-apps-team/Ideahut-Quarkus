[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# API

* Security untuk mengakses **_CRUD Resource_** dan **_Request Mapping Resource_** berdasarkan ApiRole (User) atau ApiProvider (Service).
* Antar service / provider bisa berkomunikasi menggunakan token / signature.

## ApiHandler

* Menangani data yang tersimpan di database dan [redis](./18-redis.md).

``` java
@Singleton
@DefaultBean
ApiHandler apiHandler(
    AppProperties appProperties,
    EntityTrxManager entityTrxManager,
    BinarySerializer binarySerializer,
    RedisDataSource redisDataSource,
    TaskHandler taskHandler,
    RequestInfoCollector requestInfoCollector
) {
    ApiDefinition.Handler handler = appProperties.api().orElseThrow().handler().orElseThrow();
    
    return new ApiHandlerImpl()
    
    // Serialize & deserialize byte array ke redis
    .setBinarySerializer(binarySerializer)
    
    // Jumlah thread pada saat reload bean (menyiapkan data crud & request mapping), default: 3 thread
    .setConfigureThreads(handler.configureThreads().orElse(null))
    
    // Flag apakah bisa di-consume oleh Api Service / Provider lain
    .setEnableConsumer(handler.enableConsumer().orElse(null))
    
    // Flag apakah Api Crud aktif atau tidak
    .setEnableCrud(handler.enableCrud().orElse(null))
    
    // Flag apakah data Crud & Request Mapping di database dihapus jika tidak tersedia di aplikasi
    .setEnableSync(handler.enableSync().orElse(null))
    
    // Daftar Entity class dan nama trxManager yang terkait dengan ApiHandler
    // default semua class di package net.ideahut.quarkus.api.entity
    .setEntityClass(null)
    
    // EntityTrxManager
    .setEntityTrxManager(entityTrxManager)
    
    // RedisTemplate dan definisi penamaan key-nya
    .setRedisParam(
        new RedisParam(StorageKeyDefinition.convert(handler.redisParam().orElse(null)))
        .setDataSource(redisDataSource)
    )
    
    // Untuk mendapatkan daftar RequestInfo / endpoint yang tersedia
    .setRequestInfoCollector(requestInfoCollector)
    
    // TaskHandler, untuk asinkronus
    .setTaskHandler(taskHandler);
}
```

## ApiTokenService

* Membuat _consumer token_ untuk komunikasi antar service / provider.
* Memvalidasi signature (secret & digest) untuk komunikasi antar service / provider.
* Membuat _access token_ untuk ApiProcessor yang bertipe JWT.
* _SignatureTimeSpan_ adalah batas min / max timestamp terhadap waktu server.

``` java
@Singleton
@DefaultBean
ApiTokenService apiTokenService(
    AppProperties appProperties,
    DataMapper dataMapper,
    SysParamHandler sysParamHandler
) {
    ApiDefinition.Token token = appProperties.api().orElseThrow().token().orElseThrow();
    
    return new ApiTokenServiceImpl()
    
    // Secara default sudah ada, diperlukan untuk mendapatkan ApiToken custom		
    .setApiTokenRetriever(null)
    
    // Default Consumer (komunikasi antar service) secret, digest , & expiry, jika tidak diset di database
    .setConsumerJwtParam(ApiDefinition.JwtParam.convert(token.consumerJwtParam().orElse(null)))
    
    // index redisTemplate untuk meyimpan consumer token jika menggunakan MultipleRedisTemplate, default index 1
    .setConsumerTokenStorageIndex(token.consumerTokenStorageIndex().orElse(null))
    
    // DataMapper
    .setDataMapper(dataMapper)
    
    // Default JwtProcessor secret, digest , & expiry, jika tidak diset di database
    .setProcessorJwtParam(ApiDefinition.JwtParam.convert(token.processorJwtParam().orElse(null)))
    
    // Batas atas & bawah timestamp signature yang dikirim
    // Contoh: jika diset 1 menit, berarti signature yang dikirim valid jika timestamp client dalam range -+ 1 menit
    .setSignatureTimeSpan(TimeValueDefinition.convert(token.signatureTimeSpan().orElse(null)))
    
    // Untuk menyimpan ApiToken di SysParam, sysCode = API_TOKEN, paramCode = [ApiName]
    //.setSysParamHandler(sysParamHandler)
    
    // Pakai ApiToken di dalam ApiProcessor untuk request ke ApiSource lain, 
    // jika false atau ApiToken tidak ada, maka akan digunakan Signature (berdasarkan secret dan digest) 
    .setUseApiTokenInProcessor(token.useApiTokenInProcessor().orElse(null));
}
```

## ApiAccessInternalService

* Untuk mendapatkan ApiAccess jika ApiName token dan service sama (proses dilakukan di server penerbit token).

``` java
@Singleton
@DefaultBean
ApiAccessInternalService apiAccessInternalService(
    AuthService authService
) {
    return authService::getApiAccessForInternal;
}
```

## ApiService

* Menangani semua yang terkait dengan API CRUD dan Request Mapping.
* Untuk mendapatkan ApiAccess custom bisa menggunakan setApiAccessRemoteService.
* Untuk mendapatkan ApiSource custom bisa menggunakan setApiSourceService.
* Untuk definisi request header custom bisa menggunakan setHeader.
* Untuk ApiName custom bisa menggunakan setApiName (default diambil dari 'quarkus.application.name').

```` java
@Singleton
@DefaultBean
ApiService apiService(
    AppProperties appProperties,
    BinarySerializer binarySerializer,
    RedisDataSource redisDataSource,
    ApiHandler apiHandler,
    RestHandler restHandler,
    ApiTokenService apiTokenService,
    ApiAccessInternalService apiAccessInternalService
) {
    ApiDefinition.Service service = appProperties.api().orElseThrow().service().orElseThrow();
    return new ApiServiceImpl()
            
    // Untuk mendapatkan ApiAccess, jika token diterbitkan oleh service ini sendiri
    .setApiAccessInternalService(apiAccessInternalService)
    
    // Untuk mendapatkan ApiAccess ke service yang menerbitkan token
    // Secara default sudah dihandle, fungsi ini diperlukan jika ada custom
    .setApiAccessRemoteService(null)
    
    // ApiHandler
    .setApiHandler(apiHandler)
    
    // ApiName, akan didaftarkan ke database di setiap service yang terhubung
    // Jika tidak diset akan digunakan ID dari application context (property 'quarkus.application.name')
    .setApiName(service.apiName().orElse(null))
    
    // Daftar ApiProcessor yang digunakan
    .setApiProcessors(ModuleApi.getDefaultProcessors())
    
    // Untuk custom RestClient, seperti menambahkan sertifikat, timeout, dll.
    // Secara default sudah ada, hanya diperlukan jika custom
    .setApiRestClientCreator(null)
    
    // Secara default sudah ada, hanya diperlukan untuk mendapatkan ApiSource custom
    // misalnya daftar ApiSource tersimpan di file
    .setApiSourceService(null)
    
    // Service untuk meng-handle token
    .setApiTokenService(apiTokenService)
    
    // Serialize & deserialize byte array ke redis
    .setBinarySerializer(binarySerializer)
    
    // Custom Header name
    .setHeaderName(null)
    
    // Custom redis expiry, untuk data access & consumer baik yang null maupun tidak
    .setRedisExpiry(ApiDefinition.RedisExpiry.convert(service.redisExpiry().orElse(null)))
    
    // RedisTemplate dan definisi penamaan key-nya
    .setRedisParam(
        new RedisParam(StorageKeyDefinition.convert(service.redisParam().orElse(null)))
        .setDataSource(redisDataSource)
    )
    
    // RestHandler
    .setRestHandler(restHandler);
}
````

## Default HTTP Header

* `Api-Token`: token access atau token consumer.
* `Api-Auth`: request authorization.
* `Api-Type`: kode tipe ApiProcessor.
* `Api-Signature`: signature untuk komunikasi antar service, hash-digest(secret + ApiName + timestamp).
* `Api-Timestamp`: timestamp dari service yang me-request.
* `Api-ZoneOffset`: time zone offset (bisa dalam detik ataupun kode), contoh: bisa diisi GMT+7 atau 25200.
* `Api-From`: ApiName dari service yang me-request.
* `Api-Data`: data tambahan (bisa dalam format json / xml).
* `Api-AppId`: ID aplikasi.

## ApiProcessor

* Menangani token access dan token consumer.
* Untuk menambah processor lain, dengan cara meng-_extends_ class 'net.ideahut.quarkus.api.ApiProcessor' dan gunakan ApiType yang belum terpakai.
* Daftar default ApiProcessor bisa dilihat di table di bawah (validasi bisa berdasarkan User-Agent & Remote-Host pada saat token dibuat).

||ApiType|Token Header|Scheme|User Agent|Remote Host|
|---|:---:|:---:|:---:|:---:|:---:|
|`StandardAuthApiProcessor`|A00|Authorization||&#x2612;|&#x2612;|
|`StandardHeaderApiProcessor`|A01|Api-Auth||&#x2612;|&#x2612;|
|`StandardJwtApiProcessor`|A02|Authorization|Bearer|&#x2612;|&#x2612;|
|`AgentAuthApiProcessor`|A03|Authorization||&#9745;|&#x2612;|
|`AgentHeaderApiProcessor`|A04|Api-Auth||&#9745;|&#x2612;|
|`AgentJwtApiProcessor`|A05|Authorization|Bearer|&#9745;|&#x2612;|
|`HostAuthApiProcessor`|A06|Authorization||&#x2612;|&#9745;|
|`HostHeaderApiProcessor`|A07|Api-Auth||&#x2612;|&#9745;|
|`HostJwtApiProcessor`|A08|Authorization|Bearer|&#x2612;|&#9745;|
|`AgentHostAuthApiProcessor`|A09|Authorization||&#9745;|&#9745;|
|`AgentHostHeaderApiProcessor`|A10|Api-Auth||&#9745;|&#9745;|
|`AgentHostJwtApiProcessor`|A11|Authorization|Bearer|&#9745;|&#9745;|

### Contoh Token

```` md
// A00
{
    "time": 328332417,
    "status": 0,
    "data": {
        "token": "OGNjNjc0ZDViZjhlNTU3NTNkNDBhZTc1YmEyYTc3NWNiZTZhYmFjYmI3MjM3N2ZlMDU0M2QwOGU3ODM5Nzk2NTo6QTAwOjptb2JpbGUtYmFja2VuZA==::49d22562-794e-425e-bb20-53f49ae7ea63",
        "scheme": "",
        "headers": {
            "Authorization": "OGNjNjc0ZDViZjhlNTU3NTNkNDBhZTc1YmEyYTc3NWNiZTZhYmFjYmI3MjM3N2ZlMDU0M2QwOGU3ODM5Nzk2NTo6QTAwOjptb2JpbGUtYmFja2VuZA==::49d22562-794e-425e-bb20-53f49ae7ea63"
        }
    }
}

// A01
{
    "time": 196847667,
    "status": 0,
    "data": {
        "token": "NDc2MDM0ZGU5ODU5YjZmNGU1NTkzY2VhYTNmYmI1M2ZhNDk3NmVmNDAxYzAzYjRhNDhmYTRkOWE3ZGEwOGE3OTo6QTAxOjptb2JpbGUtYmFja2VuZA==::70bd30ec-7c83-4467-90a5-c762ca7ed8bc",
        "headers": {
            "Access-Token": "NDc2MDM0ZGU5ODU5YjZmNGU1NTkzY2VhYTNmYmI1M2ZhNDk3NmVmNDAxYzAzYjRhNDhmYTRkOWE3ZGEwOGE3OTo6QTAxOjptb2JpbGUtYmFja2VuZA==::70bd30ec-7c83-4467-90a5-c762ca7ed8bc"
        }
    }
}

// A02
{
    "time": 253228583,
    "status": 0,
    "data": {
        "token": "MTBmZjMyZjk3YzhjMTNiMjBmZTcyN2NiNmFlZjQ3YjJkYjhjMTQ5Mjo6QTAyOjptb2JpbGUtYmFja2VuZA==::eyJhbGciOiJIUzI1NiJ9.eyJhdHRyaWJ1dGVzIjp7ImRldmljZUlkIjoiNmY5Yzg4NDJhNDk5Nzk5NmVlMzZiZjIyMTcwNWQ0OTRkN2JmMjQ1YzcyZjhiNzNkYTUxNTY2NGY3ZDYxYjY2YyJ9LCJhcGlLZXkiOiIxMGZmMzJmOTdjOGMxM2IyMGZlNzI3Y2I2YWVmNDdiMmRiOGMxNDkyIiwiYXBpVXNlciI6eyJhdHRyaWJ1dGVzIjp7ImZ1bGxuYW1lIjoiQWRtaW4ifSwiaWQiOiJVU1IwMDAwLTAwMDAwLTAwMDAtMDAwMDAtMC0wMDAwIiwidXNlcm5hbWUiOiJhZG1pbiJ9LCJ2YWxpZFVudGlsIjoxNzI0NDgyNDk0Nzk1LCJzZXJ2aWNlUm9sZSI6eyJpZGVhaHV0LXNwcmluZ2Jvb3QtdGVtcGxhdGUiOiJURVNUIiwibW9iaWxlLWJhY2tlbmQiOiJBRE1JTiJ9fQ.wGaGv8ZN7KTdOOoW6tcR8oFFUsCfrXHlru8QiSS0mt0",
        "scheme": "Bearer",
        "headers": {
            "Authorization": "Bearer MTBmZjMyZjk3YzhjMTNiMjBmZTcyN2NiNmFlZjQ3YjJkYjhjMTQ5Mjo6QTAyOjptb2JpbGUtYmFja2VuZA==::eyJhbGciOiJIUzI1NiJ9.eyJhdHRyaWJ1dGVzIjp7ImRldmljZUlkIjoiNmY5Yzg4NDJhNDk5Nzk5NmVlMzZiZjIyMTcwNWQ0OTRkN2JmMjQ1YzcyZjhiNzNkYTUxNTY2NGY3ZDYxYjY2YyJ9LCJhcGlLZXkiOiIxMGZmMzJmOTdjOGMxM2IyMGZlNzI3Y2I2YWVmNDdiMmRiOGMxNDkyIiwiYXBpVXNlciI6eyJhdHRyaWJ1dGVzIjp7ImZ1bGxuYW1lIjoiQWRtaW4ifSwiaWQiOiJVU1IwMDAwLTAwMDAwLTAwMDAtMDAwMDAtMC0wMDAwIiwidXNlcm5hbWUiOiJhZG1pbiJ9LCJ2YWxpZFVudGlsIjoxNzI0NDgyNDk0Nzk1LCJzZXJ2aWNlUm9sZSI6eyJpZGVhaHV0LXNwcmluZ2Jvb3QtdGVtcGxhdGUiOiJURVNUIiwibW9iaWxlLWJhY2tlbmQiOiJBRE1JTiJ9fQ.wGaGv8ZN7KTdOOoW6tcR8oFFUsCfrXHlru8QiSS0mt0"
        }
    }
}
````

## Screenshot

<div>
   <img src="./assets/api-01.png" alt="" width="800" />
</div>
<br/>
<div>
   <img src="./assets/api-02.png" alt="" width="800" />
</div>
<br/>
<div>
   <img src="./assets/api-03.png" alt="" width="800" />
</div>
<br/>
<div>
   <img src="./assets/api-04.png" alt="" width="800" />
</div>
<br/>
<div>
   <img src="./assets/api-05.png" alt="" width="800" />
</div>
<br/>
<div>
   <img src="./assets/api-06.png" alt="" width="800" />
</div>
<br/>
<div>
   <img src="./assets/api-07.png" alt="" width="800" />
</div>

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
