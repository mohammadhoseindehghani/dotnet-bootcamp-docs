---
title: جلسهٔ دوازدهم · تحلیل پروژهٔ گلستان
---

# جلسهٔ دوازدهم · کنار هم گذاشتن مفاهیم OOP در پروژهٔ گلستان

<div class="lesson-intro">تاریخ برگزاری: <bdi dir="ltr">1405/09/03</bdi><br>در این جلسه مفهوم تازه‌ای تدریس نشد؛ یک سناریوی واقعی‌تر را مدل کردیم و کلاس‌ها، ارث‌بری، کپسوله‌سازی، سازنده‌ها، چندریختی، اعضای static و List را کنار هم قرار دادیم.</div>

کد این درس بر اساس پروژه‌ای است که مدرس با نام `Golestan` در اختیار دانشجوها قرار داده است. هدف این صفحه فقط خواندن چند کلاس نیست؛ می‌خواهیم بفهمیم هر تصمیم به کدام مفهوم قبلی مربوط است، شیءها چگونه با هم ارتباط دارند و اجرای برنامه از کجا به کجا می‌رود.

> پروژه در مرحلهٔ آموزشی است. ثبت دانشجو و ورود دانشجو بخشی از رفتار واقعی دارند، اما ورود استاد و اپراتور، انتخاب کلاس و بعضی قواعد هنوز کامل نشده‌اند. ناقص بودن این بخش‌ها اشکال درس نیست؛ مرز فعلی پیاده‌سازی را نشان می‌دهد.

## ۱. مسئله‌ای که مدل می‌کنیم

سامانهٔ گلستان باید با چند موجودیت دانشگاهی کار کند:

- کاربر (`User`) با نام، نام خانوادگی، نام کاربری، رمز و شمارهٔ همراه؛
- دانشجو (`Student`) با شمارهٔ دانشجویی، ترم، تعداد واحد و کلاس‌ها؛
- استاد (`Teacher`)؛
- اپراتور (`Operator`)؛
- درس (`Course`) با نام و تعداد واحد؛
- کلاس ارائه‌شده (`Class`) با ظرفیت، درس، استاد و دانشجوها؛
- یک محل موقت برای نگه‌داری داده‌ها (`InMemoryDB`).

پیش از کد، رابطه‌ها را این‌طور ببین:

```text
BaseEntity
├── User (abstract)
│   ├── Student
│   ├── Teacher
│   └── Operator
├── Course
└── Class
    ├── یک Course دارد
    ├── یک Teacher دارد
    └── چند Student دارد

InMemoryDB
├── List<Student>
├── List<Teacher>
└── List<Operator>
```

این نمودار دو نوع رابطه را نشان می‌دهد:

- پیکان عمودی «نوعی از» است: `Student` نوعی `User` است و `User` نوعی `BaseEntity` است.
- عبارت «دارد» ارث‌بری نیست: یک `Class` یک درس و استاد دارد؛ کلاس دانشگاهی نوعی درس یا استاد نیست.

## ۲. ساختار فایل‌های پروژه

| فایل | مسئولیت فعلی |
| --- | --- |
| `BaseEntity.cs` | شناسهٔ مشترک موجودیت‌ها |
| `User.cs` | داده و رفتار مشترک کاربران |
| `Student.cs` | اطلاعات و قوانین مخصوص دانشجو |
| `Teacher.cs` | نوع کاربر استاد |
| `Operator.cs` | نوع کاربر اپراتور |
| `Course.cs` | مدل یک درس |
| `Class.cs` | مدل یک ارائهٔ درس با استاد و دانشجوها |
| `InMemoryDB.cs` | فهرست‌های مشترک و موقت برنامه |
| `Program.cs` | منو، گرفتن ورودی و هدایت جریان اجرا |

جدا بودن کلاس‌ها در فایل‌های مختلف باعث می‌شود هر فایل یک مسئولیت روشن‌تر داشته باشد. نام فایل معمولاً با نام کلاس اصلی داخل آن یکسان است.

## ۳. `BaseEntity`: ریشهٔ موجودیت‌های سیستم

```csharp
namespace Golestan;

public abstract class BaseEntity
{
    public int Id { get; set; }
}
```

چند کلاس مختلف به شناسه نیاز دارند. به‌جای تکرار `Id` در `Student`، `Course` و `Class`، آن را در `BaseEntity` قرار داده‌ایم.

### مفاهیم استفاده‌شده

**ارث‌بری:** هر فرزند `BaseEntity` پراپرتی `Id` را دریافت می‌کند.

**Abstraction:** عبارت `abstract` می‌گوید `BaseEntity` یک مفهوم عمومی برای موجودیت‌های دارای شناسه است و قرار نیست مستقیم بنویسیم:

```csharp
// BaseEntity entity = new BaseEntity(); // مجاز نیست.
```

اما می‌توانیم از فرزند کامل آن نمونه بسازیم:

```csharp
Course course = new Course();
course.Id = 10;
```

کلاس abstract مجبور نیست حتماً متد abstract داشته باشد. این کلاس فقط کد مشترک می‌دهد و ساخت مستقیم مفهوم عمومی `BaseEntity` را ممنوع می‌کند.

### چرا `Id` در کلاس پایه است؟

