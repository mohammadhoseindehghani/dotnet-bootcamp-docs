---
title: جلسهٔ دهم · static، overload، sealed و abstraction
---

# جلسهٔ دهم · از مقدار مشترک تا کلاس انتزاعی

<div class="lesson-intro">تاریخ برگزاری: <bdi dir="ltr">1405/08/26</bdi><br>تکمیل مباحث <code>static</code> و چندریختی، آشنایی با <code>sealed</code> و ورود مرحله‌به‌مرحله به abstraction، کلاس abstract و اعضای <code>object</code>.</div>

این جلسه چند بخش مهم OOP را به هم وصل می‌کند. ابتدا می‌فهمیم دادهٔ مشترکِ کلاس چه زمانی مقدار می‌گیرد، بعد تفاوت overriding و overloading را دقیق می‌کنیم، مسیر ارث‌بری را با `sealed` کنترل می‌کنیم و در پایان به abstraction می‌رسیم. مثال‌های هر بخش مستقل‌اند؛ نسخه‌های متفاوت یک کلاس را هم‌زمان در یک پروژه قرار نده.

## ۱. عضو static در کلاس معمولی

یک کلاس معمولی می‌تواند هم عضوهای **نمونه‌ای** و هم عضوهای **static** داشته باشد:

- عضو نمونه‌ای متعلق به یک شیء مشخص است؛ هر شیء مقدار خودش را دارد.
- عضو static متعلق به خودِ نوع است؛ یک مقدار مشترک دارد و با نام کلاس استفاده می‌شود.

برای مثال، هر دانشجو نام خودش را دارد، ولی سقف تعداد واحد در این سامانه برای همهٔ دانشجوها مشترک است:

```csharp
public class Student
{
    public string Name { get; set; } = "";
    public static int MaxUnit { get; set; }
}
```

```csharp
Student first = new Student();
Student second = new Student();

first.Name = "Sara";
second.Name = "Ali";

Student.MaxUnit = 20;

Console.WriteLine(first.Name);      // Sara
Console.WriteLine(second.Name);     // Ali
Console.WriteLine(Student.MaxUnit); // 20
```

`Name` را با شیء می‌خوانیم، چون باید مشخص کنیم نام کدام دانشجو را می‌خواهیم. `MaxUnit` را با `Student.MaxUnit` می‌خوانیم، چون به یک دانشجوی خاص تعلق ندارد.

## ۲. چرا مقداردهی عضو static در سازندهٔ نمونه‌ای مشکل دارد؟

فرض کن `MaxUnit` را داخل سازندهٔ معمولی مقدار بدهیم:

**`Student.cs`**

```csharp
public class Student
{
    public static int MaxUnit { get; private set; }
    public string Name { get; private set; }

    public Student(string name)
    {
        Name = name;
        MaxUnit = 20;
    }
}
```

**`Program.cs`**

```csharp
Console.WriteLine(Student.MaxUnit); // 0

Student student = new Student("Sara");

Console.WriteLine(Student.MaxUnit); // 20
```

چرا بار اول `0` چاپ شد؟ `MaxUnit` یک `int` است و مقدار اولیهٔ پیش‌فرض آن صفر است. سازندهٔ معمولی فقط هنگام `new Student(...)` اجرا می‌شود. در خط اول هنوز هیچ شیئی نساخته‌ایم، پس کدی که `MaxUnit = 20` دارد اجرا نشده است.

این طراحی دو مشکل دارد:

1. مقدار مشترک کلاس به ساخته شدن یک شیء نامرتبط وابسته شده است.
2. سازندهٔ نمونه‌ای با ساخت **هر دانشجو** دوباره همان مقدار مشترک را تنظیم می‌کند.

اگر `MaxUnit` قانون مشترک نوع `Student` است، منطقی نیست برای آماده شدنش منتظر اولین دانشجو بمانیم. حتی در این مثال ساده می‌توانستیم همان موقع تعریف بنویسیم `public static int MaxUnit { get; } = 20;`. اما اگر مقداردهی به چند مرحله یا محاسبه نیاز داشته باشد، سازندهٔ static ابزار مناسبی است.

## ۳. سازندهٔ static

**Static constructor** برای آماده کردن داده یا انجام کاری است که به خودِ نوع مربوط می‌شود و فقط یک بار باید انجام شود.

**`Student.cs`**

```csharp
public class Student
{
    public static int MaxUnit { get; private set; }
    public string Name { get; private set; }

    static Student()
    {
        Console.WriteLine("Static constructor");
        MaxUnit = 20;
    }

    public Student(string name)
    {
        Console.WriteLine("Instance constructor");
        Name = name;
    }
}
```

**`Program.cs` — حالت اول: اولین استفاده، عضو static است**

