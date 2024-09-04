# SSC-Internship-Dev-Dapper
# Hướng Dẫn Về Dapper trong .NET Core
## 1. Giới thiệu về Dapper
Dapper là một micro ORM (Object-Relational Mapper) cho .NET, được phát triển bởi Stack Overflow. Dapper giúp bạn làm việc với cơ sở dữ liệu bằng cách cung cấp một cách dễ dàng và hiệu quả để thực hiện các truy vấn SQL thuần túy. Khác với Entity Framework Core, Dapper không ẩn đi các truy vấn SQL phức tạp; thay vào đó, nó tập trung vào hiệu suất cao và truy vấn trực tiếp, điều này làm cho Dapper trở nên lý tưởng khi bạn cần kiểm soát hoàn toàn các truy vấn SQL của mình.

Ưu điểm của Dapper:

- Hiệu suất cao: Dapper được biết đến là một trong những micro ORM nhanh nhất cho .NET.
- Dễ sử dụng: Dapper rất dễ tích hợp và sử dụng, với cú pháp đơn giản.
- Kiểm soát SQL: Bạn có quyền kiểm soát hoàn toàn các truy vấn SQL của mình.
- Tương thích với nhiều cơ sở dữ liệu: Dapper hoạt động tốt với nhiều hệ quản trị cơ sở dữ liệu khác nhau như SQL Server, MySQL, PostgreSQL, và SQLite.
## 2. Cài đặt Dapper
Để sử dụng Dapper trong một ứng dụng .NET Core, bạn cần cài đặt package Dapper. Bạn có thể cài đặt nó qua NuGet Package Manager hoặc sử dụng lệnh sau trong .NET CLI:

```bash
dotnet add package Dapper
```
## 3. Thiết lập kết nối cơ sở dữ liệu
Dapper hoạt động trên các kết nối cơ sở dữ liệu ADO.NET. Vì vậy, bước đầu tiên là thiết lập một kết nối cơ sở dữ liệu.
## 4. Sử dụng Dapper để thực hiện CRUD Operations
Dapper cho phép bạn thực hiện các thao tác CRUD (Create, Read, Update, Delete) một cách dễ dàng với cơ sở dữ liệu.

### 4.1. Model
Đầu tiên, tạo một model đại diện cho bảng trong cơ sở dữ liệu:

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public string Grade { get; set; }
}
```
### 4.2. Repository
Tạo một repository để thực hiện các thao tác CRUD:

```csharp
public interface IStudentRepository
{
    Task<IEnumerable<Student>> GetAllStudentsAsync();
    Task<Student> GetStudentByIdAsync(int id);
    Task<int> AddStudentAsync(Student student);
    Task<int> UpdateStudentAsync(Student student);
    Task<int> DeleteStudentAsync(int id);
}

public class StudentRepository : IStudentRepository
{
    private readonly IDbConnection _dbConnection;

    public StudentRepository(IDbConnection dbConnection)
    {
        _dbConnection = dbConnection;
    }

    public async Task<IEnumerable<Student>> GetAllStudentsAsync()
    {
        var sql = "SELECT * FROM Students";
        return await _dbConnection.QueryAsync<Student>(sql);
    }

    public async Task<Student> GetStudentByIdAsync(int id)
    {
        var sql = "SELECT * FROM Students WHERE Id = @Id";
        return await _dbConnection.QueryFirstOrDefaultAsync<Student>(sql, new { Id = id });
    }

    public async Task<int> AddStudentAsync(Student student)
    {
        var sql = "INSERT INTO Students (Name, Age, Grade) VALUES (@Name, @Age, @Grade)";
        return await _dbConnection.ExecuteAsync(sql, student);
    }

    public async Task<int> UpdateStudentAsync(Student student)
    {
        var sql = "UPDATE Students SET Name = @Name, Age = @Age, Grade = @Grade WHERE Id = @Id";
        return await _dbConnection.ExecuteAsync(sql, student);
    }

    public async Task<int> DeleteStudentAsync(int id)
    {
        var sql = "DELETE FROM Students WHERE Id = @Id";
        return await _dbConnection.ExecuteAsync(sql, new { Id = id });
    }
}
```
### 4.3. Service
Tạo một service để quản lý logic nghiệp vụ, sử dụng repository:

```csharp
public interface IStudentService
{
    Task<IEnumerable<Student>> GetAllStudentsAsync();
    Task<Student> GetStudentByIdAsync(int id);
    Task<int> AddStudentAsync(Student student);
    Task<int> UpdateStudentAsync(Student student);
    Task<int> DeleteStudentAsync(int id);
}

public class StudentService : IStudentService
{
    private readonly IStudentRepository _studentRepository;

    public StudentService(IStudentRepository studentRepository)
    {
        _studentRepository = studentRepository;
    }

    public async Task<IEnumerable<Student>> GetAllStudentsAsync()
    {
        return await _studentRepository.GetAllStudentsAsync();
    }

    public async Task<Student> GetStudentByIdAsync(int id)
    {
        return await _studentRepository.GetStudentByIdAsync(id);
    }

    public async Task<int> AddStudentAsync(Student student)
    {
        return await _studentRepository.AddStudentAsync(student);
    }

    public async Task<int> UpdateStudentAsync(Student student)
    {
        return await _studentRepository.UpdateStudentAsync(student);
    }

    public async Task<int> DeleteStudentAsync(int id)
    {
        return await _studentRepository.DeleteStudentAsync(id);
    }
}
```
### 4.4. Controller
Cuối cùng, tạo một controller để xử lý các yêu cầu HTTP và gọi service:

```csharp
[ApiController]
[Route("api/[controller]")]
public class StudentsController : ControllerBase
{
    private readonly IStudentService _studentService;

    public StudentsController(IStudentService studentService)
    {
        _studentService = studentService;
    }

    [HttpGet]
    public async Task<IActionResult> GetAllStudents()
    {
        var students = await _studentService.GetAllStudentsAsync();
        return Ok(students);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetStudentById(int id)
    {
        var student = await _studentService.GetStudentByIdAsync(id);
        if (student == null)
            return NotFound();
        return Ok(student);
    }

    [HttpPost]
    public async Task<IActionResult> AddStudent(Student student)
    {
        await _studentService.AddStudentAsync(student);
        return CreatedAtAction(nameof(GetStudentById), new { id = student.Id }, student);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateStudent(int id, Student student)
    {
        if (id != student.Id)
            return BadRequest();
        await _studentService.UpdateStudentAsync(student);
        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteStudent(int id)
    {
        await _studentService.DeleteStudentAsync(id);
        return NoContent();
    }
}
```
## 5. Tổng kết
Dapper là một công cụ mạnh mẽ và linh hoạt cho việc truy vấn cơ sở dữ liệu trong .NET. Với cú pháp đơn giản và hiệu suất cao, Dapper giúp bạn quản lý dữ liệu một cách dễ dàng mà không cần phải làm việc với những lớp trừu tượng phức tạp. Tuy nhiên, vì Dapper chỉ cung cấp những tiện ích cơ bản và đòi hỏi bạn phải viết các truy vấn SQL thủ công, nên nó phù hợp hơn cho những ứng dụng mà hiệu suất và khả năng kiểm soát SQL là ưu tiên hàng đầu.
