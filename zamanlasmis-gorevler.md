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


[[Ana sayfaya dön!]](README.md)
