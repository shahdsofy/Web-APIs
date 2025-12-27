# 🌐 ASP.NET Core Web API (.NET 6) 


---

## 1️⃣ Introduction

**ما هي Web API؟**
Web API هي واجهة بتسمح للتطبيقات المختلفة إنها تتواصل مع بعض عن طريق HTTP.

* تعتمد على HTTP Methods (GET, POST, PUT, DELETE)
* بترجع بيانات غالبًا بصيغة JSON
* تُستخدم مع Frontend (Angular, React, Mobile Apps)

🎯 الهدف: فهم فكرة الـ API ودورها في أي سيستم حديث.

---

## 2️⃣ Install Needed Tools

الأدوات المطلوبة للعمل على ASP.NET Core Web API:

* Visual Studio 2022
* .NET 6 SDK
* SQL Server
* Postman (اختياري للاختبار)

🎯 الهدف: تجهيز بيئة العمل قبل كتابة أي كود.

---

## 3️⃣ Create Our API and Go Through its Structure

هنا بننشئ مشروع Web API جديد ونتعرف على Structure المشروع.

### أهم الملفات:

* `Program.cs` → نقطة تشغيل التطبيق
* `Controllers` → Endpoints
* `appsettings.json` → الإعدادات

🎯 الهدف: فهم شكل المشروع ومسؤولية كل جزء.

---

## 4️⃣ Swagger Overview

**Swagger** هو أداة لتوثيق واختبار الـ API.

* يعرض كل Endpoints
* يسمح بتجربتها مباشرة
* يولد Documentation تلقائي

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme()
    {
        Type = SecuritySchemeType.ApiKey,
        Name = "api_key",
        Description = "API Key Authentication",
        BearerFormat = "JWT",
        Scheme = "Bearer",
        In = ParameterLocation.Header
    });
});
```
🎯 الهدف: اختبار الـ API بدون أدوات خارجية.

---

## 5️⃣ Add Options / Authorization Token With Swagger

هنا بنضيف دعم **Authorization** داخل Swagger.

* إضافة Bearer Token
* إرسال التوكن مع كل Request

🎯 الهدف: تجربة Endpoints المحمية من Swagger.

---

## 6️⃣ Enable CORS

CORS بتحدد مين مسموح له يستهلك الـ API.
Enabled by default.

### مثال:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowAll",
        builder => builder.AllowAnyOrigin()
                          .AllowAnyMethod()
                          .AllowAnyHeader());
});
```

🎯 الهدف: السماح لتطبيقات Frontend بالتواصل مع الـ API.

---

## 7️⃣ Add Database Model

بنبدأ نضيف Models اللي هتمثل جداول قاعدة البيانات.

* كل Class = Table
* كل Property = Column

🎯 الهدف: تجهيز الـ Data Layer.

---

## 8️⃣ Add Genres Model (Table)

إضافة جدول **Genres** كمثال عملي.

```csharp
public class Genre
{
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int Id { get; set; }
    [MaxLength(100)]
    public string Name { get; set; }
}
```

🎯 الهدف: تطبيق عملي على إنشاء Model.

---

## 9️⃣ Add GET Genres Endpoint

Endpoint لإرجاع كل الـ Genres.

```csharp
[HttpGet]
public IActionResult GetAll()
{
    return Ok(_context.Genres.ToList());
}
```

🎯 الهدف: فهم HTTP GET وإرجاع البيانات.

---

## 🔟 Add POST (Create) Genres Endpoint

Endpoint لإضافة Genre جديد.

```csharp
[HttpPost]
public IActionResult Create(Genre genre)
{
    _context.Genres.Add(genre);
    _context.SaveChanges();
    return Ok(genre);
}
```

🎯 الهدف: فهم HTTP POST وإنشاء بيانات جديدة.

---


## 1️⃣1️⃣ Add PUT (Update) Genres Endpoint

Endpoint لتحديث Genre موجود باستخدام HTTP PUT.

```csharp
[HttpPut("{id}")]
public IActionResult Update(int id, Genre genre)
{
    var existing = _context.Genres.Find(id);
    if (existing == null) return NotFound();

    existing.Name = genre.Name;
    _context.SaveChanges();
    return Ok(existing);
}
```

