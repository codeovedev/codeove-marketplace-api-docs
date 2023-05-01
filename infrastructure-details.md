# Altyapı Detayları

## Program.cs
API tarafındaki `program.cs` sınıfı içindeki kurulum da gerekli tüm bağımlılıklar işlenmiştir. Bağımlılıkların tümü bu metot içinde değildir. Her katmandaki bağımlılıklar(dependency injection) kendi projesi içerisindeki `startup.cs` içerisindeki `ConfigureServices` içerisinde toplanmıştır. Örneğin repository sınıfları ve interface leri `DataAccess` projesinde olduğundan dependency injection tanımları bu proje içerisinde ki `startup.cs` içinde tanımlıdır.

API projesindeki `program.cs` içerisinde business projesi `startup.cs` tetiklemesi ve bağımlılıklarının yüklenmesi, business projesi `startup.cs` içinde dataAccess projesi `startup.cs` tetiklenmesi ve bağımlılıklarının yüklenmesi sağlanmaktadır.

### Marketplace.API > Program.cs
```csharp
// Adding business dependecies..
new Business.Startup(builder.Configuration).ConfigureServices(builder.Services);
```

### Marketplace.Business > Startup.cs
```csharp            
new DataAccess.Startup(Configuration).ConfigureServices(services);
```

### Startup.CustomConfiguration Metotları
`Startup.cs` sınıfları içerisindeki `CustomConfiguration` metotları ise. API projesindeki bağımlılıkların yüklenmesi sonrasıdaki build işlemi sonrası çalıştırılması istenilen işlemler için kurgulanmıştır. Şu an sadece veri tabanı migration işlemi bu metotlarda tetiklenir. **API > Business > DataAccess** şeklinde metotların çağırımı yapılır.

API da,
```csharp
...

// WebApplication build.
var app = builder.Build();

// Database Migration and Seed Data Operations
#region Applying Custom Configuration

var scope = app.Services.CreateScope();

try
{
    AppStatic.ServiceProviderInstance = scope.ServiceProvider;
    AppStatic.Common = new WebCommonApi(AppStatic.ServiceProviderInstance.GetRequiredService<IHttpContextAccessor>());

    Business.Startup.CustomConfiguration(scope.ServiceProvider);
}
catch (System.Exception ex)
{
    Debug.WriteLine("");
    Debug.WriteLine("Error : " + ex.Message);
    Debug.WriteLine("");
}

...
```
Business da,
```csharp
public static void CustomConfiguration(IServiceProvider serviceProvider)
{
    DataAccess.Startup.CustomConfiguration(serviceProvider);
}
```

DataAccess de,
```csharp
public static void CustomConfiguration(IServiceProvider serviceProvider)
{
    IConfiguration configuration = serviceProvider.GetRequiredService<IConfiguration>();
    IMongoMigrationRepository migrationRepository = serviceProvider.GetRequiredService<IMongoMigrationRepository>();

    #region EntityFramework

    var contextEF = serviceProvider.GetRequiredService<DatabaseContext>();

    if (contextEF.Database.GetPendingMigrations().Count() > 0)
        contextEF.Database.Migrate();

    DatabaseInitializer initializerEF = new DatabaseInitializer();
    initializerEF.Seed(contextEF);

    #endregion

    #region MongoDB

    var context = serviceProvider.GetRequiredService<MongoDBContext>();
    MongoDatabaseInitializer initializer = new MongoDatabaseInitializer(configuration, serviceProvider);

    #endregion

    initializer.Migrate(context);
    initializer.Seed(context);
}
```

## Entities
Altyapı mimarisi oldukça basit ve anlaşılır kurgulanmıştır. `Codeove.Marketplace.Entities` projesi içerisinde entity sınıflarınız `EntityFramework` ya da `Mongo` tarafı için sınıflar oluşturulur. `EntityBase` sınıfından türetilirler ve `PrimaryKey` prop ları `ObjectId` türündedir. Örnek olarak `Brand` entity sınıfı.

```csharp
public class Brand : EntityBase<ObjectId>
{
    public string Name { get; set; }
    public string Slug { get; set; }
}
```

## Models
Models projesinde entity ler için gerekli işlemlere ait sınıflar oluşturulmuştur. Bu projenin ayrı yapılma sebebi, marketplace in ileride müşteriler için API sunulması noktasında bu DLL in onlara sunulması ve sınıfları oluşturma işlemleri için ekstra çabaya girmemeleri içindir.

