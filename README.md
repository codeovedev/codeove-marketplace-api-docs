# Codeove Marketplace API Docs - OpenAPI 3.0

Marketplace API önemli noktalara ilişkin dokümantasyon içerikleri.

https://editor.swagger.io/ adresinden api projesinin swagger.json verisi ile swagger editor üzerinde api incelemesi yapabilirsiniz. Hatta json formatını yaml olarak güncelleyerek(`editor.swagger.io da bu işlem json dan yaml a convert şeklinde yapılıyor`) bu repo daki `marketplace-api.yaml` dosyasını güncellerseniz aşağıdaki link ile online editor üzerinden görüntüleme yapabilirsiniz.

**Güncelleme Tarihi :**
25.04.2023

[Open Online Swagger Editor - Marketplace API](https://editor.swagger.io?url=https://raw.githubusercontent.com/muratbaseren/codeove-marketplace-api-docs/master/marketplace-api.yaml)

## Sayfalar
[UI App Secret Key Code Tanım Düzenlemesi](page-ui-app-secret-key-code.md)<br>
[Veri Tabanı Oluşum ve Seed Data](page-ui-app-secret-key-code.md)<br>
[Token Oluşturma](page-ui-app-secret-key-code.md)<br>
[Yapı Hakkında Önemli Detaylar](page-ui-app-secret-key-code.md)<br>
[Yeni Entity Oluşumu ve API ye Bağlama](page-ui-app-secret-key-code.md)<br>
[Kategori Tanımları](kategori-tanimlari.md)<br>
[Ürün Oluşturma](urun-olusturma.md)<br>
[Ürün Oluşturma](urun-varyanti.md)<br>

## Bazı Hata Gidermeleri
Elinizdeki API kodlarında aşağıdaki görsel de yer alan metotlar da, görsel de belirtilen şekilde değişiklikler yapılmıştır. **Bu güncellemelerin varlığını kontrol ediniz. UI tarafındaki olası hata çıkarma durumlarını kontrol ediniz.**

![fixes](/images/fixes.jpg)

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







## Temp Tablolar
Pazaryeri sitesinde ürünleri göstermek için satıcı satıcı gezip satışta olan ürünlerini tek tek bulup aynı ürünleri gruplayarak liste fiyatını hesaplayıp ekranda göstermek gerekiyor. Bu filtrelemeleri ürün listeleme aşamasında yapmak yerine iki tane temp tablo oluşturarak tüm bu verilerin bu iki tabloda merge edilerek saklanması sağlanmıştır. Web sitesinde ürün listeleme yaparken diğer tablolarda kayıt aramak yerine doğrudan bu tablolar üzerinden ürün listemele işlemleri yapılmaktadır. Bu tabloların verileri bir `scheduler service` aracılığı ile günde bir sefer tümü oluşturulmaktadır. **Bunun haricinden gün için bir sipariş geldiğinde veya herhangi bir sebeple ürün güncellemesi yapıldığında bu tablodaki ürün bilgileri update edilmektedir.**

`tempMerchantProductLists` : Satıcı bazında ürünlerin listesinin bulunduğu tablodur. Aynı ürün farklı satıcılarda varsa bu tabloda satıcı bazında çoklayarak tutulmaktadır.

`tempProductLists` : Tüm ürünler tekil olarak bu tabloda tutulmaktadır. En düşük satış fiyatına sahip satıcının bilgileri ile bu tabloda tekil olarak ürün kaydı bulunur.



## Komisyon Oranları
Komisyon oranları kategori özelinde tanımlanmaktadır. Kategoriye ait komisyon oranı ile ürün satışlarından pazaryeri kazançları oluşmaktadır.



## Sipariş İşlemleri
Pazaryeri sitesi üzerinden bir sipariş verildiğinde sırasıyla aşağıdaki süreçlerden geçmektedir;

`1.Adım` Sipariş kaydı oluşturulur ve iyzico üzerinde ödeme kaydı açılır. Iyzico dan gelen bir anahtar değer sayesinde sipariş ile iyzico ödeme kaydı ilişkilendirilir.

`2.Adım` Sipariş satın alınan satıcının onayına düşer. Onaylama işlemi yapılana kadar site üzerinden kullanıcı sipariş siparişi iptal ederek iyzico üzerinden istek atılarak ücret iadesini alabilir.

`3.Adım` Satıcı siparişi iptal edebilir veya onaylayabilir. İptal etmesi durumunda ücret iadesi için iyzico üzerinden istek atılır. Onay verildiği takdirde satın alan kullanıcı sipariş iptal etme işlemi yapamaz. Ancak ürün eline ulaştıktan sonra iade süreci başlatabilir.

`4.Adım` Satıcı tarafından onaylanan sipariş kargolanmayı bekler. Satıcı göndereceği ürünleri kargoya vermek için panel üzerinden paket oluşturarak paketlere özel barkodu olan bir çıktı üretir. Bu aşamada kargo entegrasyonu ile kargo firmasının veritabanına bilgiler aktarılır. Satıcı kargo firmasına gittiğinde görevli aldığı barkod okutarak teslimat adresi vb tüm bilgileri sistemlerinde görerek gönderiyi hızlıca kabul eder. 

`5.Adım` Belirli sürelerle çalışan bir servis sayesinde ürünün kargoya verilme durumu kargo entegrasyon servisleri aracılığı ile kontrol edilerek durum alanı güncellenir. Durum değişikliklerinde son kullanıcıya bildirimler gönderilir.

`6.Adım` Kargo firması paket teslim ettiğinde yine aynı entegrasyon servisi ile kontrol edilerek Teslim Edildi olarak güncellenir ve 15 günlük iade süreci başlar.

`7.Adım` Bu süre zarfı içerisinden son kullanıcı iade talebinde bulunabilir.

`8.Adım` İade talebinde bulunmak için sistemden bir kayıt açarak kendisine İade Kodu üretililir. Son kullanıcı kargo şubesine gittiğinde bu kod ile yine hiç bir adres bilgisi vermeden kargo entegrasyon servisleri ile satıcıya ulaşması sağlanır. Satıcının iadeyi onaylaması durumunda ücret iadesi iyzico servisleri ile yapılır ve sipariş sonuçlanmış olur. Satıcı iadeyi kabul etmez ise pazaryeri yönetimi ile sipariş itilaflı durumuna düşer ve pazaryeri yönetimi ile sipariş nihai sonuca ulaştırılır.

`9.Adım` Alıcı iade talebinden bulunmadığı takdirde 14 günlük iade süreci tamamlandıktan sonra, belirli aralıklarla çalışan bir servis ile siparişin ücretlerinin (satıcıya aktrılacak ücret ve pazaryeri komisyonu ücreti) gerekli hesaplara aktarılması için iyzico ya onaylama işlemi yapılır. Bu onaylamanın sonrasında iyzico ödeme prosedürlerine uygun olarak ödeler iyzico tarafından ilgili hesaplara aktarılır.


## Excel Dosyasından Ürün Import Etme

Satıcı ve admin panelinden import etmek istediğinin kategoriye özel örnek bir excel dosyası indirilir. Örnek dosyaya uygun olarak dosya hazırlanır. Exceldeki tüm kolonlar girilerek import işlemi yapılabilir. (ÜrünId/VaryanId alanına sayısal kendi db nizdeki ID yi yazabilirsiniz bu değer ObjectId olarak convert edilir)


## Zamanlaşmış Görevler

`CreateTempProductList.CreateList` Web sitesinde ürünleri hızlıca gösterebilmek adına otomatik dolduran tablodur. Yeni ürün ekleme, güncelleme işleminde de bu tabloya hızlıca veriler eklenmekte ve güncellenmektedir. (Tablo içerikleri Temp Tablolar başlığı altında detaylı olarak anlatılmıştır)
  
`CreateTempMerchantProductList.CreateList` Diğer tablonun aynı amaçla olan ve Satıcı bilgilerinin de içinde bulunduğu hali diyebiliriz. Yeni ürün ekleme, güncelleme işleminde de bu tabloya hızlıca veriler eklenmekte ve güncellenmektedir. (Tablo içerikleri Temp Tablolar başlığı altında detaylı olarak anlatılmıştır)
  
`SendAlarmMail.Send` Gelince Haber Ver, Fiyat Alarm Listesi gibi müşterilerin kurmuş olduğu alarmları kontrol edip tetikleyerek ilgili mail gönderimlerin yapılmasını sağlamaktadır.
  
`SitemapGenerate.Generate` Site map dosyasını yeniden üretir.
  
`SitemapGenerate.GenerateCimriXML` Cimri uygulamasının istediği formatta xml dosyasını yeniden üretir.
  
`MngKargoUpdateOrderDelivered.UpdateStatus` İsmi mng ancak entegrasyon olan tüm kargo firmaları için çalışmaktadır. Kargoya verildiğini, teslim edildiğini anlayıp sipariş durumlarını günceller.
  
`IyzicoCreateApprovalRequest.CreateApprovalReques` Sipariş tarihlerine uygun olacak şekilde iyzico ödemelerinin satıcılara ödenmesi için onay verme işlemini iyzico ya bildirir.
  
`ProductExcelExport.Process` azure blob storage üzerine "products-{DateTime.Now.ToShortDateString()}.xlsx" ismiyle tüm ürünlerin excel dosyasını oluşturur. Geçici bir kontrol için eklenmişti sistem işlleyişi ile bir ilgisi yok.
  
`IsNewProductStatusChange.CheckProduct` Ürün ekleme tarihi 3 günü geçen ve yeni ürün olarak işaretlenen ürünlerin yeni ürün etiketini kaldırma işlemini yapar.
