# Token Oluşturma
Token oluşturma örneği için `UserManager.cs` içinde `Login` metodunda ilgili işlemi bulabilirsiniz. Bu metot ile token oluşturma işlemi gerçekleştirilerek `users` koleksiyonunda ilgili kullanıcı kaydında ayrıca saklanmaktadır. Token üretme işlemi standart net core bearer token üretim kodudur. Kullanıcıya ait roller de claim olarak işlenmektedir. `SymmetricSecurityKey` değeri ise `AppSettings.json` da `Authentication:JwtKey` altında bulunmaktadır. Base64 olarak encoded olarak konulmuştur. Decode ederek değeri görebilir ve değiştirebilirsiniz ama Base64 olarak oluşturmayı unutmayınız.

Authorize yapısı NetCore un kendi base yapısı olarak kullanılmıştır. Role yapısı kullanılarak metotlara yetki kısıtlaması eklenmiştir. Kullanıcı rolüne göre ilgili metotları çalıştırabilmektedir.

```csharp
public UserQuery Login(string email, string password)
{
    var user = repository.Queryable().FirstOrDefault(x => x.Email == email || x.Email == email.ToUpper() && x.Active);

    if (user == null)
    {
        _Clogger.Error($"Email:{email}", "Login Failed", title: "Admin Login", desc: $"Email:{email}");
        return null;
    }

    PasswordHasher<IdentityUser> passwordHasher = new PasswordHasher<IdentityUser>();

    IdentityUser identityUser = new IdentityUser(email);
    PasswordVerificationResult verificationResult =
        passwordHasher.VerifyHashedPassword(identityUser, user.Password, password);

    if (verificationResult == PasswordVerificationResult.Failed)
    {
        _Clogger.Error($"Email:{email}", "Login Failed", title: "Admin Login", desc: $"Email:{email}");
        return null;
    }

    List<Claim> claims = new List<Claim> {
                new Claim(ClaimNames.Id,user.Id.ToString()),
                new Claim(ClaimNames.Email,user.Email),
                new Claim(ClaimNames.Name,user.Name ?? string.Empty)
            };

    claims.AddRange(user.Roles.Select(ur => new Claim(ClaimTypes.Role, ur)));

    var key = new SymmetricSecurityKey(Convert.FromBase64String(_configuration["Authentication:JwtKey"]));

    var token = new JwtSecurityToken(
        claims: claims,
        expires: DateTime.UtcNow.AddMonths(1),
        signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256)
    );
    var generatedToken = new JwtSecurityTokenHandler().WriteToken(token);

    user.Token = generatedToken;
    Update<dynamic>(user.Id, new { Token = generatedToken }.ToExpando());

    _Clogger.Info($"Email:{email}", "Login", title: "Admin Login", desc: $"Email:{email}");

    return mapper.Map<UserQuery>(user);
}
```


[[Ana sayfaya dön!]](README.md)
