---
title: جلسهٔ یازدهم · interface، extension method و List
---

# جلسهٔ یازدهم · قراردادها و مجموعه‌های پویا

<div class="lesson-intro">تاریخ برگزاری: <bdi dir="ltr">1405/08/28</bdi><br>مرور کوتاه finalizer و indexer، آشنایی عمیق با interface و قراردادهای نرم‌افزاری، نوشتن extension method و شروع کار با List و Queue.</div>

این جلسه از دو نکتهٔ تکمیلی شروع می‌شود و سپس به یکی از مهم‌ترین ابزارهای OOP یعنی `interface` می‌رسد. در ادامه یاد می‌گیریم بدون تغییر کد اصلی یک نوع، extension method بنویسیم و در پایان با `List` به‌عنوان مجموعه‌ای با اندازهٔ قابل تغییر کار کنیم.

مثال‌های هر بخش مستقل‌اند. اگر در دو بخش کلاس‌هایی با نام یکسان دیدی، آن‌ها را در پروژه‌های جدا اجرا کن یا نسخهٔ قبلی را با نسخهٔ تازه جایگزین کن.

## ۱. مطالعهٔ تکمیلی: finalizer و مدیریت حافظه

وقتی با `new` از یک کلاس شیء می‌سازیم، runtime برای آن حافظه در نظر می‌گیرد. در مدل سادهٔ آموزشی، دادهٔ خود شیء معمولاً در heap قرار می‌گیرد و متغیر ما reference آن را نگه می‌دارد. وقتی دیگر هیچ reference قابل‌استفاده‌ای به آن شیء وجود ندارد، آن شیء می‌تواند برای جمع‌آوری حافظه آماده شود.

در .NET مدیریت حافظهٔ شیءهای managed معمولاً بر عهدهٔ **Garbage Collector (GC)** است. لازم نیست پس از پایان کار با هر `Student` یا `Course`، حافظهٔ آن را دستی آزاد کنیم. GC در زمان مناسب شیءهای غیرقابل‌دسترسی را پیدا می‌کند و حافظهٔ آن‌ها را پس می‌گیرد.

### destructor در C# دقیق‌تر چه نام دارد؟

چیزی که در C# با شکل `~ClassName()` نوشته می‌شود، در منابع جدید معمولاً **finalizer** نامیده می‌شود؛ واژهٔ destructor نیز به‌صورت تاریخی برای آن استفاده شده است:

```csharp
public class NativeResource
{
    private IntPtr unmanagedHandle;

    public NativeResource(IntPtr handle)
    {
        unmanagedHandle = handle;
    }

    ~NativeResource()
    {
        if (unmanagedHandle != IntPtr.Zero)
        {
            // اینجا فقط API واقعیِ آزادسازی همان منبع unmanaged صدا زده می‌شود.
            // ReleaseNativeHandle(unmanagedHandle);
            unmanagedHandle = IntPtr.Zero;
        }
    }
}
```

نکتهٔ مهم این است که **ما finalizer را صدا نمی‌زنیم و زمان اجرای آن را تعیین نمی‌کنیم**. GC تصمیم می‌گیرد چه زمانی شیء واجد شرایط جمع‌آوری است و اگر کلاس finalizer داشته باشد، آن را در فرایند finalization اجرا می‌کند. حتی پایان برنامه تضمین ساده‌ای برای دیدن پیام finalizer نیست. پس نباید منطق اصلی برنامه، ذخیرهٔ اطلاعات در دیتابیس یا کار ضروری را به اجرای finalizer وابسته کنیم.

!!! warning "این کد فقط برای دیدن شکل finalizer است"
    کد بالا فقط شکل و محل پاک‌سازی را نشان می‌دهد؛ تابع آزادسازی واقعی به منبع و API خارجی بستگی دارد. در finalizer نباید به شیءهای managed دیگر تکیه کنیم، چون وضعیت آن‌ها هنگام finalization قابل اتکا نیست. در برنامه‌های روزمره معمولاً finalizer نمی‌نویسیم و برای handleهای unmanaged نیز استفاده از `SafeHandle` روش ترجیحی است. برای آزادسازی قابل‌پیش‌بینی فایل، اتصال و منابع مشابه معمولاً الگوی `IDisposable` و `using` به کار می‌رود که در درس مناسب جداگانه بررسی خواهد شد.

### سه مفهوم را از هم جدا کن

| رویداد | چه زمانی رخ می‌دهد؟ | چه کسی کنترل می‌کند؟ |
| --- | --- | --- |
| اجرای constructor | هنگام ساخت نمونه | کد با `new` آغازش می‌کند. |
| غیرقابل‌دسترسی شدن شیء | وقتی دیگر reference قابل‌استفاده‌ای به آن نمانده | نتیجهٔ جریان برنامه است. |
| جمع‌آوری حافظه و احتمال اجرای finalizer | در زمانی که runtime مناسب بداند | Garbage Collector |

ذخیره کردن دادهٔ شیء در دیتابیس به معنی نابود شدن خودکار شیء نیست؛ دیتابیس و شیء داخل حافظه دو چیز جدا هستند. شیء وقتی برای GC قابل جمع‌آوری می‌شود که دیگر از مسیر کد قابل‌دسترسی نباشد، نه صرفاً چون اطلاعاتش ذخیره شده است.

