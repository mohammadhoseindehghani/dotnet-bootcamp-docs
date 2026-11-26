---
title: جلسهٔ سیزدهم · پروژهٔ گلستان، Casting، Boxing و Service
---

# جلسهٔ سیزدهم · توسعهٔ پروژهٔ گلستان، Casting، Boxing و لایهٔ Service

<div class="lesson-intro">تاریخ برگزاری: <bdi dir="ltr">1405/09/05</bdi><br>در این جلسه نسخهٔ دوم پروژهٔ گلستان را با نسخهٔ قبلی مقایسه می‌کنیم، مسیر ورود و انتخاب کلاس را قدم‌به‌قدم می‌خوانیم، با دلیل استفاده از Service آشنا می‌شویم و Upcasting، Downcasting، Boxing و Unboxing را با مثال‌های ساده یاد می‌گیریم.</div>

این جلسه هنوز مقدماتی است. هدف فعلاً ساختن معماری بی‌نقص یا سامانهٔ کامل دانشگاه نیست؛ می‌خواهیم مسئولیت بخش‌های مختلف برنامه را تشخیص بدهیم و تبدیل نوع‌ها را درست بفهمیم.

> نکتهٔ مهم: تبدیل `(Student)InMemoryDB.CurrentUser` در پروژه **Unboxing نیست**؛ این یک Downcasting بین دو reference type است. در ادامه تفاوتشان را دقیق می‌بینیم.

## ۱. نسخهٔ دوم چه چیزهایی اضافه کرده است؟

نسخهٔ قبل بیشتر روی موجودیت‌ها، ثبت‌نام و ورود جداگانهٔ نقش‌ها تمرکز داشت. نسخهٔ دوم برنامه را به یک جریان قابل‌استفاده‌تر نزدیک کرده است.

| موضوع | نسخهٔ قبل | نسخهٔ دوم | دلیل تغییر |
| --- | --- | --- | --- |
| ساختار فایل‌ها | همهٔ فایل‌ها در ریشه | پوشه‌های `Entities`، `Repositories` و `Services` | پیدا کردن مسئولیت هر فایل آسان‌تر می‌شود. |
| نقطهٔ شروع | کلاس `Program` و متد `Main` | top-level statements | کد شروع برنامه کوتاه‌تر شده است. |
| کاربران | سه لیست جدا برای دانشجو، استاد و اپراتور | یک `List<User>` | به کمک ارث‌بری، همهٔ کاربران در یک مجموعه نگهداری می‌شوند. |
| کاربر واردشده | وجود نداشت | `CurrentUser` | برنامه می‌فهمد اکنون چه کسی وارد شده است. |
| دادهٔ آزمایشی | لیست‌های خالی | متد `Seed` | چند درس، استاد، کلاس و دانشجو برای آزمایش آماده می‌شوند. |
| ورود | متد جدا برای هر نقش | یک `LoginPage` برای همه | کد تکراری کمتر می‌شود. |
| تشخیص نقش | کاربر خودش نقش را انتخاب می‌کرد | نوع واقعی شیء بررسی می‌شود | برنامه نقش واقعی کاربر را تشخیص می‌دهد. |
| منوها | فقط ثبت‌نام و ورود | صفحهٔ دانشجو، استاد و اپراتور | جریان برنامه توسعه پیدا کرده است. |
| کلاس‌های دانشجو | لیست ممکن بود `null` باشد | مقدار اولیهٔ `[]` | می‌توان بدون خطا از `Add` استفاده کرد. |
| ظرفیت باقی‌مانده | محاسبه نمی‌شد | `RemainingCapacity` | مقدار از روی ظرفیت و تعداد دانشجوها محاسبه می‌شود. |
| نمایش کلاس | خروجی پیش‌فرض شیء | override کردن `ToString` | اطلاعات کلاس خوانا چاپ می‌شود. |
| منطق عملیات | بیشتر داخل `Program` یا موجودیت‌ها | کلاس `Service` | رابط کاربری از بخشی از منطق برنامه جدا می‌شود. |
| خروج | وجود نداشت | `LogOff` | `CurrentUser` پاک و برنامه به صفحهٔ اصلی برمی‌گردد. |

### ساختار تازهٔ پروژه

