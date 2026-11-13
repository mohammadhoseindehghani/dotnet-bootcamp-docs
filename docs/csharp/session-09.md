---
title: جلسهٔ نهم · ارث‌بری، چندریختی و اعضای static
---

# جلسهٔ نهم · ساخت نوع‌های تخصصی از یک نوع عمومی

<div class="lesson-intro">تاریخ برگزاری: <bdi dir="ltr">1405/08/21</bdi><br>ادامهٔ OOP با ارث‌بری، سطح دسترسی <code>protected</code>، زنجیرهٔ سازنده‌ها، <code>virtual</code> و <code>override</code>، چندریختی و کلاس‌های <code>static</code>.</div>

در جلسه‌های قبل `Student` را به‌عنوان یک نوع مستقل ساختیم. حالا در سامانهٔ دانشگاه، دانشجو و استاد هر دو **کاربر** هستند: نام کاربری و رمز عبور دارند و می‌توانند وارد سامانه شوند. آیا باید این بخش‌های مشترک را در هر دو کلاس دوباره بنویسیم؟ ارث‌بری یکی از ابزارهایی است که برای چنین رابطه‌ای در اختیار داریم.

مثال‌های هر بخش مستقل‌اند. فایل‌های معرفی‌شده در همان بخش را کنار `Program.cs` قرار بده و نسخه‌های مختلف یک کلاس را هم‌زمان در یک پروژه نگذار.

## ۱. ارث‌بری در دنیای واقعی و در برنامه

در زندگی روزمره می‌گوییم فرزند ممکن است بعضی ویژگی‌ها را از پدر و مادر به ارث ببرد؛ مثلاً رنگ چشم یا بعضی ویژگی‌های ظاهری. فرزند در عین شباهت، خصوصیات و رفتارهای خودش را هم دارد. این تشبیه برای شروع مفید است، اما ارث‌بری در برنامه دقیقاً ژنتیک یا رابطهٔ خانوادگی نیست.

در برنامه‌نویسی، **Inheritance (ارث‌بری)** یعنی یک کلاس تخصصی‌تر، داده‌ها و رفتارهای یک کلاس عمومی‌تر را دریافت کند و در صورت نیاز چیزهای تازه‌ای به آن اضافه کند یا بعضی رفتارهای قابل تغییر را به شکل مخصوص خودش انجام دهد.

```text
                    User
          username / ChangePassword
                   /    \
                  /      \
             Student    Teacher
          studentNumber  personnelCode
```

- `User` کلاس پایه، والد یا **base class** است.
- `Student` و `Teacher` کلاس مشتق‌شده، فرزند یا **derived class** هستند.
- دانشجو **نوعی کاربر است** و استاد نیز **نوعی کاربر است**.
- شمارهٔ دانشجویی فقط به `Student` و کد پرسنلی فقط به `Teacher` تعلق دارد.

### چرا ارث‌بری مفید است؟

اگر نام کاربری و رفتار مشترک را در `Student` و `Teacher` جداگانه بنویسیم، کد تکرار می‌شود. اگر قانون مشترک تغییر کند، باید چند جای مختلف را اصلاح کنیم. کلاس `User` می‌تواند مفهوم مشترک را یک‌بار تعریف کند و کلاس‌های تخصصی آن را دوباره استفاده کنند.

ارث‌بری می‌تواند این کمک‌ها را بدهد:

1. **استفادهٔ دوباره از کد مشترک:** داده و رفتار مشترک یک‌بار در کلاس پایه نوشته می‌شود.
2. **مدل روشن‌تر:** کد نشان می‌دهد دانشجو و استاد هر دو نوعی کاربرند.
3. **تخصصی کردن:** هر فرزند می‌تواند عضوهای مخصوص خودش را اضافه کند.
4. **رفتار متفاوت زیر یک عنوان مشترک:** همه `ChangePassword` دارند، ولی قانون دانشجو و استاد می‌تواند فرق کند.
5. **کار گروهی با نوع پایه:** می‌توانیم آرایه‌ای از `User` داشته باشیم که هم دانشجو و هم استاد در آن قرار بگیرند.

### چه زمانی ارث‌بری انتخاب خوبی است؟

پیش از ارث‌بری جملهٔ «X نوعی Y است» را امتحان کن:

| رابطه | جمله | نتیجهٔ ابتدایی |
| --- | --- | --- |
| `Student : User` | دانشجو نوعی کاربر سامانه است. | در این مدل منطقی است. |
| `Teacher : User` | استاد نوعی کاربر سامانه است. | در این مدل منطقی است. |
| `Car : Vehicle` | خودرو نوعی وسیلهٔ نقلیه است. | می‌تواند منطقی باشد. |
| `SavingsAccount : BankAccount` | حساب پس‌انداز نوعی حساب بانکی است. | می‌تواند منطقی باشد. |
| `Car : Engine` | خودرو نوعی موتور است. | نادرست؛ خودرو **موتور دارد**. |
| `Student : Course` | دانشجو نوعی درس است. | نادرست؛ دانشجو **درس انتخاب می‌کند**. |

وجود شباهت به‌تنهایی کافی نیست. اگر رابطه «دارد» یا «استفاده می‌کند» باشد، معمولاً باید آن موجودیت را به‌عنوان یک عضو یا ارتباط جدا در نظر بگیریم، نه والد. مثلاً دانشگاه استاد دارد، اما `University : Teacher` معنی ندارد.

> ارث‌بری برای حذف هر کد تکراری نیست. ابتدا باید یک رابطهٔ واقعی و پایدارِ «نوعی از» در مدل مسئله وجود داشته باشد.

## ۲. اولین ارث‌بری در C#

پس از نام کلاس فرزند، `:` و نام کلاس پایه را می‌نویسیم.

**`User.cs`**

```csharp
public class User
{
    public string Username { get; set; } = "";

    public void ShowUsername()
    {
        Console.WriteLine(Username);
    }
}
```

**`Student.cs`**

```csharp
public class Student : User
{
    public string StudentNumber { get; set; } = "";
}
```

**`Teacher.cs`**

```csharp
public class Teacher : User
{
    public string PersonnelCode { get; set; } = "";
}
```

**`Program.cs`**

```csharp
Student student = new Student();
student.Username = "sara.student";
student.StudentNumber = "40101";
student.ShowUsername();

Teacher teacher = new Teacher();
teacher.Username = "ali.teacher";
teacher.PersonnelCode = "T-120";
teacher.ShowUsername();
```

`Student` خودش `Username` و `ShowUsername` را ننوشته، اما آن‌ها را از `User` به ارث برده است. هم‌زمان عضو مخصوص خودش، یعنی `StudentNumber`، را هم دارد. سازنده‌های کلاس پایه به ارث نمی‌رسند؛ کمی بعد تفاوت این موضوع را می‌بینیم.

C# برای کلاس‌ها **single inheritance** دارد: یک کلاس فقط می‌تواند مستقیماً از یک کلاس پایه ارث ببرد. البته زنجیره ممکن است ادامه پیدا کند؛ مثلاً یک نوع خاص‌تر از `Student` می‌تواند عضوهای `Student` و `User` را دریافت کند. فعلاً به همان یک سطح اکتفا می‌کنیم.

## ۳. فرزند به کدام اعضای والد دسترسی دارد؟

عضو `public` کلاس پایه از بیرون و درون فرزند در دسترس است. عضو `private` فقط در خود کلاس پایه قابل دسترسی است؛ کلاس فرزند نمی‌تواند مستقیم از آن استفاده کند. برای عضوی که باید از بیرون پنهان باشد اما کلاس‌های فرزند به آن دسترسی داشته باشند، می‌توانیم از `protected` استفاده کنیم.

| سطح دسترسی عضو پایه | داخل `User` | داخل `Student` | داخل `Program.cs` |
| --- | --- | --- | --- |
| `public` | بله | بله | بله |
| `protected` | بله | بله | خیر |
| `private` | بله | خیر | خیر |

### مثال `protected`

**`User.cs`**

```csharp
public class User
{
    protected string username = "";

    public void SetUsername(string value)
    {
        if (value != "")
        {
            username = value;
        }
    }
}
```

**`Student.cs`**

```csharp
public class Student : User
{
    public string StudentNumber { get; set; } = "";

    public void PrintProfile()
    {
        Console.WriteLine($"{username} - {StudentNumber}");
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student();
student.SetUsername("sara.student");
student.StudentNumber = "40101";
student.PrintProfile();

// student.username = "other"; // خطا: protected از Program قابل دسترسی نیست.
```

