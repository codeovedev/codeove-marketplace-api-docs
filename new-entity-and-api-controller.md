# Yeni Entity Oluşumu ve API ye Bağlama

## Marketplace.Entities
Veri tabanı için entities sınıfları oluşturulur.

```csharp
public class Student : EntityBase<ObjectId>
{
    public string Name { get; set; }
    public string Surname { get; set; }
    public int Number { get; set; }
}
```

## Marketplace.Models
DTO veya ViewModel sınıfları oluşturulur.

```csharp
// Verileri listelerken kullanılacak olan sınıf. 
// Fazladan Code isimli bir prop ile dönen bir kod alanına sahip. 
// Bunun gibi alanlar içerebilir/içermeyebilir de.
public class StudentQuery : ModelBase<ObjectId>
{
    public string Name { get; set; }
    public string Surname { get; set; }
    public int Number { get; set; }
    public string Code
    {
        get
        {
            return $"{Name}-{Surname}:{Number}";
        }
    }
}

// Insert işlemi için kullanılacak olan model.
public class StudentCreate
{
    public string Name { get; set; }
    public string Surname { get; set; }
}

// Update işlemi için kullanılacak olan model.
public class StudentUpdate
{
    public string Name { get; set; }
    public string Surname { get; set; }
    public int Number { get; set; }
}
```

## Marketplace.DataAccess
Repository sınıfı oluşturulur. `MongoStudentRepository.cs` olarak isimlendirildi. Veri tabanı işlemi yapıldığı an da otomatik koleksiyon oluşacaktır.

```csharp
public interface IMongoStudentRepository : IMyMongoRepository<Student, ObjectId>
{

}

[Collection("students")]
public class MongoStudentRepository : MyMongoRepository<Student, ObjectId>, IMongoStudentRepository
{
    public MongoStudentRepository(MongoDBContext mongoDbContext) : base(mongoDbContext)
    {
    }
}
```

Ardından bu proje de `Startup.cs` sınıfında bağımlılık ayarları yapılır.
```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddScoped<IMongoStudentRepository, MongoStudentRepository>();
    ...
}
```

## Marketplace.Business
Manager sınıfı yapılandırması. Ekstra `GenerateSchoolNumber` metodu eklenmiştir. Olmasa da olabilir. Varsa yeni metotlarınız eklenmelidir.

```csharp
public interface IStudentManager : IMyMongoManager<Student, ObjectId>
{
    int GenerateSchoolNumber(int min, int max);
}

public class StudentManager : MyMongoManager<Student, ObjectId, IMongoStudentRepository>, IStudentManager
{
    public StudentManager(IMongoStudentRepository repository, IMapper mapper) : base(repository, mapper)
    {
    }

    public int GenerateSchoolNumber(int min, int max)
    {
        return Random.Shared.Next(min, max);
    }
}
```

Ardından bu proje de `Startup.cs` sınıfında bağımlılık ayarları yapılır.
```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddAutoMapper(System.Reflection.Assembly.GetExecutingAssembly());
            ...
            services.AddScoped<IStudentManager, StudentManager>();
            ...
        }
```

`AutoMapperConfig.cs` içine ViewModel ya da DTO ile Entity çevrimi için tanımlamalar yazılır.
```csharp
public class AutoMapperConfig : Profile
{
    public AutoMapperConfig()
    {
        ...
        StudentsMappings();
        ...
    }
    
    ...
    ...
    
    private void StudentsMappings()
    {
        CreateMap<Student, StudentCreate>().ReverseMap();
        CreateMap<Student, StudentQuery>().ReverseMap();
        CreateMap<Student, StudentUpdate>().ReverseMap();
    }
}
```

