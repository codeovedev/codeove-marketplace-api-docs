# Veri Tabanı Oluşum ve Seed Data
`Codeove.Marketplace.DataAccess` projesi içerisinde `startup.cs` içinde `CustomConfiguration` metodunda hem Entity framework(`DatabaseInitializer`) hem de mongodb için `MongoDatabaseInitializer` sınıflarında işlemler gerçekleitirilir. Hem migration hem de seed veri işlemi proje çalıştırıldığında uygulanır. Seed metodunda örnek verilerin ya da başlangıçta olması gereken verilerin veri tabanına işlenmesi gerekli durumlarda kullanılabilir bir metotdur.

## Entity Framework Kısmı
Proje genelinde EF kullanılmadığından `EntityFramework` için kodlar şimdilik açıklama satırı yapılabilir. Bunlar eğer EF nin desteklediği bir RDBMS kullanılmak istenirse açılabilir. Örnek olması için kodlar bırakılmıştır ve Album tipinde bir tablo oluşturarak, bu tablo içerisine seed veri işlenmesine ait kodlar bulunmaktadır.

Bu kodlar açıklama satırı yapılabilir. Projenin RDBMS üzerinde bir işlemi bulunmamaktadır.

> DİKKAT : Eğer EF için bir migration yaptıysanız ama henüz update-database komutunu çalıştırmadıysanız. Projeyi çalıştırdığınız da bu aşağıdaki kod otomatik olarak beklemede olan migration ları tespit edip çalıştıracak ve update-database komutu otomatik devreye girecektir. O sebeple hemen veri tabanı güncellenmesi(oluşturulan migration ların uygulanması) istenmiyorsa bu kısma dikkat edilmelidir.

```csharp
#region EntityFramework

var contextEF = serviceProvider.GetRequiredService<DatabaseContext>();

// varsa migration 
if (contextEF.Database.GetPendingMigrations().Count() > 0)
    contextEF.Database.Migrate();

DatabaseInitializer initializerEF = new DatabaseInitializer();
initializerEF.Seed(contextEF);

#endregion
```

## MongoDB Kısmı
MongoDB için bildiğiniz gibi bir migration yapısı aslında bulunmamaktadır. Bu yapı EF den kopya çekilerek kurgulanmıştır. Amaç; güncelleme yapılan koleksiyonlarda(tablolar) yapılan değişikliklerin var olan verilere uygulanması ve migrate edilmesi için düşünülmüştür. 

Burada örnek iki tane migration metodu bırakılmıştır. `Migrate_AlbumSummaryAdded` ve `Migrate_AlbumDescriptionRemove` metotları örnektir. Örneğin bir migration yapmak istediğiniz de bu metotlar gibi bir metot oluşturarak bu metot içinde istediğiniz bir mongodb güncellemesi yapabilirsiniz(örneğin, bir entity e yeni property eklediniz ya da sildiniz ve var olan verileri güncelleyerek bu property nin onlara da eklenmesi ya da silinmesi gibi). Bu metotlardan birinin içeriğini inceleyebilirsiniz.

```csharp
private void Migrate_AlbumSummaryAdded(MongoDBContext context)
{
    //todo : Summary prop Album tipine eklenir.

    IMongoAlbumRepository albumRepository = serviceProvider.GetRequiredService<IMongoAlbumRepository>();
    var list = albumRepository.List();

    list.ForEach(x =>
    {
        x.Summary = TextData.GetSentence();
        albumRepository.Update(x.Id, x);
    });
}
```

Migration için benzer metodunuzu oluşturduktan sonra mutlaka bu class içindeki public `Migrate` metoduna ilgili migration metodunun çalışması için satır eklenmelidir. Aşağıda metot içeriğinde örnek migration metotlarının satırları eklenmiştir. Bu metot proje her çalıştığında mongodb  de `_migrationsHistory` isimli koleksiyona burada yazılı migration metotlarını uygulayıp, uyguladığına dair işler. Aynı EF de olduğu gibi. Dolayısı ile aynı migration tekrar tekrar her migration da çalışmaz.

> DİKKAT : Yeni migration metotlarınız aşağıdaki metot içinde en sona eklenmelidir. Zamanla migration metotlarında kodlar değiştiğinden kodlama hataları oluşabilir. Örneğin silinen bir property artık olmadığından önceki bir migration da kullanıldı ise compile-time hatası olabilir. Bu durumda eski mig metotları silinebilir, bu bir sorun yaratmayacaktır.

```csharp
public void Migrate(MongoDBContext context)
{
    IMongoMigrationRepository mongoMigration = serviceProvider.GetRequiredService<IMongoMigrationRepository>();

    //Migrate(context, mongoMigration, Migrate_AlbumSummaryAdded);    // migrate Migrate_AlbumSummaryAdded
    //Migrate(context, mongoMigration, Migrate_AlbumDescriptionRemove);   // migrate Migrate_AlbumSummaryRemove
}
```

[[Ana sayfaya dön!]](README.md)
