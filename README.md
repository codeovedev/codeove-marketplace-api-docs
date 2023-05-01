# Codeove Marketplace API Docs - OpenAPI 3.0

Marketplace API önemli noktalara ilişkin dokümantasyon içerikleri.

https://editor.swagger.io/ adresinden api projesinin swagger.json verisi ile swagger editor üzerinde api incelemesi yapabilirsiniz. Hatta json formatını yaml olarak güncelleyerek(`editor.swagger.io da bu işlem json dan yaml a convert şeklinde yapılıyor`) bu repo daki `marketplace-api.yaml` dosyasını güncellerseniz aşağıdaki link ile online editor üzerinden görüntüleme yapabilirsiniz.

**Güncelleme Tarihi :**
25.04.2023

[Open Online Swagger Editor - Marketplace API](https://editor.swagger.io?url=https://raw.githubusercontent.com/muratbaseren/codeove-marketplace-api-docs/master/marketplace-api.yaml)

## Sayfalar
[UI App Secret Key Code Tanım Düzenlemesi](page-ui-app-secret-key-code.md)<br>
[Veri Tabanı Oluşum ve Seed Data](create-database-seed-data.md)<br>
[Token Oluşturma](token-olusturma.md)<br>
[Alt Yapı Detayları](infrastructure-details.md)<br>
[Yeni Entity Oluşumu ve API ye Bağlama](new-entity-and-api-controller.md)<br>
[Kategori Tanımları](kategori-tanimlari.md)<br>
[Ürün Oluşturma](urun-olusturma.md)<br>
[Ürün Varyantları](urun-varyanti.md)<br>
[Temp Tablolar](temp-tablolar.md)<br>
[Komisyon Oranları](komisyon-oranlari.md)<br>
[Sipariş Süreçleri](siparis-surecleri.md)<br>
[Excel Dosyasından Ürün Import Etme](excel-import.md)<br>
[Zamanlaşmış Görevler (Hangfire)](zamanlasmis-gorevler.md)<br>

## Bazı Hata Gidermeleri
Elinizdeki API kodlarında aşağıdaki görsel de yer alan metotlar da, görsel de belirtilen şekilde değişiklikler yapılmıştır. **Bu güncellemelerin varlığını kontrol ediniz. UI tarafındaki olası hata çıkarma durumlarını kontrol ediniz.**

![fixes](/images/fixes.jpg)

## Swagger Üzerindeki Detaylar
Swagger üzerinde endpoint açıklamaları detaylandırıldı.
![swagger comments](/images/swagger-comments.jpg)

Swagger üzerinde post verisi için `schema` verilerinden zorunlu alan ve alan tipleri hakkında bilgi edinilebilir. Ayrıca her endpoint için `200(ok)(success)` response tipi detay işlendi. Başarılı işlem sonucunda dönen schema görülebilir.
![swagger details](/images/swagger-detail.jpg)

## Solution Yapısı
Projenin solution yapısı görseldeki gibidir. 

![Proje solution mimarisi](/images/solution-projects.jpg?raw=true)

- `Logging` klasöründe Loglama ile ilgili proje örnekleri yer alır. 
- `MFramework` klasöründe api altyapısında kullanılan base yapılar ve kütüphanelere ait projeler yer almaktadır. 
- `Services` klasörü altında ödeme sistemi olan(Iyzico) ve kargo sistemi olan (Ceva,MNG) entegrasyonlarına ait kodlar yer almaktadır.
- `Codeove.Marketplace.API` api projesidir.
- `Codeove.Marketplace.Business` manager(service) gibi yapıları içeren projedir.
- `Codeove.Marketplace.DataAccess` veri tabanı işlemleri için Repository Pattern, UnitOfWork gibi yapıları içeren projedir.
- `Codeove.Marketplace.Entities` veri tabanı varlık nesneleri gibi yapıları içeren projedir.
- `Codeove.Marketplace.Models` DTO ya da ViewModel gibi yapıları içeren projedir.
- `Codeove.Marketplace.Common` proje genelinde ortak kullanılan  yapıları içeren projedir.

## Projenin İşleyiş Mimarisi
Projedeki katmanlı yapılar belirli bir kullanım mantığına göre oluşturulmuştur.<br>
`Codeove.Marketplace.API > Codeove.Marketplace.Business > Codeove.Marketplace.DataAccess` ieltişimi üzerine kurgulanmıştır.<br><br>
Özetle, yapı temel olarak 3 katmanlı bir mimari üzerine kurgulanmış ve katmanların ortak kullandığı diğer projeler olarak geliştirilmiştir. Projelerin çokluluğu tamamen yapının SOLID temellerine uygun olması için projelere ayrılmıştır.

## Codeove.Marketplace.API Projesi 
`Codeove.Marketplace.Business` projesi ile etkileşime geçerek `*Manager` sınıfları ile işlemleri ilerletir. Aynı zamanda `Codeove.Marketplace.Entities` , `Codeove.Marketplace.Models` ve `Codeove.Marketplace.Common` projelerini de kullanır.

## Codeove.Marketplace.Business Projesi 
`Codeove.Marketplace.DataAccess` projesi ile etkileşime geçerek `*Repository` ve `UnitOfWork` sınıfları ile işlemleri ilerletir. Aynı zamanda `Codeove.Marketplace.Entities` , `Codeove.Marketplace.Models`, `Iyzipay` ve `Codeove.Marketplace.CargoOperations` projelerini de kullanır.

## Codeove.Marketplace.DataAccess Projesi 
**MongoClient** ya da **EntityFramework**(kullanılmadı ama hazır olarak bulunmaktadır) sınıfları ile işlemleri ilerletir. Aynı zamanda `Codeove.Marketplace.Entities` , `Codeove.Marketplace.Logger`, `Codeove.Services.Logger` ve `Codeove.Marketplace.Common` projelerini de kullanır. Sadece veri tabanı mantık nesneleri ile çalışarak veri tabanı işlemlerini gerçekleştirir. (İşlem yapılan veri tiplerinin ilgili DTO ya da ViewModel nesnelerine dönüşümü ile ilgilenmez.)
