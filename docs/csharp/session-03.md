---
title: جلسهٔ سوم · آرایه، کنترل حلقه و پردازش متن
---

# جلسهٔ سوم · کار با چند داده و بخش‌های متن

<div class="lesson-intro">تاریخ برگزاری: <bdi dir="ltr">1405/07/30</bdi><br>مسیر حلقه را کنترل می‌کنیم، چند مقدار را در آرایه نگه می‌داریم و بخش دلخواه یک متن را پیدا و جدا می‌کنیم.</div>

## هدف‌های جلسه

- تشخیص تفاوت `break` و `continue`.
- تعریف آرایه، خواندن و تغییر عضوهای آن.
- پیمایش آرایه با `for` و `foreach`.
- جست‌وجو در متن با `IndexOf()` و جدا کردن متن با `Substring()`.
- ترکیب این ابزارها در مسئله‌های کوچک و کاربردی.

**پیش‌نیاز:** متغیر و تبدیل نوع از [جلسهٔ اول](session-01.md)، شرط و حلقه از [جلسهٔ دوم](session-02.md). هر مثال را جداگانه اجرا کن. برای نمونه‌های جست‌وجوی متن از حروف سادهٔ انگلیسی استفاده کرده‌ایم تا جایگاه‌ها روشن باشند.

## ۱. خروج از حلقه با break

`break` اجرای نزدیک‌ترین حلقه‌ای را که داخل آن هستیم تمام می‌کند. برنامه از اولین دستور بعد از آن حلقه ادامه پیدا می‌کند.

```csharp
for (int number = 1; number <= 5; number++)
{
    if (number == 3)
    {
        break;
    }

    Console.WriteLine(number);
}

Console.WriteLine("Finished");
```

خروجی:

```text
1
2
Finished
```

وقتی شمارنده به ۳ می‌رسد، `break` پیش از دستور چاپ اجرا می‌شود؛ پس خود ۳ هم چاپ نمی‌شود. `break` کل برنامه را نمی‌بندد و فقط خروج از `if` نیست.

**تمرین کوتاه:** جای دستور چاپ را به قبل از شرط منتقل کن. پیش‌بینی کن آیا ۳ چاپ می‌شود یا نه و بعد نتیجه را بررسی کن.

## ۲. رد کردن یک دور با continue

`continue` باقی دستورهای **همین دور** را رد می‌کند؛ خود حلقه تمام نمی‌شود.

```csharp
for (int number = 1; number <= 5; number++)
{
    if (number == 3)
    {
        continue;
    }

    Console.WriteLine(number);
}
```

خروجی:

```text
1
2
4
5
```

در `for`، پس از `continue` بخش تغییر شمارنده اجرا می‌شود و بعد شرط دور بعد بررسی می‌شود. در `while`، اجرا مستقیم به بررسی شرط برمی‌گردد. در `do while` هم به بررسی شرط انتهای حلقه می‌رسیم.

### مراقب شمارنده در while باش

اگر افزایش شمارنده را بعد از `continue` بگذاریم، ممکن است در یک مقدار گیر کنیم. در این مثال افزایش قبل از تصمیم به رد کردن دور انجام شده است:

```csharp
int number = 0;

while (number < 5)
{
    number++;

    if (number == 3)
    {
        continue;
    }

    Console.WriteLine(number);
}
```

این مثال هم ۱، ۲، ۴ و ۵ را چاپ می‌کند.

| دستور | اثر |
| --- | --- |
| `break` | پایان نزدیک‌ترین حلقه |
| `continue` | رد کردن ادامهٔ دور فعلی و رفتن به مرحلهٔ بعد حلقه |

## ۳. آرایه؛ چند مقدار زیر یک نام

اگر چهار نمره داشته باشیم، تعریف چهار متغیر جداگانه مدیریتشان را سخت می‌کند. آرایه چند مقدار **از یک نوع** را زیر یک نام نگه می‌دارد:

```csharp
int[] scores = { 12, 18, 9, 20 };
string[] names = { "Sara", "Ali", "Mina" };

Console.WriteLine(scores[0]); // 12
Console.WriteLine(names[1]);  // Ali
```

`[]` کنار نوع یعنی متغیر آرایه است. عدد داخل کروشه هنگام خواندن، **اندیس** یا جایگاه عضو است. اندیس از صفر شروع می‌شود.

| اندیس | مقدار در scores |
| --- | --- |
| `0` | `12` |
| `1` | `18` |
| `2` | `9` |
| `3` | `20` |

### طول و تغییر عضو

```csharp
int[] scores = { 12, 18, 9, 20 };
Console.WriteLine(scores.Length); // 4

scores[2] = 15;
Console.WriteLine(scores[2]);     // 15
Console.WriteLine(scores[scores.Length - 1]); // 20
```