🎯 الهدف: فهم Update operations والتحقق من وجود البيانات.

---

## 1️⃣2️⃣ Add DELETE Genres Endpoint

Endpoint لحذف Genre.

```csharp
[HttpDelete("{id}")]
public IActionResult Delete(int id)
{
    var genre = _context.Genres.Find(id);
    if (genre == null) return NotFound();

    _context.Genres.Remove(genre);
    _context.SaveChanges();
    return NoContent();
}
```

🎯 الهدف: تطبيق عملية الحذف بشكل آمن.

---

## 1️⃣3️⃣ Add Movies Model (Table)

إضافة جدول Movies وربطه بالـ Genres.

```csharp
public class Movie
{
     public int Id { get; set; }
     [MaxLength(250)]
     public  string Title { get; set; }
     public int Year { get; set; }
     public double Rate { get; set; }
     [MaxLength(2500)]
     public  string StoreLine { get; set; }
     public byte[] Poster { get; set; }
     public byte GenreId { get; set; }
     public Genre Genre { get; set; }
}
```

🎯 الهدف: إنشاء Model بعلاقة One-To-Many.

---

## 1️⃣4️⃣ Add [POST] Create Movie Endpoint – Part 1

بداية إنشاء Endpoint لإضافة Movie.

* استقبال البيانات
* التحقق من GenreId
```csharp
public class MoviesController : ControllerBase
{
    private readonly ApplicationDbContext _dbContext;

    private new List<string> _allowedExtensions=new List<string>{".jpg",".png"};
    private const long _maxAllowedPosterSize=1048576;

    public MoviesController(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    [HttpPost]
    public async Task<IActionResult> CreateMovie([FromForm] MovieDTO dto)
    {
        if (!_allowedExtensions.Contains(Path.GetExtension(dto.Poster.FileName).ToLower()))
        {
            return BadRequest("Only .jpg and .png files are allowed.");
        }
        if (dto.Poster.Length > _maxAllowedPosterSize)
        {
            return BadRequest("Poster size exceeds the maximum allowed size of 1 MB.");
        }

        using var memoryStream = new MemoryStream();
        await dto.Poster.CopyToAsync(memoryStream);

        if(!_dbContext.Genres.Any(d=>d.Id==dto.GenreId))
            return BadRequest("Invalid GenreId.");

        var movie = new Movie
        {
            Title = dto.Title,
            Year = dto.Year,
            Rate = dto.Rate,
            StoreLine = dto.StoreLine,
            Poster = memoryStream.ToArray(),
            GenreId = dto.GenreId
        };

        await _dbContext.Movies.AddAsync(movie);
        await _dbContext.SaveChangesAsync();
        return Ok(movie);
    }
}

```
🎯 الهدف: بناء Create Endpoint بشكل صحيح

.

---

## 1️⃣5️⃣ Add [POST] Create Movie Endpoint – Part 2

استكمال إنشاء Movie:

* حفظ البيانات
* إرجاع Response مناسب

```csharp
 await _dbContext.Movies.AddAsync(movie);
 await _dbContext.SaveChangesAsync();
```

🎯 الهدف: فصل المنطق والتحقق قبل الحفظ.

---

## 1️⃣6️⃣ Add [GET] All Movies Endpoint

Endpoint لإرجاع كل الأفلام.

```csharp
[HttpGet]
public async Task<IActionResult> GetAllMovies()
{
    var movies = await _dbContext.Movies.Include(m => m.Genre)
        .Select(m=> new MovieDetailsDTO
        {
            Id = m.Id,
            Title = m.Title,
            Year = m.Year,
            Rate = m.Rate,
            StoreLine = m.StoreLine,
            Poster = m.Poster,
            GenreId = m.GenreId,
            Genre = m.Genre.Name
        }).ToListAsync();

    return Ok(movies);
}
```

🎯 الهدف: جلب البيانات مع العلاقات.

---

