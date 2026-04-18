[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Annotation

Daftar _annotation_ yang tersedia.

## @ApiExclude

Penggunaan di entity / model (CRUD endpoint), dan di controller (request mapping).

```java
// Entity / Model tidak disertakan ke CRUD Resource
@ApiExclude
@Audit
@Entity
@Table(name = "api_provider")
@Setter
@Getter
@SuppressWarnings("serial")
public class ApiProvider extends EntityAudit {
    @Id
    @Column(name = "api_name", unique = true, nullable = false, length = 128)
    private String apiName;

    @Column(name = "secret")
    private String secret;
}
```

```java
// Semua request mapping di Controller ini tidak disertakan ke API Resource
@ApiExclude
@ComponentScan
@RestController
@RequestMapping("/admin")
class AdminController extends WebMvcAdminController {
    private final DataMapper dataMapper;
    private final AdminHandler adminHandler;
    @Autowired
    AdminController(
       DataMapper dataMapper,
       AdminHandler adminHandler
    ) {
       this.dataMapper = dataMapper;
       this.adminHandler = adminHandler;
    }
    @Override
    protected AdminHandler adminHandler() {
       return adminHandler;
    }
    @Override
    protected DataMapper dataMapper() {
       return dataMapper;
    }

}
```

## @ApiSkip

_Request mapping_ akan dilewati oleh ApiService (tidak ada pengecekan security)

```java
@ApiSkip
@PostMapping("/access")
ApiAccess access(HttpServletRequest request) {
    return authService.getApiAccessForExternal(request);
}
```

## @Audit

Untuk mengidentifikasi suatu entity / model akan diaudit setiap perubahannya (CREATE / UPDATE / DELETE).

``` java
@Audit
@Entity
public class User extends EntityAuditSoftDelete {
    @Id
    @GeneratedValue(generator = OdtIdGenerator.NAME)
    @GenericGenerator(name = OdtIdGenerator.NAME, strategy = OdtIdGenerator.STRATEGY)
    @Column(name = "user_id", unique = true, nullable = false, length = 64)
    private String userId;
}
```

## @Body

* Request body di-_consume_ di level filter atau controller (karena body request hanya bisa di-_consume_ sekali). Untuk stream request, sebaiknnya di-_consume_ di level controller.
* Response body di-_produce_ oleh _response writer_ atau manual di controller. Untuk stream response, sebaiknya di-_produce_ secara manual di level controller.

``` java
// request body di-consume di controller / resource
@Body(request = true, response = false)
@POST
@Path("/upload")
Uni<Result> upload(@Context ContainerRequestContext request) {
    byte[] data = WebHelper.getBodyAsBytes(request);
}

// response body di-produce di controller / resource
@Body(request = false, response = true)
@POST
@Path("/stream")
Multi<byte[]> stream(@QueryParam("id") String id) {
}

// request body di-consume di controller / resource, dan response body di-produce di controller / resource
@Body(request = true, response = true)
@GET
ResponseEntity<StreamingResponseBody> report(@Context ContainerRequestContext request, @Context ContainerResponseContext response) throws Exception {
   // get stream request
   // send stream response
}
```

## @ForeignKeyEntity

Alternatif jika entity / model tidak menggunakan @ManyToOne tapi foreign key tetap ingin didefinisikan. Ada kasus, terjadi kegagalan pada saat _build_ Native Image jika package entity (yang punya @ManyToOne) berbeda dengan package aplikasi.

``` java
@Entity
@Table(
    name = "api_role_request",
    indexes = {
        @Index( name = "idx_api_role_request__api_role", columnList = "api_role_code"),
    }
)
@ForeignKeyEntity(
    name = "fk_api_role_request__api_role", 
    referencedEntity = ApiRole.class, 
    fieldJoins = { 
        @ForeignKeyJoin(field = "apiRoleCode", referencedField = "apiRoleCode"),
    },
    onDeleteAction = ForeignKeyAction.CASCADE,
    onUpdateAction = ForeignKeyAction.CASCADE
)
@Setter
@Getter
@SuppressWarnings("serial")
public class ApiRoleRequest extends EntityAudit {
}
```

## @IdPrefix

Menambah prefix ke id yang _auto generated_.

``` java
@Audit
@IdPrefix("USR-")
@Entity
public class User extends EntityAuditSoftDelete {
    @Id
    @GeneratedValue(generator = OdtIdGenerator.NAME)
    @GenericGenerator(name = OdtIdGenerator.NAME, strategy = OdtIdGenerator.STRATEGY)
    @Column(name = "user_id", unique = true, nullable = false, length = 64)
    private String userId;
}
```

Dari contoh di atas, jika id yang digenerate "abcd012989", maka yang tersimpan di table menjadi "USR-abcd012989".

## @Login

Sebagai tanda apakah API yang diakses wajib login atau tidak. Default: true.

``` java
@Login(false)
@PostMapping(value = "/login")
public Result login(
    @RequestParam("username") String username,
    @RequestParam("password") String password
) {

}
```

## @Public

Sebagai tanda apakah API yang diakses public atau tidak. Default: true.

``` java
@Public
@PostMapping(value = "/info")
public Result info() {
}
```

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
