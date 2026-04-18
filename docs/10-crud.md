[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# CRUD

- Secara otomatis semua entity / model yang terdeteksi oleh [EntityTrxManager](./09-trxmanager.md) dapat diakses menggunakan modul ini.
- Untuk membatasi akses dapat dipasangkan dengan ApiRole (lihat: modul [API](./21-api.md)).

## Bean

- `CrudHandler`: menangani pengambilan resource, permission, dan eksekusi.
- `CrudResource`: mendapatkan CrudProperties yang akan dieksekusi oleh `CrudHandler`.
- `CrudPermission`: mengecek apakah eksekusi diijinkan atau tidak.

``` java
@Singleton
CrudHandler crudHandler(
    AppProperties appProperties,
    EntityTrxManager entityTrxManager,
    DataMapper dataMapper,
    CrudResource resource,
    CrudPermission permission
) {
    CrudDefinition crud = appProperties.crud().orElseThrow();
    return new CrudHandlerImpl()
            
    // semua query menggunakan sql (native) atau tidak
    .setAlwaysUseNative(crud.alwaysUseNative().orElse(null))
    
    // default maksimum jumlah data saat retrieve (PAGE, LIST, MAP)
    .setDefaultMaxLimit(crud.defaultMaxLimit().orElse(null))
    
    // EntityTrxManager
    .setEntityTrxManager(entityTrxManager)
    
    // Informasi aksi disertakan di respon atau tidak
    .setInfoEnabled(crud.infoEnabled().orElse(null))
    
    // CrudPermission
    .setPermission(permission)
    
    // CrudResource
    .setResource(resource)
    
    // Daftar filter specific yang akan disertakan saat query
    .setSpecificValueGetters(CrudSupport.getSpecificValueGetters())
    
    // Flag apakah bulk diaktifkan atau tidak
    .setBulkEnabled(crud.bulkEnabled().orElse(null))
    
    // Maksimum jumlah operasi / aksi CRUD yang dibolehkan, diisi 0 untuk tak terbatas
    .setMaxBulkSize(crud.maxBulkSize().orElse(null))
    
    // Maksimum jumlah dependensi / layer CRUD yang dibolehkan, diisi 0 untuk tak terbatas
    .setMaxBulkLayer(crud.maxBulkLayer().orElse(null));
    
}

// Contoh CrudResource berdasarkan nama class yang didefinisikan di CrudRequest
@Singleton
CrudResource crudResource(
    EntityTrxManager entityTrxManager
) {
    return new CrudResource() {
        @Override
        public CrudProperties getCrudProperties(String manager, String name) {
            try {
                Class<?> clazz = ObjectHelper.classOf("net.ideahut.quarkus.template.entity." + name);
                TrxManagerInfo trxManagerInfo = entityTrxManager.getDefaultTrxManagerInfo();
                if (manager != null && !manager.isEmpty()) {
                    trxManagerInfo = entityTrxManager.getTrxManagerInfo(manager);
                }
                EntityInfo entityInfo = trxManagerInfo.getEntityInfo(clazz);
                CrudProperties properties = new CrudProperties();
                properties.setEntityInfo(entityInfo);
                properties.setMaxLimit(200);
                properties.setUseNative(false);
                return properties;
            } catch (Exception e) {
                throw ErrorHelper.exception(e);
            }
        }
    };
}

// Contoh CrudResource yang diambil menggunakan ApiService
@Singleton
CrudResource crudResource(
    ApiService apiService
) {
    return (manager, name) -> {
        ApiAccess apiAccess = ApiAccess.fromContext();
        CrudProperties properties = apiService.getApiCrudProperties(apiAccess, name);
        Assert.notNull(properties, "CrudProperties is not found: " + name);
        return properties;
    };
}

// Contoh CrudPermission mengijinkan semua CrudRequest
@Singleton
CrudPermission crudPermission() {
    return (action, request) -> true;
}

// Contoh CrudPermission yang mengecek apakah action didefinisikan di table / ApiService
@Singleton
CrudPermission crudPermission() {
    return (action, request) -> {
        CrudProperties properties = request.getProperties();
        Set<CrudAction> actions = properties.getActions();
        return actions != null && actions.contains(action);
    };
}
```

## Controller / Resource

``` java
@Public
@Path("/crud")
class CrudController extends net.ideahut.quarkus.crud.CrudController {
    
    @POST
	@Path("/action/{action}")
	public Result action(
		@NotBlank @PathParam("action") String action,
		@Context ContainerRequestContext request
	) {
		byte[] data = WebHelper.getBodyAsBytes(request);
		return super.body(CrudAction.valueOf(action.toUpperCase()), data);
	}

    @POST
	@Path("/bulk/list")
	public List<Result> bulkList(@Context ContainerRequestContext request) {
		byte[] data = WebHelper.getBodyAsBytes(request);
		return super.bulkList(data);
	}

    @POST
	@Path("/bulk/map")
	public Map<String, Result> bulkMap(@Context ContainerRequestContext request) {
		byte[] data = WebHelper.getBodyAsBytes(request);
		return super.bulkMap(data);
	}
}
```

## Action

- `UNIQUE` mendapatkan satu data unik (akan error jika data lebih dari satu).
- `SINGLE` mendapatkan hanya satu data.
- `PAGE` mendapatkan koleksi data disertai informasi halaman.
- `LIST` mendapatkan koleksi data.
- `MAP` mendapatkan koleksi data dalam bentuk _key_-_value_.
- `CREATE` membuat data baru.
- `UPDATE` memperbaharui data yang sudah ada.
- `SAVE` membuat / memperbaharui data.
- `DELETE` menghapus satu data.
- `DELETES` menghapus koleksi data.

## Condition

- `ANY_LIKE` mengandung kalimat / kata (huruf besar / kecil tidak masalah).
- `ANY_START` dimulai dengan kalimat / kata (huruf besar / kecil tidak masalah).
- `ANY_END` diakhiri dengan kalimat / kata (huruf besar / kecil tidak masalah).
- `ANY_EQUAL` sama dengan kalimat / kata (huruf besar / kecil tidak masalah).
- `LIKE` mengandung kalimat / kata.
- `START` dimulai dengan kalimat / kata.
- `END` diakhiri dengan kalimat / kata.
- `NOT_ANY_LIKE` tidak mengandung kalimat / kata (huruf besar / kecil tidak masalah).
- `NOT_ANY_START` tidak dimulai dengan kalimat / kata (huruf besar / kecil tidak masalah).
- `NOT_ANY_END` tidak diakhiri dengan kalimat / kata (huruf besar / kecil tidak masalah).
- `NOT_ANY_EQUAL` tidak sama dengan kalimat / kata (huruf besar / kecil tidak masalah).
- `NOT_LIKE` tidak mengandung kalimat / kata.
- `NOT_START` tidak dimulai dengan kalimat / kata.
- `NOT_END` tidak diakhiri dengan kalimat / kata.
- `NOT_EQUAL` tidak sama dengan.
- `BETWEEN` diantara, min - max.
- `NOT_NULL` bukan null.
- `IS_NULL` hanya yang null.
- `GREATER_THAN` lebih besar.
- `GREATER_EQUAL` lebih besar atau sama dengan.
- `LESS_THAN` lebih kecil.
- `LESS_EQUAL` lebih kecil atau sama dengan.
- `IN` hanya yang di dalam.
- `NOT_IN` selain yang di dalam.
- `EQUAL` sama dengan.

## Request

- `name` nama resource.
- `manager` nama transaction manager.
- `replica` nomor table dari entity.
- `ids` array id entity.
- `id` id entity.
- `map` definisi key-value untuk action MAP.
- `page` informasi halaman.
- `start` start offset.
- `limit` batas jumlah data.
- `filters` array filter.
- `orders` array order (descending dimulai dengan karakter '-').
- `fields` daftar field yang ditampilkan.
- `excludes` daftar field yang tidak ditampilkan.
- `loads` daftar field yang datanya diambil dari entity lain (_Lazy Load_).
- `values` array value, untuk insert & update.
- `joins` array join dengan entity lain.
- `stacks` array stack, proses bertingkat. Contoh: setelah user disimpan, bisa dilanjutkan dengan menyimpan data detail atau otentikasi.

## Bulk
- Untuk mengeksekusi banyak aksi CRUD dalam satu kali request.
- Mendukung suatu aksi akan dieksekusi ketika aksi lain sudah selesai prosesnya (bertingkat).


### Page

``` js
// Request
{
    "name": "app.AutoGenStrIdHardDel",
    "page": {
        "index": 1,
        "size": 10,
        "count": true
    },
    "filters": [
        {
            "field": "name",
            "condition": "NOT_NULL"
        },
        {
            "field": "date",
            "condition": "BETWEEN",
            "values": ["2024-01-01 00:00:00", "2024-01-31 23:59:59"]
        }
    ],
    "orders":["-createdOn"]
}
// Response
{
    "time": 82150375,
    "status": 0,
    "info": {
        "action": "PAGE",
        "manager": "mainTransactionManager",
        "name": "app.AutoGenStrIdHardDel"
    },
    "data": {
        "index": 1,
        "size": 10,
        "total": 1,
        "records": 1,
        "count": true,
        "data": [
            {
                "createdOn": 1763717045176,
                "updatedOn": 1763717045176,
                "id": "2025-411216-1624-05174-25200-0001",
                "name": "COBA",
                "description": "description",
                "isActive": "Y",
                "date": "2024-01-02 18:43:24",
                "condition": "ANY_LIKE"
            }
        ]
    }
}

## Contoh menggunakan replica
// Request (replica = 1)
{
    "name": "app.Information",
    "replica": 1,
    "page": {
        "index": 1,
        "size": 100,
        "count": true
    },
    "orders":["-createdOn"]
}
// Response (replica = 1)
{
    "time": 68235208,
    "status": 0,
    "info": {
        "action": "PAGE",
        "manager": "mainTransactionManager",
        "name": "app.Information"
    },
    "data": {
        "index": 1,
        "size": 100,
        "total": 1,
        "records": 1,
        "count": true,
        "data": [
            {
                "createdOn": 1764043087781,
                "updatedOn": 1764043087781,
                "informationId": "INF2025-411253-1058-07774-25200-0000",
                "title": "admin",
                "image": "A",
                "description": "description",
                "content": "So, you'll think that something is wrong with the mapping - and thats right! It seems like Hibernate maps LOBs to OIDs unless others specified. Duh..",
                "isExternal": "N",
                "isActive": "Y",
                "seqno": 1
            }
        ]
    }
}
// Request (replica = 2)
{
    "name": "app.Information",
    "replica": 2,
    "page": {
        "index": 1,
        "size": 100,
        "count": true
    },
    "orders":["-createdOn"]
}
// Response (replica = 2)
{
    "time": 69575250,
    "status": 0,
    "info": {
        "action": "PAGE",
        "manager": "mainTransactionManager",
        "name": "app.Information"
    },
    "data": {
        "index": 1,
        "size": 100,
        "total": 0,
        "records": 0,
        "count": true
    }
}
```

### Map

``` js
// Request
{
    "name": "app.AutoGenStrIdHardDel",
    "start": 0,
    "limit": 10,
    "map": {
        "keys": ["id"],
        "flat": true
    },
    "filters": [
        {
            "field": "createdOn",
            "condition": "not_null"
        }        
    ],
    "orders":["-createdOn"]

}
// Response
{
    "time": 48872709,
    "status": 0,
    "info": {
        "action": "MAP",
        "manager": "mainTransactionManager",
        "name": "app.AutoGenStrIdHardDel"
    },
    "data": {
        "id:2025-411216-1624-05174-25200-0001": {
            "createdOn": 1763717045176,
            "updatedOn": 1763717045176,
            "id": "2025-411216-1624-05174-25200-0001",
            "name": "COBA",
            "description": "description",
            "isActive": "Y",
            "date": "2024-01-02 18:43:24",
            "condition": "ANY_LIKE"
        }
    }
}
```

### Unique

``` js
// Request
{
    "name": "app.AutoGenStrIdHardDel",
    "id": "2025-411216-1624-05174-25200-0001"
}
// Response
{
    "time": 41023458,
    "status": 0,
    "info": {
        "action": "UNIQUE",
        "manager": "mainTransactionManager",
        "name": "app.AutoGenStrIdHardDel"
    },
    "data": {
        "createdOn": 1763717045176,
        "updatedOn": 1763717045176,
        "id": "2025-411216-1624-05174-25200-0001",
        "name": "COBA",
        "description": "description",
        "isActive": "Y",
        "date": "2024-01-02 18:43:24",
        "condition": "ANY_LIKE"
    }
}
```

### Create

``` js
// Request
{
    "name": "app.AutoGenStrIdHardDel",
    "value": {
        "name": "COBA",
        "isActive": "Y",
        "description": "description",
        "date": "2025-11-25 18:43:24",
        "condition": "ANY_LIKE"
    }
}
// Response
{
    "time": 62282041,
    "status": 0,
    "info": {
        "action": "CREATE",
        "manager": "mainTransactionManager",
        "name": "app.AutoGenStrIdHardDel"
    },
    "data": {
        "createdOn": 1764044505853,
        "updatedOn": 1764044505853,
        "id": "2025-411253-1121-45852-25200-0019",
        "name": "COBA",
        "description": "description",
        "isActive": "Y",
        "date": "2025-11-25 18:43:24",
        "condition": "ANY_LIKE"
    }
}
```

### Update

``` js
// Request
{
    "name": "app.AutoGenStrIdHardDel",
    "id": "2025-411253-1121-45852-25200-0019",
    "value": {
        "name": "yyyyyTest (Edited)--",
        "description": null,
        "date": "2025-11-30 18:00:02",
        "condition": "BETWEEN"
    }
}
// Response
{
    "time": 44078625,
    "status": 0,
    "info": {
        "action": "UPDATE",
        "manager": "mainTransactionManager",
        "name": "app.AutoGenStrIdHardDel"
    },
    "data": {
        "createdOn": 1764044505853,
        "updatedOn": 1764044611933,
        "id": "2025-411253-1121-45852-25200-0019",
        "name": "yyyyyTest (Edited)--",
        "isActive": "Y",
        "date": "2025-11-30 18:00:02",
        "condition": "BETWEEN"
    }
}
```

### Delete

``` js
// Request
{
    "name": "app.AutoGenStrIdHardDel",
    "id": "2025-411253-1121-44949-25200-0018"
}
// Response
{
    "time": 82880958,
    "status": 0,
    "info": {
        "action": "DELETE",
        "manager": "mainTransactionManager",
        "name": "app.AutoGenStrIdHardDel"
    },
    "data": {
        "createdOn": 1764044504953,
        "updatedOn": 1764044504953,
        "id": "2025-411253-1121-44949-25200-0018",
        "name": "COBA",
        "description": "description",
        "isActive": "Y",
        "date": "2025-11-25 18:43:24",
        "condition": "ANY_LIKE"
    }
}
```

### Deletes

``` js
// Request
{
    "name": "app.AutoGenStrIdHardDel",
    "ids": [
        "2025-411253-1121-41518-25200-0015",
        "2025-411253-1121-42583-25200-0016",
        "2025-411253-1121-44042-25200-0017"
    ]
}
// Response
{
    "time": 160827584,
    "status": 0,
    "info": {
        "action": "DELETES",
        "manager": "mainTransactionManager",
        "name": "app.AutoGenStrIdHardDel"
    },
    "data": [
        {
            "createdOn": 1764044501525,
            "updatedOn": 1764044501525,
            "id": "2025-411253-1121-41518-25200-0015",
            "name": "COBA",
            "description": "description",
            "isActive": "Y",
            "date": "2025-11-25 18:43:24",
            "condition": "ANY_LIKE"
        },
        {
            "createdOn": 1764044502586,
            "updatedOn": 1764044502586,
            "id": "2025-411253-1121-42583-25200-0016",
            "name": "COBA",
            "description": "description",
            "isActive": "Y",
            "date": "2025-11-25 18:43:24",
            "condition": "ANY_LIKE"
        },
        {
            "createdOn": 1764044504044,
            "updatedOn": 1764044504044,
            "id": "2025-411253-1121-44042-25200-0017",
            "name": "COBA",
            "description": "description",
            "isActive": "Y",
            "date": "2025-11-25 18:43:24",
            "condition": "ANY_LIKE"
        }
    ]
}
```

### Bulk (List)
``` js
// Request
[
    {
        "action": "save",
        "name": "app.CompositeHardDel",
        "value": {
            "type": 1,
            "code": "A",
            "name": "COBA",
            "isActive": "Y",
            "description": "description"
        }
    },
    {
        "action": "create",
        "after": 0,
        "name": "app.LongIdJoinComposite",
        "value": {
            "name": "COBA",
            "isActive": "Y",
            "description": "description",
            "composite": {
                "type": "{[0].type}",
                "code": "{[0].code}"
            }
        }
    },
    {
        "action": "page",
        "after": 0,
        "name": "app.CompositeHardDel",
        "page": {
            "index": 1,
            "size": 1,
            "count": false
        },
        "filters": [
            {
                "field": "name",
                "condition": "NOT_null"
            }       
        ],
        "orders":["-createdOn"]
    },
    {
        "action": "page",
        "after": 1,
        "name": "app.LongIdJoinComposite",
        "page": {
            "index": 1,
            "size": 1,
            "count": false
        },
        "joins": [
            {
                "name": "app.CompositeHardDel",
                "store": "composite",
                "relations": [
                    {
                        "target": "type",
                        "value": "{[0].type}"
                    },
                    {
                        "target": "code",
                        "value": "{[0].code}"
                    }
                ]
            }
        ],
        "filters": [
            {
                "field": "name",
                "condition": "NOT_NULL"
            }   
        ],
        "orders":["-createdOn"]
    }
]
// Response
{
    "time": 184308166,
    "status": 0,
    "data": [
        {
            "time": 53422916,
            "status": 0,
            "info": {
                "action": "SAVE",
                "manager": "mainTransactionManager",
                "name": "app.CompositeHardDel"
            },
            "data": {
                "createdOn": 1764045165036,
                "updatedOn": 1764045165036,
                "type": 1,
                "code": "A",
                "name": "COBA",
                "description": "description",
                "isActive": "Y"
            }
        },
        {
            "time": 118973958,
            "status": 0,
            "info": {
                "action": "CREATE",
                "manager": "mainTransactionManager",
                "name": "app.LongIdJoinComposite"
            },
            "data": {
                "createdOn": 1764045205486,
                "updatedOn": 1764045205486,
                "id": 67,
                "name": "COBA",
                "description": "description",
                "isActive": "Y",
                "composite": {
                    "type": 1,
                    "code": "A"
                }
            }
        },
        {
            "time": 100862791,
            "status": 0,
            "info": {
                "action": "PAGE",
                "manager": "mainTransactionManager",
                "name": "app.CompositeHardDel"
            },
            "data": {
                "index": 1,
                "size": 1,
                "count": false,
                "data": [
                    {
                        "createdOn": 1764045165036,
                        "updatedOn": 1764045165036,
                        "type": 1,
                        "code": "A",
                        "name": "COBA",
                        "description": "description",
                        "isActive": "Y"
                    }
                ]
            }
        },
        {
            "time": 170593208,
            "status": 0,
            "info": {
                "action": "PAGE",
                "manager": "mainTransactionManager",
                "name": "app.LongIdJoinComposite"
            },
            "data": {
                "index": 1,
                "size": 1,
                "count": false,
                "data": [
                    {
                        "createdOn": 1764045205486,
                        "updatedOn": 1764045205486,
                        "id": 67,
                        "name": "COBA",
                        "description": "description",
                        "isActive": "Y",
                        "composite": {
                            "createdOn": 1764045165036,
                            "updatedOn": 1764045165036,
                            "type": 1,
                            "code": "A",
                            "name": "COBA",
                            "description": "description",
                            "isActive": "Y"
                        }
                    }
                ]
            }
        }
    ]
}
```

### Bulk (Map)
``` js
// Request
{
    "CompositeHardDel-Save":
    {
        "action": "save",
        "name": "app.CompositeHardDel",
        "value": {
            "type": 1,
            "code": "A",
            "name": "COBA",
            "isActive": "Y",
            "description": "description"
        }
    },
    "LongIdJoinComposite-Create":
    {
        "action": "create",
        "after": "CompositeHardDel-Save",
        "name": "app.LongIdJoinComposite",
        "value": {
            "name": "COBA",
            "isActive": "Y",
            "description": "description",
            "composite": {
                "type": "{[CompositeHardDel-Save].type}",
                "code": "{[CompositeHardDel-Save].code}"
            }
        }
    },
    "CompositeHardDel-Page":
    {
        "action": "page",
        "after": "CompositeHardDel-Save",
        "name": "app.CompositeHardDel",
        "page": {
            "index": 1,
            "size": 1,
            "count": false
        },
        "filters": [
            {
                "field": "name",
                "condition": "NOT_null"
            }       
        ],
        "orders":["-createdOn"]
    },
    "LongIdJoinComposite-Page":
    {
        "action": "page",
        "after": "LongIdJoinComposite-Create",
        "name": "app.LongIdJoinComposite",
        "page": {
            "index": 1,
            "size": 1,
            "count": false
        },
        "joins": [
            {
                "name": "app.CompositeHardDel",
                "store": "composite",
                "alias": "CompositeHardDel",
                "relations": [
                    {
                        "target": "type",
                        "source": "composite.type"
                    },
                    {
                        "target": "code",
                        "source": "composite.code"
                    }
                ]
            }
        ],
        "filters": [
            {
                "field": "CompositeHardDel.type",
                "condition": "EQUAL",
                "value": "{[CompositeHardDel-Save].type}"
            },
            {
                "field": "CompositeHardDel.code",
                "condition": "EQUAL",
                "value": "{[CompositeHardDel-Save].code}"
            }   
        ],
        "orders":["-createdOn"]
    }
}
// Response
{
    "time": 167108625,
    "status": 0,
    "data": {
        "CompositeHardDel-Save": {
            "time": 53876250,
            "status": 0,
            "info": {
                "action": "SAVE",
                "manager": "mainTransactionManager",
                "name": "app.CompositeHardDel"
            },
            "data": {
                "createdOn": 1764045165036,
                "updatedOn": 1764045165036,
                "type": 1,
                "code": "A",
                "name": "COBA",
                "description": "description",
                "isActive": "Y"
            }
        },
        "LongIdJoinComposite-Create": {
            "time": 108882042,
            "status": 0,
            "info": {
                "action": "CREATE",
                "manager": "mainTransactionManager",
                "name": "app.LongIdJoinComposite"
            },
            "data": {
                "createdOn": 1764045373587,
                "updatedOn": 1764045373587,
                "id": 88,
                "name": "COBA",
                "description": "description",
                "isActive": "Y",
                "composite": {
                    "type": 1,
                    "code": "A"
                }
            }
        },
        "CompositeHardDel-Page": {
            "time": 113667292,
            "status": 0,
            "info": {
                "action": "PAGE",
                "manager": "mainTransactionManager",
                "name": "app.CompositeHardDel"
            },
            "data": {
                "index": 1,
                "size": 1,
                "count": false,
                "data": [
                    {
                        "createdOn": 1764045165036,
                        "updatedOn": 1764045165036,
                        "type": 1,
                        "code": "A",
                        "name": "COBA",
                        "description": "description",
                        "isActive": "Y"
                    }
                ]
            }
        },
        "LongIdJoinComposite-Page": {
            "time": 155756917,
            "status": 0,
            "info": {
                "action": "PAGE",
                "manager": "mainTransactionManager",
                "name": "app.LongIdJoinComposite"
            },
            "data": {
                "index": 1,
                "size": 1,
                "count": false,
                "data": [
                    {
                        "createdOn": 1764045373587,
                        "updatedOn": 1764045373587,
                        "id": 88,
                        "name": "COBA",
                        "description": "description",
                        "isActive": "Y",
                        "composite": {
                            "createdOn": 1764045165036,
                            "updatedOn": 1764045165036,
                            "type": 1,
                            "code": "A",
                            "name": "COBA",
                            "description": "description",
                            "isActive": "Y"
                        }
                    }
                ]
            }
        }
    }
}
```

## Screenshot

<div>
   <img src="./assets/crud.png" alt="" width="800" />
</div>

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