```text
Golestan
├── Entities
│   ├── BaseEntity.cs
│   ├── User.cs
│   ├── Student.cs
│   ├── Teacher.cs
│   ├── Operator.cs
│   ├── Course.cs
│   └── Class.cs
├── Repositories
│   └── InMemoryDB.cs
├── Services
│   └── Service.cs
└── Program.cs
```

- `Entities` می‌گوید داده‌ها و موجودیت‌های مسئله چه شکلی دارند.
- `Repositories` فعلاً داده‌ها را در حافظه نگه می‌دارد.
- `Services` عملیات و قواعد برنامه را اجرا می‌کند.
- `Program.cs` ورودی و خروجی کنسول و جابه‌جایی بین صفحه‌ها را مدیریت می‌کند.

پوشه‌بندی به‌تنهایی رفتار برنامه را عوض نمی‌کند. namespaceها نیز تغییر کرده‌اند تا با ساختار تازه هماهنگ شوند:

```csharp
using Golestan.Entities;
using Golestan.Repositories;
using Golestan.Services;
```

## ۲. شروع برنامه در نسخهٔ جدید

ابتدای `Program.cs` این سه دستور اجرا می‌شود:

```csharp
Service _service = new Service();
InMemoryDB.Seed();
MainPage();
```

ترتیب مهم است:

1. یک شیء از `Service` ساخته می‌شود.
2. `Seed` داده‌های آزمایشی را می‌سازد.
3. صفحهٔ اصلی نشان داده می‌شود.

در این نسخه دیگر `class Program` و `static void Main` را در فایل نمی‌بینیم. این قابلیت **top-level statements** است. کامپایلر پشت صحنه نقطهٔ شروع لازم را می‌سازد. یعنی برنامه همچنان `Main` دارد، فقط ما شکل کوتاه‌تری نوشته‌ایم.

## ۳. چرا لیست کاربران یکی شد؟

نسخهٔ قبل سه لیست داشت:

```csharp
List<Student> Students
List<Teacher> Teachers
List<Operator> Operators
```

در نسخهٔ جدید هر سه نقش از `User` ارث می‌برند، پس می‌توان آن‌ها را در یک لیست نگه داشت:

```csharp
public static List<User> Users { get; set; } = [];
```

```csharp
Users.Add(student);
Users.Add(teacher);
Users.Add(operator);
```

این نمونه‌ای از polymorphism است: نوع متغیر یا جایگاه لیست `User` است، اما شیء واقعی می‌تواند `Student` یا `Teacher` یا `Operator` باشد.

```text
متغیر از نوع User
        ↓ اشاره می‌کند به
شیء واقعی از نوع Student
```

وقتی در حلقه می‌نویسیم `user.CheckPassword(password)`، رفتار مشترک تعریف‌شده در `User` برای همهٔ نقش‌ها در دسترس است؛ بنابراین به سه حلقهٔ تقریباً یکسان نیاز نداریم.

## ۴. `CurrentUser`: چه کسی وارد برنامه شده است؟

```csharp
public static User CurrentUser { get; set; }
```

پس از ورود موفق:

```csharp
InMemoryDB.CurrentUser = user;
```

این property مانند کارت شناسایی فرد واردشده عمل می‌کند. صفحه‌ها و سرویس‌ها می‌توانند بفهمند عملیات برای کدام کاربر انجام می‌شود.

نوع `CurrentUser` برابر `User` است، چون ممکن است هر کدام از نقش‌ها وارد شوند. اما گاهی عملیات مخصوص دانشجو است و به propertyای مثل `Classes` نیاز داریم که فقط در `Student` وجود دارد. آن‌جا تبدیل نوع مطرح می‌شود:

```csharp
Student student = (Student)InMemoryDB.CurrentUser;
```

این کد فقط وقتی امن است که شیء واقعی واقعاً `Student` باشد. اگر استاد وارد شده باشد، در زمان اجرا `InvalidCastException` رخ می‌دهد.

شکل امن‌تر برای کدهای بعدی:

```csharp
if (InMemoryDB.CurrentUser is Student student)
{
    Console.WriteLine(student.StudentNO);
}
```

فعلاً همین‌قدر بدانیم که `is` هم نوع را بررسی می‌کند و هم در صورت درست بودن، متغیر `student` را آماده می‌کند.

## ۵. داده‌سازی با `Seed`

متد `Seed` چند دادهٔ ازپیش‌آماده می‌سازد:

- سه درس؛
- دو استاد؛
- چهار کلاس؛
- دو دانشجو؛
- ارتباط دانشجوها با کلاس‌ها.

هدف `Seed` این است که برای هر بار آزمایش مجبور نباشیم همهٔ اطلاعات را دستی وارد کنیم. مثلاً این دو خط یک ارتباط دوطرفه را ثبت می‌کنند:

```csharp
student.Classes.Add(classItem);
classItem.Students.Add(student);
```

خط اول می‌گوید این کلاس در فهرست کلاس‌های دانشجو است. خط دوم می‌گوید همین دانشجو در فهرست دانشجوهای کلاس است. اگر فقط یکی را انجام دهیم، دو سمت مدل با هم ناسازگار می‌شوند.

> `InMemoryDB` پایگاه دادهٔ واقعی نیست. داده‌ها فقط تا وقتی برنامه باز است در حافظه می‌مانند و با بسته شدن برنامه از بین می‌روند.

## ۶. property محاسبه‌شده و `ToString`

### ظرفیت باقی‌مانده

```csharp
public int RemainingCapacity
{
    get
    {
        return Capacity - Students.Count;
    }
}
```

برای `RemainingCapacity` مقدار جداگانه ذخیره نشده است. هر بار آن را بخوانیم، از اطلاعات فعلی محاسبه می‌شود.

مثال: اگر ظرفیت `30` و تعداد دانشجوها `8` باشد:

```text
RemainingCapacity = 30 - 8 = 22
```

این روش جلوی ناسازگاری دو مقدار را می‌گیرد. اگر هم `Students.Count` تغییر کند، نتیجهٔ property نیز خودکار تغییر می‌کند.

### نمایش خوانای کلاس

```csharp
public override string ToString()
{
    return $"{Id}- {Course.CourseName} " +
           $"({Teacher.FirstName} {Teacher.LastName})";
}
```

بدون override، `Console.WriteLine(classItem)` معمولاً نام نوع را چاپ می‌کند. با override کردن `ToString`، خودمان تعیین می‌کنیم نمایش متنی شیء چه باشد.

## ۷. Service چیست و چرا از آن استفاده می‌کنیم؟

در برنامهٔ کنسولی کوچک می‌توان همهٔ کدها را در `Program.cs` نوشت؛ اما خیلی زود این فایل هم ورودی می‌گیرد، هم پیام چاپ می‌کند، هم کاربر را پیدا می‌کند، هم ظرفیت را بررسی می‌کند و هم ثبت کلاس انجام می‌دهد. در نتیجه تغییر و آزمایش آن سخت می‌شود.

Service را فعلاً این‌طور تعریف کنیم:

> Service بخشی است که عملیات برنامه و قواعد مربوط به آن‌ها را در یک جای مشخص قرار می‌دهد.

تقسیم مسئولیت ساده:

```text
Program.cs
ورودی بگیر و نتیجه را نمایش بده
        ↓ درخواست
Service
عملیات و قانون را اجرا کن
        ↓ خواندن/نوشتن
InMemoryDB
داده را نگه دار
```

مثلاً صفحهٔ «کلاس‌های من» فقط نتیجه را می‌خواهد:

```csharp
void MyClassesPage()
{
    List<Class> classes = _service.GetListOfMyClass();

    foreach (Class classItem in classes)
    {
        Console.WriteLine(classItem);
    }
}
```

این صفحه لازم نیست بداند کاربر جاری چگونه به `Student` تبدیل می‌شود یا کلاس‌ها کجا نگهداری شده‌اند. این جزئیات در Service قرار می‌گیرند:

```csharp
public List<Class> GetListOfMyClass()
{
    Student student = (Student)InMemoryDB.CurrentUser;
    return student.Classes.ToList();
}
```

### فایده‌های Service در همین سطح مقدماتی

1. `Program.cs` خواناتر می‌شود.
2. منطق تکراری در یک نقطه قرار می‌گیرد.
3. تغییر قواعد آسان‌تر می‌شود؛ مثلاً قانون ظرفیت را در Service تغییر می‌دهیم.
4. بعداً آزمایش منطق بدون کار با `Console.ReadLine` و `Console.WriteLine` ساده‌تر است.
5. رابط دیگری مثل وب یا موبایل می‌تواند از همان منطق استفاده کند.