```csharp
Console.WriteLine(Student.MaxUnit);
Console.WriteLine(Student.MaxUnit);
```

خروجی:

```text
Static constructor
20
20
```

سازندهٔ static پیش از اولین دسترسی به عضو static اجرا شد و با دسترسی دوم دوباره اجرا نشد.

**`Program.cs` — حالت دوم: اولین استفاده، ساخت شیء است**

```csharp
Student first = new Student("Sara");
Student second = new Student("Ali");
```

خروجی:

```text
Static constructor
Instance constructor
Instance constructor
```

پیش از ساخته شدن اولین نمونه، نوع `Student` باید آماده شود؛ پس سازندهٔ static اول اجرا می‌شود. با ساخته شدن نمونهٔ دوم، فقط سازندهٔ نمونه‌ای دوباره اجرا می‌شود.

### قواعد سازندهٔ static

- نامش با نام کلاس یکی است و پیش از آن `static` می‌آید.
- پارامتر و access modifier مثل `public` یا `private` نمی‌گیرد.
- مستقیماً آن را صدا نمی‌زنیم؛ runtime زمان اجرای آن را مدیریت می‌کند.
- برای هر نوع حداکثر یک بار اجرا می‌شود.
- پیش از اولین ساخته شدن نمونه یا اولین دسترسی به عضو static همان نوع اجرا می‌شود.
- پیش از سازندهٔ نمونه‌ای اجرا می‌شود.

عبارت دقیق «پیش از اولین استفادهٔ لازم از نوع» بهتر از «همیشه ابتدای برنامه» است. ممکن است برنامه اجرا شود ولی هرگز سراغ `Student` نرود؛ در این حالت لزومی ندارد سازندهٔ static آن اجرا شود.

> برای یک مقدار ساده مثل `20`، مقداردهی مستقیم (`= 20`) کوتاه‌تر است. هدف مثال، فهم زمان اجرای سازندهٔ static است؛ هر عضو static الزاماً به سازندهٔ static نیاز ندارد.

## ۴. چرا سازندهٔ static به عضو نمونه‌ای دسترسی مستقیم ندارد؟

این کد معتبر نیست:

```csharp
public class Student
{
    public static int MaxUnit { get; private set; }
    public string Name { get; private set; } = "";

    static Student()
    {
        MaxUnit = 20;
        // Name = "Sara"; // خطای کامپایل
    }
}
```

دلیل را با حفظ کردن جملهٔ «static به non-static دسترسی ندارد» یاد نگیر. سؤال واقعی این است: **نام کدام دانشجو؟**

```text
Student.MaxUnit → یک مقدار مشترک: 20

first.Name  → Sara
second.Name → Ali
third.Name  → Nima
```

سازندهٔ static در سطح کلاس اجرا می‌شود و هیچ شیء مشخصی به نام `this` ندارد. ممکن است هنوز اصلاً دانشجویی ساخته نشده باشد. پس `Name` بدون مشخص کردن نمونه، معنای روشنی ندارد. در مقابل `MaxUnit` دقیقاً یک مقدار متعلق به کلاس است و سازندهٔ static می‌داند کدام مقدار را باید تنظیم کند.

این به معنی ممنوع بودن هر نوع استفاده از شیء در کد static نیست. اگر خودمان یک نمونهٔ مشخص بسازیم یا نمونه‌ای را به متد static بدهیم، می‌توانیم از طریق همان متغیر به عضوهایش دسترسی پیدا کنیم:

```csharp
public class Student
{
    public string Name { get; set; } = "";

    public static void PrintName(Student student)
    {
        Console.WriteLine(student.Name);
    }
}
```

```csharp
Student sara = new Student();
sara.Name = "Sara";

Student.PrintName(sara);
```

متد static هنوز مستقیم `Name` را نخوانده است؛ یک شیء مشخص با نام `student` دریافت کرده و `student.Name` را خوانده است.

## ۵. آیا متد static هم همین محدودیت را دارد؟

بله. متد static نیز روی یک نمونهٔ مشخص اجرا نمی‌شود و `this` ندارد:

```csharp
public class Student
{
    public static int MaxUnit { get; private set; } = 20;
    public string Name { get; set; } = "";

    public static void PrintMaxUnit()
    {
        Console.WriteLine(MaxUnit); // درست: عضو static
        // Console.WriteLine(Name); // خطا: نام کدام شیء؟
    }

    public void PrintStudent()
    {
        Console.WriteLine(Name);    // عضو همان شیء
        Console.WriteLine(MaxUnit); // مقدار مشترک کلاس
    }
}
```