`Student` می‌تواند در بدنهٔ خودش `username` را ببیند، چون فرزند `User` است؛ `Program.cs` نمی‌تواند مستقیم آن را تغییر دهد. پس `protected` از `public` محدودتر و از `private` برای فرزند قابل‌دسترس‌تر است.

اما هر فیلدی را برای راحتی `protected` نکن. وقتی فرزند مستقیم فیلد والد را تغییر می‌دهد، ممکن است قانون‌های والد را دور بزند. در بسیاری از طراحی‌ها، فیلد `private` باقی می‌ماند و والد یک property یا متد `protected` کنترل‌شده در اختیار فرزند می‌گذارد. این جلسه `protected` را ساده نشان می‌دهد تا معنی سطح دسترسی روشن شود.

## ۴. سازندهٔ والد و خطای کلاس فرزند

فرض کن هر `User` باید هنگام ساخته شدن نام کاربری و رمز اولیه داشته باشد. پس `User` فقط یک سازندهٔ پارامتردار دارد:

**`User.cs`**

```csharp
public class User
{
    public string Username { get; private set; }
    protected string password;

    public User(string username, string initialPassword)
    {
        Username = username;
        password = initialPassword;
    }
}
```

اگر فرزند را این‌طور بنویسیم، کامپایل نمی‌شود:

```csharp
public class Student : User
{
    public string StudentNumber { get; set; } = "";
}
```

چرا؟ هر سازندهٔ فرزند باید قبل از آماده کردن بخشِ مخصوص فرزند، **بخش `User` همان شیء** را هم بسازد. وقتی چیزی ننویسیم، کامپایلر تلاش می‌کند `User()` را صدا بزند؛ اما `User` سازندهٔ بدون پارامتر ندارد. تنها سازندهٔ موجود دو ورودی می‌خواهد، پس خطای کامپایل می‌گیریم.

```text
new Student(...)
      │
      ├─ اول: ساخت بخش User با username و password
      └─ بعد: ساخت بخش Student با studentNumber
```

فرزند سازندهٔ والد را **به ارث نمی‌برد**. فرزند سازندهٔ خودش را دارد و با `base(...)` مشخص می‌کند کدام سازندهٔ والد اجرا شود.

**`Student.cs` — نسخهٔ درست**

```csharp
public class Student : User
{
    public string StudentNumber { get; private set; }

    public Student(
        string username,
        string initialPassword,
        string studentNumber)
        : base(username, initialPassword)
    {
        StudentNumber = studentNumber;
    }
}
```

**`Teacher.cs`**

```csharp
public class Teacher : User
{
    public string PersonnelCode { get; private set; }

    public Teacher(
        string username,
        string initialPassword,
        string personnelCode)
        : base(username, initialPassword)
    {
        PersonnelCode = personnelCode;
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student("sara", "student-pass", "40101");
Teacher teacher = new Teacher("ali", "teacher-pass", "T-120");

Console.WriteLine(student.Username);
Console.WriteLine(student.StudentNumber);
Console.WriteLine(teacher.Username);
Console.WriteLine(teacher.PersonnelCode);
```

`base(username, initialPassword)` پیش از بدنهٔ سازندهٔ `Student` اجرا می‌شود. بنابراین `User` نام کاربری و رمز را آماده می‌کند و بعد `StudentNumber` مقدار می‌گیرد. این رمزها فقط دادهٔ تمرینی‌اند؛ در سامانهٔ واقعی رمز عبور را به شکل متن ساده نگه نمی‌داریم.

### اگر والد سازندهٔ بدون پارامتر داشته باشد

اگر `User()` وجود داشته باشد و در سازندهٔ فرزند `base(...)` ننویسیم، `base()` به‌صورت ضمنی اجرا می‌شود. نوشتن این دو از نظر انتخاب سازندهٔ والد یک معنی دارد:

```csharp
public Student()
{
}
```

```csharp
public Student() : base()
{
}
```

ولی در مثال اصلی ما `User()` وجود ندارد، پس باید آرگومان‌های لازم را صریح با `base(username, initialPassword)` بفرستیم.

## ۵. چند سازنده، `this` و `base`

یک کلاس فرزند می‌تواند چند سازنده داشته باشد. با `this(...)` از یک سازنده، **سازندهٔ دیگری در همان کلاس** را صدا می‌زنیم. با `base(...)`، **سازندهٔ کلاس والد** را صدا می‌زنیم.