Service محل ریختن هر کدی نیست. گرفتن ورودی و رنگ‌کردن کنسول کار رابط کاربری است؛ نگهداری لیست‌ها کار مخزن داده است؛ اجرای عملیاتی مثل «کلاس‌های قابل ثبت را پیدا کن» کار Service است.

## ۸. خواندن `GetAvailableClasses` قدم‌به‌قدم

نسخهٔ فعلی ابتدا کلاس دارای ظرفیت را پیدا می‌کند:

```csharp
foreach (Class classItem in InMemoryDB.Classes)
{
    if (classItem.Capacity - classItem.Students.Count > 0)
    {
        availableClasses.Add(classItem);
    }
}
```

سپس کلاس‌هایی را که دانشجو قبلاً دارد، حذف می‌کند:

```csharp
foreach (Class classItem in currentStudent.Classes)
{
    availableClasses.Remove(classItem);
}
```

پس «قابل ثبت» فعلاً یعنی:

```text
کلاس ظرفیت خالی دارد
و
دانشجو قبلاً عضو آن نیست
```

هنوز قواعدی مثل سقف واحد، مشروطی، تداخل زمانی و پیش‌نیاز بررسی نمی‌شوند. این‌ها می‌توانند در مراحل بعد به Service اضافه شوند.

همچنین می‌توان از property تازه استفاده کرد و شرط را خواناتر نوشت:

```csharp
if (classItem.RemainingCapacity > 0)
{
    availableClasses.Add(classItem);
}
```

## ۹. Boxing چیست؟

ابتدا دو خانوادهٔ نوع را یادآوری کنیم:

- `int`، `double`، `bool` و `char` نمونه‌هایی از value type هستند.
- `string`، آرایه، کلاس‌ها و `object` reference type هستند.

`object` ریشهٔ مشترک همهٔ نوع‌ها در سی‌شارپ است. پس می‌توان یک مقدار عددی را داخل متغیر `object` قرار داد:

```csharp
int score = 18;
object boxedScore = score;
```

در خط دوم Boxing اتفاق می‌افتد:

1. یک شیء روی heap ساخته می‌شود.
2. مقدار `18` داخل آن کپی می‌شود.
3. متغیر `boxedScore` به آن شیء اشاره می‌کند.

```text
int score = 18
      │
      │ Boxing و کپی مقدار
      ▼
object boxedScore ───► [ 18 ] روی heap
```

کلمهٔ «جعبه‌کردن» کمک می‌کند تصویر ذهنی خوبی بسازیم: مقدار value type داخل جعبه‌ای از نوع `object` قرار می‌گیرد.

### Boxing ضمنی است

برای Boxing معمولاً cast صریح لازم نیست:

```csharp
int number = 10;
object value = number;
```

کامپایلر می‌داند هر نوعی می‌تواند به `object` تبدیل شود.

### Boxing یک کپی می‌سازد

```csharp
int number = 10;
object boxed = number;

number = 20;

Console.WriteLine(number); // 20
Console.WriteLine(boxed);  // 10
```

تغییر `number` مقدار داخل جعبه را تغییر نمی‌دهد، چون هنگام Boxing مقدار کپی شده است.

## ۱۰. Unboxing چیست؟

برای بیرون آوردن value type از `object` باید نوع دقیق آن را مشخص کنیم:

```csharp
object boxedScore = 18;
int score = (int)boxedScore;
```

خط دوم Unboxing است. این تبدیل صریح است، چون ممکن است نوع داخل `object` چیز دیگری باشد.

```text
object boxedScore ───► [ 18 ]
                           │
                           │ Unboxing و کپی مقدار
                           ▼
                     int score = 18
```

### نوع دقیق اهمیت دارد

```csharp
object value = 18;
int number = (int)value;       // درست
// long bigNumber = (long)value; // خطای زمان اجرا
```

عدد داخل جعبه با نوع `int` ذخیره شده است. این‌که `int` می‌تواند بعداً به `long` تبدیل شود، اجازه نمی‌دهد مستقیم آن را به `long` unbox کنیم.

راه درست:

```csharp
object value = 18;
int number = (int)value;
long bigNumber = number;
```

ابتدا با نوع واقعی `int` از جعبه خارج می‌کنیم، سپس تبدیل عددی انجام می‌دهیم.

### مثال با `double`

```csharp
double average = 17.5;
object boxedAverage = average;          // Boxing
double result = (double)boxedAverage;   // Unboxing
```