متد نمونه‌ای روی شیئی مثل `sara.PrintStudent()` اجرا می‌شود؛ پس می‌داند `Name` مربوط به `sara` است و در کنار آن می‌تواند مقدار مشترک `MaxUnit` را هم بخواند. جهت دسترسی را این‌طور تحلیل کن:

| محل کد | عضو static | عضو نمونه‌ای |
| --- | --- | --- |
| کد static | مستقیم قابل دسترسی است | فقط از راه یک نمونهٔ مشخص |
| کد نمونه‌ای | قابل دسترسی است | مربوط به `this` یا یک نمونهٔ مشخص است |

## ۶. مرور چندریختی و امضای متد

برای صحبت دربارهٔ overriding و overloading باید بدانیم «امضای متد» چیست. در بحث overload در C#، امضا شامل **نام متد و فهرست پارامترها** است؛ در فهرست پارامترها، تعداد، نوع و ترتیب نوع‌ها اهمیت دارند.

```csharp
void Print(string text)
void Print(int number)
void Print(string text, int count)
void Print(int count, string text)
```

این چهار متد امضاهای متفاوت دارند.

نام پارامتر برای تشخیص overload مهم نیست:

```csharp
void Print(int number)
// void Print(int value) // خطا: امضای متفاوتی ایجاد نمی‌کند.
```

### اصلاح مهم: نوع خروجی جزو تمایز overload نیست

نمی‌توانیم فقط با تغییر نوع خروجی دو overload بسازیم:

```csharp
int Calculate(int number)
{
    return number * 2;
}

// خطا: فقط نوع خروجی عوض شده است.
// double Calculate(int number)
// {
//     return number * 2.5;
// }
```

کامپایلر هنگام دیدن `Calculate(5)` نمی‌تواند از مقصد احتمالیِ خروجی برای انتخاب این دو استفاده کند. پس برای overload، **return type بخشی نیست که دو امضا را متفاوت کند**.

در overriding، متد فرزند باید با عضو پایه سازگار باشد: نام و پارامترها همان‌اند و نوع خروجی نیز باید مطابق قواعد override سازگار باشد. برای سطح فعلی، همان نوع خروجی را نگه دار. بنابراین نوع خروجی در درست بودن override مهم است، اما با تغییرِ تنها نوع خروجی نمی‌توان overload ساخت.

## ۷. Method overriding

در overriding، رابطهٔ ارث‌بری داریم. والد رفتار `virtual` تعریف می‌کند و فرزند همان رفتار را با `override` تخصصی می‌کند:

```csharp
public class User
{
    public virtual bool ChangePassword(string newPassword)
    {
        return newPassword.Length >= 6;
    }
}
```

```csharp
public class Student : User
{
    public override bool ChangePassword(string newPassword)
    {
        return newPassword.Length >= 8;
    }
}
```

```csharp
User user = new Student();
Console.WriteLine(user.ChangePassword("1234567")); // False
```

نوع متغیر `User` است، ولی شیء واقعی `Student` است؛ نسخهٔ overrideشدهٔ دانشجو اجرا می‌شود. این **runtime polymorphism** است: انتخاب اجرای نهایی هنگام اجرا و با توجه به نوع واقعی شیء اتفاق می‌افتد.

## ۸. Method overloading

در overloading، چند متد هم‌نام داریم که پارامترهای متفاوت دارند. وجود ارث‌بری لازم نیست. کامپایلر با آرگومان‌های فراخوانی، overload مناسب را انتخاب می‌کند.

**`Calculator.cs`**

```csharp
public class Calculator
{
    public int Add(int first, int second)
    {
        return first + second;
    }

    public double Add(double first, double second)
    {
        return first + second;
    }

    public int Add(int first, int second, int third)
    {
        return first + second + third;
    }

    public string Add(string first, string second)
    {
        return first + second;
    }
}
```

**`Program.cs`**

```csharp
Calculator calculator = new Calculator();

Console.WriteLine(calculator.Add(2, 3));          // 5
Console.WriteLine(calculator.Add(2.5, 3.1));      // 5.6
Console.WriteLine(calculator.Add(2, 3, 4));       // 9
Console.WriteLine(calculator.Add("C# ", "Bootcamp"));
```

راه‌های متداول متفاوت شدن overloadها:

- تعداد پارامترها متفاوت باشد.
- نوع یک یا چند پارامتر متفاوت باشد.
- ترتیب **نوع** پارامترها متفاوت باشد، مثل `(string, int)` و `(int, string)`.

فقط تغییر نام پارامتر یا نوع خروجی کافی نیست. همچنین overloadها بهتر است یک مفهوم مشترک داشته باشند؛ هم‌نام کردن کارهای کاملاً نامرتبط، کد را گیج‌کننده می‌کند.

### `Console.WriteLine` چگونه ورودی‌های مختلف را می‌پذیرد؟