Bu proje de yer alan class lar tamamen işlemlere ait(örneğin, ürün insert, kategori insert ya da update vs) gerekli validation kurallarını da içeren şekilde DTO ya da ViewModel lerin oluşturulması esasına dayanır. Bu sınıfların isimlendirilmesinde tabii ki esneksiniz ama genel olarak şöyle bir standart oluşturulmuştur. Hepsinde entity adı **ön eki** kullanılır(`Brand...`). Veri listeleme ile alakalı sınıfları `...Query` **son eki** ile bitiririz. Diğer sınıflar için ilgili işlem ismi ile bitebilir. `...Create` ya da `...Update` gibi **Insert** ve **Update** işlemleri için gerekli modellerdir. Örnek olarak `Brand` entity ait işlemler için gerekli sınıflar.

```csharp
public class BrandCreate
{
    public string Name { get; set; }
    public string Slug { get; set; }
}
```

```csharp
public class BrandQuery : ModelBase<string>
{
    public string Name { get; set; }
    public string Slug { get; set; }
    public decimal ProductCount { get; set; }
}

```

```csharp
public class BrandUpdate
{
    public string Name { get; set; }
    public string Slug { get; set; }
}
```

## Business
Bu proje içerisinde `Model` ve `Entity` sınıflarının birbirine dönüşmesi için `AutoMapperConfig.cs` isimli sınıf yer alır. Ayrıca tüm `...Manager.cs` isimli sınıflar bu proje de yer alır. Bu sınıflar da ilgili entity ler için gerekli eylemlere ait metotlar olmalıdır. Entity sınıfları ve Model sınıfları yapıldıktan sonra burada Manager sınıfı oluşturulmalı ve `AutoMapperConfig` e tipler arası dönüşüm tanımlaması yapılmalıdır.

AutoMapperConfig.cs içinde örnek,
```csharp
private void SectorMappings()
{
    CreateMap<Sector, SectorQuery>().ForMember(
        dest => dest.Id,
        opts => opts.MapFrom((x, y) => x.Id.ToString())).ReverseMap();

    CreateMap<SectorQuery, Sector>().ForMember(
        dest => dest.Id,
        opt => opt.MapFrom((x, y) => x.Id.ToObjectId())).ReverseMap();
    CreateMap<Sector, SectorCreate>().ReverseMap();
    CreateMap<Sector, SectorUpdate>().ReverseMap();
}
```

Örnek bir manager kodu; manager sınıfları için hazır temel işlemlere ait kodların gelmesi için `MyMongoManager<Sector, ObjectId, IMongoSectorRepository>` base sınıfından miras alınır. Bu sınıf içinde Create, List, Find, Update, Delete işlemlerine ait hazır metotlar gelmektedir. Aslında sadece  `MyMongoManager<..>` sınıfından miras alınarak manager sınıfı tasarımı tamamlananbilir. Fakat her manager sınıfı için özel işlemler gerekeceği için ilgili manager sınıfının `interface` i de yazılır ve manager sınıfına eklenir. Interface içinde `IMyMongoManager<Sector, ObjectId>` şeklinde hazır bir base interface vardır(Bu base sınıflar `MFramework` projelerinden gelmektedir). Aşağıdaki SectorManager örneğinde görebilirsiniz. Bu şekilde bir manager sınıfını DataAccess deki repository ile çalışacak şekilde temel işlemler için hızlıca ayağa kaldırabilirsiniz.
```csharp
public interface ISectorManager : IMyMongoManager<Sector, ObjectId>
{

}

public class SectorManager : MyMongoManager<Sector, ObjectId, IMongoSectorRepository>, ISectorManager
{
    public SectorManager(IMongoSectorRepository repository, IMapper mapper) : base(repository, mapper)
    {
    }
}
```

> DİKKAT : DataAccess içindeki metotlarda sadece entity tipleri kullanılmalıdır. Herhangi bir DTO ya da ViewModel olmamalıdır. Entity tiplerinden DTO ya da ViewModel e dönüşüm business sınıfındaki metotlarda olmalıdır!

## DataAccess
DataAccess projesinde ilgili repository sınıfları ve context sınıfı bulunmaktadır.

Context sınıfı, appsetting.json dan ilgili bağlantı cümlesini okuyarak veri tabanı bağlantısı için gerekli yapıları oluşturur. Bunu da `MongoDBContextBase` sınıfı içinde otomatik yapmaktadır.
```csharp
public class MongoDBContext : MongoDBContextBase
{
    public MongoDBContext(IConfiguration configuration) : base(configuration, configuration.GetConnectionString("MongoDatabaseConnectionString"), configuration.GetConnectionString("MongoDatabaseConnectionString").Split('/').Last())
    {

    }
}
```