شناسه به نوع خاصی از کاربر محدود نیست. اگر بعداً موجودیت دیگری مثل `Department` نیز شناسه داشته باشد، می‌تواند از همین پایه استفاده کند. البته هر کلاس دارای `Id` الزاماً باید از این پایه ارث ببرد؛ این تصمیم به مدل پروژه بستگی دارد.

## ۴. `User`: داده و رفتار مشترک کاربران

نسخهٔ اصلی کلاس:

```csharp
public abstract class User : BaseEntity
{
    public User(string userName, string password)
    {
        UserName = userName;
        Password = password;
    }

    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string UserName { get; set; }
    private string Password { get; set; }
    public string Mobile { get; set; }

    public virtual bool ChangePassword(
        string oldPassword,
        string newPassword)
    {
        if (Password == oldPassword && newPassword.Length > 3)
        {
            Password = newPassword;
            return true;
        }
        else
        {
            return false;
        }
    }

    public bool CheckPassword(string pass)
    {
        if (pass == Password)
            return true;
        else
            return false;
    }
}
```

### `User : BaseEntity`

کلاس `User` از `BaseEntity` ارث می‌برد؛ بنابراین همهٔ کاربران `Id` دارند. خود `User` نیز abstract است، چون در این مدل کاربر باید نقش مشخصی مانند دانشجو، استاد یا اپراتور داشته باشد.

```text
BaseEntity.Id
      ↓
User.Id + اطلاعات حساب
      ↓
Student / Teacher / Operator
```

### سازنده و وضعیت معتبر اولیه

سازنده می‌گوید هر کاربر هنگام ساخت حداقل باید نام کاربری و رمز اولیه داشته باشد:

```csharp
public User(string userName, string password)
```

چون `User` abstract است، این سازنده مستقیم با `new User(...)` صدا زده نمی‌شود. سازندهٔ فرزند با `base(userName, password)` آن را اجرا می‌کند.

### Encapsulation برای رمز عبور

```csharp
private string Password { get; set; }
```

رمز `private` است؛ کد بیرون کلاس و حتی کلاس فرزند نمی‌تواند مستقیم بنویسد:

```csharp
// student.Password = "1234"; // غیرمجاز
```

تغییر و بررسی رمز از دو رفتار کنترل‌شده عبور می‌کند:

- `ChangePassword` رمز قدیمی و طول رمز تازه را بررسی می‌کند.
- `CheckPassword` فقط نتیجهٔ درست یا غلط بودن را برمی‌گرداند.

این همان ایدهٔ «منشی کلاس» است: بیرون کلاس درخواست می‌دهد و خود کلاس تصمیم می‌گیرد عملیات مجاز است یا نه.

### متد `virtual`

```csharp
public virtual bool ChangePassword(...)
```

کلمهٔ `virtual` اجازه می‌دهد فرزند قانون دقیق‌تری تعریف کند. اجرای عمومی `User` حداقل طول بیشتر از ۳ را می‌خواهد، اما `Student` در ادامه حداقل ۶ نویسه را الزام می‌کند.

### ساده‌سازی `CheckPassword`

نسخهٔ مدرس درست کار می‌کند. چون نتیجهٔ مقایسه خودش `bool` است، می‌توان همان رفتار را کوتاه‌تر نوشت:

```csharp
public bool CheckPassword(string pass)
{
    return pass == Password;
}
```

این تغییر فقط کوتاه‌نویسی است و منطق را عوض نمی‌کند. برای شروع، شکل `if/else` گاهی خواناتر است.

## ۵. `Student`: تخصصی کردن User

بخش اصلی کلاس دانشجو:

```csharp
public class Student : User
{
    public Student(
        string firstName,
        string lastName,
        string userName,
        string password)
        : base(userName, password)
    {
        FirstName = firstName;
        LastName = lastName;
    }

    public static int MaxUnit { get; set; } = 20;
    public static int MaxUnitInMAshroot { get; set; } = 14;
    public int StudentNO { get; set; }
    public int Term { get; set; }
    public bool IsMashroot { get; set; } = false;
    public int UnitCount { get; set; }
    public int UnitCountInCurrentTerm { get; set; }
    public List<Class> Classes { get; set; }
}
```

### زنجیرهٔ سازنده‌ها

هنگام ساخت دانشجو این ترتیب اتفاق می‌افتد:

```csharp
Student student = new Student(
    "Sara",
    "Ahmadi",
    "sara1405",
    "123456");
```

```text
new Student(...)
    ↓
base(userName, password)
    ↓
سازندهٔ User نام کاربری و رمز را مقدار می‌دهد.
    ↓
بدنهٔ سازندهٔ Student نام و نام خانوادگی را مقدار می‌دهد.
```

اگر `base(userName, password)` حذف شود، کامپایلر تلاش می‌کند `User()` را اجرا کند. چنین سازنده‌ای وجود ندارد، پس خطای کامپایل می‌گیریم.

### عضوهای static

```csharp
public static int MaxUnit { get; set; } = 20;
public static int MaxUnitInMAshroot { get; set; } = 14;
```

حداکثر تعداد واحد متعلق به یک دانشجوی خاص نیست؛ قانون مشترک نوع `Student` است:

```csharp
Console.WriteLine(Student.MaxUnit);          // 20
Console.WriteLine(Student.MaxUnitInMAshroot); // 14
```

