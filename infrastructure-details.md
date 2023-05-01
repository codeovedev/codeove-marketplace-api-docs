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
using MFramework.Services.Entities.Abstract;
using MongoDB.Bson;

namespace Codeove.Marketplace.Entities.Brands
{
    public class Brand : EntityBase<ObjectId>
    {
        public string Name { get; set; }
        public string Slug { get; set; }
    }
}
```

## Models
Models projesinde entity ler için gerekli işlemlere ait sınıflar oluşturulmuştur. Bu projenin ayrı yapılma sebebi, marketplace in ileride müşteriler için API sunulması noktasında bu DLL in onlara sunulması ve sınıfları oluşturma işlemleri için ekstra çabaya girmemeleri içindir.

Bu proje de yer alan class lar tamamen işlemlere ait(örneğin, ürün insert, kategori insert ya da update vs) gerekli validation kurallarını da içeren şekilde DTO ya da ViewModel lerin oluşturulması esasına dayanır. Bu sınıfların isimlendirilmesinde tabii ki esneksiniz ama genel olarak şöyle bir standart oluşturulmuştur. Hepsinde entity adı **ön eki** kullanılır(`Brand...`). Veri listeleme ile alakalı sınıfları `...Query` **son eki** ile bitiririz. Diğer sınıflar için ilgili işlem ismi ile bitebilir. `...Create` ya da `...Update` gibi **Insert** ve **Update** işlemleri için gerekli modellerdir. Örnek olarak `Brand` entity ait işlemler için gerekli sınıflar.

```csharp
namespace Codeove.Marketplace.Models.Brands
{
    public class BrandCreate
    {
        public string Name { get; set; }
        public string Slug { get; set; }
    }
}
```

```csharp
using Codeove.Marketplace.Models.Abstract;

namespace Codeove.Marketplace.Models.Brands
{
    public class BrandQuery : ModelBase<string>
    {
        public string Name { get; set; }
        public string Slug { get; set; }
        public decimal ProductCount { get; set; }
    }
}

```

```csharp
namespace Codeove.Marketplace.Models.Brands
{
    public class BrandUpdate
    {
        public string Name { get; set; }
        public string Slug { get; set; }
    }
}
```

[[Ana sayfaya dön!]](README.md)