این کد اشتباه است:

```csharp
object boxedAverage = 17.5;
// int result = (int)boxedAverage;
```

مقدار داخل جعبه `double` است، نه `int`.

### مثال با `bool`

```csharp
bool isActive = true;
object boxedState = isActive;
bool state = (bool)boxedState;
```

Boxing فقط برای عددها نیست؛ هر value type می‌تواند box شود.

## ۱۱. مثال‌های روزمره‌تر Boxing و Unboxing

### لیست قدیمی و غیرجنریک

```csharp
using System.Collections;

ArrayList values = new ArrayList();
values.Add(10);              // Boxing
values.Add(20);              // Boxing

int first = (int)values[0];  // Unboxing
```

چون `ArrayList` اعضا را به شکل `object` نگه می‌دارد، عددها box می‌شوند. اما `List<int>` نوع عدد را می‌شناسد:

```csharp
List<int> values = new List<int>();
values.Add(10);
int first = values[0];
```

در استفادهٔ معمول `List<int>` به Boxing و Unboxing نیاز ندارد. این یکی از مزیت‌های collectionهای generic است: type safety بهتر و هزینهٔ کمتر.

### متدی که `object` می‌گیرد

```csharp
static void Show(object value)
{
    Console.WriteLine(value);
}

int age = 25;
Show(age); // age برای ارسال به پارامتر object، box می‌شود.
```

### رشته Boxing نمی‌شود

```csharp
string name = "Sara";
object value = name;
```

اینجا Boxing رخ نمی‌دهد، چون `string` از ابتدا reference type است. فقط reference آن در متغیر `object` قرار می‌گیرد.

### شیء دانشجو هم Boxing نمی‌شود

```csharp
Student student = new Student("Sara", "Ahmadi", "sara", "123456");
User user = student;
```

هر دو `Student` و `User` reference type هستند. این Upcasting است، نه Boxing.

## ۱۲. Upcasting و Downcasting

در پروژهٔ گلستان، `Student` و `Teacher` از `User` ارث‌بری کرده‌اند. پس هر دانشجو یک کاربر هم هست؛ اما هر کاربر لزوماً دانشجو نیست. همین جمله، منطق دو تبدیل مهم را روشن می‌کند.

### Upcasting: نگاه‌کردن به شیء فرزند از زاویهٔ کلاس والد

وقتی reference یک شیء از کلاس فرزند را داخل متغیری از نوع کلاس والد قرار می‌دهیم، **Upcasting** انجام شده است:

```csharp
Student student = new Student("Sara", "Ahmadi", "sara", "123456");
User user = student; // Upcasting
```

این تبدیل خودکار و امن است، چون هر `Student` حتماً یک `User` است. در اینجا:

- شیء تازه‌ای ساخته نمی‌شود؛
- اطلاعات دانشجو حذف نمی‌شود؛
- هر دو متغیر به همان یک شیء اشاره می‌کنند؛
- از طریق متغیر `user` فقط اعضایی را می‌بینیم که در نوع `User` تعریف شده‌اند.

```text
Student student ────┐
                    ├──► یک شیء واقعی از نوع Student
User user ──────────┘
```

برای مثال چون `StudentNO` فقط در کلاس `Student` تعریف شده است، خط زیر کامپایل نمی‌شود:

```csharp
// Console.WriteLine(user.StudentNO);
```

نوع متغیر `user` برابر `User` است؛ بنابراین کامپایلر از این مسیر فقط اعضای `User` را در اختیار ما می‌گذارد. بااین‌حال نوع واقعی شیء همچنان `Student` است. اگر یک متد `virtual` در `Student`، `override` شده باشد، نسخهٔ مربوط به `Student` اجرا می‌شود:

```csharp
User user = new Student("Sara", "Ahmadi", "sara", "123456");
bool changed = user.ChangePassword("123456", "newPassword");
// چون ChangePassword در Student بازنویسی شده، نسخهٔ Student اجرا می‌شود.
```

این ویژگی به ما اجازه می‌دهد کاربران مختلف را در یک مجموعه نگه داریم:

```csharp
User[] users =
{
    new Student("Sara", "Ahmadi", "sara", "123456"),
    new Teacher("Ali", "Karimi", "ali", "987654")
};

foreach (User item in users)
{
    Console.WriteLine(item.UserName);
}
```