اگر این قوانین در زمان اجرای برنامه نباید تغییر کنند، setter عمومی مناسب نیست. در نسخهٔ بعدی می‌توان آن‌ها را فقط‌خواندنی یا ثابت‌تر طراحی کرد. نام `MaxUnitInMAshroot` نیز بهتر است از نظر املایی و convention اصلاح شود؛ مثلاً `MaxUnitWhenConditional` یا نام فارسی‌ـ‌انگلیسی مورد توافق تیم.

### پراپرتی‌های مخصوص دانشجو

| پراپرتی | معنی فعلی |
| --- | --- |
| `StudentNO` | شمارهٔ دانشجویی |
| `Term` | ترم فعلی |
| `IsMashroot` | آیا دانشجو مشروط است؟ |
| `UnitCount` | تعداد کل واحدها |
| `UnitCountInCurrentTerm` | واحدهای ترم جاری |
| `Classes` | کلاس‌های انتخاب‌شده |

مقادیر پیش‌فرض نوع‌ها را به یاد بیاور:

- `int` در این propertyها ابتدا `0` است.
- `bool` ابتدا `false` است؛ نوشتن `= false` برای `IsMashroot` توضیحی است، اما ضروری نیست.
- `Classes` چون reference type است، بدون مقداردهی `null` می‌ماند و همین یکی از warningهای build است.

شکل امن‌تر در سطح فعلی دوره:

```csharp
public List<Class> Classes { get; set; } = new List<Class>();
```

یا با syntax جدید پروژه:

```csharp
public List<Class> Classes { get; set; } = [];
```

### Method overriding و استفاده از `base`

```csharp
public override bool ChangePassword(
    string oldPassword,
    string newPassword)
{
    if (newPassword.Length >= 6)
    {
        return base.ChangePassword(oldPassword, newPassword);
    }
    else
    {
        return false;
    }
}
```

این متد دو لایهٔ قانون دارد:

1. `Student` بررسی می‌کند رمز تازه حداقل ۶ نویسه باشد.
2. `base.ChangePassword` اجرای والد را صدا می‌زند تا رمز قبلی بررسی و مقدار private واقعاً تغییر داده شود.

چرا فرزند خودش `Password = newPassword` نمی‌نویسد؟ چون `Password` در `User` خصوصی است. والد مالک دادهٔ رمز و مسیر تغییر آن است. اینجا inheritance، encapsulation و polymorphism هم‌زمان دیده می‌شوند.

### `RegisterClass` هنوز placeholder است

```csharp
public bool RegisterClass(Class cl)
{
    return true;
}
```

امضای رفتار طراحی شده، اما منطق هنوز نوشته نشده است. در نسخهٔ آینده باید سؤال‌هایی مثل این بررسی شوند:

- آیا ظرفیت کلاس پر شده است؟
- آیا دانشجو قبلاً همین کلاس را گرفته است؟
- مجموع واحدهای ترم از حد مجاز بیشتر می‌شود؟
- آیا دانشجوی مشروط از سقف ۱۴ واحد عبور می‌کند؟
- در صورت موفقیت، آیا دانشجو به `Class.Students` و کلاس به `Student.Classes` اضافه می‌شود؟

برگرداندن همیشگی `true` یعنی متد هنوز واقعاً ثبت‌نام را انجام نمی‌دهد.

## ۶. `Teacher` و `Operator`

کلاس استاد:

```csharp
public class Teacher : User
{
    public Teacher(
        string firstName,
        string lastName,
        string userName,
        string password)
        : base(userName, password)
    {
        FirstName = firstName;
        LastName = lastName;
    }
}
```

کلاس اپراتور:

```csharp
public class Operator : User
{
    public Operator(string userName, string password)
        : base(userName, password)
    {
    }
}
```

هر دو عضوهای `Id`, `FirstName`, `LastName`, `UserName`, `Mobile`, `ChangePassword` و `CheckPassword` را از زنجیرهٔ ارث‌بری می‌گیرند.

در مرحلهٔ فعلی:

- `Teacher` فقط روش ساخت مخصوص خود را دارد و رفتار تازه‌ای اضافه نکرده است.
- `Operator` فقط نام کاربری و رمز می‌گیرد؛ نام و نام خانوادگی آن مقداردهی نمی‌شوند.

این کلاس‌های کوچک بی‌فایده نیستند. وجود نوع جدا به برنامه اجازه می‌دهد بعداً رفتارهای مخصوص اضافه کند؛ مثلاً استاد نمره ثبت کند و اپراتور درس یا کاربر بسازد.

## ۷. `Course`: مدل درس

```csharp
public class Course : BaseEntity
{
    public string CourseName { get; set; }
    public int Unit { get; set; }
}
```

`Course` از `BaseEntity` شناسه می‌گیرد و دو دادهٔ خودش را اضافه می‌کند:

- نام درس؛
- تعداد واحد درس.

مثال استفاده:

```csharp
Course course = new Course();
course.Id = 10;
course.CourseName = "C# Basics";
course.Unit = 3;
```

چون `CourseName` در سازنده یا تعریف property مقدار ندارد، nullable analysis هشدار می‌دهد. در سطح فعلی می‌توان نوشت:

```csharp
public string CourseName { get; set; } = "";
```

یا سازنده‌ای تعریف کرد که نام و تعداد واحد را اجباری بگیرد. انتخاب بهتر به قانون مسئله بستگی دارد: آیا درس بدون نام اصلاً معتبر است؟ اگر نه، سازنده معنای قوی‌تری دارد.