**`User.cs`**

```csharp
public class User
{
    public string Username { get; private set; }
    protected string password;

    public User(string username, string initialPassword)
    {
        Username = username;
        password = initialPassword;
    }
}
```

**`Student.cs`**

```csharp
public class Student : User
{
    public string StudentNumber { get; private set; }
    public string Major { get; private set; }

    public Student(
        string username,
        string initialPassword,
        string studentNumber)
        : this(username, initialPassword, studentNumber, "Unknown")
    {
    }

    public Student(
        string username,
        string initialPassword,
        string studentNumber,
        string major)
        : base(username, initialPassword)
    {
        StudentNumber = studentNumber;
        Major = major;
    }
}
```

**`Program.cs`**

```csharp
Student first = new Student("sara", "pass-123", "40101");
Student second = new Student("nima", "pass-456", "40102", "Computer");

Console.WriteLine(first.Major);  // Unknown
Console.WriteLine(second.Major); // Computer
```

مسیر ساخت `first` چنین است:

```text
Student با 3 ورودی
        ↓ this(...)
Student با 4 ورودی
        ↓ base(...)
User با 2 ورودی
        ↓
بدنهٔ Student چهارورودی
        ↓
بدنهٔ Student سه‌ورودی
```

در هر سازنده فقط یکی از `this(...)` یا `base(...)` را می‌توان به‌عنوان initializer نوشت. زنجیره در نهایت باید به سازندهٔ والد برسد. `this` در اینجا جلوی تکرار مقداردهی `StudentNumber` و `Major` را می‌گیرد.

## ۶. `virtual` و `override`: یک رفتار مشترک، اجرای متفاوت

اکنون `User` یک رفتار `ChangePassword` دارد. قانون عمومی می‌گوید رمز حداقل ۶ نویسه باشد، اما برای دانشجو حداقل ۸ و برای استاد حداقل ۱۰ نویسه می‌خواهیم.

- `virtual` روی متد والد یعنی فرزند **اجازه دارد** اجرای مخصوص خودش را ارائه کند.
- `override` روی متد فرزند یعنی این همان رفتار مجازیِ والد است که برای این نوع بازنویسی می‌شود.
- نام، نوع خروجی و پارامترهای متد override باید با متد والد هماهنگ باشند.

**`User.cs`**

```csharp
public class User
{
    public string Username { get; private set; }
    protected string password;

    public User(string username, string initialPassword)
    {
        Username = username;
        password = initialPassword;
    }

    public virtual bool ChangePassword(string newPassword)
    {
        if (newPassword.Length < 6)
        {
            return false;
        }

        password = newPassword;
        return true;
    }

    public int GetPasswordLength()
    {
        return password.Length;
    }
}
```

**`Student.cs`**

```csharp
public class Student : User
{
    public Student(string username, string initialPassword)
        : base(username, initialPassword)
    {
    }

    public override bool ChangePassword(string newPassword)
    {
        if (newPassword.Length < 8)
        {
            return false;
        }

        password = newPassword;
        return true;
    }
}
```

**`Teacher.cs`**

```csharp
public class Teacher : User
{
    public Teacher(string username, string initialPassword)
        : base(username, initialPassword)
    {
    }

    public override bool ChangePassword(string newPassword)
    {
        if (newPassword.Length < 10)
        {
            return false;
        }

        password = newPassword;
        return true;
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student("sara", "student-old");
Teacher teacher = new Teacher("ali", "teacher-old");

Console.WriteLine(student.ChangePassword("1234567"));  // False
Console.WriteLine(student.ChangePassword("12345678")); // True

Console.WriteLine(teacher.ChangePassword("123456789"));  // False
Console.WriteLine(teacher.ChangePassword("1234567890")); // True
```

متد `GetPasswordLength` عمومی است تا نتیجهٔ تمرین را بدون چاپ خودِ رمز بررسی کنیم. نگه‌داری متن سادهٔ رمز فقط برای نمایش مفهوم است و الگوی امنیتیِ برنامهٔ واقعی نیست.

### استفاده از رفتار والد در override

گاهی فرزند نمی‌خواهد تمام رفتار را از صفر بنویسد؛ می‌خواهد کار والد را انجام دهد و یک کار اضافه کند. در بدنهٔ متد override می‌توانیم `base.ChangePassword(...)` را صدا بزنیم:

```csharp
public override bool ChangePassword(string newPassword)
{
    if (newPassword.Length < 8)
    {
        return false;
    }

    bool changed = base.ChangePassword(newPassword);

    if (changed)
    {
        Console.WriteLine("Student password changed.");
    }

    return changed;
}
```

اینجا قانون مخصوص دانشجو اول بررسی می‌شود، سپس اجرای والد واقعاً مقدار را تغییر می‌دهد. `base` در سازنده برای انتخاب سازندهٔ والد و در متد برای دسترسی به اجرای والد استفاده می‌شود؛ کاربرد را از محل نوشتن آن می‌فهمیم.

## ۷. Polymorphism یا چندریختی

**Polymorphism (چندریختی)** یعنی بتوانیم چند نوع مرتبط را از راه نوع عمومی‌تر ببینیم، ولی هنگام اجرای رفتار مجازی، رفتار واقعیِ همان شیء انجام شود. یک شیء `Student` هم دانشجو است و هم نوعی `User`؛ بنابراین می‌توانیم آن را در متغیری از نوع `User` قرار دهیم:

```csharp
User user = new Student("sara", "student-old");
bool changed = user.ChangePassword("12345678");
```

نوع متغیر `user` برابر `User` است، اما شیء واقعی که با `new` ساخته‌ایم `Student` است. چون `ChangePassword` در والد `virtual` و در فرزند `override` شده، نسخهٔ `Student` اجرا می‌شود. تصمیم بر اساس **نوع واقعی شیء در زمان اجرا** است.

### قدرت اصلی: کار با چند نوع از یک مسیر

فایل‌های `User.cs`، `Student.cs` و `Teacher.cs` بخش قبل را نگه دار و این `Program.cs` را اجرا کن:

```csharp
User[] users =
{
    new User("normal", "normal-old"),
    new Student("sara", "student-old"),
    new Teacher("ali", "teacher-old")
};

string[] newPasswords =
{
    "1234567",
    "1234567",
    "1234567"
};

for (int i = 0; i < users.Length; i++)
{
    bool changed = users[i].ChangePassword(newPasswords[i]);
    Console.WriteLine($"{users[i].Username}: {changed}");
}
```

خروجی:

```text
normal: True
sara: False
ali: False
```

همهٔ خانه‌های آرایه از دید کد `User` هستند و یک دستور مشترک روی آن‌ها اجرا می‌شود. با این حال شیء معمولی قانون ۶ نویسه، دانشجو قانون ۸ نویسه و استاد قانون ۱۰ نویسه را اجرا می‌کند. لازم نبود در حلقه بپرسیم «اگر دانشجو بود این کار، اگر استاد بود آن کار». این همان بخش از چندریختی است که در این جلسه با **method overriding** یاد می‌گیریم.

### override با overload یکی نیست

این دو واژه شبیه‌اند، اما یک مفهوم نیستند:

| مفهوم | معنی کوتاه | مثال |
| --- | --- | --- |
| Overriding | فرزند، اجرای متد `virtual` والد را با همان امضا تخصصی می‌کند. | `Student.ChangePassword(string)` |
| Overloading | چند متد هم‌نام با فهرست پارامترهای متفاوت داریم. | `Print(string)` و `Print(string, int)` |

در جلسهٔ قبل overload سازنده‌ها را مختصر دیدیم. **Operator overloading** و **abstract class / abstract method** نیز به خانوادهٔ مباحث شی‌ءگرایی مربوط‌اند، اما هنوز تدریس نشده‌اند و جزئیاتشان را به جلسهٔ مربوط می‌سپاریم. برای این جلسه باید ارتباط `virtual`، `override`، نوع پایه و نوع واقعی شیء را خوب بفهمی.

## ۸. کلاس `static` چیست؟

گاهی یک کلاس قرار نیست نمایندهٔ یک موجودیت با نمونه‌های جدا باشد. فقط می‌خواهیم چند عملیات مرتبط را کنار هم بگذاریم؛ عملیات‌هایی که تمام دادهٔ لازم را از پارامترها می‌گیرند و به وضعیت یک شیء خاص نیاز ندارند. در این حالت یک **static class** می‌تواند مناسب باشد.