`Length` تعداد عضوهاست؛ آخرین اندیس، یک واحد کمتر از آن است. `scores[4]` برای این آرایه معتبر نیست و هنگام اجرا خطا می‌دهد. برای خواندن عضو، اندیس باید بین صفر و `Length - 1` باشد.

طول یک آرایه بعد از ساخته شدن تغییر نمی‌کند، ولی مقدار عضوهای آن قابل تغییر است. فعلاً وارد ابزارهای افزایش اندازه نمی‌شویم.

### ساخت آرایه با تعداد مشخص

```csharp
int[] scores = new int[3];
Console.WriteLine(scores[0]); // 0

scores[0] = 14;
scores[1] = 16;
scores[2] = 18;
Console.WriteLine(scores.Length); // 3
```

`new int[3]` آرایه‌ای با سه عضو عدد صحیح می‌سازد و مقدار اولیهٔ هر عضو آن صفر است. در اینجا عدد `3` تعداد عضوهاست، نه آخرین اندیس. این رفتار مقدار اولیهٔ عضوهای آرایه را با متغیر محلی بدون مقداردهی جلسهٔ اول اشتباه نگیر.

## ۴. پیمایش آرایه با for

برای دسترسی به تک‌تک عضوها می‌توانیم اندیس را از صفر تا قبل از طول آرایه جلو ببریم:

```csharp
int[] scores = { 12, 18, 9, 20 };

for (int index = 0; index < scores.Length; index++)
{
    Console.WriteLine($"Score {index + 1}: {scores[index]}");
}
```

خروجی:

```text
Score 1: 12
Score 2: 18
Score 3: 9
Score 4: 20
```

برای نمایش انسانی از `index + 1` استفاده کرده‌ایم، اما برای دسترسی به آرایه هنوز `index` لازم است. شرط `index <= scores.Length` اشتباه است؛ در دور آخر به عضو خارج از محدوده می‌رسد.

## ۵. پیمایش ساده‌تر با foreach

وقتی فقط مقدار هر عضو را می‌خواهیم و به اندیس نیاز نداریم، `foreach` ساده‌تر است:

```csharp
string[] names = { "Sara", "Ali", "Mina" };

foreach (string name in names)
{
    Console.WriteLine($"Hello, {name}!");
}
```

در هر دور، مقدار عضو بعدی در `name` قرار می‌گیرد. برای این آرایه سه دور اجرا می‌شود. آرایهٔ خالی هیچ دوری ندارد.

در `foreach` معمولی این جلسه، به متغیر پیمایش مقدار تازه نده؛ مثلاً `name = "Reza"` در بدنه مجاز نیست. اگر می‌خواهی عضوهای آرایهٔ عدد یا متن را عوض کنی یا شمارهٔ جایگاه را لازم داری، از `for` و اندیس استفاده کن.

## ۶. پیدا کردن جایگاه با IndexOf

در این بخش، منظور `IndexOf()` روی **رشته** است. این ابزار جایگاه اولین رخدادِ متن یا نشانهٔ مورد نظر را برمی‌گرداند. اگر پیدا نشود، نتیجه `-1` است.

```csharp
string text = "CSharp Bootcamp";

Console.WriteLine(text.IndexOf("Boot")); // 7
Console.WriteLine(text.IndexOf(' '));    // 6
Console.WriteLine(text.IndexOf("Java")); // -1
```

فاصله هم جایگاه دارد. در این متن، `C` در اندیس صفر و `B` در اندیس ۷ است.

```text
C S h a r p [space] B o o t c a m p
0 1 2 3 4 5    6    7 8 9 ...
```

برای مثال‌های سادهٔ انگلیسی، حروف بزرگ و کوچک را یکسان فرض نکن؛ `IndexOf("Boot")` و `IndexOf("boot")` در این متن نتیجهٔ یکسانی ندارند. فعلاً همان متن دقیق مورد جست‌وجو را وارد کن و وارد تنظیمات پیشرفتهٔ مقایسه نشو.

### بررسی نتیجه قبل از استفاده

```csharp
string email = "sara@example.com";
int atIndex = email.IndexOf('@');

if (atIndex != -1)
{
    Console.WriteLine($"Found @ at index {atIndex}");
}
else
{
    Console.WriteLine("No @ found");
}
```

شرط `atIndex > 0` برای «پیدا شدن در هر جای متن» کافی نیست؛ ممکن است چیزی در اندیس صفر باشد. برای بررسی وجود، از `atIndex != -1` یا `atIndex >= 0` استفاده کن.

## ۷. جدا کردن بخشی از متن با Substring

`Substring` بخشی از متن را در یک رشتهٔ جدید برمی‌گرداند؛ متن اصلی تغییر نمی‌کند.

