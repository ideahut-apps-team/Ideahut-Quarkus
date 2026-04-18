[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Binary Serializer
* Untuk _serialize_ / _deserialize_ object ke / dari byte array.
* Berguna untuk menyimpan data biner ke penyimpanan seperti database dan [redis](./18-redis.md).
* Berguna untuk service berbasis data biner.
``` java
public interface BinarySerializer {
	<T> byte[] serialize(Class<? extends T> type, T object);
	<T> void serialize(Class<? extends T> type, T object, OutputStream outputStream);
	
	<T> T deserialize(Class<T> type, byte[] bytes);
	<T> T deserialize(Class<T> type, InputStream inputStream);
}
```

## DataMapperBinarySerializer
[(Sumber)](https://github.com/FasterXML/jackson/) -> Menggunakan [DataMapper](./04-mapper.md) untuk byte array dalam format json / xml.
``` java
// JSON
BinarySerializer binarySerializer = new DataMapperBinarySerializer()
.setMapper(dataMapper)
.setFormat(DataMapper.JSON);

// XML
BinarySerializer binarySerializer = new DataMapperBinarySerializer()
.setMapper(dataMapper)
.setFormat(DataMapper.XML);
```
* `setMapper`: [DataMapper](./04-mapper.md) bean.
* `setFormat`: Format DataMapper.JSON atau DataMapper.XML.

## HessianBinarySerializer
[(Sumber)](http://hessian.caucho.com/) -> Mendukung versi 1 & 2.
``` java
BinarySerializer binarySerializer = new HessianBinarySerializer()
.setSerializerFactory(null)
.setVersion(null);
```
* `setSerializerFactory`: Untuk custom serializer factory.
* `setVersion`: Versi hessian yang digunakan 1 atau 2.

## JdkBinarySerializer
[(Sumber)](https://docs.oracle.com/javase/7/docs/api/java/io/ObjectOutputStream.html) -> Menggunakan bawaan JDK.
``` java
BinarySerializer binarySerializer = new JdkBinarySerializer();
```

## KryoBinarySerializer
[(Sumber)](https://github.com/EsotericSoftware/kryo)
``` java
BinarySerializer binarySerializer = new KryoBinarySerializer()
.setAutoReset(null)
.setBufferSize(null)
.setClassLoader(null)
.setCopyReferences(null)
.setDefaultSerializerClass(null)
.setDefaultSerializerFactory(null)
.setDefinition(null)
.setInstantiatorStrategy(null)
.setKryo(null)
.setMaxBufferSize(null)
.setMaxDepth(null)
.setOptimizedGenerics(null)
.setReferenceResolver(null)
.setReferences(null)
.setRegistrationRequired(null)
.setWarnUnregisteredClasses(null);
```
* `setKryo`: Jika ingin menggunakan object Kryo yang sudah ada atau custom.
* Untuk setter lain sama dengan penjelasan di [(Sumber)](https://github.com/EsotericSoftware/kryo).

## ForyBinarySerializer
[(Sumber)](https://fory.apache.org/)
``` java
BinarySerializer binarySerializer = new ForyBinarySerializer()
.setAsyncCompilation(null)
.setBufferSizeLimitBytes(null)
.setClassLoader(null)
.setClassVersionCheck(null)
.setCodegen(null)
.setCompatible(null)
.setCompatibleMode(null)
.setDefinition(null)
.setDeserializeUnknownClass(null)
.setDeserializeUnknownEnumValueAsNull(null)
.setDisableLogging(null)
.setFory(null)
.setIgnoreStringRef(null)
.setIgnoreTimeRef(null)
.setInstanceType(null)
.setIntArrayCompressed(null)
.setIntCompressed(null)
.setJdkClassSerializableCheck(null)
.setLanguage(null)
.setLongArrayCompressed(null)
.setLongCompressed(null)
.setLongEncoding(null)
.setMapRefLoadFactor(null)
.setMaxDepth(null)
.setMaxPoolSize(null)
.setMetaCompressor(null)
.setMetaShare(null)
.setMinPoolSize(null)
.setName(null)
.setNumberCompressed(null)
.setPoolExpiry(null)
.setRefCopy(null)
.setRefTracking(null)
.setRegisterGuavaTypes(null)
.setRequireClassRegistration(null)
.setScalaOptimizationEnabled(null)
.setScopedMetaShare(null)
.setSerializeEnumByName(null)
.setSerializerFactory(null)
.setStringCompressed(null)
.setSuppressClassRegistrationWarnings(null)
.setTypeChecker(null)
.setUnknownEnumValueStrategy(null)
.setWriteNumUtf16BytesForUtf8Encoding(null)
.setXlang(null);
```
* `setFory`: Jika ingin menggunakan object Fory yang sudah ada, atau custom. Bisa berguna untuk integrasi dengan [GraalVM](https://fory.apache.org/docs/guide/serialization).
* Untuk setter lain sama dengan penjelasan di [(Sumber)](https://fory.apache.org/docs/guide/java/).

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