## ۲. مطالعهٔ تکمیلی: indexer

در آرایه با `numbers[0]` به خانهٔ اول دسترسی داشتیم. **Indexer** اجازه می‌دهد یک شیء از کلاس خودمان نیز با شکل `object[index]` داده‌ای را در اختیار بگذارد.

کلاس با تعریف indexer به آرایه تبدیل نمی‌شود. indexer یک **property ویژه و پارامتردار** است که مشخص می‌کند عبارت داخل `[]` چگونه به داده برسد. طبیعی‌ترین کاربردش کلاس‌هایی است که مجموعه‌ای از داده‌ها را نگه می‌دارند.

**`Student.cs`**

```csharp
public class Student
{
    public string FirstName { get; set; } = "";

    public Student(string firstName)
    {
        FirstName = firstName;
    }
}
```

**`Classroom.cs`**

```csharp
public class Classroom
{
    private Student[] students;

    public Classroom(Student first, Student second)
    {
        students = new Student[] { first, second };
    }

    public Student this[int index]
    {
        get
        {
            return students[index];
        }
        set
        {
            students[index] = value;
        }
    }
}
```

**`Program.cs`**

```csharp
Student sara = new Student("Sara");
Student ali = new Student("Ali");

Classroom classroom = new Classroom(sara, ali);

Console.WriteLine(classroom[0].FirstName); // Sara

classroom[1] = new Student("Nima");
Console.WriteLine(classroom[1].FirstName); // Nima
```

در تعریف indexer به‌جای نام property از `this` استفاده می‌کنیم و پارامتر index داخل `[]` قرار می‌گیرد. `get` برای خواندن و `set` برای جایگزین کردن مقدار است. قوانین index نامعتبر در این مثال همان قوانین آرایه‌اند و دسترسی بیرون محدوده خطا ایجاد می‌کند.

!!! info "Indexer متد نیست"
    Indexer نوعی property است، نه متد. البته کد `get` و `set` می‌تواند متدهای دیگر کلاس را صدا بزند. ممکن است indexer با `int`، `string` یا نوع مناسب دیگری تعریف شود، اما باید استفاده از `[]` برای مدل مسئله طبیعی و قابل‌فهم باشد. برای کلاس `Student` معمولاً `student.FirstName` از `student[0]` روشن‌تر است؛ indexer برای `Classroom` یا مجموعه‌ها منطقی‌تر است.

## ۳. Interface یعنی قرارداد

معنی لغوی interface «واسط» است. در برنامه‌نویسی، interface معمولاً یک **قرارداد قابلیت** تعریف می‌کند: می‌گوید چه عملیات یا اعضایی باید وجود داشته باشند، بدون اینکه مصرف‌کننده مجبور باشد بداند آن عملیات دقیقاً چگونه انجام شده است.

### قرارداد در دنیای واقعی

فرض کن با یک شرکت خدماتی قرارداد می‌بندیم که هر شنبه از ساعت ۸ تا ۱۲ خانه را تمیز کند. قرارداد مشخص می‌کند:

- چه کاری باید انجام شود؛
- چه زمانی انجام شود؛
- نتیجهٔ مورد انتظار چیست؛
- دو طرف چه تعهدی دارند.

قرارداد لازم نیست تعیین کند کدام کارمند دقیقاً از کدام وسیله استفاده کند. ممکن است شرکت نیروی خود را عوض کند، اما تا وقتی قرارداد رعایت می‌شود، دریافت‌کنندهٔ خدمت نتیجهٔ مورد انتظار را می‌گیرد.

مثال نان ساعت ۸ نیز همین مفهوم را نشان می‌دهد: سفارش‌دهنده می‌گوید دو نان تازه باید ساعت ۸ روی میز باشد. اینکه مسئول خرید از کدام نانوایی می‌خرد، چقدر در صف می‌ایستد یا چه مسیر داخلی‌ای طی می‌کند، جزئیات پیاده‌سازی اوست. مصرف‌کننده به **قرارداد و نتیجه** وابسته است.

### قرارداد میان بخش‌های نرم‌افزار

فرض کن برنامه از سرویسی نرخ ارز می‌گیرد. قرارداد باید روشن کند:

- نام عملیات چیست؛
- چه پارامترهایی می‌گیرد؛
- چه نوع نتیجه‌ای برمی‌گرداند؛
- مصرف‌کننده چگونه آن را فراخوانی می‌کند.

یک سرویس ممکن است داده را از اینترنت و سرویس دیگر از فایل آزمایشی بخواند. اگر هر دو قرارداد یکسانی را پیاده کنند، کد مصرف‌کننده می‌تواند با هر دو کار کند. Interface میان این بخش‌ها قرار می‌گیرد و **آنچه باید ارائه شود** را از **چگونگی انجام آن** جدا می‌کند.

> Interface در مباحث آینده مانند SOLID و dependency injection بسیار دیده می‌شود. فعلاً لازم نیست آن موضوع‌ها را جداگانه جست‌وجو کنی؛ هدف این جلسه فهم خودِ قرارداد و امکان چند پیاده‌سازی است.

## ۴. تعریف اولین interface

طبق convention رایج C#، نام interface را با حرف `I` شروع می‌کنیم؛ مثلاً `IStudentService`. این حرف بخشی اجباری از زبان نیست، اما کد را خواناتر می‌کند.