ما از اول دوره این‌ها را نوشته‌ایم:

```csharp
Console.WriteLine(25);
Console.WriteLine(19.5);
Console.WriteLine(true);
Console.WriteLine("Hello");
```

`WriteLine` overloadهای متعددی برای نوع‌های مختلف دارد. کامپایلر با توجه به نوع آرگومان، overload مناسب را انتخاب می‌کند. در فرایند نمایش، مقدار به نمایش متنی مناسب تبدیل و در خروجی نوشته می‌شود؛ لازم نیست ما قبل از هر بار چاپ، دستی `Convert.ToString` بنویسیم.

### overriding و overloading در یک نگاه

| ویژگی | Overriding | Overloading |
| --- | --- | --- |
| به ارث‌بری نیاز دارد؟ | بله | خیر |
| نام متد | همان نام | همان نام |
| پارامترها | همان امضای عضو پایه | باید تفاوت قابل تشخیص داشته باشند |
| کلمه‌های اصلی | `virtual` و `override` | کلمهٔ کلیدی ویژه‌ای ندارد |
| زمان انتخاب | بر اساس نوع واقعی شیء هنگام اجرا | معمولاً توسط کامپایلر از روی آرگومان‌ها |

هر دو اجازه می‌دهند یک نام رفتار در موقعیت‌های مختلف شکل مناسب داشته باشد؛ سازوکارشان یکسان نیست.

## ۹. مطالعهٔ بیشتر: Operator overloading

!!! tip "مطالعهٔ بیشتر · Operator overloading"
    عملگرها هم می‌توانند برای نوع‌های مختلف رفتار متناسب داشته باشند. `+` برای دو عدد جمع عددی و برای دو `string` اتصال متن انجام می‌دهد. **Operator overloading** یعنی برای یک نوعِ ساخته‌شده توسط خودمان تعریف کنیم عملگری مانند `+` چه معنایی دارد.

    این مفهوم با method overloading مرتبط است، اما شکل استفاده فرق دارد: متد را با `calculator.Add(...)` صدا می‌زنیم، ولی عملگر در عبارتی مثل `left + right` دیده می‌شود. پشت صحنه، تعریف عملگر نیز عضو مخصوصی از نوع است. این بخش در کلاس کدنویسی نشده؛ کد زیر فقط برای آشنایی و مطالعهٔ بیشتر است.

    ```csharp
    public class Score
    {
        public int Value { get; }

        public Score(int value)
        {
            Value = value;
        }

        public static Score operator +(Score left, Score right)
        {
            return new Score(left.Value + right.Value);
        }
    }
    ```

    ```csharp
    Score first = new Score(17);
    Score second = new Score(18);
    Score total = first + second;

    Console.WriteLine(total.Value); // 35
    ```

    در این مدل، جمع دو `Score` یعنی ساخت یک `Score` تازه با مجموع مقدارها. باید معنایی را انتخاب کنیم که برای مسئله طبیعی باشد؛ تعریف عجیبِ `+` فقط چون امکانش وجود دارد، طراحی خوبی نیست.

    بعضی عملگرها **دودویی (binary)** هستند و دو عملوند دارند، مثل `left + right`. بعضی **یکانی (unary)** هستند و روی یک عملوند کار می‌کنند، مثل `++value` یا `-value`. شکل تعریف آن‌ها متفاوت است و جزئیاتشان به درس بعدی مربوط می‌شود.

در دسته‌بندی آموزشی این جلسه، overriding، method overloading و operator overloading سه شکل نمایش رفتارهای چندگانه معرفی می‌شوند. در منابع C# معمولاً overriding را چندریختی زمان اجرا و overloading را چندریختی زمان کامپایل می‌نامند؛ operator overloading نیز نوع تخصصی overloading برای عملگرهاست.

## ۱۰. متوقف کردن override با `sealed`

متد `virtual` در زنجیرهٔ ارث‌بری همچنان قابل override است:

```text
User → Student → Test → Test2
```

اگر `User.ChangePassword` مجازی باشد، `Student` می‌تواند آن را override کند و کلاس‌های بعدی نیز در حالت عادی می‌توانند دوباره رفتار را تغییر دهند. اگر `Student` بخواهد بگوید «اجرای من آخرین نسخه در ادامهٔ این مسیر است»، از `sealed override` استفاده می‌کند.

```csharp
public class User
{
    public virtual bool ChangePassword(string password)
    {
        return password.Length >= 6;
    }
}
```

```csharp
public class Student : User
{
    public sealed override bool ChangePassword(string password)
    {
        return password.Length >= 8;
    }
}
```

```csharp
public class Test : Student
{
    // خطا: Student ادامهٔ override این متد را مهر و موم کرده است.
    // public override bool ChangePassword(string password)
    // {
    //     return true;
    // }
}
```