## Marketplace.API
Controller sınıfı yazılır. Gereken endpoint ler eklenir. Yetkilendirme türüne göre `Authorize` ve `Role` tanımları yapılır. Get, Get(id), Post, Put, Delete metotları eklenmiştir. Daha fazla endpoint ihtiyaca bağlı olarak eklenebilir.
```csharp
[Authorize(Roles = RoleNames.Admin + "," + RoleNames.Merchant)]
public class StudentsController : BaseApiController
{
    private readonly IStudentManager _studentManager;
    private readonly IMapper _mapper;

    /// <summary>
    /// Initializes a new instance of the <see cref="StudentsController"/> class.
    /// </summary>
    /// <param name="studentManager">The student manager.</param>
    /// <param name="mapper">The mapper(optional).</param>
    public StudentsController(IStudentManager studentManager, IMapper mapper)
    {
        _studentManager = studentManager;
        _mapper = mapper;
    }


    /// <summary>
    /// Tüm öğrencileri getirir.
    /// </summary>
    [HttpGet]
    [AllowAnonymous]
    [ProducesResponseType((int)HttpStatusCode.OK, Type = typeof(ApiResponseObject<List<StudentQuery>, string>))]
    public IActionResult Get()
    {
        var list = _studentManager.Query().ToList();

        return Success("Tüm öğrenciler listelendi", "Tüm öğrenciler listelendi",
            _mapper.Map<List<StudentQuery>>(list), "");
    }


    /// <summary>
    /// Id'si verilen öğrenci getirilir.
    /// </summary>
    /// <param name="id">Student id.</param>
    [HttpGet("{id}")]
    [ServiceFilter(typeof(NotFoundMongoFilter<IStudentManager, Student>))]
    [ProducesResponseType((int)HttpStatusCode.OK, Type = typeof(ApiResponseObject<StudentQuery, string>))]
    public IActionResult Get(string id)
    {
        var sectorId = id.ToObjectId();
        var sector = _studentManager.Find<StudentQuery>(sectorId);

        return Success("Öğrenci getirildi", "Öğrenci getirildi", sector, id);
    }


    /// <summary>
    /// Öğrenci oluşturulur.
    /// </summary>
    /// <param name="model">The model.</param>
    [HttpPost]
    [ValidateModel]
    [ProducesResponseType((int)HttpStatusCode.OK, Type = typeof(ApiResponseObject<StudentQuery, StudentCreate>))]
    public IActionResult Post([FromBody] StudentCreate model)
    {
        var created = _studentManager.Create(model);

        if (created != null)
        {
            return Success("Öğrenci oluşturuldu", "Öğrenci oluşturuldu", _mapper.Map<StudentQuery>(created), model);
        }

        return Error("Öğrenci oluşturulamadı", "Öğrenci oluşturulamadı", "", model);
    }


    /// <summary>
    /// Öğrenci güncellenir
    /// </summary>
    /// <param name="id">Student id</param>
    /// <param name="model">The model.</param>
    [HttpPut("{id}")]
    [ServiceFilter(typeof(NotFoundMongoFilter<IStudentManager, Student>))]
    [ValidateModel]
    [ProducesResponseType((int)HttpStatusCode.OK, Type = typeof(ApiResponseObject<StudentQuery, StudentUpdate>))]
    public IActionResult Put(string id, [FromBody] StudentUpdate model)
    {
        var studentid = id.ToObjectId();
        var updated = _studentManager.Update<StudentUpdate>(studentid, model);

        if (updated > 0)
        {
            return Success("Öğrenci güncellendi", "Öğrenci güncellendi", _studentManager.Find<StudentQuery>(studentid), model);
        }

        return Error("Öğrenci güncellenemedi", "Öğrenci güncellenemedi", "", model);
    }


    /// <summary>Öğrenciyi siler.</summary>
    /// <param name="id">Student id.</param>
    [ServiceFilter(typeof(NotFoundMongoFilter<IStudentManager, Student>))]
    [HttpDelete("{id}")]
    [ProducesResponseType((int)HttpStatusCode.OK, Type = typeof(ApiResponseObject<string, string>))]
    public IActionResult Delete(string id)
    {
        var studentid = id.ToObjectId();
        var deleted = _studentManager.Delete(studentid);
        
        if (deleted > 0)
        {
            return Success("Öğrenci silindi.", "Öğrenci silindi.", "", id);
        }

        return Error("Öğrenci silinemedi.", "Öğrenci silinemedi.", "", id);
    }
}
```

[[Ana sayfaya dön!]](README.md)
