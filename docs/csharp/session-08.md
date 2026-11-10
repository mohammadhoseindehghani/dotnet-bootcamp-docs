---
title: جلسهٔ هشتم · سازنده، کپسوله‌سازی و پراپرتی
---

# جلسهٔ هشتم · ساختن و مراقبت از یک شیء

<div class="lesson-intro">تاریخ برگزاری: <bdi dir="ltr">1405/08/19</bdi><br>ادامهٔ OOP با همان کلاس <code>Student</code>: هنگام ساخت شیء چه اتفاقی می‌افتد و چگونه جلوی مقدارهای نامعتبر را می‌گیریم؟</div>

در [جلسهٔ هفتم](session-07.md) کلاس `Student` را ساختیم، فیلدهایش را در دسترس گذاشتیم و با `new Student()` از آن نمونه گرفتیم. حالا می‌خواهیم **لحظهٔ ساخت** و **راه دسترسی به داده‌ها** را دقیق‌تر بشناسیم. مثال‌های هر بخش، مرحله‌ای از تکامل کلاس هستند؛ هر نمونهٔ `Student.cs` را جداگانه و با `Program.cs` همان بخش اجرا کن، نه همهٔ نسخه‌ها را هم‌زمان.

## ۱. Constructor یا سازنده چیست؟

وقتی می‌نویسیم `new Student()`، یک شیء تازه ساخته می‌شود و **سازنده (constructor / ctor)** آن اجرا می‌شود. سازنده بخشی از کلاس است که کارهای آغازینِ یک نمونه را انجام می‌دهد: مثلاً نام اولیه را بگیرد یا سن را بررسی کند. نام سازنده **دقیقاً مثل نام کلاس** است و **نوع خروجی ندارد**؛ حتی `void` هم نمی‌نویسیم. فراخوانی سازنده در همان عبارت `new` انجام می‌شود.

```text
new Student("Sara", 22)
        │
        └─ ساخت نمونه → اجرای Student("Sara", 22) → شیء آمادهٔ استفاده
```

### سازندهٔ بدون پارامترِ نانوشته

در کلاس جلسهٔ قبل، خودمان سازنده‌ای ننوشته بودیم؛ با این حال `new Student()` کار می‌کرد. **اگر برای یک کلاس هیچ سازنده‌ای ننویسیم، کامپایلر یک سازندهٔ عمومیِ بدون پارامتر برایش فراهم می‌کند.** به آن سازندهٔ بدون پارامتر (`parameterless constructor`) می‌گوییم. فیلدها مقدار اولیهٔ تعریف‌شده در کلاس را می‌گیرند، و فیلدی مثل `int age` که مقدار نداده‌ایم ابتدا `0` است.

**`Student.cs`**

```csharp
public class Student
{
    public string firstName = "";
    public int age;
}
```

**`Program.cs`**

```csharp
Student student = new Student();
Console.WriteLine(student.firstName == ""); // True
Console.WriteLine(student.age);               // 0
```

می‌توانیم سازندهٔ بدون پارامتر را **خودمان** بنویسیم تا هنگام ساخت کاری انجام دهد:

**`Student.cs` — جایگزین نسخهٔ بالا**

```csharp
public class Student
{
    public string firstName = "";
    public int age;

    public Student()
    {
        firstName = "Unknown";
        age = 18;
    }
}
```

اکنون با همان `Program.cs` خروجی نام و سن به‌ترتیب `Unknown` و `18` می‌شود. سازنده یک متد معمولی نیست که بنویسیم `student.Student()`؛ همراه با `new Student()` اجرا می‌شود.

### اگر نام و سن از همان ابتدا لازم باشند