`sealed` روی این متد، خودِ ارث‌بری از `Student` را ممنوع نمی‌کند؛ فقط ادامهٔ override همان عضو را متوقف می‌کند. این شکل فقط روی عضوی معنی دارد که در حال override شدن است: `sealed override`.

## ۱۱. کلاس sealed

کاربرد دوم `sealed` روی خود کلاس است. اگر نخواهیم هیچ کلاس دیگری از یک کلاس ارث ببرد، آن کلاس را مهر و موم می‌کنیم:

```csharp
public sealed class FinalReport
{
    public string Title { get; set; } = "";
}
```

```csharp
// خطا: از کلاس sealed نمی‌توان ارث برد.
// public class SpecialReport : FinalReport
// {
// }
```

در حالت عادی یک کلاس غیر sealed می‌تواند کلاس پایه باشد. برای ارث‌بری در کد «اجازه گرفتن» در زمان اجرا وجود ندارد؛ تعریف کلاس تعیین می‌کند ارث‌بری مجاز هست یا نه. `sealed` را زمانی به کار می‌بریم که ادامهٔ ارث‌بری با قواعد یا طراحی نوع سازگار نیست. لازم نیست همهٔ کلاس‌ها را بی‌دلیل sealed کنیم.

| شکل | چه چیزی متوقف می‌شود؟ |
| --- | --- |
| `sealed class Student` | هیچ کلاسی نمی‌تواند از `Student` ارث ببرد. |
| `sealed override ChangePassword` | فرزند می‌تواند از کلاس ارث ببرد، ولی آن متد را دوباره override نمی‌کند. |

## ۱۲. Abstraction یا انتزاع چیست؟

**Abstraction (انتزاع)** یعنی روی ویژگی‌های ضروری و مشترک تمرکز کنیم و جزئیاتی را که برای سطح فعلی لازم نیست کنار بگذاریم. این کار کمک می‌کند با یک مفهوم عمومی کار کنیم، بدون اینکه در همان لحظه درگیر تمام جزئیات نوع‌های مشخص شویم.

### مثال وسیلهٔ نقلیه

خودرو، موتور، قایق و دوچرخه همه وسیلهٔ نقلیه‌اند. مفهوم `Vehicle` می‌تواند بگوید هر وسیله نام دارد و باید بتواند حرکت کند. اما سؤال «یک Vehicle معمولی دقیقاً چگونه حرکت می‌کند؟» جواب مشخصی ندارد:

- خودرو با موتور و چرخ حرکت می‌کند.
- قایق روی آب حرکت می‌کند.
- دوچرخه با رکاب زدن حرکت می‌کند.

اگر `Vehicle` فقط مفهوم مشترک این خانواده باشد، ساخت `new Vehicle()` ممکن است در مدل ما معنی نداشته باشد. ما باید یک نوع مشخص مثل `Car` یا `Boat` بسازیم.

### مثال موجود زنده

`LivingThing` می‌تواند مفهوم عمومی موجود زنده باشد و گیاه، حیوان و انسان نوع‌های مشخص‌تر آن باشند. اگر برنامه فقط دربارهٔ ویژگی‌های مشترک صحبت می‌کند، `LivingThing` پایهٔ خوبی است؛ ولی «این موجود زندهٔ نامشخص دقیقاً چگونه غذا می‌گیرد؟» بسته به نوع آن فرق دارد.

### انتزاع بودن به سناریو بستگی دارد

هیچ فهرست همیشگی وجود ندارد که بگوید `User` یا `Student` همیشه abstract است:

- در سامانه‌ای که کاربر عمومی هم واقعاً ثبت می‌شود، `User` می‌تواند کلاس معمولی باشد.
- در سامانهٔ دانشگاه که فقط دانشجو و استاد وارد می‌شوند، `User` شاید فقط مفهوم مشترک باشد و ساخت کاربر عمومی معنی نداشته باشد.
- در یک مدل ساده، `Student` نوع مشخص است.
- در سامانه‌ای دیگر، شاید `Student` فقط پایه‌ای برای `UndergraduateStudent` و `GraduateStudent` باشد و خودش abstract طراحی شود.

انتخاب abstraction از دامنهٔ مسئله، نیاز برنامه و تصمیم طراحی می‌آید. تجربه کمک می‌کند، ولی سؤال ساده‌ای برای شروع داریم: **آیا ساخت نمونهٔ مستقیم از این مفهوم، در سناریوی من یک موجود معتبر و کامل ایجاد می‌کند؟**

## ۱۳. کلاس abstract

با `abstract` اعلام می‌کنیم کلاس برای استفاده به‌عنوان پایه طراحی شده و نمی‌توان مستقیم از آن شیء ساخت:

**`Vehicle.cs`**

```csharp
public abstract class Vehicle
{
    public string Name { get; private set; }

    public Vehicle(string name)
    {
        Name = name;
    }

    public void PrintName()
    {
        Console.WriteLine(Name);
    }
}
```

```csharp
// Vehicle vehicle = new Vehicle("Unknown"); // خطای کامپایل
```

کلاس abstract هنوز می‌تواند سازنده، فیلد، property و متد معمولیِ دارای بدنه داشته باشد. سازنده‌اش هنگام ساخت شیء فرزند از راه `base(...)` اجرا می‌شود. ممنوع بودن `new Vehicle(...)` به معنی بی‌استفاده بودن کلاس نیست؛ این کلاس بخش مشترک فرزندان را تعریف می‌کند.

### اصلاح مهم: وجود عضو abstract اجباری نیست

یک کلاس abstract **می‌تواند هیچ عضو abstractی نداشته باشد**. شاید فقط بخواهیم ساخت مستقیم آن را ممنوع کنیم و کد مشترک بدهیم. برعکس، هر کلاس که عضو abstract دارد باید خودش abstract باشد، چون هنوز یک رفتار ناقص در آن وجود دارد.

کلاس abstract نمی‌تواند هم‌زمان `sealed` باشد: `abstract` می‌گوید «برای کامل شدن از من ارث ببر» و `sealed` می‌گوید «از من ارث نبر»؛ این دو هدف با هم تناقض دارند.

## ۱۴. متد abstract: قرارداد بدون اجرای پایه

گاهی کلاس پایه می‌داند همهٔ فرزندان باید رفتاری داشته باشند، اما یک اجرای عمومی و معنادار برای آن ندارد. در این حالت فقط قرارداد را با متد abstract مشخص می‌کند:

**`Vehicle.cs`**

```csharp
public abstract class Vehicle
{
    public string Name { get; private set; }

    public Vehicle(string name)
    {
        Name = name;
    }

    public void PrintName()
    {
        Console.WriteLine(Name);
    }

    public abstract void Move();
}
```

متد `Move` بدنه ندارد و با `;` تمام می‌شود. این کلاس می‌گوید «هر وسیلهٔ نقلیهٔ کامل باید روش حرکت خودش را تعریف کند.»

**`Car.cs`**

```csharp
public class Car : Vehicle
{
    public Car(string name) : base(name)
    {
    }

    public override void Move()
    {
        Console.WriteLine("The car moves on the road.");
    }
}
```

**`Boat.cs`**

```csharp
public class Boat : Vehicle
{
    public Boat(string name) : base(name)
    {
    }

    public override void Move()
    {
        Console.WriteLine("The boat moves on water.");
    }
}
```

کلاس غیر abstract فرزند باید همهٔ عضوهای abstract به‌ارث‌رسیده را پیاده‌سازی کند. اگر هنوز آن‌ها را پیاده نکرده باشد، خودش نیز باید abstract بماند.

### abstract و virtual چه تفاوتی دارند؟

| عضو | بدنه در کلاس پایه | وظیفهٔ فرزند |
| --- | --- | --- |
| `virtual` | دارد | می‌تواند override کند، ولی مجبور نیست. |
| `abstract` | ندارد | فرزند concrete باید override کند. |

متد abstract از نظر قابلیت override شدن، مجازی است؛ کلمهٔ `virtual` را کنار `abstract` نمی‌نویسیم. متد abstract کلاس نمی‌تواند `static` باشد، چون overriding بر اساس شیء واقعی و زنجیرهٔ ارث‌بریِ نمونه‌ها انجام می‌شود، در حالی که عضو static به نمونه تعلق ندارد.

property هم می‌تواند abstract باشد. indexer نیز چنین قابلیتی دارد، اما چون هنوز indexer نخوانده‌ایم، وارد آن نمی‌شویم.

## ۱۵. متغیر از نوع abstract و شیء از نوع فرزند

نمی‌توانیم `new Vehicle()` بنویسیم، اما می‌توانیم متغیر یا آرایه‌ای از نوع `Vehicle` داشته باشیم و شیءهای کامل فرزند را در آن قرار دهیم:

**`Program.cs` — همراه فایل‌های بخش قبل**

```csharp
Vehicle first = new Car("Taxi");
Vehicle second = new Boat("Ferry");

Vehicle[] vehicles = { first, second };

foreach (Vehicle vehicle in vehicles)
{
    vehicle.PrintName();
    vehicle.Move();
}
```

نوع متغیر مشخص می‌کند از مسیر عمومی `Vehicle` با شیء کار می‌کنیم؛ `new Car` و `new Boat` نشان می‌دهند شیء واقعی کامل و قابل ساخت چیست. فراخوانی `Move` به override نوع واقعی می‌رسد. abstraction و polymorphism اینجا کنار هم کار می‌کنند.