## ۸. `Class`: ارتباط چند موجودیت

```csharp
public class Class : BaseEntity
{
    public int Capacity { get; set; }
    public Course Course { get; set; }
    public Teacher Teacher { get; set; }
    public List<Student> Students { get; set; }
}
```

در این پروژه، `Course` خودِ درس است؛ مثلاً «برنامه‌نویسی ۱، سه واحد». `Class` ارائهٔ آن درس است که ظرفیت، استاد و دانشجو دارد.

```text
Course: برنامه‌نویسی ۱، ۳ واحد

Class:
  درس: برنامه‌نویسی ۱
  استاد: Ali Ahmadi
  ظرفیت: ۳۰
  دانشجوها: Sara, Nima, ...
```

### Composition / Association به زبان ساده

`Class` از `Course` یا `Teacher` ارث نبرده است. آن‌ها را به‌عنوان property نگه می‌دارد. جملهٔ درست این است:

- کلاس ارائه‌شده **یک درس دارد**.
- کلاس ارائه‌شده **یک استاد دارد**.
- کلاس ارائه‌شده **چند دانشجو دارد**.

این بخش نشان می‌دهد مدل‌سازی OOP فقط inheritance نیست. بخش بزرگی از طراحی، ساختن ارتباط میان شیءهاست.

### هشدارهای مقداردهی

سه property مرجعی هنوز مقدار اولیه ندارند. یک نسخهٔ ساده و بدون warning می‌تواند سازنده داشته باشد:

```csharp
public class Class : BaseEntity
{
    public int Capacity { get; set; }
    public Course Course { get; set; }
    public Teacher Teacher { get; set; }
    public List<Student> Students { get; set; } = [];

    public Class(Course course, Teacher teacher, int capacity)
    {
        Course = course;
        Teacher = teacher;
        Capacity = capacity;
    }
}
```

اکنون ساخت کلاس بدون درس و استاد ممکن نیست و لیست دانشجوها نیز از ابتدا آماده است.

> نام `Class` در C# از نظر بزرگی حروف با keyword کوچک `class` متفاوت و مجاز است، ولی هنگام خواندن می‌تواند گیج‌کننده باشد. نامی مانند `CourseOffering` یا `UniversityClass` معنای دقیق‌تری دارد. این یک پیشنهاد نام‌گذاری است، نه خطای کامپایل.

## ۹. `InMemoryDB`: دیتابیس موقت داخل حافظه

```csharp
public static class InMemoryDB
{
    public static List<Student> Students { get; set; } = [];
    public static List<Teacher> Teachers { get; set; } = [];
    public static List<Operator> Operators { get; set; } = [];
}
```

### چرا کلاس static است؟

برنامه یک محل مشترک برای فهرست کاربران می‌خواهد. لازم نیست بنویسیم:

```csharp
// InMemoryDB db = new InMemoryDB();
```

مستقیم با نام کلاس کار می‌کنیم:

```csharp
InMemoryDB.Students.Add(student);
```

هر جای برنامه به همان لیست مشترک دسترسی دارد. `[]` syntax جدید ساخت مجموعهٔ خالی است و در این پروژه به دلیل نسخهٔ جدید C# قابل استفاده است. شکل آشناتر آن چنین است:

```csharp
public static List<Student> Students { get; set; }
    = new List<Student>();
```

### چرا این دیتابیس واقعی نیست؟

این داده‌ها فقط تا زمان اجرای برنامه در RAM هستند. با بسته شدن برنامه همه‌چیز از بین می‌رود. نام `InMemoryDB` دقیقاً همین محدودیت را می‌گوید.

مزیت آموزشی آن این است که بدون یادگیری دیتابیس واقعی می‌توانیم ثبت، جست‌وجو و ورود را تمرین کنیم. بعداً می‌توان این بخش را با سرویس یا repository واقعی جایگزین کرد.

### نکتهٔ طراحی

setter عمومی اجازه می‌دهد کل لیست بیرون کلاس تعویض شود:

```csharp
InMemoryDB.Students = new List<Student>();
```

اگر فقط افزودن و حذف عضوها لازم است، می‌توان setter را خصوصی کرد:

```csharp
public static List<Student> Students { get; private set; } = [];
```

هنوز خود لیست قابل تغییر است، ولی کد بیرون نمی‌تواند reference آن را با لیستی دیگر جایگزین کند.

## ۱۰. `Program`: نقطهٔ شروع و هدایت جریان

برخلاف مثال‌های top-level statement جلسه‌های اول، این پروژه کلاس `Program` و متد `Main` دارد:

```csharp
internal class Program
{
    static void Main(string[] args)
    {
        // نقطهٔ شروع برنامه
    }
}
```

`Main` اولین متدی است که هنگام اجرای برنامه صدا زده می‌شود. متدهای `Register`, `LoginAs` و متدهای ورود نیز static هستند، چون `Main` آن‌ها را بدون ساختن شیء `Program` صدا می‌زند.

### منوی اول

```csharp
Console.Write("Please Select Action (1- Register , 2- Login) :");
string Action = Console.ReadLine();

switch (Action)
{
    case "1":
        Register();
        break;
    case "2":
        LoginAs();
        break;
    default:
        Console.WriteLine("Error");
        break;
}
```