با وجود متفاوت‌بودن نوع واقعی اعضای آرایه، می‌توانیم اعضای مشترک تعریف‌شده در `User` را برای همهٔ آن‌ها استفاده کنیم. این یکی از کاربردهای اصلی Upcasting در کنار polymorphism است.

### Downcasting: برگشتن از نوع والد به نوع واقعی فرزند

گاهی متغیر ما از نوع `User` است، اما برای کاری مشخص به اعضای مخصوص `Student` نیاز داریم. اگر شیء واقعی واقعاً دانشجو باشد، می‌توانیم **Downcasting** انجام دهیم:

```csharp
User user = new Student("Sara", "Ahmadi", "sara", "123456");
Student student = (Student)user; // Downcasting

Console.WriteLine(student.StudentNO);
```

Downcasting باید صریح نوشته شود، چون هر `User` الزاماً `Student` نیست. cast فقط نوع نگاه ما به همان شیء را تغییر می‌دهد و شیء تازه‌ای نمی‌سازد.

### خطر Downcasting مستقیم

کد زیر از نظر کامپایلر ممکن است، اما هنگام اجرا خطا می‌دهد:

```csharp
User user = new Teacher("Ali", "Karimi", "ali", "987654");
Student student = (Student)user; // InvalidCastException
```

شیء واقعی `Teacher` است و نمی‌توان آن را `Student` در نظر گرفت. برای جلوگیری از این خطا، قبل از تبدیل نوع را بررسی می‌کنیم.

### روش امن‌تر با `is`

Pattern matching با `is` هم نوع واقعی را بررسی می‌کند و هم در صورت درست‌بودن، متغیر موردنیاز را می‌سازد:

```csharp
User user = InMemoryDB.CurrentUser;

if (user is Student student)
{
    Console.WriteLine(student.StudentNO);
}
else
{
    Console.WriteLine("کاربر واردشده دانشجو نیست.");
}
```

در پروژهٔ گلستان، این روش از cast مستقیم زیر امن‌تر است:

```csharp
// ممکن است در زمان اجرا خطا بدهد:
Student student = (Student)InMemoryDB.CurrentUser;
```

### روش دیگر با `as`

کلمهٔ `as` در صورت ناموفق‌بودن تبدیل، به‌جای ایجاد `InvalidCastException` مقدار `null` برمی‌گرداند:

```csharp
User user = InMemoryDB.CurrentUser;
Student? student = user as Student;

if (student != null)
{
    Console.WriteLine(student.StudentNO);
}
```

برای کدهای جدید، الگوی `is Student student` معمولاً کوتاه‌تر و خواناتر است. `as` فقط برای reference typeها و nullable typeها قابل استفاده است.

### مقایسه با Boxing و Unboxing

این تبدیل‌ها ظاهر مشابهی دارند، اما مفهومشان یکسان نیست:

| تبدیل | مبدأ | مقصد | خودکار یا صریح؟ | مثال |
| --- | --- | --- | --- | --- |
| Upcasting | کلاس فرزند | کلاس والد | خودکار و امن | `User u = student;` |
| Downcasting | کلاس والد | کلاس فرزند واقعی | صریح یا همراه بررسی | `Student s = (Student)u;` |
| Boxing | value type | `object` یا interface | معمولاً خودکار | `object x = 10;` |
| Unboxing | object حاوی value type | همان value type دقیق | صریح | `int n = (int)x;` |

در Upcasting و Downcasting میان دو reference type حرکت می‌کنیم و referenceها به همان شیء اشاره دارند. در Boxing، مقدار یک value type داخل یک object تازه کپی می‌شود. بنابراین این خط:

```csharp
User user = student;
```

**Boxing نیست**؛ یک Upcasting ساده است.

## ۱۳. خطاهای رایج در تبدیل‌ها

### Unboxing با نوع اشتباه

```csharp
object value = 12;
// double number = (double)value; // InvalidCastException
```

### Downcasting اشتباه

```csharp
User user = new Teacher("Ali", "Ahmadi", "ali", "123456");
// Student student = (Student)user; // InvalidCastException
```

این دو خطا نام یکسانی دارند، اما دلیل مفهومی آن‌ها فرق دارد. اولی object حاوی `int` را با نوع غلط unbox می‌کند. دومی می‌خواهد شیء واقعی `Teacher` را `Student` در نظر بگیرد.