ابتدا مدل سادهٔ دانشجو را می‌سازیم:

**`Student.cs`**

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; } = "";

    public Student(int id, string name)
    {
        Id = id;
        Name = name;
    }

    public override string ToString()
    {
        return $"{Id} - {Name}";
    }
}
```

حالا قرارداد سرویس دانشجو را تعریف می‌کنیم:

**`IStudentService.cs`**

```csharp
public interface IStudentService
{
    Student GetStudent(int id);
    Student GetStudent(string name);
    void DeleteStudent(int id);
}
```

این interface سه قرارداد دارد:

- گرفتن دانشجو بر اساس شناسه؛
- گرفتن دانشجو بر اساس نام؛
- حذف دانشجو بر اساس شناسه.

دو متد `GetStudent` یک نمونه از method overloading هستند. Interface نگفته دانشجو از آرایه، فایل، دیتابیس یا سرویس اینترنتی پیدا شود؛ فقط امضا و قابلیت مورد انتظار را گفته است.

اعضای قراردادی interface در این شکل بدنه ندارند و با `;` تمام می‌شوند. متدهای قراردادی آن به‌طور معمول public هستند و کلاس پیاده‌ساز باید اجرای مناسب را ارائه کند.

## ۵. پیاده‌سازی قرارداد

برای اعلام پیاده‌سازی interface نیز بعد از نام کلاس `:` می‌نویسیم، اما اصطلاح دقیق این است که کلاس **interface را پیاده‌سازی می‌کند**، نه اینکه از آن مانند یک کلاس والد ارث‌بری می‌کند.

**`StudentService.cs`**

```csharp
public class StudentService : IStudentService
{
    private Student[] students =
    {
        new Student(1, "Sara"),
        new Student(2, "Ali"),
        new Student(3, "Nima")
    };

    public Student GetStudent(int id)
    {
        foreach (Student student in students)
        {
            if (student.Id == id)
            {
                return student;
            }
        }

        return new Student(0, "Not found");
    }

    public Student GetStudent(string name)
    {
        foreach (Student student in students)
        {
            if (student.Name == name)
            {
                return student;
            }
        }

        return new Student(0, "Not found");
    }

    public void DeleteStudent(int id)
    {
        Console.WriteLine($"Delete request for student {id}");
    }
}
```

این پیاده‌سازی برای ساده ماندن مثال از آرایه استفاده می‌کند و در حالت پیدا نشدن یک دانشجوی قراردادی با شناسهٔ صفر برمی‌گرداند. این تصمیم فقط آموزشی است؛ در یک برنامهٔ واقعی باید روش روشن‌تری برای «پیدا نشد» و حذف واقعی طراحی کنیم.

اگر یکی از متدهای interface را ننویسیم یا امضایش را اشتباه کنیم، کلاس قرارداد را کامل نکرده و کامپایلر خطا می‌دهد. کامپایلر کمک می‌کند تعهدهای قرارداد فراموش نشوند.

## ۶. چند پیاده‌سازی برای یک قرارداد

فقط `StudentService` اجازهٔ پیاده‌سازی قرارداد را ندارد. یک سرویس آزمایشی هم می‌تواند همان interface را پیاده کند:

**`TestStudentService.cs`**

```csharp
public class TestStudentService : IStudentService
{
    public Student GetStudent(int id)
    {
        return new Student(id, "Test Student");
    }

    public Student GetStudent(string name)
    {
        return new Student(100, name);
    }

    public void DeleteStudent(int id)
    {
        Console.WriteLine($"Test delete: {id}");
    }
}
```

حالا مصرف‌کننده می‌تواند از طریق خود قرارداد با هرکدام کار کند:

**`Program.cs` — همراه فایل‌های بالا**

```csharp
IStudentService studentService = new StudentService();
Student first = studentService.GetStudent(2);
Console.WriteLine(first);

studentService = new TestStudentService();
Student second = studentService.GetStudent(2);
Console.WriteLine(second);
```

سمت چپ می‌گوید متغیر باید شیئی را بپذیرد که قرارداد `IStudentService` را اجرا می‌کند. سمت راست پیاده‌سازی واقعی را می‌سازد. چون هر دو کلاس قرارداد را رعایت کرده‌اند، می‌توانیم پیاده‌سازی را عوض کنیم و همان متدهای قراردادی را صدا بزنیم.

این شبیه کدی است که با کلاس پایه دیده بودیم:

```csharp
User user1 = new StudentUser();
User user2 = new TeacherUser();
```

اما در interface، رابطه روی **قابلیت و قرارداد** تمرکز دارد. `StudentService` «نوعی interface» به معنای موجودیت واقعی نیست؛ این کلاس تعهدهای آن interface را اجرا می‌کند.

!!! note "این موضوع boxing نیست"
    قرار دادن شیء یک کلاس reference type در متغیر interface مربوط به چندریختی و پیاده‌سازی قرارداد است. Boxing موضوع دیگری است و عمدتاً وقتی یک value type مثل `int` به `object` یا یک interface تبدیل می‌شود مطرح است. فعلاً این دو مفهوم را با هم یکی ندان.

### مثال درگاه پرداخت

برنامه می‌تواند قراردادی مثل `IPaymentGateway` داشته باشد:

```csharp
public interface IPaymentGateway
{
    bool Pay(int amount);
}
```

```csharp
public class MellatGateway : IPaymentGateway
{
    public bool Pay(int amount)
    {
        Console.WriteLine($"Mellat payment: {amount}");
        return true;
    }
}
```

```csharp
public class MelliGateway : IPaymentGateway
{
    public bool Pay(int amount)
    {
        Console.WriteLine($"Melli payment: {amount}");
        return true;
    }
}
```

```csharp
IPaymentGateway gateway = new MellatGateway();
gateway.Pay(500000);