جریان اجرا بر اساس متن ورودی به یکی از دو مسیر می‌رود. چون `ReadLine()` ممکن است `null` برگرداند، این خط warning دارد. شکل ساده‌تر و بدون warning:

```csharp
string action = Console.ReadLine() ?? "";
```

همچنین طبق convention متغیر محلی با حرف کوچک شروع می‌شود؛ `action` از `Action` مناسب‌تر است.

## ۱۱. جریان ثبت دانشجو

متد `Register` چهار ورودی می‌گیرد، دانشجو می‌سازد، آن را در حافظه ذخیره می‌کند و سپس صفحهٔ انتخاب نوع ورود را نشان می‌دهد:

```csharp
static void Register()
{
    Console.Write("Please Enter Your First Name :");
    string firstName = Console.ReadLine();

    Console.Write("Please Enter Your Last Name :");
    string lastName = Console.ReadLine();

    Console.Write("Please Enter Your UserName :");
    string userName = Console.ReadLine();

    Console.Write("Please Enter Your Password :");
    string password = Console.ReadLine();

    Student st = new Student(
        firstName,
        lastName,
        userName,
        password);

    InMemoryDB.Students.Add(st);
    LoginAs();
}
```

### ردگیری مرحله‌به‌مرحله

فرض کن کاربر این داده‌ها را وارد کند:

```text
Sara
Ahmadi
sara1405
123456
```

اتفاق‌ها:

1. چهار متغیر محلی مقدار می‌گیرند.
2. `new Student(...)` سازندهٔ دانشجو را اجرا می‌کند.
3. سازندهٔ دانشجو با `base` نام کاربری و رمز را به سازندهٔ `User` می‌فرستد.
4. یک شیء کامل در متغیر `st` قرار می‌گیرد.
5. `Add` reference همان شیء را به `InMemoryDB.Students` اضافه می‌کند.
6. `LoginAs()` اجرا می‌شود.

لیست یک کپی جدا از تمام داده‌های شیء نمی‌سازد؛ reference دانشجو را نگه می‌دارد. اگر بعداً همان شیء تغییر کند، تغییر از طریق reference داخل لیست نیز دیده می‌شود.

### چه اعتبارسنجی‌هایی هنوز وجود ندارند؟

- خالی نبودن نام و نام کاربری؛
- معتبر بودن رمز؛
- تکراری نبودن نام کاربری؛
- تولید `Id` و `StudentNO`؛
- معتبر بودن شمارهٔ همراه؛
- تأیید موفقیت ثبت برای کاربر.

این موارد فرصت توسعهٔ مرحلهٔ بعد هستند.

## ۱۲. انتخاب نقش برای ورود

```csharp
static void LoginAs()
{
    Console.Write(
        "You Want Login As " +
        "(1- Student, 2- Teacher, 3- Operator): ");

    string action = Console.ReadLine();

    switch (action)
    {
        case "1":
            LoginStudent();
            break;
        case "2":
            LoginTeacher();
            break;
        case "3":
            LoginOperator();
            break;
        default:
            Console.WriteLine("Error");
            break;
    }
}
```

این متد فقط تصمیم می‌گیرد کدام جریان اجرا شود. خودش عملیات احراز هویت را انجام نمی‌دهد. این تفکیک کوچک باعث می‌شود `Main` شلوغ نشود.

در نسخهٔ فعلی، فقط `LoginStudent` کامل‌تر است. دو متد دیگر ورودی می‌گیرند اما هنوز لیست را جست‌وجو نمی‌کنند.

## ۱۳. جریان ورود دانشجو

```csharp
static void LoginStudent()
{
    Console.Write("Please Enter Your UserName :");
    string userName = Console.ReadLine();

    Console.Write("Please Enter Your Password :");
    string password = Console.ReadLine();

    foreach (var student in InMemoryDB.Students)
    {
        if (userName == student.UserName)
        {
            if (student.CheckPassword(password))
            {
                Console.WriteLine("Login Ok");
                break;
            }
            else
            {
                Console.WriteLine("Not Ok");
            }
        }
    }
}
```

### الگوریتم ورود

```text
نام کاربری و رمز را بگیر
        ↓
روی همهٔ Students حرکت کن
        ↓
آیا UserName برابر است؟
   ├── خیر: دانشجوی بعدی
   └── بله: CheckPassword
              ├── true  → Login Ok و پایان حلقه
              └── false → Not Ok
```

### مفاهیم قبلی در همین متد

- `foreach` برای پیمایش `List<Student>`؛
- شرط‌های تو در تو؛
- دسترسی به property عمومی `UserName`؛
- کپسوله‌سازی رمز با `CheckPassword`؛
- `break` برای توقف پس از ورود موفق؛
- استفاده از دادهٔ مشترک کلاس static.

### چرا مقایسهٔ رمز در `Program` انجام نشده؟

`Program` نمی‌تواند و نباید به `Password` خصوصی دسترسی مستقیم داشته باشد. از خود شیء می‌پرسد:

```csharp
student.CheckPassword(password)
```

این همان پیام OOP است: داده و رفتاری که به آن مربوط است کنار هم باشند.

## ۱۴. رفتارهای فعلی برنامه را دقیق بدانیم

### مسیر موفق

1. برنامه را اجرا می‌کنیم.
2. گزینهٔ ثبت را انتخاب می‌کنیم.
3. دانشجو به لیست static اضافه می‌شود.
4. ورود به‌عنوان دانشجو را انتخاب می‌کنیم.
5. همان نام کاربری و رمز را می‌زنیم.
6. برنامه `Login Ok` چاپ می‌کند.