همین مدل برای سامانهٔ دانشگاه ممکن است چنین باشد:

```csharp
public abstract class User
{
    public string Username { get; private set; }

    public User(string username)
    {
        Username = username;
    }

    public abstract string GetRoleTitle();
}
```

```csharp
public class Student : User
{
    public Student(string username) : base(username)
    {
    }

    public override string GetRoleTitle()
    {
        return "Student";
    }
}
```

```csharp
public class Teacher : User
{
    public Teacher(string username) : base(username)
    {
    }

    public override string GetRoleTitle()
    {
        return "Teacher";
    }
}
```

```csharp
User[] users =
{
    new Student("sara"),
    new Teacher("ali")
};

foreach (User user in users)
{
    Console.WriteLine($"{user.Username}: {user.GetRoleTitle()}");
}
```

## ۱۶. پدر نهایی کلاس‌ها: `object`

در C# همهٔ کلاس‌ها در انتهای زنجیره به `System.Object` یا شکل کوتاه آن، `object`، می‌رسند. اگر والد ننویسیم، کلاس ما باز هم در نهایت از `object` می‌آید:

```text
object
  └── User
       ├── Student
       └── Teacher
```

به همین دلیل همهٔ شیءها عضوهایی مانند `ToString()`, `Equals(...)`, `GetHashCode()` و `GetType()` دارند.

### `ToString`: نمایش متنی معنادار

اجرای پیش‌فرض `ToString` برای کلاس خودمان معمولاً نام نوع را برمی‌گرداند. می‌توانیم آن را override کنیم تا نمایش کاربردی‌تری بسازیم:

```csharp
public class Student
{
    public string Name { get; set; } = "";
    public string StudentNumber { get; set; } = "";

    public override string ToString()
    {
        return $"{Name} ({StudentNumber})";
    }
}
```

```csharp
Student student = new Student();
student.Name = "Sara";
student.StudentNumber = "40101";

Console.WriteLine(student.ToString()); // Sara (40101)
Console.WriteLine(student);            // Sara (40101)
```

`Console.WriteLine(student)` برای نمایش شیء از نمایش متنی آن استفاده می‌کند؛ override کردن `ToString` برای گزارش، دیباگ و نمایش خلاصهٔ شیء مفید است. بهتر است خروجی آن کوتاه، قابل‌فهم و بدون اطلاعات حساس مثل رمز عبور باشد.

### چند کلاس، چند نمایش مناسب

```csharp
public class Course
{
    public string Title { get; set; } = "";
    public int Unit { get; set; }

    public override string ToString()
    {
        return $"{Title} - {Unit} units";
    }
}
```

```csharp
public class Teacher
{
    public string Name { get; set; } = "";
    public string PersonnelCode { get; set; } = "";

    public override string ToString()
    {
        return $"{Name} [{PersonnelCode}]";
    }
}
```

هر کلاس بر اساس نیاز خودش نمایش مناسب را تعیین می‌کند.

### `GetType`: فهمیدن نوع واقعی شیء

`GetType()` نوع واقعی شیء را در زمان اجرا برمی‌گرداند:

```csharp
User user = new Student("sara");

Console.WriteLine(user.GetType().Name); // Student
Console.WriteLine(user.GetType());      // نام کامل نوع
```

اگر نوع متغیر `User` باشد ولی شیء واقعی `Student` باشد، `GetType()` نوع واقعی یعنی `Student` را نشان می‌دهد. **`GetType` را override نمی‌کنیم**؛ این متد virtual نیست. آن را برای مشاهدهٔ نوع runtime استفاده می‌کنیم.

### `Equals` و `GetHashCode`

`Equals` برای بیان برابری دو شیء و `GetHashCode` برای تولید یک کد عددی مرتبط با برابری استفاده می‌شود. این دو virtual هستند و می‌توان آن‌ها را override کرد، اما باید با هم سازگار باشند: اگر دو شیء را با `Equals` برابر می‌دانیم، باید hash code برابر هم تولید کنند.

این قرارداد در مجموعه‌هایی مثل دیکشنری و مجموعهٔ بدون تکرار اهمیت زیادی دارد. چون دانشجویان هنوز این collectionها را نخوانده‌اند، پیاده‌سازی کامل را به درس بعدی می‌سپاریم. فعلاً این نکته را نگه دار: **فقط override کردن `Equals` و فراموش کردن `GetHashCode` طراحی ناقصی است.**