**`PasswordRules.cs`**

```csharp
public static class PasswordRules
{
    public static bool HasMinimumLength(string password, int minimumLength)
    {
        return password.Length >= minimumLength;
    }

    public static bool ContainsNumber(string password)
    {
        foreach (char character in password)
        {
            if (character >= '0' && character <= '9')
            {
                return true;
            }
        }

        return false;
    }
}
```

**`Program.cs`**

```csharp
Console.WriteLine(PasswordRules.HasMinimumLength("pass1234", 8)); // True
Console.WriteLine(PasswordRules.ContainsNumber("pass1234"));      // True
```

نام کلاس را مستقیم می‌نویسیم؛ نمونه نمی‌سازیم:

```csharp
// PasswordRules rules = new PasswordRules(); // خطای کامپایل
```

### محدودیت‌های کلاس static

- با `new` از آن نمونه ساخته نمی‌شود.
- فقط می‌تواند عضوهای `static` داشته باشد؛ فیلد، پراپرتی یا متد نمونه‌ای در آن نداریم.
- سازندهٔ نمونه‌ای ندارد؛ می‌تواند سازندهٔ static مخصوص آماده‌سازی دادهٔ مشترک داشته باشد که موضوع امروز نیست.
- نمی‌تواند کلاس پایهٔ `Student` یا کلاس دیگری باشد و کلاسی هم نمی‌تواند از آن ارث ببرد.
- متد static به `this` یا دادهٔ یک نمونهٔ خاص دسترسی ندارد؛ اگر داده‌ای لازم دارد باید از پارامتر بگیرد یا از عضو static استفاده کند.

کلاس static برای موجودیت‌هایی مثل `Student` مناسب نیست، چون هر دانشجو نام، شماره و وضعیت جدا دارد. برای مجموعه‌ای از عملیات مستقل مثل قوانین سادهٔ رمز یا محاسبات، انتخاب قابل‌فهم‌تری است.

### قبلاً هم کلاس‌های static دیده بودیم

این فراخوانی‌ها برایت آشنا هستند:

```csharp
Console.WriteLine("Hello");
double result = Math.Sqrt(25);
int number = Convert.ToInt32("42");
```

`Console`، `Math` و `Convert` در .NET کلاس‌های static هستند. هیچ‌وقت ننوشته‌ایم `new Math()`؛ عضو را با نام کلاس صدا زده‌ایم. حالا می‌دانیم دلیل شکل `Math.Sqrt(...)` چیست.

## ۹. عضو static در یک کلاس معمولی

یک کلاس معمولی می‌تواند **هم عضوهای نمونه‌ای و هم عضوهای static** داشته باشد. عضو نمونه‌ای به هر شیء جدا تعلق دارد؛ عضو static به خود کلاس تعلق دارد و میان همهٔ نمونه‌ها مشترک است.

**`Student.cs`**

```csharp
public class Student
{
    public static int CreatedCount { get; private set; }

    public string Name { get; private set; }

    public Student(string name)
    {
        Name = name;
        CreatedCount++;
    }

    public void PrintName()
    {
        Console.WriteLine(Name);
    }

    public static void PrintCreatedCount()
    {
        Console.WriteLine(CreatedCount);
    }
}
```

**`Program.cs`**

```csharp
Student first = new Student("Sara");
Student second = new Student("Ali");

first.PrintName();
second.PrintName();

Console.WriteLine(Student.CreatedCount); // 2
Student.PrintCreatedCount();             // 2
```

`Name` برای هر شیء فرق دارد و با `first.Name` یا `second.Name` خوانده می‌شود. `CreatedCount` یک مقدار مشترک برای خود کلاس است و با `Student.CreatedCount` خوانده می‌شود. متد `PrintCreatedCount` static است، پس بدون نمونه صدا زده می‌شود.

متد static نمی‌تواند مستقیم `Name` را بخواند، چون معلوم نیست نامِ **کدام نمونه** منظور است. متد نمونه‌ای `PrintName` این مشکل را ندارد؛ روی `first` یا `second` اجرا می‌شود و می‌داند با کدام شیء کار می‌کند. اگر یک متد static به یک دانشجوی مشخص نیاز داشته باشد، می‌توانیم آن دانشجو را به‌عنوان پارامتر بفرستیم.