برای بعضی برنامه‌ها، ساخت دانشجو بدون نام و سنِ اولیه معنی ندارد. پارامترهای سازنده کمک می‌کنند فراخواننده این داده‌ها را **هنگام ساخت** بدهد. در مثال زیر برای سن نامعتبر، فعلاً مقدار معتبر `0` را جایگزین می‌کنیم تا یک نمونه با سن منفی نسازیم؛ شیوهٔ گزارش خطا و رد کامل ساخت را بعداً می‌آموزیم.

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    public string firstName = "";
    public int age;

    public Student(string name, int initialAge)
    {
        firstName = name;

        if (initialAge >= 0 && initialAge <= 120)
        {
            age = initialAge;
        }
        else
        {
            age = 0;
        }
    }
}
```

**`Program.cs`**

```csharp
Student sara = new Student("Sara", 22);
Student ali = new Student("Ali", -5);

Console.WriteLine($"{sara.firstName}: {sara.age}"); // Sara: 22
Console.WriteLine($"{ali.firstName}: {ali.age}");   // Ali: 0
```

دقت کن: این شرط **سن را اصلاح می‌کند، ساخت شیء را رد نمی‌کند**. اگر بعداً `ali.age = -5` بنویسیم، این نسخه هنوز اجازه می‌دهد! سازنده فقط آغاز کار است؛ برای مراقبت از تغییرهای بعدی به کپسوله‌سازی نیاز داریم.

داشتن پارامتر `name` یعنی هنگام فراخوانی باید آرگومانی برای نام بدهیم؛ اما هنوز می‌توانیم `""` بدهیم. اگر خالی نبودن نام هم قانون مسئله است، باید آن را جداگانه بررسی کنیم. «اجباری بودن ورودی» و «معتبر بودن مقدار ورودی» دو پرسش متفاوت‌اند.

> **نکتهٔ مهم:** به محض اینکه برای کلاس خودت سازنده‌ای بنویسی، سازندهٔ بدون پارامترِ نانوشته دیگر اضافه نمی‌شود. پس در نسخهٔ بالا `new Student()` خطای کامپایل دارد. اگر هر دو شیوهٔ ساخت را بخواهی، باید سازندهٔ بدون پارامتر را هم خودت بنویسی.

### چند سازنده برای یک کلاس

می‌توانیم سازنده‌هایی با **فهرست پارامترهای متفاوت** داشته باشیم. به داشتن چند عضو هم‌نام با ورودی‌های متفاوت، **overload (اورلود)** می‌گویند؛ فعلاً فقط همین ایده و شیوهٔ انتخاب سازنده را ببین.

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    public string firstName = "";
    public int age;

    public Student()
    {
        firstName = "Unknown";
    }

    public Student(string name, int initialAge)
    {
        firstName = name;
        age = initialAge >= 0 && initialAge <= 120 ? initialAge : 0;
    }
}
```

**`Program.cs`**

```csharp
Student first = new Student();
Student second = new Student("Sara", 22);

Console.WriteLine(first.firstName);  // Unknown
Console.WriteLine(second.firstName); // Sara
```

کامپایلر از **تعداد و نوع ورودی‌ها** می‌فهمد کدام سازنده را اجرا کند. فقط عوض کردن نام پارامترها، overload تازه‌ای نمی‌سازد.

**کجا به درد می‌خورد؟** یک `Course` را می‌توان با عنوان و ظرفیت اولیه ساخت، یک درخواست مرخصی را با نام درخواست‌دهنده و تعداد روزها، یا یک `Student` را با نام و سن. سازنده جای خوبی برای آماده کردن وضعیت اولیه است؛ قوانین مربوط به **تغییرهای بعدی** را هم باید در راه‌های تغییر داده رعایت کنیم.

## ۲. مشکل فیلدهای عمومی چیست؟

تا اینجا برای سادگی همه‌چیز را `public` نوشته بودیم:

**`Student.cs`**

```csharp
public class Student
{
    public string firstName = "";
    public int age;
}
```

**`Program.cs`**

```csharp
Student student = new Student();
student.firstName = "Sara";
student.age = -30;
Console.WriteLine(student.age); // -30
```