| عضو `object` | virtual است؟ | استفادهٔ فعلی |
| --- | --- | --- |
| `ToString()` | بله | نمایش متنی شیء؛ قابل override |
| `Equals(...)` | بله | تعریف برابری معنایی؛ قابل override |
| `GetHashCode()` | بله | کد سازگار با برابری؛ قابل override |
| `GetType()` | خیر | مشاهدهٔ نوع واقعی؛ قابل override نیست |

## ۱۷. تمرین‌های مرحله‌ای

1. مثال `MaxUnit` را یک بار با مقداردهی در سازندهٔ نمونه‌ای اجرا کن. قبل و بعد از `new Student(...)` مقدار را چاپ و دلیل تفاوت را توضیح بده.
2. سازندهٔ static و نمونه‌ای را با دو پیام متفاوت بساز. سه دانشجو ایجاد کن و پیش از اجرا ترتیب و تعداد پیام‌ها را حدس بزن.
3. در یک متد static تلاش کن property نمونه‌ای را مستقیم بخوانی. پس از دیدن خطا، یک `Student` به متد پارامتر بده و از راه آن نام را چاپ کن.
4. برای `Calculator` overloadهای جمع دو `int`، دو `double` و سه `int` بنویس. سپس تلاش کن overloadی بسازی که فقط return type متفاوت دارد و پیام خطا را تحلیل کن.
5. کلاس‌های `User → Student → Test` بساز. متد `ChangePassword` را در `Student` با `sealed override` ببند و تلاش کن در `Test` دوباره override کنی.
6. یک کلاس `sealed` به نام `SystemSettings` بساز و تلاش کن از آن ارث ببری. خطای کامپایل را بخوان و تفاوتش را با sealed کردن یک متد توضیح بده.
7. `Shape` را abstract کن و متد abstract به نام `GetArea()` بساز. کلاس‌های `Rectangle` و `Circle` را پیاده کن. برای دایره از `Math.PI` استفاده کن.
8. آرایه‌ای از `Vehicle` بساز و `Car` و `Boat` در آن قرار بده. با حلقه `Move()` را صدا بزن و توضیح بده abstraction و polymorphism هرکدام کجای مثال دیده می‌شوند.
9. برای `Student`، `Course` و `Teacher` نمایش‌های مناسب `ToString` بنویس. اطلاعات حساس را در خروجی قرار نده.
10. با `User user = new Student("sara");` تفاوت نوع متغیر و نتیجهٔ `user.GetType().Name` را با زبان خودت توضیح بده.

این قواعد را حفظ نکن؛ هر بار سؤال مناسب را بپرس: «این داده متعلق به نوع است یا هر نمونه؟»، «کد static دربارهٔ کدام شیء حرف می‌زند؟»، «این مفهوم به‌تنهایی قابل ساخت است؟»، «فرزند باید رفتار را داشته باشد یا فقط اجازهٔ تغییر آن را دارد؟» و «انتخاب رفتار هنگام کامپایل است یا بر اساس شیء واقعی؟».

## جمع‌بندی

عضو static یک مقدار یا رفتار مشترکِ نوع است. سازندهٔ static پیش از اولین استفادهٔ لازم از نوع، حداکثر یک بار و پیش از سازندهٔ نمونه‌ای اجرا می‌شود؛ چون شیء مشخصی ندارد، عضو نمونه‌ای را بدون یک reference مشخص نمی‌شناسد. Overloading چند امضای متفاوت برای یک نام می‌سازد و return type به‌تنهایی overload ایجاد نمی‌کند. Overriding رفتار مجازی والد را در فرزند تخصصی می‌کند. `sealed` می‌تواند ادامهٔ override یا خودِ ارث‌بری را متوقف کند. کلاس abstract نمونهٔ مستقیم ندارد، ولی می‌تواند کد مشترک و قرارداد abstract داشته باشد؛ کلاس concrete فرزند آن قرارداد را کامل می‌کند. در نهایت همهٔ کلاس‌ها به `object` می‌رسند و می‌توانند نمایش و برابری خود را با اعضای virtual آن تعریف کنند، در حالی که `GetType` فقط نوع واقعی را گزارش می‌دهد.

## منابع مطالعهٔ بیشتر

- [Microsoft Learn · Static constructors](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors) و [Static classes and members](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members).
- [Microsoft Learn · Methods](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods) برای امضای متد و overload.
- [Microsoft Learn · Abstract and sealed classes](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/abstract-and-sealed-classes-and-class-members).
- [Microsoft Learn · Polymorphism](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/object-oriented/polymorphism).
- [Microsoft Learn · `System.Object`](https://learn.microsoft.com/en-us/dotnet/api/system.object) برای عضوهای پایهٔ همهٔ کلاس‌ها.

[→ جلسهٔ قبل: ارث‌بری، چندریختی و static](session-09.md) · [بازگشت به فهرست جلسات](index.md)
