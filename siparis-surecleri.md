## Sipariş Süreçleri
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


[[Ana sayfaya dön!]](README.md)