gateway = new MelliGateway();
gateway.Pay(500000);
```

هر بانک جزئیات خودش را دارد، ولی برنامه از عملیات قراردادی `Pay` استفاده می‌کند. در سامانهٔ واقعی قرارداد پرداخت بسیار کامل‌تر است؛ این مثال فقط اصلِ جابه‌جایی پیاده‌سازی‌ها را نشان می‌دهد.

## ۷. پیاده‌سازی چند interface

یک کلاس در C# فقط از یک کلاس به‌طور مستقیم ارث می‌برد، ولی می‌تواند چند interface را پیاده‌سازی کند؛ چون یک موجودیت می‌تواند چند قابلیت مستقل را متعهد شود.

**`IStudentPrinter.cs`**

```csharp
public interface IStudentPrinter
{
    void Print(Student student);
}
```

**`StudentService.cs` — شکل ساده برای نمایش چند قرارداد**

```csharp
public class StudentService : IStudentService, IStudentPrinter
{
    public Student GetStudent(int id)
    {
        return new Student(id, "Sara");
    }

    public Student GetStudent(string name)
    {
        return new Student(1, name);
    }

    public void DeleteStudent(int id)
    {
        Console.WriteLine($"Deleted: {id}");
    }

    public void Print(Student student)
    {
        Console.WriteLine(student);
    }
}
```

این کلاس باید اعضای قراردادی هر دو interface را کامل کند. می‌توانیم همان شیء را از دید هر قرارداد ببینیم:

```csharp
StudentService service = new StudentService();

IStudentService reader = service;
IStudentPrinter printer = service;