```csharp
string course = "CSharp Bootcamp";

string firstPart = course.Substring(0, 6);
string secondPart = course.Substring(7);

Console.WriteLine(firstPart);  // CSharp
Console.WriteLine(secondPart); // Bootcamp
Console.WriteLine(course);     // CSharp Bootcamp
```

دو شکل مورد استفاده در این دوره:

| شکل | معنی |
| --- | --- |
| `Substring(startIndex)` | از اندیس شروع تا انتهای رشته |
| `Substring(startIndex, length)` | از اندیس شروع، به تعداد مشخص |

عدد دوم **تعداد نویسه‌ها** است، نه اندیس پایان. مثلاً `"Bootcamp".Substring(1, 3)` برابر `"oot"` است.

برای جلوگیری از خطا، اندیس شروع باید از صفر تا `Length` باشد، طول درخواستی منفی نباشد و مجموع شروع و طول از `Length` بیشتر نشود. شروع از خود `Length` با طول صفر، متن خالی می‌دهد. برای مثال `"Ali".Substring(5)` نامعتبر است.

### ترکیب IndexOf و Substring

در این مثال دو بخش یک متن ساده را از روی فاصله جدا می‌کنیم:

```csharp
string fullName = "Sara Ahmadi";
int spaceIndex = fullName.IndexOf(' ');

if (spaceIndex > 0 && spaceIndex < fullName.Length - 1)
{
    string firstName = fullName.Substring(0, spaceIndex);
    string lastName = fullName.Substring(spaceIndex + 1);

    Console.WriteLine(firstName); // Sara
    Console.WriteLine(lastName);  // Ahmadi
}
else
{
    Console.WriteLine("Enter two parts separated by a space.");
}
```

اینجا `spaceIndex > 0` عمدی است: فاصله نباید اولین نشانه باشد. شرط دوم هم تضمین می‌کند بعد از فاصله متن داریم. `+ 1` باعث می‌شود خود فاصله وارد بخش دوم نشود. این فقط یک تمرین با قالب ساده است؛ راهکار کامل پردازش همهٔ نام‌های واقعی نیست.

## مثال کاربردی ۱ · گزارش نمره‌ها

در دادهٔ تمرینی زیر، `-1` نشانهٔ نمرهٔ ثبت‌نشده است. آن را از محاسبه کنار می‌گذاریم و تعداد قبولی‌ها را می‌شماریم:

```csharp
int[] scores = { 12, -1, 18, 7, 20 };
int total = 0;
int validCount = 0;
int passedCount = 0;

foreach (int score in scores)
{
    if (score == -1)
    {
        continue;
    }

    total += score;
    validCount++;

    if (score >= 10)
    {
        passedCount++;
    }
}

if (validCount > 0)
{
    double average = (double)total / validCount;
    Console.WriteLine($"Average: {average}");
    Console.WriteLine($"Passed: {passedCount}");
}
else
{
    Console.WriteLine("No scores recorded.");
}
```

نتیجه: میانگین `14.25` و تعداد قبولی `3`. از پنج عضو آرایه، فقط چهار نمره در محاسبه شرکت می‌کنند؛ به همین دلیل تقسیم بر `scores.Length` درست نیست. بررسی `validCount > 0` هم از تقسیم بر صفر جلوگیری می‌کند.

**آزمایش:** همهٔ عضوها را `-1` کن. برنامه باید پیام `No scores recorded.` را نمایش دهد.

## مثال کاربردی ۲ · پیدا کردن اولین نام

هدف این است که به محض پیدا شدن اولین نام برابر، جست‌وجو تمام شود:

```csharp
string[] names = { "Sara", "Ali", "Mina", "Ali" };
string target = "Ali";
int foundIndex = -1;

for (int index = 0; index < names.Length; index++)
{
    if (names[index] == target)
    {
        foundIndex = index;
        break;
    }
}

if (foundIndex == -1)
{
    Console.WriteLine("Not found");
}
else
{
    Console.WriteLine($"First match at index: {foundIndex}");
}
```

خروجی `First match at index: 1` است. `-1` را انتخاب کرده‌ایم چون اندیس معتبر آرایه نیست. اینجا جست‌وجوی آرایه را خودمان با حلقه نوشته‌ایم؛ `IndexOf` بخش قبلی روی متن کار می‌کرد.

## مثال کاربردی ۳ · جدا کردن دو بخش یک نشانی

یک متن با قالب سادهٔ `name@domain` می‌گیریم و دو بخشش را جدا می‌کنیم:

```csharp
Console.Write("Enter a value like sara@example.com: ");
string email = Console.ReadLine() ?? "";
int atIndex = email.IndexOf('@');

if (atIndex > 0 && atIndex < email.Length - 1)
{
    string userName = email.Substring(0, atIndex);
    string domain = email.Substring(atIndex + 1);

    Console.WriteLine($"User: {userName}");
    Console.WriteLine($"Domain: {domain}");
}
else
{
    Console.WriteLine("Use the name@domain format.");
}
```

برای `sara@example.com`، نتیجه دو بخش `sara` و `example.com` است. برای `sara`، `@example.com` یا `sara@` پیام قالب نمایش داده می‌شود. این مثال فقط وجود جداکننده و خالی نبودن دو طرف آن را بررسی می‌کند؛ **اعتبارسنجی کامل ایمیل نیست**.

## اشتباه‌های رایج

- استفاده از `break` وقتی فقط می‌خواهیم یک عضو را رد کنیم.
- استفاده از `continue` قبل از افزایش شمارندهٔ `while`.
- تصور اینکه اولین اندیس آرایه ۱ است.
- استفاده از `<= array.Length` در حلقهٔ دسترسی به عضوها.
- اشتباه گرفتن طول رشته با آخرین اندیس آن.
- استفاده از نتیجهٔ `IndexOf` بدون بررسی `-1`.
- تصور اینکه عدد دوم `Substring`، جایگاه پایان است.

## تمرین و مرور

۱. **رد کردن مضرب‌ها:** عددهای ۱ تا ۱۰ را با `for` چاپ کن، ولی مضرب‌های ۳ را با `continue` رد کن. انتظار: `1, 2, 4, 5, 7, 8, 10`.

۲. **توقف با صفر:** حداکثر پنج عدد صحیح بگیر. اگر کاربر صفر وارد کرد با `break` متوقف شو و مجموع عددهای قبل از صفر را چاپ کن. برای `4`، `6` و `0`، انتظار `10` است. نیازی به آرایه نیست.

۳. **آرایهٔ نمره‌ها:** یک `int[4]` بساز و چهار نمرهٔ معتبر را با `for` در آن ذخیره کن. سپس با `foreach` همه را نمایش بده. برای ورود نمرهٔ اول از اندیس صفر استفاده کن.

۴. **بزرگ‌ترین عدد:** بزرگ‌ترین عضو `{ -8, -3, -12, -5 }` را با حلقه پیدا کن. مقدار اولیهٔ بزرگ‌ترین را از عضو اول بگیر، نه از صفر. انتظار: `-3`. فرض کن آرایه خالی نیست.

۵. **اولین عدد منفی:** در `{ 4, 7, -2, 9, -5 }` اولین عدد منفی و اندیس آن را پیدا کن و جست‌وجو را متوقف کن. انتظار: مقدار `-2` و اندیس `2`.

۶. **پیش‌بینی متن:** برای `string text = "Ali-Reza";` نتیجهٔ `text.IndexOf('-')`، `text.Substring(0, 3)` و `text.Substring(4)` را قبل از اجرا بنویس. انتظار: `3`، `Ali` و `Reza`.

۷. **تفکیک کد کالا:** متن با قالب `BOOK-125` را از اولین خط تیره به دو بخش تقسیم کن. خروجی باید `BOOK` و `125` به صورت متن باشد. اگر جداکننده نبود یا در ابتدا یا انتهای متن بود، پیام مناسب بده. با `BOOK125`، `-125` و `BOOK-` هم بررسی کن. قالب تمرین فقط یک جداکننده دارد.

۸. **گزارش بدون داده:** مثال گزارش نمره‌ها را با آرایهٔ خالی `int[] scores = { };` آزمایش کن. بدون دسترسی به عضو اول، همان پیام نبود نمره باید نمایش داده شود.

**معیار پایان جلسه:** بتوانی یک آرایه را بدون خروج از محدوده پیمایش کنی و پیش از جدا کردن متن، وجود و جایگاه جداکننده را بررسی کنی.

## منابع مطالعه

- [W3Schools · break و continue](https://www.w3schools.com/cs/cs_break.php)، [آرایه‌ها](https://www.w3schools.com/cs/cs_arrays.php) و [foreach](https://www.w3schools.com/cs/cs_foreach_loop.php)
- [W3Schools · دسترسی به رشته و جست‌وجو](https://www.w3schools.com/cs/cs_strings_access.php)
- [Microsoft Learn · String.IndexOf](https://learn.microsoft.com/en-us/dotnet/api/system.string.indexof) و [String.Substring](https://learn.microsoft.com/en-us/dotnet/api/system.string.substring)
- [Microsoft Learn · آرایه‌ها](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/arrays)؛ برای این جلسه فقط بخش آرایهٔ یک‌بعدی را مرور کن.

[→ جلسهٔ قبل: عملگرها، شرط و حلقه‌ها](session-02.md) · [بازگشت به فهرست جلسات](index.md)