از نظر زبان، این کد مجاز است؛ از نظر مسئلهٔ ما، سن منفی دانشجو بی‌معنی است. اگر برنامه در ده نقطه به `age` مقدار بدهد، در هر نقطه باید قانون سن را تکرار کنیم و ممکن است جایی فراموش شود. **Encapsulation (کپسوله‌سازی)** یعنی داده و رفتار مربوط را در کلاس کنار هم بگذاریم و راه دسترسی و تغییر را کنترل کنیم. کلاس مثل یک دفتر ثبت است: مراجعه‌کننده درخواست می‌دهد، «منشی» قانون را بررسی می‌کند، و فقط تغییر مجاز ثبت می‌شود. این تشبیه برای فهم مسیر دسترسی است؛ در کد، کنترل را با اعضای کلاس و سطح دسترسی پیاده می‌کنیم.

### Access modifier یا تعیین‌کنندهٔ سطح دسترسی

`public` و `private` مشخص می‌کنند چه کدی می‌تواند به یک عضو دسترسی پیدا کند:

| واژه | در این درس چه بدانیم؟ |
| --- | --- |
| `public` | کد بیرون کلاس، مثل `Program.cs`، می‌تواند عضو را ببیند و در صورت امکان استفاده کند. |
| `private` | عضو فقط از داخل همان کلاس قابل دسترسی است. |
| `internal` | یکی دیگر از سطح‌های دسترسی در C# است؛ جزئیاتش برای درس‌های بعد. |
| `protected` | یکی دیگر از سطح‌های دسترسی است که با بحث ارث‌بری روشن‌تر می‌شود؛ جزئیاتش برای بعد. |

سطح‌های دسترسی C# به همین چهار واژه محدود نمی‌شوند؛ فعلاً فقط `public` و `private` را **به کار می‌بریم**. اگر پیشِ عضوی در کلاس هیچ سطح دسترسی ننویسیم، پیش‌فرض آن عضو `private` است؛ برای یادگیری، اینجا آن را صریح می‌نویسیم.

## ۳. اولین راه کنترل: فیلد `private` و متدهای Set / Get