Repository sınıfı, içinde ilgili koleksiyon için gerekli veri tabanı işlemlerine ait metotlar tanımlanır. Yine burada manager sınıflarında olduğu gibi base sınıflar(`MyMongoRepository`) kullanılarak class lar ve base interface(`IMyMongoRepository`) ler kullanılarak ilgili sınıf için gerekli interface tanımı mutlaka yapılır. Çünkü manager lar da da belirttiğimiz gibi koleksiyona ait özel veri tabanı işlemleri mutlaka bu sınıf içinde yazılır(Bu base sınıflar `MFramework` projelerinden gelmektedir). `Collection` attribute u ise mongodb de ilgili entity koleksiyonun(tablo) adını tanımlamak içindir.
```csharp
public interface IMongoBrandRepository : IMyMongoRepository<Brand, ObjectId>
{

}

[Collection("brands")]
public class MongoBrandRepository : MyMongoRepository<Brand, ObjectId>, IMongoBrandRepository
{
    public MongoBrandRepository(MongoDBContext mongoDbContext) : base(mongoDbContext)
    {
    }
}
```

> DİKKAT : DataAccess içindeki metotlarda sadece entity tipleri kullanılmalıdır. Herhangi bir DTO ya da ViewModel olmamalıdır. Entity tiplerinden DTO ya da ViewModel e dönüşüm business sınıfındaki metotlarda olmalıdır!

## API
Controller sınıfları burada yer alır. Bu sınıflarda ilgili manager sınıfları injection ile eklenerek kullanılır. `Authorize(Roles="...")` attribute ile ilgili role kısıtlamaları yapılır. Bu Controller sınıfları içerisindeki metotlarda sadece manager sınıflar ile işlem yapılmalıdır. 

Endpoint metotlarına işlemin başarılı olması durumunda döndürdüğü tip tanımlaması(`ProducesResponseType`) yapılmıştır. Bu swagger üzerinde de görünecektir.
```csharp
[Authorize(Roles = RoleNames.Admin + "," + RoleNames.Merchant + "," + RoleNames.ProductConfirmation)]
public class BrandsController : BaseApiController
{
    private readonly IBrandManager _brandManager;
    private readonly IProductManager _productManager;

    /// <summary>Initializes a new instance of the <see cref="BrandsController" /> class.</summary>
    /// <param name="brandManager">The brand manager.</param>
    /// <param name="productManager">The product manager.</param>
    public BrandsController(IBrandManager brandManager, IProductManager productManager)
    {
        _brandManager = brandManager;
        _productManager = productManager;
    }

    #region GET

    /// <summary>Gets brand list.</summary>
    [HttpGet]
    [ProducesResponseType((int)HttpStatusCode.OK, Type = typeof(ApiResponseObject<List<BrandQuery>, string>))]
    public IActionResult Get()
    {
        var brands = _brandManager.List<BrandQuery>();

        ...
        ...

        return Success("Markalar listelendi.", null, brands, "");
    }
    
    /// <summary>
    /// Get specific brand by id.
    /// </summary>
    /// <param name="id">Brand id.</param>
    [ServiceFilter(typeof(NotFoundMongoFilter<IBrandManager, Brand>))]
    [HttpGet("{id}")]
    [ProducesResponseType((int)HttpStatusCode.OK, Type = typeof(ApiResponseObject<BrandQuery, string>))]
    public IActionResult Get(string id)
    {
        var objectId = ObjectId.Parse(id);
        var brand = _brandManager.Find<BrandQuery>(objectId);
        return Success("Marka bulundu.", null, brand, id);
    }
...
...
...
```

Bazı metotlarda `ServiceFilter` olarak `NotFoundMongoFilter` gibi filter lar kullanılmıştır. Örnek olarak yazılmış bu filter da mongodb de ilgili id li kayıt varlığı kontrolü yapılır. Yoksa endpoint den `NotFound` dönmesi sağlanır.

`BaseApiController` sınıfı kullanılarak endpoint lerden dönecek tiplerin standart bir tipte response sınıfları olmasını sağlaycak şekilde `success, created, nocontent, badrequest, vs`  metotları oluşturulmuştur. Tüm bu metotlar `ApiResponseObject<TResult, TModel>` tipinde dönüş türü sağlar. `TResult` endpoint den geriye dönen tip türü. `TModel` ise endpoint e gönderilen model varsa onun türü. Yoksa `string` ya da `object` yapılabilir.

örneğin success metodu,
```csharp
...
...

protected IActionResult Success<TResult, TModel>(string message, string internalMessage, TResult result, TModel model)
{
    return Success(new ApiResponseObject<TResult, TModel>
    {
        Success = true,
        Message = message,
        InternalMessage = internalMessage,
        Result = result,
        Model = model
    });
}

...
...
```

[[Ana sayfaya dön!]](README.md)
