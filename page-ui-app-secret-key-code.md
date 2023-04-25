# UI App Secret Key Code Tanım Düzenlemesi
Bu tanımlama bazı metotlarda UI uygulamalarından gelen isteklerde ekstra güvenlik olarak düşünülmüştür. API içerisinde hard-coded olarak yazılmıştır. Dilerseniz bunu appsettings.json dosyasından okunacak şekilde değiştirebilirsiniz. Böylesi daha doğru ve ideal olacaktır.

- `LogsController.cs > Post`
- `WebSiteOrdersController.cs > GetIyzicoSetting`
- `WebSiteOrdersController.cs > OrderUpdatePaymentId`
- `WebSiteOrdersController.cs > SetOrderPaymentFailed`
- `OrderManager.cs > OrderCreate`
- `OrderManager.cs > OrderSuccess`
