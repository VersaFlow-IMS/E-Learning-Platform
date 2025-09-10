# Backend Documentation V1 - .NET Core eLearning API

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Project Structure](#project-structure)
3. [Database Design](#database-design)
4. [API Endpoints](#api-endpoints)
5. [Authentication & Security](#authentication--security)
6. [Data Models](#data-models)
7. [Services & Business Logic](#services--business-logic)
8. [Configuration](#configuration)
9. [Testing Strategy](#testing-strategy)
10. [Deployment](#deployment)

## Architecture Overview

### Technology Stack
- **.NET 7.0** - Web API Framework
- **Entity Framework Core 7.0** - ORM
- **SQL Server** - Database
- **ASP.NET Core Identity** - Authentication
- **JWT Bearer Tokens** - Authorization
- **AutoMapper** - Object-to-Object Mapping
- **FluentValidation** - Input Validation
- **Serilog** - Logging

### Design Patterns
- **Repository Pattern** - Data access abstraction
- **Unit of Work** - Transaction management
- **CQRS** - Command Query Responsibility Segregation
- **Dependency Injection** - IoC container
- **Clean Architecture** - Layered approach

## Project Structure

```
eLearning.API/
├── Controllers/           # API Controllers
│   ├── AuthController.cs
│   ├── UsersController.cs
│   ├── ClassesController.cs
│   ├── StudentsController.cs
│   └── ContentController.cs
├── Services/              # Business Logic
│   ├── IAuthService.cs
│   ├── AuthService.cs
│   ├── IUserService.cs
│   ├── UserService.cs
│   ├── IClassService.cs
│   ├── ClassService.cs
│   └── IContentService.cs
├── Models/                # Data Models
│   ├── Entities/          # Database Entities
│   ├── DTOs/              # Data Transfer Objects
│   └── ViewModels/        # API Response Models
├── Data/                  # Database Context
│   ├── ApplicationDbContext.cs
│   ├── Repositories/      # Repository Pattern
│   └── Configurations/    # Entity Configurations
├── Infrastructure/        # Cross-cutting Concerns
│   ├── Extensions/
│   ├── Middleware/
│   └── Helpers/
└── Program.cs             # Application Entry Point
```

## Database Design

### Core Entities

#### Users Table
```sql
CREATE TABLE Users (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    Email NVARCHAR(256) NOT NULL UNIQUE,
    PasswordHash NVARCHAR(MAX) NOT NULL,
    Role NVARCHAR(20) NOT NULL, -- Admin, Teacher, Student, SupportDeveloper
    ProfileImageUrl NVARCHAR(500),
    TeacherLogoUrl NVARCHAR(500), -- Only for Teachers
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);
```

#### Classes Table
```sql
CREATE TABLE Classes (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Name NVARCHAR(100) NOT NULL,
    Description NVARCHAR(500),
    TeacherId UNIQUEIDENTIFIER NOT NULL,
    QRCode NVARCHAR(MAX), -- Generated QR code for attendance
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    FOREIGN KEY (TeacherId) REFERENCES Users(Id)
);
```

#### StudentClasses Table (Many-to-Many)
```sql
CREATE TABLE StudentClasses (
    StudentId UNIQUEIDENTIFIER NOT NULL,
    ClassId UNIQUEIDENTIFIER NOT NULL,
    EnrolledAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    PRIMARY KEY (StudentId, ClassId),
    FOREIGN KEY (StudentId) REFERENCES Users(Id),
    FOREIGN KEY (ClassId) REFERENCES Classes(Id)
);
```

#### Content Table
```sql
CREATE TABLE Content (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Title NVARCHAR(200) NOT NULL,
    Description NVARCHAR(1000),
    FileUrl NVARCHAR(500) NOT NULL,
    FileName NVARCHAR(255) NOT NULL,
    FileType NVARCHAR(50) NOT NULL, -- PDF, DOCX, Image, Video, etc.
    FileSize BIGINT NOT NULL,
    TeacherId UNIQUEIDENTIFIER NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    FOREIGN KEY (TeacherId) REFERENCES Users(Id)
);
```

#### ContentVisibility Table
```sql
CREATE TABLE ContentVisibility (
    ContentId UNIQUEIDENTIFIER NOT NULL,
    ClassId UNIQUEIDENTIFIER NOT NULL,
    PRIMARY KEY (ContentId, ClassId),
    FOREIGN KEY (ContentId) REFERENCES Content(Id),
    FOREIGN KEY (ClassId) REFERENCES Classes(Id)
);
```

#### TeacherContracts Table
```sql
CREATE TABLE TeacherContracts (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    TeacherId UNIQUEIDENTIFIER NOT NULL,
    StartDate DATE NOT NULL,
    EndDate DATE NOT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    FOREIGN KEY (TeacherId) REFERENCES Users(Id)
);
```

## API Endpoints

### Authentication Endpoints

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    // POST: api/auth/login
    [HttpPost("login")]
    public async Task<ActionResult<AuthResponseDto>> Login([FromBody] LoginDto model)

    // POST: api/auth/refresh-token
    [HttpPost("refresh-token")]
    public async Task<ActionResult<AuthResponseDto>> RefreshToken([FromBody] RefreshTokenDto model)

    // POST: api/auth/logout
    [HttpPost("logout")]
    [Authorize]
    public async Task<ActionResult> Logout()
}
```

### Users Management Endpoints

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class UsersController : ControllerBase
{
    // GET: api/users/profile
    [HttpGet("profile")]
    public async Task<ActionResult<UserProfileDto>> GetProfile()

    // PUT: api/users/profile
    [HttpPut("profile")]
    public async Task<ActionResult<UserProfileDto>> UpdateProfile([FromBody] UpdateProfileDto model)

    // POST: api/users/teachers (Admin only)
    [HttpPost("teachers")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult<TeacherDto>> CreateTeacher([FromBody] CreateTeacherDto model)

    // GET: api/users/teachers (Admin only)
    [HttpGet("teachers")]
    [Authorize(Roles = "Admin")]
    public async Task<ActionResult<PagedResult<TeacherDto>>> GetTeachers([FromQuery] PagedRequest request)
}
```

### Classes Management Endpoints

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ClassesController : ControllerBase
{
    // GET: api/classes
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ClassDto>>> GetClasses()

    // POST: api/classes (Teacher only)
    [HttpPost]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult<ClassDto>> CreateClass([FromBody] CreateClassDto model)

    // PUT: api/classes/{id} (Teacher only)
    [HttpPut("{id}")]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult<ClassDto>> UpdateClass(Guid id, [FromBody] UpdateClassDto model)

    // POST: api/classes/{id}/students (Teacher only)
    [HttpPost("{id}/students")]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult> AddStudent(Guid id, [FromBody] AddStudentToClassDto model)

    // DELETE: api/classes/{classId}/students/{studentId} (Teacher only)
    [HttpDelete("{classId}/students/{studentId}")]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult> RemoveStudent(Guid classId, Guid studentId)

    // GET: api/classes/{id}/qr-code (Teacher only)
    [HttpGet("{id}/qr-code")]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult<QRCodeDto>> GenerateQRCode(Guid id)
}
```

### Content Management Endpoints

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ContentController : ControllerBase
{
    // GET: api/content
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ContentDto>>> GetContent()

    // POST: api/content (Teacher only)
    [HttpPost]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult<ContentDto>> UploadContent([FromForm] UploadContentDto model)

    // PUT: api/content/{id}/visibility (Teacher only)
    [HttpPut("{id}/visibility")]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult> UpdateVisibility(Guid id, [FromBody] ContentVisibilityDto model)

    // DELETE: api/content/{id} (Teacher only)
    [HttpDelete("{id}")]
    [Authorize(Roles = "Teacher")]
    public async Task<ActionResult> DeleteContent(Guid id)
}
```

## Authentication & Security

### JWT Configuration

```csharp
// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });
```

### Role-based Authorization

```csharp
public class AuthService : IAuthService
{
    public async Task<AuthResponseDto> LoginAsync(LoginDto loginDto)
    {
        var user = await _userRepository.GetByEmailAsync(loginDto.Email);

        if (user == null || !VerifyPassword(loginDto.Password, user.PasswordHash))
            throw new UnauthorizedException("Invalid credentials");

        var token = GenerateJwtToken(user);
        var refreshToken = GenerateRefreshToken();

        // Store refresh token in database
        await _userRepository.UpdateRefreshTokenAsync(user.Id, refreshToken);

        return new AuthResponseDto
        {
            Token = token,
            RefreshToken = refreshToken,
            User = _mapper.Map<UserDto>(user)
        };
    }

    private string GenerateJwtToken(User user)
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role),
            new Claim("firstName", user.FirstName),
            new Claim("lastName", user.LastName)
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwtSettings.Key));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _jwtSettings.Issuer,
            audience: _jwtSettings.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwtSettings.ExpirationMinutes),
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

## Data Models

### DTOs (Data Transfer Objects)

```csharp
// Authentication DTOs
public class LoginDto
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [MinLength(6)]
    public string Password { get; set; }
}

public class AuthResponseDto
{
    public string Token { get; set; }
    public string RefreshToken { get; set; }
    public UserDto User { get; set; }
}

// User DTOs
public class UserDto
{
    public Guid Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public string Role { get; set; }
    public string ProfileImageUrl { get; set; }
    public string TeacherLogoUrl { get; set; }
}

public class CreateTeacherDto
{
    [Required]
    [MaxLength(50)]
    public string FirstName { get; set; }

    [Required]
    [MaxLength(50)]
    public string LastName { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [MinLength(6)]
    public string Password { get; set; }

    public DateTime ContractStartDate { get; set; }
    public DateTime ContractEndDate { get; set; }
}

// Class DTOs
public class ClassDto
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public Guid TeacherId { get; set; }
    public string TeacherName { get; set; }
    public int StudentCount { get; set; }
    public DateTime CreatedAt { get; set; }
}

public class CreateClassDto
{
    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    [MaxLength(500)]
    public string Description { get; set; }
}

// Content DTOs
public class ContentDto
{
    public Guid Id { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public string FileName { get; set; }
    public string FileType { get; set; }
    public long FileSize { get; set; }
    public string FileUrl { get; set; }
    public DateTime CreatedAt { get; set; }
    public List<Guid> VisibleToClasses { get; set; }
}
```

## Services & Business Logic

### User Service Implementation

```csharp
public class UserService : IUserService
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;
    private readonly ILogger<UserService> _logger;

    public UserService(IUnitOfWork unitOfWork, IMapper mapper, ILogger<UserService> logger)
    {
        _unitOfWork = unitOfWork;
        _mapper = mapper;
        _logger = logger;
    }

    public async Task<UserDto> GetProfileAsync(Guid userId)
    {
        var user = await _unitOfWork.UserRepository.GetByIdAsync(userId);
        if (user == null)
            throw new NotFoundException("User not found");

        return _mapper.Map<UserDto>(user);
    }

    public async Task<TeacherDto> CreateTeacherAsync(CreateTeacherDto createTeacherDto)
    {
        // Check if email already exists
        var existingUser = await _unitOfWork.UserRepository.GetByEmailAsync(createTeacherDto.Email);
        if (existingUser != null)
            throw new ConflictException("Email already exists");

        var teacher = new User
        {
            FirstName = createTeacherDto.FirstName,
            LastName = createTeacherDto.LastName,
            Email = createTeacherDto.Email,
            PasswordHash = HashPassword(createTeacherDto.Password),
            Role = UserRole.Teacher
        };

        await _unitOfWork.UserRepository.AddAsync(teacher);

        // Create contract
        var contract = new TeacherContract
        {
            TeacherId = teacher.Id,
            StartDate = createTeacherDto.ContractStartDate,
            EndDate = createTeacherDto.ContractEndDate
        };

        await _unitOfWork.ContractRepository.AddAsync(contract);
        await _unitOfWork.SaveChangesAsync();

        _logger.LogInformation("Teacher created with ID: {TeacherId}", teacher.Id);

        return _mapper.Map<TeacherDto>(teacher);
    }
}
```

### Class Service Implementation

```csharp
public class ClassService : IClassService
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;
    private readonly IQRCodeService _qrCodeService;

    public async Task<ClassDto> CreateClassAsync(Guid teacherId, CreateClassDto createClassDto)
    {
        var classEntity = new Class
        {
            Name = createClassDto.Name,
            Description = createClassDto.Description,
            TeacherId = teacherId
        };

        await _unitOfWork.ClassRepository.AddAsync(classEntity);
        await _unitOfWork.SaveChangesAsync();

        // Generate QR code for the class
        var qrCode = _qrCodeService.GenerateQRCode(classEntity.Id);
        classEntity.QRCode = qrCode;

        await _unitOfWork.SaveChangesAsync();

        return _mapper.Map<ClassDto>(classEntity);
    }

    public async Task AddStudentToClassAsync(Guid classId, Guid studentId, Guid teacherId)
    {
        // Verify teacher owns the class
        var classEntity = await _unitOfWork.ClassRepository.GetByIdAsync(classId);
        if (classEntity?.TeacherId != teacherId)
            throw new ForbiddenException("You can only add students to your own classes");

        // Check if student is already in the class
        var existingEnrollment = await _unitOfWork.StudentClassRepository
            .GetByStudentAndClassAsync(studentId, classId);

        if (existingEnrollment != null)
            throw new ConflictException("Student is already enrolled in this class");

        var enrollment = new StudentClass
        {
            StudentId = studentId,
            ClassId = classId
        };

        await _unitOfWork.StudentClassRepository.AddAsync(enrollment);
        await _unitOfWork.SaveChangesAsync();
    }
}
```

## Configuration

### appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=eLearningDB;Trusted_Connection=true;MultipleActiveResultSets=true",
    "TestConnection": "Server=(localdb)\\mssqllocaldb;Database=eLearningTestDB;Trusted_Connection=true;MultipleActiveResultSets=true"
  },
  "Jwt": {
    "Key": "your-super-secure-key-that-should-be-at-least-256-bits",
    "Issuer": "eLearning-API",
    "Audience": "eLearning-Client",
    "ExpirationMinutes": 60
  },
  "FileStorage": {
    "BasePath": "wwwroot/uploads",
    "MaxFileSize": 52428800,
    "AllowedExtensions": [".pdf", ".docx", ".doc", ".jpg", ".jpeg", ".png", ".mp4", ".avi"]
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

### Dependency Injection Setup

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Database
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Repositories and Unit of Work
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IClassRepository, ClassRepository>();
builder.Services.AddScoped<IContentRepository, ContentRepository>();

// Services
builder.Services.AddScoped<IAuthService, AuthService>();
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IClassService, ClassService>();
builder.Services.AddScoped<IContentService, ContentService>();
builder.Services.AddScoped<IQRCodeService, QRCodeService>();

// AutoMapper
builder.Services.AddAutoMapper(typeof(MappingProfile));

// FluentValidation
builder.Services.AddFluentValidationAutoValidation();
builder.Services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
```

## Testing Strategy

### Unit Testing Setup

```csharp
[TestClass]
public class UserServiceTests
{
    private Mock<IUnitOfWork> _mockUnitOfWork;
    private Mock<IMapper> _mockMapper;
    private Mock<ILogger<UserService>> _mockLogger;
    private UserService _userService;

    [TestInitialize]
    public void Setup()
    {
        _mockUnitOfWork = new Mock<IUnitOfWork>();
        _mockMapper = new Mock<IMapper>();
        _mockLogger = new Mock<ILogger<UserService>>();
        _userService = new UserService(_mockUnitOfWork.Object, _mockMapper.Object, _mockLogger.Object);
    }

    [TestMethod]
    public async Task GetProfileAsync_WithValidUserId_ReturnsUserDto()
    {
        // Arrange
        var userId = Guid.NewGuid();
        var user = new User { Id = userId, FirstName = "John", LastName = "Doe" };
        var userDto = new UserDto { Id = userId, FirstName = "John", LastName = "Doe" };

        _mockUnitOfWork.Setup(x => x.UserRepository.GetByIdAsync(userId))
            .ReturnsAsync(user);
        _mockMapper.Setup(x => x.Map<UserDto>(user))
            .Returns(userDto);

        // Act
        var result = await _userService.GetProfileAsync(userId);

        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(userId, result.Id);
        Assert.AreEqual("John", result.FirstName);
    }
}
```

## Deployment

### Environment Setup

```bash
# Development
dotnet run --environment Development

# Staging
dotnet run --environment Staging

# Production
dotnet run --environment Production
```

### Database Migration Commands

```bash
# Add new migration
dotnet ef migrations add InitialCreate

# Update database
dotnet ef database update

# Generate SQL script
dotnet ef migrations script
```

This backend documentation provides a comprehensive foundation for implementing the .NET Core API for the eLearning Management System V1, covering all essential aspects from architecture to deployment.
