## Temp Tablolar
Pazaryeri sitesinde ürünleri göstermek için satıcı satıcı gezip satışta olan ürünlerini tek tek bulup aynı ürünleri gruplayarak liste fiyatını hesaplayıp ekranda göstermek gerekiyor. Bu filtrelemeleri ürün listeleme aşamasında yapmak yerine iki tane temp tablo oluşturarak tüm bu verilerin bu iki tabloda merge edilerek saklanması sağlanmıştır. Web sitesinde ürün listeleme yaparken diğer tablolarda kayıt aramak yerine doğrudan bu tablolar üzerinden ürün listemele işlemleri yapılmaktadır. Bu tabloların verileri bir `scheduler service` aracılığı ile günde bir sefer tümü oluşturulmaktadır. **Bunun haricinden gün için bir sipariş geldiğinde veya herhangi bir sebeple ürün güncellemesi yapıldığında bu tablodaki ürün bilgileri update edilmektedir.**

`tempMerchantProductLists` : Satıcı bazında ürünlerin listesinin bulunduğu tablodur. Aynı ürün farklı satıcılarda varsa bu tabloda satıcı bazında çoklayarak tutulmaktadır.

`tempProductLists` : Tüm ürünler tekil olarak bu tabloda tutulmaktadır. En düşük satış fiyatına sahip satıcının bilgileri ile bu tabloda tekil olarak ürün kaydı bulunur.


[[Ana sayfaya dön!]](README.md)