### اگر نام کاربری پیدا نشود

حلقه تمام می‌شود و هیچ پیام نهایی چاپ نمی‌شود. کاربر نمی‌فهمد نام کاربری اشتباه بوده است. این یک مورد مناسب برای اصلاح است.

### اگر رمز اشتباه باشد

`Not Ok` چاپ می‌شود. چون `break` فقط در حالت موفق وجود دارد، حلقه ادامه پیدا می‌کند. اگر نام‌های کاربری یکتا باشند، بهتر است پس از پیدا شدن همان کاربر و تشخیص رمز غلط نیز جریان به‌صورت روشن تمام شود.

### اگر برنامه تازه با Login شروع شود

لیست `Students` خالی است، چون داده‌ها دائمی نیستند. بنابراین هیچ دانشجویی برای ورود وجود ندارد. برای آزمایش موفق باید در همان اجرای برنامه ابتدا ثبت انجام شود یا دادهٔ اولیه اضافه کنیم.

## ۱۵. نسخهٔ خواناتر الگوریتم ورود

این نسخه هنوز فقط از مفاهیم خوانده‌شده استفاده می‌کند، ولی برای همهٔ حالت‌ها پاسخ مشخص دارد:

```csharp
static void LoginStudent()
{
    Console.Write("Please Enter Your UserName: ");
    string userName = Console.ReadLine() ?? "";

    Console.Write("Please Enter Your Password: ");
    string password = Console.ReadLine() ?? "";

    bool userFound = false;

    foreach (Student student in InMemoryDB.Students)
    {
        if (student.UserName == userName)
        {
            userFound = true;

            if (student.CheckPassword(password))
            {
                Console.WriteLine("Login Ok");
            }
            else
            {
                Console.WriteLine("Wrong password");
            }

            break;
        }
    }

    if (!userFound)
    {
        Console.WriteLine("User not found");
    }
}
```

`userFound` وضعیت جست‌وجو را نگه می‌دارد. این نسخه میان «کاربر پیدا نشد» و «رمز اشتباه بود» تفاوت می‌گذارد. در سامانه‌های واقعی ممکن است برای امنیت پیام عمومی‌تری نمایش دهند، اما برای تمرین الگوریتم، تفاوت دو حالت مفید است.

## ۱۶. نقشهٔ مفاهیم OOP در پروژه

| مفهوم | محل استفاده | نتیجه |
| --- | --- | --- |
| Class / Object | همهٔ مدل‌ها و `new Student(...)` | تعریف نوع و ساخت نمونه |
| Entity modeling | Student, Teacher, Course, Class | تبدیل مفاهیم دانشگاه به کد |
| Inheritance | `User : BaseEntity`, `Student : User` | اشتراک داده و رفتار |
| Abstraction | `BaseEntity`, `User` | جلوگیری از ساخت مفهوم عمومی و ناقص |
| Encapsulation | `private Password` | جلوگیری از دسترسی مستقیم به رمز |
| Constructor | User, Student, Teacher, Operator | آماده کردن دادهٔ اولیه |
| Constructor chaining | `: base(userName, password)` | ساخت درست بخش والد پیش از فرزند |
| Property | داده‌های همهٔ کلاس‌ها | خواندن و نوشتن وضعیت شیء |
| Static member | `Student.MaxUnit` | قانون مشترک همهٔ دانشجوها |
| Static class | `InMemoryDB` | یک محل مشترک بدون نمونه |
| Method overriding | `Student.ChangePassword` | قانون تخصصی‌تر برای دانشجو |
| `base` در متد | `base.ChangePassword(...)` | استفاده از منطق کپسوله‌شدهٔ والد |
| List | Students, Teachers, Classes | نگه‌داری تعداد متغیر شیءها |
| Association | Class → Course/Teacher/Students | ارتباط میان موجودیت‌ها |

### چه چیزهایی عمداً هنوز استفاده نشده‌اند؟

در نسخهٔ فعلی interface و extension method وجود ندارند، با اینکه در جلسهٔ قبل خوانده شدند. هر پروژه لازم نیست همهٔ ابزارها را فقط برای نمایش استفاده کند. بعداً می‌توان عملیات کاربران یا ذخیره‌سازی را پشت interface برد، ولی افزودن interface بدون نیاز واقعی صرفاً کد را بیشتر می‌کند.

## ۱۷. نتیجهٔ build و معنی warningها

پروژه با تنظیم `Nullable` فعال ساخته می‌شود:

```xml
<Nullable>enable</Nullable>
```

نتیجهٔ بررسی نسخهٔ مدرس:

```text
Build succeeded.
0 Error(s)
25 Warning(s)
```

پس برنامه قابل کامپایل است، اما کامپایلر ۲۵ نقطه را نشان می‌دهد که ممکن است `null` وارد property یا متغیر غیرnullable شود.

دو گروه اصلی warning:

### ۱. Propertyهای مقداردهی‌نشده

نمونه‌ها:

- `User.FirstName`, `LastName`, `Mobile`؛
- `Student.Classes`؛
- `Course.CourseName`؛
- `Class.Course`, `Teacher`, `Students`.

راه‌حل آموزشی وابسته به معنی داده است:

```csharp
public string Mobile { get; set; } = "";
public List<Student> Students { get; set; } = [];
```

یا دادهٔ ضروری را در سازنده بگیریم.

### ۲. خروجی احتمالی null از `Console.ReadLine`

```csharp
string userName = Console.ReadLine() ?? "";
```

`?? ""` می‌گوید اگر ورودی `null` بود، متن خالی قرار بده. بعد باید خالی بودن را نیز اعتبارسنجی کنیم؛ حذف warning به‌تنهایی داده را معتبر نمی‌کند.

> Warning با error فرق دارد. پروژه ساخته شد، ولی warningها اطلاعات مفیدی دربارهٔ ریسک‌های احتمالی می‌دهند. هدف خوب این است که دلیل هر warning را بفهمیم و آگاهانه رفع کنیم، نه اینکه فقط نمایش warningها را خاموش کنیم.

## ۱۸. مرور فنی و پیشنهادهای مرحلهٔ بعد

این موارد ایراد به روند تدریس نیستند؛ backlog طبیعی پروژه برای جلسه‌های بعد هستند.

### اولویت ۱: مقداردهی collectionها

`Student.Classes` و `Class.Students` باید از ابتدا لیست خالی داشته باشند تا `Add` باعث `NullReferenceException` نشود.

### اولویت ۲: کامل کردن بازخورد Login

برای کاربر پیدا‌نشده، رمز اشتباه و ورود موفق مسیر نهایی و پیام روشن داشته باشیم.

### اولویت ۳: یکتا بودن نام کاربری

قبل از ثبت، لیست دانشجوها بررسی شود تا دو کاربر نام کاربری یکسان نداشته باشند. در غیر این صورت ورود مبهم می‌شود.

### اولویت ۴: تولید شناسه

در ثبت فعلی `Id` و `StudentNO` صفر می‌مانند. باید روشی ساده برای تولید شناسهٔ یکتا طراحی شود.

### اولویت ۵: کامل کردن نقش‌ها

`LoginTeacher` و `LoginOperator` فعلاً فقط ورودی می‌گیرند. می‌توان منطق مشترک ورود را بعداً طوری طراحی کرد که تکرار کمتر شود.

### اولویت ۶: پیاده‌سازی انتخاب کلاس

`RegisterClass` باید ظرفیت، سقف واحد، مشروطی و ثبت دوطرفهٔ ارتباط را بررسی کند.

### اولویت ۷: رمز عبور در برنامهٔ واقعی

این پروژه برای آموزش کپسوله‌سازی، رمز را به‌صورت `string` نگه می‌دارد. در محصول واقعی رمز نباید به‌شکل متن ساده ذخیره شود؛ باید از روش استاندارد hashing همراه salt و ابزار معتبر استفاده شود. پیاده‌سازی امنیت رمز موضوع این مرحله از بوت‌کمپ نیست.

### اولویت ۸: اصلاح نام‌ها و متن‌ها

- `StudentNO` بهتر است `StudentNumber` باشد.
- `MaxUnitInMAshroot` نیاز به اصلاح املا و نام روشن‌تر دارد.
- `Class` می‌تواند `CourseOffering` باشد.
- خطاهای تایپی متن‌هایی مثل `Lagin` اصلاح شوند.
- متغیرهای محلی با camelCase نام‌گذاری شوند.

### اولویت ۹: حذف usingهای بدون استفاده

چند فایل `using`هایی دارند که در کد استفاده نشده‌اند؛ مثلاً `System.ComponentModel.Design` و `System.Reflection.Metadata.Ecma335`. حذف آن‌ها رفتار برنامه را تغییر نمی‌دهد و فایل را تمیزتر می‌کند.

## ۱۹. مثال تکمیلی: ثبت کلاس با قواعد فعلی

کد زیر یک مسیر ساده برای کامل کردن `RegisterClass` نشان می‌دهد. این نسخه فرض می‌کند `Classes` و `cl.Students` از قبل لیست خالی هستند:

```csharp
public bool RegisterClass(Class cl)
{
    if (cl.Students.Count >= cl.Capacity)
    {
        return false;
    }

    int allowedUnit;

    if (IsMashroot)
    {
        allowedUnit = MaxUnitInMAshroot;
    }
    else
    {
        allowedUnit = MaxUnit;
    }

    int newUnitCount =
        UnitCountInCurrentTerm + cl.Course.Unit;

    if (newUnitCount > allowedUnit)
    {
        return false;
    }

    foreach (Class registeredClass in Classes)
    {
        if (registeredClass.Id == cl.Id)
        {
            return false;
        }
    }

    Classes.Add(cl);
    cl.Students.Add(this);
    UnitCountInCurrentTerm = newUnitCount;

    return true;
}
```

### حل مسئله پیش از کد

ورودی متد یک کلاس دانشگاهی است. خروجی `bool` می‌گوید ثبت موفق بود یا نه. مراحل:

1. ظرفیت را بررسی کن.
2. سقف واحد را با توجه به مشروطی مشخص کن.
3. مجموع تازهٔ واحدها را حساب کن.
4. تکراری نبودن کلاس را بررسی کن.
5. ارتباط را در هر دو طرف ثبت کن.
6. تعداد واحد ترم را به‌روزرسانی کن.