> مشترک بودن دادهٔ static قدرت و مسئولیت دارد. هر بخش برنامه که آن را تغییر دهد، نتیجه برای همه دیده می‌شود. فقط داده‌ای را static کن که واقعاً متعلق به کل نوع باشد، نه برای فرار از ساختن شیء.

## ۱۰. تمرین‌های مرحله‌ای

1. کلاس‌های `User` و `Student` بخش اول را بنویس. یک property به `User` و یک property مخصوص به `Student` اضافه کن و هر دو را از شیء دانشجو چاپ کن.
2. در `User` یک فیلد `private` و یک فیلد `protected` بساز. داخل `Student` تلاش کن هر دو را بخوانی و تفاوت خطای کامپایل را توضیح بده.
3. سازندهٔ `User(string username)` بنویس و عمداً در `Student` سازنده ننویس. خطا را بخوان، سپس با سازندهٔ `Student(string username) : base(username)` آن را اصلاح کن.
4. برای `Teacher` دو سازنده بساز: یکی با نام کاربری و کد پرسنلی، دیگری فقط با نام کاربری. با `this(...)` برای حالت دوم کد پیش‌فرض `"Unknown"` بفرست.
5. در متد `ChangePassword` والد حداقل ۶ و در `Student` حداقل ۸ نویسه بخواه. سه رمز با طول ۵، ۷ و ۸ را امتحان و قبل از اجرا نتیجه را حدس بزن.
6. یک `User` و یک `Student` در آرایهٔ `User[]` بگذار. متد overrideشده را در حلقه صدا بزن و توضیح بده چرا نسخهٔ دانشجو اجرا شد.
7. کلاس static به نام `NumberHelper` بساز که متدهای `IsEven(int number)` و `IsPositive(int number)` داشته باشد. بدون `new` هر دو را صدا بزن.
8. به کلاس معمولی `Course` یک عضو نمونه‌ای `Title` و یک عضو static به نام `CreatedCount` اضافه کن. سه درس بساز و تعداد را با نام کلاس چاپ کن.
9. برای هر رابطه تصمیم بگیر ارث‌بری مناسب است یا نه و دلیل بنویس: استاد/کاربر، کتاب/نویسنده، خودرو/وسیلهٔ نقلیه، دانشگاه/دانشجو، حساب پس‌انداز/حساب بانکی.

هنگام حل از خودت بپرس: «آیا فرزند واقعاً نوعی والد است؟»، «کدام عضو باید برای بیرون، فرزند یا فقط خود والد دیده شود؟»، «کدام سازندهٔ والد باید اجرا شود؟» و «رفتار بر اساس نوع واقعی شیء باید فرق کند؟».

## جمع‌بندی

ارث‌بری یک نوع تخصصی را از نوع عمومی‌تر می‌سازد و رابطهٔ «نوعی از» را مدل می‌کند. عضو `protected` در خود کلاس و فرزندان قابل دسترسی است، اما کد عادی بیرون کلاس آن را نمی‌بیند. سازنده‌ها ارث نمی‌رسند؛ سازندهٔ فرزند با `base(...)` والد و با `this(...)` سازندهٔ دیگری از خودش را صدا می‌زند. `virtual` اجازهٔ تغییر رفتار را می‌دهد و `override` اجرای مخصوص فرزند را تعریف می‌کند. وقتی یک `User` به شیء واقعی `Student` اشاره می‌کند، فراخوانی متد مجازی به نسخهٔ دانشجو می‌رسد؛ این شکلِ method overriding از چندریختی است. کلاس static نمونه ندارد، ولی کلاس معمولی می‌تواند عضو static مشترک در کنار عضوهای نمونه‌ای داشته باشد.

## منابع مطالعهٔ بیشتر

- [Microsoft Learn · Inheritance](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/inheritance) برای رابطهٔ کلاس پایه و مشتق‌شده.
- [Microsoft Learn · `base`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/base) و [Using constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/using-constructors) برای زنجیرهٔ سازنده‌ها.
- [Microsoft Learn · Polymorphism](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/polymorphism) برای `virtual`، `override` و نوع واقعی شیء.
- [Microsoft Learn · Static classes and members](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members) برای کلاس static و عضوهای مشترک.

[→ جلسهٔ قبل: سازنده، کپسوله‌سازی و پراپرتی](session-08.md) · [بازگشت به فهرست جلسات](index.md)