نام فیلد خصوصی را `_age` می‌گذاریم تا از نام پارامتر و متد تشخیص داده شود. از بیرون دیگر نمی‌توان مستقیم به آن مقدار داد؛ یک متد عمومی برای درخواست تغییر و یک متد عمومی برای خواندنش می‌نویسیم.

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    private int _age;

    public void SetAge(int newAge)
    {
        if (newAge >= 0 && newAge <= 120)
        {
            _age = newAge;
        }
    }

    public int GetAge()
    {
        return _age;
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student();
student.SetAge(20);
Console.WriteLine(student.GetAge()); // 20

student.SetAge(-5);
Console.WriteLine(student.GetAge()); // 20
// student._age = -5; // خطای کامپایل: فیلد private است.
```

اگر مقدار نامعتبر باشد، این نسخه **مقدار قبلی را نگه می‌دارد**؛ پیام خطا نمی‌دهد. این یک تصمیم سادهٔ آموزشی است و در برنامهٔ واقعی باید روشن کنیم چگونه خطا را به فراخواننده گزارش دهیم. مزیت اصلی: قانون سن در **یک مسیر مشخص** قرار گرفته است. سازندهٔ کلاس هم می‌تواند از همین مسیر استفاده کند:

```csharp
// این سازنده را داخل همان کلاس Student بالا اضافه کن.
public Student(int initialAge)
{
    SetAge(initialAge);
}
```

با این سازنده، `new Student(20)` مجاز است، اما چون دیگر سازندهٔ نانوشته نداریم، `new Student()` موجود نیست؛ اگر مثال قبلی را دوباره اجرا می‌کنی، آن را نیز تغییر بده. فیلد `private` از **داخل خود کلاس**، از جمله سازنده، قابل مقداردهی است. `private` به معنی «هیچ‌کس حتی خود کلاس نمی‌تواند تغییرش دهد» نیست.

## ۴. Property: همان مسیر کنترل، با شکل خواناتر

برای هر داده، نوشتن `SetAge(...)` و `GetAge()` تکراری می‌شود. **Property (پراپرتی)** عضوی است که از بیرون شبیه یک داده خوانده و نوشته می‌شود، ولی درونش می‌تواند کدِ خواندن (`get`) و نوشتن (`set`) داشته باشد. این راه، همان نقش «منشی» را با دستور زبان ساده‌تری انجام می‌دهد.

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    private int _age;

    public int Age
    {
        get
        {
            return _age;
        }
        set
        {
            if (value >= 0 && value <= 120)
            {
                _age = value;
            }
        }
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student();
student.Age = 20;                 // اجرای set؛ value برابر 20 است.
Console.WriteLine(student.Age);   // اجرای get؛ خروجی 20

student.Age = -5;                 // set مقدار نامعتبر را رد می‌کند.
Console.WriteLine(student.Age);   // همچنان 20
```

در `set`، واژهٔ `value` **مقداری است که سمت راست `=` آمده**. `Age` خودِ پراپرتی است و `_age` فیلدی است که واقعاً مقدار را نگه می‌دارد؛ به آن **backing field (فیلد پشتیبان)** می‌گوییم. دقت کن در بدنهٔ `set` باید `_age = value` بنویسیم، نه `Age = value`؛ دومی دوباره همان `set` را صدا می‌زند و این چرخه ادامه پیدا می‌کند.

```text
Program.cs: student.Age = 20
                      ↓
                set → بررسی 20 → _age = 20

Program.cs: student.Age
                      ↓
                get → خواندن _age → 20
```

پراپرتی معمولاً با نامی مانند `Age` و حرف بزرگ شروع می‌شود؛ فیلد پشتیبان معمولاً نامی مانند `_age` دارد. این شیوهٔ رایج نام‌گذاری کمک می‌کند این دو را اشتباه نگیریم.

### خواندن عمومی، تغییر فقط از داخل کلاس

گاهی می‌خواهیم بیرون کلاس سن را **بخوانیم**، اما نتواند آن را مستقیم عوض کند. می‌توانیم `set` را `private` کنیم و تغییر را از متدی که قانون را بررسی می‌کند انجام دهیم:

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    public int Age { get; private set; }

    public void ChangeAge(int newAge)
    {
        if (newAge >= 0 && newAge <= 120)
        {
            Age = newAge;
        }
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student();
student.ChangeAge(21);
Console.WriteLine(student.Age); // 21
// student.Age = -5; // خطای کامپایل: set فقط درون Student در دسترس است.
```

`get` عمومی است، `set` خصوصی. درون `Student`، سازنده یا متد `ChangeAge` می‌توانند `Age` را مقداردهی کنند. اگر همهٔ مسیرهای تغییر را از `ChangeAge` عبور بدهیم، قانون سن یک‌جا می‌ماند. `private set` به‌تنهایی بررسی معتبر بودن را انجام نمی‌دهد؛ فقط دسترسی بیرونی را می‌بندد.

## ۵. Auto property و فیلد پشتیبانِ ناپیدا

اگر برای خواندن و نوشتن **هیچ منطق اضافی** نداریم، این شکل کوتاه کافی است:

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    public string FirstName { get; set; } = "";
    public int Age { get; set; }
}
```

**`Program.cs`**

```csharp
Student student = new Student();
student.FirstName = "Sara";
student.Age = -5;
Console.WriteLine(student.Age); // -5
```

این‌ها **auto property** هستند. کامپایلر پشت صحنه برای هرکدام یک **فیلد پشتیبان خصوصی و بی‌نام برای ما** می‌سازد و `get` و `set` ساده را به آن وصل می‌کند. ما نام فیلد تولیدشده را در کد خودمان نمی‌نویسیم. کوتاه شدن کد به معنی اضافه شدن اعتبارسنجی نیست: `Age` در این مثال هنوز مقدار منفی می‌پذیرد. برای کنترل سن، از پراپرتی بخش قبل با فیلد `_age` و شرط استفاده کن.

### اگر `set` نداشته باشیم چه می‌شود؟

یک پراپرتی فقط با `get` از بیرون **قابل خواندن** است و نمی‌توان به آن مقدار تازه نسبت داد. برای auto property فقط‌خواندنی، می‌توان مقدار اولیه را همان‌جا یا در سازنده تعیین کرد:

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    public string StudentNumber { get; }

    public Student(string number)
    {
        StudentNumber = number;
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student("40101");
Console.WriteLine(student.StudentNumber); // 40101
// student.StudentNumber = "99999"; // خطای کامپایل
```

اینجا هر دانشجو یک شمارهٔ اولیه دارد که از بیرون تغییر نمی‌کند. دقت کن **حذف `set` در auto property** با `private set` یکی نیست: `private set` اجازهٔ تغییر از متدهای خود کلاس را می‌دهد؛ پراپرتیِ فقط دارای `get` را در این شکل می‌توان هنگام تعریف یا داخل سازنده مقداردهی کرد، نه در یک متد معمولی بعدی. این مثال فقط نحوهٔ دسترسی را نشان می‌دهد؛ اگر شمارهٔ نامعتبر نباید پذیرفته شود، باید آن را هنگام ساخت هم بررسی کنیم.

## ۶. `const` در سطح کلاس

بعضی مقدارها برای **همهٔ نمونه‌ها یکسان** هستند و هیچ‌وقت عوض نمی‌شوند؛ مثلاً یک حد ثابتِ قراردادی در برنامه. `const` برای چنین مقدار ثابتی است. مقدارش را باید **همان‌جا هنگام تعریف** بنویسیم و مقدار باید برای کامپایلر از قبل مشخص باشد.

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    public const int MaxAge = 120;

    private int _age;

    public int Age
    {
        get { return _age; }
        set
        {
            if (value >= 0 && value <= MaxAge)
            {
                _age = value;
            }
        }
    }
}
```

**`Program.cs`**

```csharp
Console.WriteLine(Student.MaxAge); // 120

Student student = new Student();
student.Age = 121;
Console.WriteLine(student.Age);    // 0؛ مقدار 121 پذیرفته نشد.
```

برای مقدار عمومی `const` در کلاس، نام **کلاس** را می‌نویسیم: `Student.MaxAge`، چون این مقدار متعلق به یک دانشجوی خاص نیست. `student.MaxAge` شکل درست استفاده نیست. `const` را بعداً نمی‌توان تغییر داد و لازم نیست برای هر شیء یک نسخهٔ قابل تغییر از آن بسازیم. عدد `120` فقط یک **قانون نمونه برای تمرین** است، نه یک قاعدهٔ همیشگی برای همهٔ سامانه‌های دانشجویی.

## ۷. فیلد `readonly`؛ یک بار تعیین، سپس ثابت

اگر مقدار را **هنگام اجرای برنامه** و موقع ساخت هر شیء می‌فهمیم، `const` مناسب نیست. `readonly` را روی **فیلد** می‌گذاریم تا مقدار آن را فقط هنگام تعریف فیلد یا در سازندهٔ همان کلاس تعیین کنیم؛ بعد از ساخت، دیگر نمی‌توانیم به آن فیلد مقدار دیگری نسبت دهیم.

**`Student.cs` — نسخهٔ مستقل**

```csharp
public class Student
{
    public readonly string studentNumber;
    public string firstName = "";

    public Student(string number, string name)
    {
        studentNumber = number;
        firstName = name;
    }

    public void ChangeName(string newName)
    {
        firstName = newName;
        // studentNumber = "other"; // خطای کامپایل
    }
}
```

**`Program.cs`**

```csharp
Student sara = new Student("40101", "Sara");
Student ali = new Student("40102", "Ali");

sara.ChangeName("Sarah");
Console.WriteLine($"{sara.firstName}: {sara.studentNumber}");
Console.WriteLine($"{ali.firstName}: {ali.studentNumber}");
// sara.studentNumber = "99999"; // خطای کامپایل
```

هر نمونه می‌تواند مقدار `readonly` **متفاوتی** داشته باشد؛ مقدار `const` در مثال قبل برای کل کلاس یکسان بود. `readonly` مانع **نسبت دادن مقدار تازه به خود فیلد** می‌شود. در این مثال فیلد از نوع `string` است؛ جزئیات تغییرپذیریِ اشیای دیگر را بعداً بررسی می‌کنیم.

**پراپرتی را نمی‌توان با واژهٔ `readonly` تعریف کرد.** اگر می‌خواهیم چیزی برای بیرون فقط خواندنی باشد، از پراپرتیِ فقط دارای `get` یا از `private set`، بسته به نیاز، استفاده می‌کنیم. این دو دقیقاً همان قواعد فیلد `readonly` نیستند:

| شکل | چه کسی می‌تواند مقدار را تعیین یا عوض کند؟ | نمونهٔ کاربرد |
| --- | --- | --- |
| `public const int MaxAge = 120;` | مقدار هنگام نوشتن کد تعیین می‌شود؛ بعد تغییر نمی‌کند. | حد ثابت مشترک |
| `public readonly string studentNumber;` | تعریف فیلد یا سازندهٔ همان کلاس؛ بعد قابل نسبت‌دادن نیست. | شمارهٔ هر دانشجو در لحظهٔ ساخت |
| `public string StudentNumber { get; }` | مقداردهی در تعریف یا سازنده؛ بیرون فقط می‌خواند. | نمایش شماره بدون setter |
| `public int Age { get; private set; }` | کد داخل کلاس می‌تواند بعداً تغییر دهد؛ بیرون فقط می‌خواند. | سن با متد تغییر کنترل‌شده |

## ۸. کنار هم گذاشتن مفهوم‌ها

اکنون یک نسخهٔ کوچک و قابل اجرا می‌سازیم. `FirstName` را auto property گذاشته‌ایم، چون فعلاً قانونی برای آن نداریم. `Age` فیلد پشتیبان و شرط دارد. شمارهٔ دانشجویی را هنگام ساخت می‌گیریم و برای خواندن بیرونی به شکل پراپرتیِ فقط دارای `get` نشان می‌دهیم. حد سن نیز در یک جای ثابت تعریف می‌شود.

**`Student.cs` — نمونهٔ نهایی این جلسه**

```csharp
public class Student
{
    public const int MaxAge = 120;

    private int _age;

    public string FirstName { get; set; } = "";
    public string StudentNumber { get; }

    public int Age
    {
        get { return _age; }
        private set
        {
            if (value >= 0 && value <= MaxAge)
            {
                _age = value;
            }
        }
    }

    public Student(string number, string name, int initialAge)
    {
        StudentNumber = number;
        FirstName = name;
        Age = initialAge;
    }

    public void ChangeAge(int newAge)
    {
        Age = newAge;
    }
}
```

**`Program.cs`**

```csharp
Student student = new Student("40101", "Sara", 20);
Console.WriteLine($"{student.FirstName}: {student.Age}"); // Sara: 20

student.ChangeAge(-2);
Console.WriteLine(student.Age); // 20؛ تغییر نامعتبر رد شد.

student.ChangeAge(21);
Console.WriteLine(student.Age);           // 21
Console.WriteLine(student.StudentNumber); // 40101
Console.WriteLine(Student.MaxAge);        // 120
```

اگر هنگام ساخت `initialAge` نامعتبر باشد، این نسخه `_age` را در مقدار اولیهٔ `0` نگه می‌دارد. اگر برای مسئله‌ات **سن صفر هم مجاز نیست** یا باید ساخت شیء نامعتبر به‌کل متوقف شود، باید قانون و نحوهٔ گزارش خطا را تغییر دهی. هدف این نسخه دیدن نقش سازنده، پراپرتی، فیلد خصوصی و متد در کنار هم است؛ هنوز قرار نیست همهٔ نیازهای یک سامانهٔ واقعی را پوشش دهد.

## ۹. تمرین‌های مرحله‌ای

1. قبل از اجرا حدس بزن خروجی سازندهٔ بدون پارامترِ نوشته‌شده چیست. سپس مقدارهای آغازین را عوض و دوباره اجرا کن.
2. برای `Student` دو سازنده بنویس: یکی بدون پارامتر و دیگری با نام. هر دو را از `Program.cs` صدا بزن و توضیح بده چرا بدون نوشتن سازندهٔ اول، `new Student()` دیگر کار نمی‌کند.
3. در نسخهٔ فیلد خصوصی و `SetAge`، مقادیر `20`، `-1`، `120` و `121` را به‌ترتیب امتحان کن. بعد از هر درخواست، مقدار `GetAge()` را پیش‌بینی کن.
4. برای شمارهٔ همراه، فیلد خصوصی `_mobile` و پراپرتی `Mobile` بساز. فعلاً فقط بررسی کن متن خالی نباشد. با مقدار معتبر و خالی امتحانش کن. اعتبارسنجی دقیق شماره، تمرین جداگانهٔ جلسهٔ سوم است.
5. پراپرتی `FirstName` در نمونهٔ نهایی را با یک فیلد خصوصی و `get` / `set` کامل جایگزین کن. اگر متن خالی بود، مقدار قبلی را نگه دار.
6. یک پراپرتی `StudentNumber { get; }` و سازنده‌ای بنویس که مقدارش را از ورودی بگیرد. در بیرون کلاس تلاش کن تغییرش دهی و پیام خطای کامپایل را بخوان؛ سپس خط را کامنت کن.
7. در نمونهٔ نهایی، `ChangeAge(121)` را اجرا کن. توضیح بده کدام بخش درخواست را بررسی می‌کند و چرا مقدار قبلی باقی می‌ماند.
8. برای کلاس خیالی `Course` مشخص کن «عنوان درس»، «ظرفیت ثابت اعلام‌شده در لحظهٔ ساخت» و «حد مشترک تعداد واحد» را با کدام ابزارِ این جلسه نمایش می‌دهی. دلیل هر انتخاب را در یک جمله بگو.

**هنگام حل:** بپرس «چه کسی باید این داده را بخواند؟»، «چه کسی اجازهٔ تغییر دارد؟»، «قانون بررسی در کدام مسیر است؟» و «مقدار در زمان نوشتن برنامه معلوم است یا هنگام ساخت شیء؟». این چهار پرسش معمولاً انتخاب میان فیلد خصوصی، پراپرتی، `const` و `readonly` را روشن می‌کند.

## جمع‌بندی

سازنده هنگام `new` اجرا می‌شود و وضعیت اولیهٔ شیء را آماده می‌کند. با `private` راهِ دسترسی مستقیم به داده را می‌بندیم و با متد یا پراپرتی، تغییر کنترل‌شده می‌دهیم. `get` می‌خواند، `set` می‌نویسد و `value` مقدار ورودیِ setter است. auto property فیلد پشتیبان پنهان دارد، اما خودبه‌خود اعتبارسنجی نمی‌کند. `const` برای مقدار ثابتِ مشترک و معلوم هنگام کامپایل است؛ فیلد `readonly` می‌تواند هنگام ساخت هر نمونه مقدار متفاوتی بگیرد. این ابزارها را بر اساس **نیاز مسئله** انتخاب کن، نه صرفاً کوتاه بودن کد.

## منابع مطالعهٔ بیشتر

- [Microsoft Learn · Using constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/using-constructors) برای سازندهٔ بدون پارامتر و سازنده‌های دیگر.
- [Microsoft Learn · Properties](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) برای `get`، `set` و سطح دسترسی.
- [Microsoft Learn · Automatically implemented properties](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/auto-implemented-properties) برای فیلد پشتیبان خودکار.
- [Microsoft Learn · `const`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/const) و [`readonly`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly) برای تفاوت مقدارهای ثابت.

[→ جلسهٔ قبل: آشنایی با شی‌ءگرایی و کلاس](session-07.md) · [بازگشت به فهرست جلسات](index.md)