این مثال برای یادگیری است. در پروژهٔ بزرگ‌تر باید دلیل شکست نیز مشخص شود؛ `false` به‌تنهایی نمی‌گوید مشکل ظرفیت بوده یا سقف واحد.

## ۲۰. مثال تکمیلی: جلوگیری از نام کاربری تکراری

پیش از ساخت دانشجو می‌توانیم جست‌وجو کنیم:

```csharp
bool userNameExists = false;

foreach (Student student in InMemoryDB.Students)
{
    if (student.UserName == userName)
    {
        userNameExists = true;
        break;
    }
}

if (userNameExists)
{
    Console.WriteLine("Username already exists.");
    return;
}

Student newStudent = new Student(
    firstName,
    lastName,
    userName,
    password);

InMemoryDB.Students.Add(newStudent);
```

این مثال همان دانش قبلیِ `foreach`, `if`, `break`, متغیر `bool`, سازنده و `List.Add` را کنار هم استفاده می‌کند.

## ۲۱. پرسش‌های ردگیری کد

پیش از اجرای برنامه، این سؤال‌ها را جواب بده:

1. هنگام `new Student(...)` کدام سازنده زودتر اجرا می‌شود؟
2. `Id` دانشجو از کدام کلاس آمده است؟
3. چرا `Program` نمی‌تواند `Password` را مستقیم بخواند؟
4. اگر `newPassword` پنج نویسه باشد، کدام متد `false` برمی‌گرداند؟
5. اگر هفت نویسه باشد ولی رمز قدیمی غلط باشد، نتیجه چیست و چرا؟
6. `InMemoryDB.Students` به چند نمونه از `InMemoryDB` تعلق دارد؟
7. بعد از بسته شدن برنامه چه اتفاقی برای دانشجوهای ثبت‌شده می‌افتد؟
8. چرا `Class` از `Course` ارث نبرده است؟
9. اگر `Student.Classes.Add(...)` را در نسخهٔ فعلی صدا بزنیم، چه خطری داریم؟
10. اگر نام کاربری وجود نداشته باشد، نسخهٔ فعلی چه پیامی نمایش می‌دهد؟

## ۲۲. تمرین‌های توسعهٔ پروژه

1. همهٔ warningهای propertyهای مقداردهی‌نشده را با مقدار اولیه یا سازندهٔ مناسب رفع کن. برای هر انتخاب دلیل بنویس.
2. همهٔ `Console.ReadLine()`ها را از نظر `null` مدیریت و متن خالی را رد کن.
3. پس از ثبت موفق، نام کاربری دانشجو را همراه پیام موفقیت چاپ کن.
4. از ثبت نام کاربری تکراری جلوگیری کن.
5. برای `Id` دانشجو یک شمارندهٔ static ساده طراحی کن و هنگام ثبت مقدار بده.
6. `LoginStudent` را طوری اصلاح کن که سه نتیجهٔ موفق، رمز غلط و کاربر پیدا‌نشده را پوشش دهد.
7. منطق مشابه ورود را برای استاد و اپراتور پیاده کن.
8. برای `Course` سازنده‌ای بنویس که نام و تعداد واحد را اجباری کند.
9. برای `Class` سازنده‌ای بنویس که درس، استاد و ظرفیت را بگیرد و لیست دانشجوها را آماده کند.
10. `RegisterClass` را با بررسی ظرفیت، سقف واحد و تکراری نبودن پیاده کن.
11. متد `ToString` را برای `Student`, `Course` و `Class` override کن تا چاپ آن‌ها معنادار شود.
12. یک منوی تکرارشونده با `while` بساز تا برنامه پس از یک عملیات فوراً تمام نشود و گزینهٔ خروج داشته باشد.

## جمع‌بندی

پروژهٔ گلستان نشان می‌دهد مفاهیم OOP جدا از هم نیستند. `BaseEntity` و `User` abstraction و inheritance را می‌سازند؛ سازنده‌ها بخش والد و فرزند را آماده می‌کنند؛ رمز private کپسوله می‌شود؛ `Student` رفتار مجازی والد را override می‌کند؛ `Class` با composition چند موجودیت را کنار هم می‌گذارد؛ `InMemoryDB` دادهٔ مشترک را در Listهای static نگه می‌دارد؛ و `Program` جریان ثبت و ورود را هدایت می‌کند.

نسخهٔ فعلی یک نقطهٔ شروع قابل‌کامپایل است، نه سامانهٔ کامل دانشگاه. ارزش اصلی این جلسه در این است که بتوانی مسیر هر داده و رفتار را توضیح بدهی: داده از کجا وارد می‌شود، کدام سازنده آن را می‌گیرد، در کدام شیء ذخیره می‌شود، چه کسی اجازهٔ دسترسی دارد و کدام کلاس مسئول اجرای هر قانون است.

## فایل مرجع و پیش‌نیازها

- کد مرجع مدرس: پروژهٔ `Golestan` ارائه‌شده در کلاس.
- [جلسهٔ هفتم: کلاس و شیء](session-07.md)
- [جلسهٔ هشتم: سازنده و کپسوله‌سازی](session-08.md)
- [جلسهٔ نهم: ارث‌بری و چندریختی](session-09.md)
- [جلسهٔ دهم: static و abstraction](session-10.md)
- [جلسهٔ یازدهم: List و interface](session-11.md)

[→ جلسهٔ قبل: interface، extension method و List](session-11.md) · [بازگشت به فهرست جلسات](index.md)