### استفاده از `Convert`

```csharp
object value = 12;
double number = Convert.ToDouble(value); // 12.0
```

`Convert.ToDouble` با cast مستقیم یکسان نیست. این متد مقدار را به عدد جدید تبدیل می‌کند؛ درحالی‌که Unboxing باید ابتدا نوع دقیق داخل جعبه را بیرون بیاورد.

## ۱۴. هزینهٔ Boxing و دلیل توجه به آن

Boxing معمولاً این هزینه‌ها را دارد:

- ساخت یک object روی heap؛
- کپی مقدار؛
- نیاز به Unboxing برای استفادهٔ دوباره به‌عنوان value type؛
- احتمال خطای زمان اجرا در cast اشتباه.

در مثال‌های کوچک این هزینه نگران‌کننده نیست. هدف فعلی شناخت رفتار است، نه بهینه‌سازی زودهنگام. انتخاب نوع مناسب مثل `List<int>` به‌جای مجموعهٔ `object` معمولاً خودبه‌خود بسیاری از Boxingهای غیرضروری را حذف می‌کند.

## ۱۵. مسیر کامل ورود دانشجو

```text
شروع برنامه
   ↓
ساخت Service
   ↓
اجرای Seed
   ↓
MainPage
   ↓ انتخاب Login
LoginPage
   ↓ پیدا کردن User و بررسی رمز
CurrentUser = user
   ↓ تشخیص نوع واقعی
StudentPage / TeacherPage / OperatorPage
```

در نسخهٔ فعلی برای تشخیص نوع از این عبارت استفاده شده است:

```csharp
user.GetType().Name
```

این عبارت نام نوع واقعی شیء را به شکل رشته برمی‌گرداند. برای شروع قابل‌فهم است، اما تصمیم‌گیری بر اساس رشته شکننده است. شکل مناسب‌تر برای ادامهٔ آموزش، pattern matching است:

```csharp
switch (user)
{
    case Student:
        StudentPage();
        break;
    case Teacher:
        TeacherPage();
        break;
    case Operator:
        OperatorPage();
        break;
}
```

## ۱۶. بخش‌های ناتمام و نکته‌های نسخهٔ فعلی

پروژه آموزشی و در حال تکمیل است. هنگام خواندن آن این موارد را بدانیم:

1. `RegisterClass` در Service هنوز بدنهٔ واقعی ندارد.
2. صفحه‌های استاد و اپراتور فعلاً فقط عنوان نمایش می‌دهند.
3. پس از انتخاب نقش در `LoginPage`، یک بار دیگر `StudentPage()` بدون شرط صدا زده شده است؛ این خط باید حذف شود، وگرنه حتی بعد از صفحهٔ استاد یا اپراتور نیز صفحهٔ دانشجو فراخوانی می‌شود.
4. اگر نام کاربری پیدا نشود، پیام مشخصی نمایش داده نمی‌شود.
5. بعد از رمز اشتباه، مسیر بازگشت یا تلاش دوباره کامل نیست.
6. `CurrentUser` می‌تواند هنگام خروج `null` شود، اما نوع property فعلاً nullable اعلام نشده است.
7. تبدیل مستقیم `CurrentUser` به `Student` بدون بررسی نوع ممکن است خطای زمان اجرا بدهد.
8. `Seed` در صورت اجرای دوباره داده‌ها را دوباره به لیست‌ها اضافه می‌کند.
9. ثبت‌نام فقط دانشجو می‌سازد و یکتا بودن نام کاربری را بررسی نمی‌کند.
10. `RegisterClassPage` خودش را دوباره فراخوانی می‌کند. برای منوی طولانی، حلقه مناسب‌تر از recursion است.

این موارد برای سرزنش کد نیستند؛ هر کدام مرز فعلی پروژه و یک تمرین خوب برای مرحلهٔ بعد را نشان می‌دهند.

## ۱۷. چند سؤال برای اطمینان از یادگیری