Student student = reader.GetStudent(1);
printer.Print(student);
```

## ۸. آیا interface می‌تواند پیاده‌سازی داشته باشد؟

در شکل کلاسیک interface، اعضای قراردادی بدنه ندارند. از **C# 8.0** امکان default interface implementation اضافه شد و بعضی عضوهای interface می‌توانند بدنه داشته باشند. این قابلیت برای سناریوهای خاص و توسعهٔ قراردادها مفید است.

برای این دوره فعلاً interface را به شکل یک قرارداد بدون بدنه بنویس و اجرای اصلی را در کلاس پیاده‌ساز قرار بده. دانستن وجود قابلیت جدید کافی است؛ برای فهم interface لازم نیست اکنون وارد جزئیات آن شویم.

## ۹. Interface، abstract class و static class

این سه ابزار کار یکسانی انجام نمی‌دهند:

| ابزار | ایدهٔ اصلی | نمونه ساخته می‌شود؟ | کد مشترک نمونه‌ای | چند مورد برای یک کلاس؟ |
| --- | --- | --- | --- | --- |
| interface | قرارداد یک یا چند قابلیت | از خود interface خیر | تمرکز فعلی ما روی قرارداد است | یک کلاس می‌تواند چند interface پیاده کند. |
| abstract class | مفهوم پایهٔ ناقص همراه داده و رفتار مشترک | مستقیم خیر | بله؛ فیلد، سازنده و متد معمولی دارد | کلاس فقط یک والد مستقیم دارد. |
| static class | مجموعه عملیات و دادهٔ مشترک بدون نمونه | خیر | دادهٔ نمونه‌ای ندارد | برای ارث‌بری و polymorphism نمونه‌ای نیست. |

### چه زمانی کدام را انتخاب کنیم؟

- اگر چند نوع متفاوت باید یک **قابلیت مشترک** را تضمین کنند، interface گزینهٔ طبیعی است: `IPaymentGateway`, `IStudentService`.
- اگر چند فرزند یک **خانوادهٔ مفهومی** هستند و کد یا وضعیت مشترک دارند، abstract class می‌تواند مناسب باشد: `Vehicle` برای `Car` و `Boat`.
- اگر فقط عملیات کمکی داریم که به هیچ نمونه‌ای تعلق ندارند، static class می‌تواند مناسب باشد: `PhoneExtensions` یا `Math`.

این انتخاب همیشه با یک فرمول حفظی انجام نمی‌شود. با دیدن پروژه‌های بیشتر، تفاوت‌ها روشن‌تر می‌شوند. فعلاً مهم است بدانیم interface می‌گوید «چه قابلیتی باید وجود داشته باشد» و کلاس پیاده‌ساز می‌گوید «آن قابلیت چگونه انجام می‌شود».

## ۱۰. Extension method چیست؟

گاهی نوعی مانند `string` را ما ننوشته‌ایم و نمی‌توانیم کد اصلی آن را تغییر دهیم. با **extension method** می‌توانیم متدی بنویسیم که هنگام استفاده شبیه یکی از متدهای همان نوع دیده شود، بدون اینکه واقعاً سورس آن کلاس را ویرایش کنیم.

برای extension method:

1. یک کلاس `static` می‌سازیم.
2. متد نیز `static` است.
3. اولین پارامتر با `this` شروع می‌شود و نوعی را مشخص می‌کند که متد برای آن قابل استفاده است.

**`StudentExtensions.cs`**

```csharp
public static class StudentExtensions
{
    public static void PrintProfile(this Student student)
    {
        Console.WriteLine($"{student.Id} - {student.Name}");
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student(1, "Sara");
student.PrintProfile();
```

شکل واقعی این فراخوانی را می‌توان این‌طور تصور کرد:

```csharp
StudentExtensions.PrintProfile(student);
```

در شکل `student.PrintProfile()` چیزی داخل پرانتز نفرستادیم، چون خودِ `student` به‌عنوان پارامتر اولِ دارای `this` ارسال می‌شود. این فقط دستور زبان راحت‌تر برای صدا زدن یک متد static است؛ متد واقعاً داخل کلاس `Student` اضافه نشده است.

### پارامترهای دیگر extension method

پارامتر `this` باید اولین پارامتر باشد، ولی بعد از آن می‌توانیم ورودی‌های دیگری داشته باشیم:

```csharp
public static class StudentExtensions
{
    public static void PrintProfile(this Student student, string title)
    {
        Console.WriteLine($"{title}: {student.Id} - {student.Name}");
    }
}
```

```csharp
Student student = new Student(1, "Sara");
student.PrintProfile("Selected student");
```

در این فراخوانی، `student` به پارامتر اول و متن به پارامتر دوم می‌رود.

اگر سورس `Student` در اختیار ماست و رفتار واقعاً متعلق به خود دانشجو است، معمولاً متد نمونه‌ای داخل همان کلاس روشن‌تر است. Extension method زمانی ارزش بیشتری دارد که نوع را نمی‌توانیم تغییر دهیم، یا می‌خواهیم عملیات کمکی را جدا و منظم نگه داریم.

### چرا کلاس و متد باید static باشند؟

هدف این است که برای اجرای عملیات کمکی، شیء جداگانه‌ای از `StudentExtensions` نسازیم. خودِ شیء اصلی، یعنی `student`، به متد static داده می‌شود. کامپایلر این الگو را می‌شناسد و اجازه می‌دهد آن را با شکل نقطه‌ای صدا بزنیم.

## ۱۱. Extension method برای نرمال‌سازی شمارهٔ همراه

نوع `string` متعلق به کتابخانهٔ .NET است. نمی‌توانیم سورس آن را تغییر دهیم، ولی می‌توانیم یک متد کمکی برای شمارهٔ ایرانی بنویسیم. قرارداد این مثال چنین است:

- فاصله و `-` حذف شوند؛
- ورودی‌های `0912...`، `912...`، `98912...`، `+98912...` و `0098912...` پذیرفته شوند؛
- خروجی همیشه با `0098` و سپس ده رقم محلی ساخته شود؛
- ورودی نامعتبر، متن خالی برگرداند.

**`PhoneExtensions.cs`**

```csharp
public static class PhoneExtensions
{
    public static string NormalizeIranianMobile(this string mobile)
    {
        string value = mobile.Replace(" ", "").Replace("-", "");

        if (value.StartsWith("0098"))
        {
            value = value.Substring(4);
        }
        else if (value.StartsWith("+98"))
        {
            value = value.Substring(3);
        }
        else if (value.StartsWith("98"))
        {
            value = value.Substring(2);
        }
        else if (value.StartsWith("0"))
        {
            value = value.Substring(1);
        }

        if (value.Length != 10 || !value.StartsWith("9"))
        {
            return "";
        }

        foreach (char character in value)
        {
            if (character < '0' || character > '9')
            {
                return "";
            }
        }

        return "0098" + value;
    }
}
```

**`Program.cs`**

```csharp
string mobile1 = "09124567788";
string mobile2 = "+98 912-456-7788";
string invalidMobile = "091278896655";

Console.WriteLine(mobile1.NormalizeIranianMobile());
Console.WriteLine(mobile2.NormalizeIranianMobile());

string result = invalidMobile.NormalizeIranianMobile();

if (result == "")
{
    Console.WriteLine("Invalid mobile number");
}
```

دو ورودی اول به `00989124567788` تبدیل می‌شوند. نمونهٔ سوم دوازده رقم دارد و معتبر نیست. این مثال از regex استفاده نمی‌کند و فقط با ابزارهای خوانده‌شده در جلسات قبل نوشته شده است.

## ۱۲. Data structure چیست؟

ساختار داده یا **data structure** روشی برای نگه‌داری و سازمان‌دهی داده‌هاست تا عملیات مورد نیاز را راحت‌تر انجام دهیم. آرایه یک ساختار داده بود: عضوهای هم‌نوع و مرتب داشت و با index به آن‌ها دسترسی پیدا می‌کردیم.

فقط آرایه و لیست وجود ندارند. ساختارهایی مانند `Queue`, `Stack`, `Dictionary` و مجموعه‌های دیگر را به‌مرور می‌بینیم. انتخاب ساختار مناسب به نوع کار بستگی دارد. در این جلسه تمرکز اصلی روی `List` است، چون در برنامه‌های روزمره بسیار استفاده می‌شود.

## ۱۳. تفاوت آرایه و List

اندازهٔ آرایه پس از ساخت ثابت است:

```csharp
int[] numbers = new int[3];
```

سه خانه داریم. می‌توانیم مقدار خانه‌ها را عوض کنیم، ولی خود آرایه ناگهان چهارخانه‌ای نمی‌شود. برای تعداد متغیر باید آرایهٔ تازه بسازیم و داده‌ها را منتقل کنیم؛ همان کاری که در تمرین حذف از آرایه انجام دادیم.

`List` اندازهٔ قابل تغییر دارد و عملیات آماده برای افزودن، حذف، جست‌وجو و مرتب‌سازی فراهم می‌کند:

```csharp
List<int> numbers = new List<int>();
```

`int` داخل `< >` یعنی این لیست عضوهای `int` نگه می‌دارد. فعلاً وارد بحث generic نمی‌شویم؛ فقط نوع اعضا را در این قسمت مشخص می‌کنیم.

| ویژگی | آرایه | `List` |
| --- | --- | --- |
| اندازه | پس از ساخت ثابت | با افزودن و حذف تغییر می‌کند. |
| دسترسی با index | دارد | دارد |
| ترتیب عضوها | حفظ می‌شود | حفظ می‌شود |
| تعداد عضوها | `Length` | `Count` |
| افزودن و حذف آماده | مستقیم ندارد | `Add`, `Insert`, `Remove`, `RemoveAt` و... |

## ۱۴. ساخت و مقداردهی List

لیست خالی:

```csharp
List<int> numbers = new List<int>();
```

لیست با مقادیر اولیه یا **collection initializer**:

```csharp
List<int> numbers = new List<int> { 10, 20, 30 };
```

ترتیب حفظ می‌شود: index صفر مقدار ۱۰، index یک مقدار ۲۰ و index دو مقدار ۳۰ دارد.

### افزودن با `Add`

```csharp
List<int> numbers = new List<int>();

numbers.Add(10);
numbers.Add(20);
numbers.Add(30);

Console.WriteLine(numbers.Count); // 3
```

هر `Add` عضو را به انتهای لیست اضافه می‌کند.

### افزودن در جای مشخص با `Insert`

```csharp
List<int> numbers = new List<int> { 10, 30 };

numbers.Insert(1, 20);

// 10, 20, 30
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

مقدار ۲۰ در index یک قرار می‌گیرد و عضوهای بعدی یک خانه جلو می‌روند. index باید در محدودهٔ مجاز باشد.

### افزودن چند عضو با `AddRange`

```csharp
List<int> first = new List<int> { 10, 20 };
List<int> second = new List<int> { 30, 40 };

first.AddRange(second);

// first: 10, 20, 30, 40
```

`AddRange` همهٔ اعضای مجموعهٔ دوم را به انتهای لیست اول اضافه می‌کند؛ لیست دوم تغییر نمی‌کند.

## ۱۵. خواندن و تغییر با index

مانند آرایه می‌توانیم عضو موجود را بخوانیم یا تغییر دهیم:

```csharp
List<int> numbers = new List<int> { 10, 20, 30 };

Console.WriteLine(numbers[0]); // 10

numbers[1] = 25;
Console.WriteLine(numbers[1]); // 25
```

اما یک تفاوت مهم وجود دارد: index فقط برای **عضوی که واقعاً وجود دارد** قابل استفاده است. این کد خطا می‌دهد:

```csharp
List<int> numbers = new List<int>();

// numbers[0] = 10; // خطا: هنوز عضو صفر وجود ندارد.
numbers.Add(10);     // راه درست برای ساخت اولین عضو
numbers[0] = 15;     // حالا می‌توان عضو موجود را تغییر داد.
```

`Count` تعداد عضوهای واقعی است. آخرین index معتبر همیشه `Count - 1` است.

```csharp
List<string> names = new List<string> { "Sara", "Ali", "Nima" };

for (int i = 0; i < names.Count; i++)
{
    Console.WriteLine($"{i}: {names[i]}");
}
```

اگر فقط مقدارها را می‌خواهیم، `foreach` ساده‌تر است:

```csharp
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

## ۱۶. حذف از List

### `Remove`: حذف بر اساس مقدار

```csharp
List<int> numbers = new List<int> { 5, 10, 5, 20 };

bool removed = numbers.Remove(5);

Console.WriteLine(removed); // True
// نتیجه: 10, 5, 20
```

`Remove(5)` اولین مقدار ۵ را حذف می‌کند و یک `bool` برمی‌گرداند که موفقیت حذف را نشان می‌دهد. اگر مقدار وجود نداشته باشد، `false` برمی‌گرداند.

### `RemoveAt`: حذف بر اساس index

```csharp
List<string> names = new List<string> { "Sara", "Ali", "Nima" };

names.RemoveAt(1);

// Sara, Nima
```

index باید معتبر باشد؛ پس برای ورودی کاربر ابتدا محدوده را بررسی کن:

```csharp
int index = 1;

if (index >= 0 && index < names.Count)
{
    names.RemoveAt(index);
}
```

### `Clear`: حذف همهٔ عضوها

```csharp
names.Clear();
Console.WriteLine(names.Count); // 0
```

خود شیء لیست هنوز وجود دارد و می‌توان دوباره به آن `Add` کرد؛ فقط اعضای آن پاک شده‌اند.

## ۱۷. جست‌وجو، بررسی و مرتب‌سازی

### `Contains`

```csharp
List<int> numbers = new List<int> { 10, 20, 30 };

Console.WriteLine(numbers.Contains(20)); // True
Console.WriteLine(numbers.Contains(99)); // False
```

### `IndexOf`

```csharp
int index = numbers.IndexOf(30);
Console.WriteLine(index); // 2

int missingIndex = numbers.IndexOf(99);
Console.WriteLine(missingIndex); // -1
```

`-1` یعنی مقدار پیدا نشده است.

### `Sort`

```csharp
List<int> scores = new List<int> { 17, 12, 20, 15 };

scores.Sort();

// 12, 15, 17, 20
foreach (int score in scores)
{
    Console.WriteLine(score);
}
```

`Sort` خودِ لیست را تغییر می‌دهد و اعداد را به‌صورت صعودی مرتب می‌کند.

### `Find` در حد آشنایی

`Find` اولین عضوی را برمی‌گرداند که یک شرط را قبول کند. نوشتن شرط در این شکل از lambda استفاده می‌کند که هنوز موضوع اصلی دوره نیست:

```csharp
List<int> scores = new List<int> { 12, 17, 20, 15 };

int firstHighScore = scores.Find(score => score >= 18);

Console.WriteLine(firstHighScore); // 20
```

عبارت `score => score >= 18` یعنی «برای هر score بررسی کن آیا حداقل ۱۸ است». اگر عددی پیدا نشود، برای `int` مقدار پیش‌فرض یعنی `0` برمی‌گردد؛ پس در مسئله‌ای که صفر می‌تواند دادهٔ واقعی باشد باید با دقت بیشتری نتیجه را بررسی کنیم. فعلاً می‌توانی همان جست‌وجوی `foreach` جلسه‌های قبل را نیز استفاده کنی.

## ۱۸. List پشت صحنه چگونه رشد می‌کند؟

در پیاده‌سازی `List<T>`، داده‌ها در یک آرایهٔ داخلی نگه‌داری می‌شوند. دو عدد مهم داریم:

- `Count`: چند عضو واقعی اکنون در لیست است.
- `Capacity`: آرایهٔ داخلی بدون resize چند عضو را می‌تواند نگه دارد.

```csharp
List<int> numbers = new List<int>();

Console.WriteLine(numbers.Count);
Console.WriteLine(numbers.Capacity);

numbers.Add(10);
numbers.Add(20);
numbers.Add(30);

Console.WriteLine(numbers.Count);    // 3
Console.WriteLine(numbers.Capacity); // حداقل 3، مقدار دقیق به پیاده‌سازی وابسته است.
```

وقتی `Add` انجام می‌دهیم و ظرفیت کافی نیست، List به‌طور خودکار:

1. آرایهٔ داخلی بزرگ‌تری تهیه می‌کند؛
2. عضوهای قبلی را به آن منتقل می‌کند؛
3. عضو تازه را اضافه می‌کند؛
4. آرایهٔ داخلی جدید را جایگزین قبلی می‌کند.

آرایهٔ قبلی «همان لحظه توسط List پاک» نمی‌شود؛ اگر دیگر referenceای به آن نباشد، در آینده برای Garbage Collector قابل جمع‌آوری است. همچنین نباید در منطق برنامه به عددی مثل «همیشه از ۴ شروع می‌شود» یا «همیشه دقیقاً دو برابر می‌شود» وابسته شویم؛ سیاست رشد یک جزئیات پیاده‌سازی است. تضمین مهم این است که ظرفیت در صورت نیاز افزایش پیدا می‌کند و `Count` تعداد واقعی عضوها را نشان می‌دهد.

رشد خودکار رایگان نیست: هنگام تغییر ظرفیت، آرایهٔ تازه و کپی لازم است. با این حال برای استفادهٔ معمول، `List` این پیچیدگی را از ما پنهان می‌کند و کار با تعداد متغیر داده‌ها را بسیار ساده‌تر می‌سازد.

## ۱۹. List از شیءهای Student

لیست فقط برای عدد نیست. می‌توانیم دانشجوها را نگه داریم:

```csharp
List<Student> students = new List<Student>();

students.Add(new Student(1, "Sara"));
students.Add(new Student(2, "Ali"));
students.Add(new Student(3, "Nima"));

foreach (Student student in students)
{
    Console.WriteLine(student);
}
```

افزودن یک دانشجو از ورودی کاربر:

```csharp
Console.Write("Student id: ");
string idText = Console.ReadLine() ?? "";

Console.Write("Student name: ");
string name = Console.ReadLine() ?? "";

if (int.TryParse(idText, out int id) && name != "")
{
    students.Add(new Student(id, name));
    Console.WriteLine("Student added.");
}
else
{
    Console.WriteLine("Invalid data.");
}
```

با `List<Student>` دیگر لازم نیست از ابتدا تعداد دقیق دانشجوها را بدانیم یا پس از هر افزودن آرایهٔ تازه بسازیم.

## ۲۰. آشنایی کوتاه با Queue

`Queue` یا صف برای داده‌هایی مناسب است که باید به ترتیب ورود پردازش شوند. مثال واقعی آن صف نانوایی یا بانک است: کسی که زودتر وارد شده، زودتر خدمت می‌گیرد. این قانون **FIFO** نام دارد:

> First In, First Out — اولین ورودی، اولین خروجی.

```csharp
Queue<string> customers = new Queue<string>();

customers.Enqueue("Sara");
customers.Enqueue("Ali");
customers.Enqueue("Nima");

Console.WriteLine(customers.Count); // 3
Console.WriteLine(customers.Peek()); // Sara؛ فقط مشاهده

string firstCustomer = customers.Dequeue();
Console.WriteLine(firstCustomer);    // Sara؛ حذف و برگرداندن
Console.WriteLine(customers.Peek()); // Ali
Console.WriteLine(customers.Count);  // 2
```

| عضو | کار |
| --- | --- |
| `Enqueue` | اضافه کردن به انتهای صف |
| `Dequeue` | حذف و برگرداندن قدیمی‌ترین عضو |
| `Peek` | مشاهدهٔ قدیمی‌ترین عضو بدون حذف |
| `Count` | تعداد عضوهای فعلی |

`Dequeue` و `Peek` روی صف خالی خطا می‌دهند، پس در سطح فعلی ابتدا `Count` را بررسی کن:

```csharp
if (customers.Count > 0)
{
    Console.WriteLine(customers.Dequeue());
}
```

تمرکز تمرین‌های این جلسه روی `List` است؛ `Queue` را فعلاً فقط به‌عنوان نمونه‌ای از ساختارهای دادهٔ دیگر بشناس.

## ۲۱. تمرین‌های مرحله‌ای

1. با زبان خودت توضیح بده چرا finalizer محل مناسبی برای ذخیرهٔ ضروری در دیتابیس نیست و چه کسی زمان اجرای آن را تعیین می‌کند.
2. برای کلاس `Team` یک indexer روی آرایهٔ دو عضو بنویس تا `team[0]` یک `Student` برگرداند. توضیح بده چرا `Team` واقعاً آرایه نشده است.
3. interface به نام `INotificationService` با متد `Send(string message)` بساز. کلاس‌های `SmsNotificationService` و `EmailNotificationService` آن را پیاده کنند.
4. یک متغیر از نوع `INotificationService` بساز، ابتدا شیء SMS و سپس شیء Email را در آن قرار بده و همان `Send` را صدا بزن.
5. interface دیگری به نام `ILogger` بساز. یک کلاس را طوری بنویس که هم `INotificationService` و هم `ILogger` را پیاده کند.
6. توضیح بده برای `Vehicle` با کد و وضعیت مشترک، abstract class مناسب‌تر است یا interface؛ سپس برای قابلیت «قابل چاپ بودن» تصمیم جداگانه بگیر.
7. extension method به نام `IsLongName` برای `string` بنویس که یک `int minimumLength` هم بگیرد و `bool` برگرداند.
8. متد `NormalizeIranianMobile` را با پنج شکل معتبر شمارهٔ همراه جلسهٔ سوم و چند ورودی نامعتبر امتحان کن.
9. یک `List<int>` بساز و با `Add`, `Insert`, `Remove`, `RemoveAt`, `Contains`, `IndexOf` و `Sort` روی آن کار کن. بعد از هر مرحله، لیست را چاپ کن.
10. یک لیست خالی بساز و عمداً `numbers[0] = 10` را امتحان کن. پس از دیدن خطا، با `Add` اصلاحش کن و دلیل را بنویس.
11. برنامه‌ای بنویس که نام دانشجوها را تا وقتی کاربر `end` وارد نکرده در `List<string>` ذخیره کند و در پایان تعداد و همهٔ نام‌ها را چاپ کند.
12. یک صف سه‌نفره بساز. قبل و بعد از `Dequeue`، نتیجهٔ `Peek` و `Count` را پیش‌بینی و سپس اجرا کن.

## جمع‌بندی

Finalizer ابزار روزمرهٔ مدیریت حافظه نیست و زمان اجرای آن در اختیار Garbage Collector است. Indexer یک property پارامتردار برای دسترسی طبیعی با `[]` است. Interface قرارداد قابلیت را تعریف می‌کند و چند کلاس می‌توانند آن را با روش‌های متفاوت پیاده کنند؛ یک کلاس نیز می‌تواند چند interface داشته باشد. Extension method یک متد static است که با پارامتر اولِ `this` از بیرون شبیه متد نمونه‌ای دیده می‌شود. `List` ترتیب و دسترسی با index را حفظ می‌کند، ولی برخلاف آرایه با `Add` و `Remove` تغییر اندازه می‌دهد و رشد آرایهٔ داخلی را خودش مدیریت می‌کند. `Queue` نیز داده‌ها را با ترتیب FIFO پردازش می‌کند.

## منابع مطالعهٔ بیشتر

- [Microsoft Learn · Finalizers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers) برای محدودیت‌ها و زمان نامعین finalization.
- [Microsoft Learn · Indexers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/indexers/) برای تعریف `this[...]`.
- [Microsoft Learn · Interfaces](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces) و [Default interface methods](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interface-implementation/default-interface-methods-versions) برای مطالعهٔ آینده.
- [Microsoft Learn · Extension methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods).
- [Microsoft Learn · `List<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1) و [`Capacity`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1.capacity).
- [Microsoft Learn · `Queue<T>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.queue-1).

[→ جلسهٔ قبل: static، overload، sealed و abstraction](session-10.md) · [بازگشت به فهرست جلسات](index.md)