## 1️⃣7️⃣ Add [GET] Movie By ID Endpoint

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var movie = _context.Movies.Find(id);
    return movie == null ? NotFound() : Ok(movie);
}
```

🎯 الهدف: جلب عنصر واحد باستخدام ID.

---

## 1️⃣8️⃣ Add [GET] Movies By GenreId Endpoint

```csharp
[HttpGet("ByGenre/{genreId}")]
public IActionResult GetByGenre(int genreId)
{
    return Ok(_context.Movies.Where(m => m.GenreId == genreId));
}
```

🎯 الهدف: Filtering باستخدام Route Parameter.

---

## 1️⃣9️⃣ Add [DELETE] Movie Endpoint

```csharp
[HttpDelete("{id}")]
public IActionResult DeleteMovie(int id)
{
    var movie = _context.Movies.Find(id);
    if (movie == null) return NotFound();

    _context.Movies.Remove(movie);
    _context.SaveChanges();
    return NoContent();
}
```

---

## 2️⃣0️⃣ Add [PUT] (Update) Movie Endpoint

```csharp
[HttpPut("{id}")]
public IActionResult UpdateMovie(int id, Movie movie)
{
    var existing = _context.Movies.Find(id);
    if (existing == null) return NotFound();

    existing.Title = movie.Title;
    existing.GenreId = movie.GenreId;
    _context.SaveChanges();
    return Ok(existing);
}
```

🎯 الهدف: تحديث بيانات Movie.

---

## 2️⃣1️⃣ Add Genres Service

نبدأ نفصل Business Logic في Services.

* إنشاء Interface
* تنفيذ Service
* تسجيله باستخدام Dependency Injection

```csharp
public class GenresService : IGenresService
    {
        private readonly ApplicationDbContext _context;

        public GenresService(ApplicationDbContext context)
        {
            _context = context;
        }

        public async Task<Genre> Add(Genre genre)
        {
            await _context.AddAsync(genre);
            _context.SaveChanges();

            return genre;
        }

        public Genre Delete(Genre genre)
        {
            _context.Remove(genre);
            _context.SaveChanges();

            return genre;
        }

        public async Task<IEnumerable<Genre>> GetAll()
        {
            return await _context.Genres.OrderBy(g => g.Name).ToListAsync();
        }

        public async Task<Genre> GetById(byte id)
        {
            return await _context.Genres.SingleOrDefaultAsync(g => g.Id == id);
        }

        public Task<bool> IsvalidGenre(byte id)
        {
            return _context.Genres.AnyAsync(g => g.Id == id);
        }

        public Genre Update(Genre genre)
        {
            _context.Update(genre);
            _context.SaveChanges();

            return genre;
        }
    }
```

🎯 الهدف: Clean Architecture.

---

## 2️⃣2️⃣ Add Movies Service – Part 1

نقل منطق Movies من Controller إلى Service.

```csharp
public class MoviesService : IMoviesService
    {
        private readonly ApplicationDbContext _context;

        public MoviesService(ApplicationDbContext context)
        {
            _context = context;
        }


        public async Task<Movie> Add(Movie movie)
        {
            await _context.AddAsync(movie);
            _context.SaveChanges();

            return movie;
        }

        public Movie Delete(Movie movie)
        {
            _context.Remove(movie);
            _context.SaveChanges();

            return movie;
        }

        public async Task<IEnumerable<Movie>> GetAll(byte genreId = 0)
        {
            return await _context.Movies
                .Where(m => m.GenreId == genreId || genreId == 0)
                .OrderByDescending(m => m.Rate)
                .Include(m => m.Genre)
                .ToListAsync();
        }

        public async Task<Movie> GetById(int id)
        {
            return await _context.Movies.Include(m => m.Genre).SingleOrDefaultAsync(m => m.Id == id);
        }

        public Movie Update(Movie movie)
        {
            _context.Update(movie);
            _context.SaveChanges();

            return movie;
        }
    }
```
🎯 الهدف: تقليل حجم Controllers.

---

## 2️⃣3️⃣ Add Movies Service – Part 2

استكمال تطبيق Service:

* CRUD Operations
* Validation

🎯 الهدف: تنظيم الكود وإعادة استخدامه.

---

## 2️⃣4️⃣ Use AutoMapper

AutoMapper يُستخدم للتحويل بين Entities و DTOs.

```csharp
CreateMap<Movie, MovieDto>();
```

🎯 الهدف: فصل الـ API Contract عن الـ Database Models.

---