1. چرا `List<User>` می‌تواند هم دانشجو و هم استاد نگه دارد؟
2. `CurrentUser` چه مسئله‌ای را حل می‌کند؟
3. چرا `Student.Classes` با `[]` مقداردهی شده است؟
4. تفاوت `Capacity` و `RemainingCapacity` چیست؟
5. چرا گرفتن ورودی کاربر را داخل Service قرار نمی‌دهیم؟
6. در `object value = 10` چه عملی رخ می‌دهد؟
7. چرا `(long)value` برای object حاوی `int` درست نیست؟
8. آیا `object name = "Sara"` Boxing است؟ چرا؟
9. تبدیل `Student` به `User` چه نام دارد؟
10. چرا تبدیل `CurrentUser` به `Student` را Unboxing نمی‌نامیم؟
11. چرا Upcasting به cast صریح نیاز ندارد، اما Downcasting ممکن است خطرناک باشد؟
12. چگونه با `is` از `InvalidCastException` جلوگیری می‌کنیم؟

## ۱۸. تمرین‌ها

### تمرین ۱: پیش‌بینی خروجی

بدون اجرا خروجی را حدس بزن:

```csharp
int a = 5;
object b = a;
a = 9;
int c = (int)b;

Console.WriteLine(a);
Console.WriteLine(b);
Console.WriteLine(c);
```

### تمرین ۲: پیدا کردن نوع تبدیل

برای هر خط بنویس Boxing، Unboxing، Upcasting یا Downcasting رخ داده است:

```csharp
object first = 25;
int second = (int)first;
User third = new Student("A", "B", "ab", "123456");
Student fourth = (Student)third;
```

### تمرین ۳: امن‌کردن Service

متد `GetListOfMyClass` را با `is` بازنویسی کن تا اگر کاربر جاری دانشجو نبود، cast مستقیم انجام نشود.

### تمرین ۴: کامل‌کردن ثبت کلاس

در `RegisterClass` این مراحل را پیاده‌سازی کن:

1. دانشجوی جاری را پیدا کن.
2. کلاس را با `classId` پیدا کن.
3. وجود ظرفیت خالی را بررسی کن.
4. تکراری نبودن کلاس را بررسی کن.
5. دانشجو را به کلاس و کلاس را به دانشجو اضافه کن.

### تمرین ۵: پیام ورود

برای سه حالت پیام مناسب نمایش بده:

- نام کاربری وجود ندارد؛
- رمز اشتباه است؛
- ورود موفق است.

### تمرین ۶: تشخیص نقش با Downcasting امن

یک آرایه از نوع `User[]` بساز و داخل آن دو `Student` و یک `Teacher` قرار بده. با یک حلقهٔ `foreach`:

1. نام کاربری همه را چاپ کن؛
2. با الگوی `is Student student` دانشجوها را تشخیص بده؛
3. فقط برای دانشجوها `StudentNO` را چاپ کن؛
4. توضیح بده چرا cast مستقیم همهٔ اعضا به `Student` اشتباه است.

## جمع‌بندی

نسخهٔ دوم گلستان مسئولیت‌ها را روشن‌تر کرده است: موجودیت‌ها مدل داده‌اند، `InMemoryDB` دادهٔ موقت و کاربر جاری را نگه می‌دارد، Service عملیات را انجام می‌دهد و `Program.cs` با کاربر ارتباط برقرار می‌کند. یک لیست از نوع `User` می‌تواند نقش‌های مختلف را نگه دارد و نوع واقعی شیء هنگام اجرا مشخص می‌کند کدام صفحه باز شود.

Boxing زمانی رخ می‌دهد که یک value type مثل `int` داخل `object` قرار بگیرد؛ Unboxing بیرون آوردن همان نوع دقیق از object است. تبدیل `Student` و `User` موضوع دیگری است: این دو reference type هستند و تبدیل میان آن‌ها Upcasting یا Downcasting نام دارد. تشخیص این تفاوت، هم فهم حافظه را بهتر می‌کند و هم جلوی خطاهای رایج در خواندن پروژه را می‌گیرد.

## فایل مرجع و پیش‌نیازها

- کد مرجع مدرس: پروژهٔ `Golestan 2` ارائه‌شده در کلاس.
- [جلسهٔ چهارم: متدها، نوع‌ها و حافظه](session-04.md)
- [جلسهٔ نهم: ارث‌بری و چندریختی](session-09.md)
- [جلسهٔ یازدهم: List و generic](session-11.md)
- [جلسهٔ دوازدهم: تحلیل نسخهٔ اول پروژهٔ گلستان](session-12.md)

[→ جلسهٔ قبل: تحلیل پروژهٔ گلستان](session-12.md) · [بازگشت به فهرست جلسات](index.md)
