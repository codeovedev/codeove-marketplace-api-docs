## Ürün Oluşturma

Satıcı ilk ürün oluşturduğunda öncelikli olarak ürün kaydı pazar yeri yöneticisine yeni ürün önerisi olarak `ProductCommits` tablosuna kayıt atılır. Sonrasında yönetici bu kaydı kontrol eder eğer yeni bir olarak yada bir ürünün varyantı olarak sisteme ürün olarak ekler. Ekledikten sonra `ProductCommits` tablosundaki kayıt silinir ve `Products` tablosuna insert edilir. Artık öneri değil bir ürün haline gelir ve ekleyen satıcı da dahil olmak üzere tüm satıcılar o ürünü satışa açabilir duruma gelir. Ürün onaylanana kadar satıcı ile yönetici ürün özelinde mesajlaşarak ürünün güncel halini birlikte oluştururlar.

[[Ana sayfaya dön!]](README.md)
