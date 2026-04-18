[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Message

- Menangani pesan text dalam banyak bahasa.
- Untuk _RedisMessageHandler_ akan dikombinasikan dengan data yang tersimpan di database.

## Bean

``` java
// Redis
@Singleton
MessageHandler messageHandler(
    AppProperties appProperties,
    DataMapper dataMapper,
    EntityTrxManager entityTrxManager,
    RedisDataSource redisDataSource
) {
    MessageDefinition message = appProperties.message().orElseThrow();
    return new RedisMessageHandler()
    .setDataMapper(dataMapper)
    .setDefaultLanguage(message.defaultLanguage().orElse(null))
    .setEntityClass(null)
    .setEntityTrxManager(entityTrxManager)
    .setLimitReloadData(message.limitReloadData().orElse(null))
    .setMaxReloadThread(message.maxReloadThread().orElse(null))
    .setRedisParam(
        new RedisParam(StorageKeyDefinition.convert(message.redisParam().orElse(null)))
        .setRedisDataSource(redisDataSource)
    );
}

// Resource
@Singleton
MessageHandler messageHandler(
    AppProperties appProperties
) {
    MessageDefinition message = appProperties.message().orElseThrow();
    SessionLocaleResolver localeResolver = new SessionLocaleResolver();
    localeResolver.setDefaultLocale(new Locale(message.defaultLanguage().orElse(null)));
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setAlwaysUseMessageFormat(false);
    messageSource.setBasenames(message.basenames().orElse(StringSet.empty()).toArray(new String[0]));
    messageSource.setDefaultEncoding(message.defaultEncoding().orElse(null));
    messageSource.setFallbackToSystemLocale(message.fallbackToSystemLocale().orElse(null));
    messageSource.setCacheSeconds(message.cacheSeconds().orElse(null));
    messageSource.setUseCodeAsDefaultMessage(message.useCodeAsDefaultMessage().orElse(null));
    return new ResourceBundleMessageHandler()
    .setLocaleResolver(localeResolver)
    .setMessageSource(messageSource);
}
```

## Screenshot

<div>
   <img src="./assets/message-01.png" alt="" width="800" />
</div>
<br/>
<div>
   <img src="./assets/message-02.png" alt="" width="800" />
</div>

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
