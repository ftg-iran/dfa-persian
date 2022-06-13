# فصل هفتم: احراز هویت کاربر

در فصل قبلی ما دسترسی‌های APIهایمان را بروز رسانی کردیم، که به آن **مجوز(authorization)** می‌گویند. در این فصل، ما **احراز هویت(authentication)** را پیاده سازی  می‌کنیم که فرآیندی است که، کاربر با آن می‌تواند برای حساب کاربری جدید ثبت نام کرده، به آن وارد یا خارج شود.

به طور سنتی، احراز هویت در وب سایت جنگو یکپارچه ساده است و متأثر از یک الگوی کوکی مبتنی بر جلسه(session based cookie pattern) است، که پایین تر آن را مورد بررسی قرار خواهیم داد. اما با وجود یک API، کارها کمی گول زننده‌تر می شود. به یاد داشته باشید که HTTP یک  **پروتکل بدون حفظ حالت(stateless protocol)**  است پس هیچ راه پیش ساخته‌ای برای به خاطر سپاری اینکه یک کاربر از یک درخواست به درخواست بعدی احراز هویت شده است یا خیر، وجود ندارد. هر بار که یک  کاربر درخواست یک منبع محدود شده را می‌کند، باید تایید کند که خودش است. 

راه حل آن، ارسال یک نشان یکتا به همراه هر درخواست می‌باشد.  به صورت گیج کننده‌ای ،یک دیدگاه  توافق شده جهانی برای این نشان یکتا تعریف نشده است  و می تواند چندین فرم داشته باشد. فریمورک رست جنگو با [چهار نوع آپشن مختلف احراز هویت پیش ساخته](https://www.django-rest-framework.org/api-guide/authentication/#api-reference) عرضه می‌شود:  پایه(basic)، جلسه(session)، توکن(token) و پیش فرض(default). پکیج‌های واسط زیادی وجود دارند که  ویژگی‌های بیشتری مانند  جیسون وب توکن‌ها را(Json Web Token یا به اختصار JWT)  ارائه می دهند.  

در این فصل به طور کامل بررسی می‌کنیم که احراز هویت API چگونه کار میکند، همچنین مزایا و معایب هر رویکرد را نیز مرور می‌کنیم و سپس یک انتخاب آگاهانه برای API *وبلاگ* خود انجام می دهیم. و در نهایت اندپوینت‌های API برای ثبت نام، ورود به حساب کاربری و  خروج از آن پیاده سازی می‌کنیم.


## احراز هویت پایه

رایج ترین فرم احراز هویت HTTP به عنوان  [احراز هویت «پایه»](https://tools.ietf.org/html/rfc7617) شناخته می‌شود. زمانی که کلاینت درخواست HTTP ارسال می‌کند، مجبور به ارسال یک اعتبارنامه(credential) احراز هویت تایید شده قبل از اعطای دسترسی است.

پروسه کامل درخواست/ پاسخ به صورت زیر است :

1. کاربر یک درخواست http ارسال می‌کند.
2. سرور یک پاسخ که حاوی کد وضعیت `401 غیرمجاز (unauthorized)` و یک هدر  `WWW-Authenticate` HTTP با جزئیات *چگونگی* دسترسی است را برمی‌گرداند.
3. کاربر اعتبارنامه خود را از طریق هدر [مجوز](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) HTTP ارسال می کند.
4. سرور اعتبارنامه را چک کرده و پاسخ را به همراه یکی از کد وضعیت‌های `200 درست` یا `403 ممنوع(403 forbidden)` را به سمت کاربر ارسال می کند.

یک بار که کاربر تایید شد، می‌تواند تمام درخواست‌های بعدی خود را با اعتبارنامه هدر `Authorization` HTTP  ارسال کند.  ما میتوانیم این فرایند را به صورت زیر نمایش دهیم:

نمودار
```code
Client                                                                                                                  Server
------                                                                                                                  ------
  
--------------------------------------->
GET / HTTP/1.1
  
                                                                                        <-------------------------------------
                                                                                                     HTTP/1.1 401 Unauthorized
                                                                                                     WWW-Authenticate: Basic
                                                                                                                               
                                                                                                                               
--------------------------------------->
GET / HTTP/1.1
Authorization: Basic d3N2OnBhc3N3b3JkMTIz
                                                                                          
                                                                                          
                                                                                         <-------------------------------------
                                                                                                                HTTP/1.1 200 OK
```

توجه داشته باشید که مجوزهای اعتبارنامه ارسال شده بر اساس  [base64 encode](https://en.wikipedia.org/wiki/Base64) رمز گذاری نشده، نسخه‌ای از `<username>:<password>` هستند. به عنوان مثال `wsv:password123` با base64 encoding  به صورت `d3N2OnBhc3N3b3JkMTIz` است.

مزیت اصلی این روش سادگی آن است اما چندین نقطه ضعف عمده نیز دارد. مورد اول، برای *هر درخواست*، سرور باید نام کاربری و رمز عبور را جستجو کرده و تایید کند که این عمل ناکارآمد است. اما روش بهتر این است که احراز هویت یکبار صورت گیرد و برای درخواست های بعدی یک توکن ارسال شود که تایید کند این کاربر احراز شده است. مورد دوم، ارسال اعتبارنامه‌های کاربر به صورت رمزگذاری نشده در سراسر اینترنت،  فوق العاده ناامن است. هر ترافیک شبکه‌ای که رمزگذاری نشده باشد خیلی راحت می‌تواند دریافت شده و مجدداً استفاده شود. بدین ترتیب احراز هویت پایه  **فقط** باید به وسیله پروتکل [HTTPS](https://en.wikipedia.org/wiki/HTTPS) مورد استفاده قرار بگیرد که نسخه امن شدۀ `HTTP` است.

 
## احراز هویت مبتنی بر جلسه

وبسایت‌های یکپارچه مانند وبسایت‌های سنتی جنگو، مدت‌هاست که از احراز هویت مبتنی بر جلسه و کوکی به صورت ترکیبی استفاده  می‌کنند. در سطح‌های بالاتر،کلاینت با اعتبارنامه خود(نام کاربری و پسورد) احراز هویت می‌کند و بعد از طرف سرور یک *شناسه جلسه(session ID)* دریافت می‌کند که به عنوان یک کوکی ذخیره می‌گردد. سپس این شناسه جلسه در هر یک از هدرهای درخواست‌های HTTP آینده ارسال می‌شود.

زمانی که  شناسه جلسه فرستاده شد، سرور از این شئ جلسه برای جستجوی تمام اطلاعات در دسترس کاربر داده شده از جمله اعتبارنامه استفاده می‌کند.

این، یک روش **با حفظ حالت(stateful)** می‌باشد،  به این دلیل که این سابقه باید در هر دو طرف یعنی شئ جلسه در سمت سرور و شناسه جلسه در سمت کلاینت حفظ و نگه‌داری گردد.

بیایید این روند را مرور کنیم:

1. کاربر با اعتبارنامه خود(نام کاربری و رمز عبور) وارد می‌شود
2. سرور اعتبارنامه را بررسی می‌کند که درست باشد و در این صورت یک شئ جلسه ایجاد کرده و آن را در دیتابیس ذخیره میکند
3. سرور یک شناسه جلسه به سمت کلاینت ارسال می‌کند(خود شئ جلسه ارسال نمی‌شود بلکه ID آن ارسال می‌شود) که به عنوان کوکی در مرورگر ذخیره می‌شود
4. در تمام درخواست‌های بعدی شناسه جلسه به عنوان هدر HTTP گنجانده شده و در صورتی که این شناسه توسط دیتابیس تایید شود، درخواست پردازش می‌شود
5. یک بار که کاربر از حساب کاربری خود خارج شود آنگاه شناسه جلسه هم از سمت کلاینت و هم از سمت سرور حذف می شود
6. .اگر کاربر مجدداً در حساب کاربری خود وارد شود آنگاه شناسه جلسه جدید توسط سرور ایجاد شده و به عنوان کوکی در سمت کلاینت ذخیره می‌شود

تنظیمات پیش فرض فریمورک رست جنگو در واقع ترکیبی از احراز هویت پایه و مبتنی بر جلسه است. سیستم احراز هویت سنتی مبتنی بر جلسه جنگو استفاده می‌شود و شناسه جلسه در هدر HTTP هر درخواست از طریق احراز هویت پایه ارسال می‌گردد. 

مزیت این روش این است که ایمن‌تر است، زیرا اعتبارنامه کاربر فقط یک بار ارسال می شود، نه مانند احراز هویت پایه در هر چرخه درخواست/پاسخ، اعتبارنامه کاربر ارسال می‌شود. از طرفی این روش کارآمدتر است زیرا سرور مجبور نیست هر بار اعتبارنامه کاربر را تأیید کند، فقط شناسه جلسه را با شئ جلسه مطابقت می‌دهد که یک جستجوی سریع است. 

با این وجود چند جنبه منفی نیز وجود دارد. اولاً آیدی جلسه فقط در مرورگری که کاربر در آن لاگین شده است معتبر است و در چندین دامنه کار نخواهد کرد. اما این یک مشکل واضح است زمانی که یک API نیاز به ساپورت چندین فرانت اند مانند یک وب سایت و یک اپلیکیشن موبایل دارد. دوماً، شی جلسه باید به روز نگه داشته شود که می‌تواند چالشی در سایت‌های بزرگ که چندین سرور دارند، باشد. چگونه درستی یک شئ جلسه را در هر سرور حفظ می‌کنید؟ و سوماً ارسال کوکی در هر درخواست، حتی درخواست‌هایی که نیاز به احراز هویت ندارند ، ناکارآمد است.

در نتیجه، عموماً استفاده از احراز هویت مبتنی بر جلسه برای APIهایی که چندین فرانت اند دارند توصیه نمی‌شود.


## احراز هویت مبتنی بر توکن

سومین دیدگاه عمده و روشی که احراز هویتی که ما برای API *وبلاگمان* پیاده‌سازی خواهیم کرد، استفاده از احراز هویت مبتنی بر توکن است. این روش یکی از محبوب ترین دیدگاه‌ها در سال‌های اخیر است که ناشی از رشد اپلیکیشن‌های تک صفحه‌ای است.

احراز هویت مبتنی بر توکن **فاقد حفظ حالت(stateless)** هستند. یک بار که اعتبارنامه اولیه کاربر را به سرور ارسال میکند، یک توکن یکتا تولید شده و سپس در سمت کلاینت به عنوان کوکی یا در  [حافظه محلی](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) ذخیره می‌شود. این توکن در هدر هر درخواست HTTP ارسال شده و سرور با آن احراز هویت کاربر را تایید می‌کند. خود سرور چه توکن معتبر باشد چه نباشد، سابقه‌ای از کاربر نگهداری نمی‌کند.

<br>
<div style="background-color:#f0f0f0; padding:5%">
<p style="font-weight:bold;">کوکی‌ها در مقابل حافظه محلی</p> کوکی‌ها برای خواندن اطلاعات از <span style="font-weight:bold;">سمت سرور</span> استفاده می‌شود و از نظر اندازه کوچک هستند (4KB) و به صورت خودکار به همراه هر درخواست HTTP  ارسال می‌شوند. حافظه محلی برای اطلاعات <span style="font-weight:bold;">سمت کلاینت</span> طراحی شده‌اند و بسیار بزرگتر هستند(5120KB) و محتوای آن‌ها به صورت پیشفرض در هر درخواست HTTP ارسال نمی‌شود. توکن‌های ذخیره شده در کوکی ها و یا حافظه‌های محلی در برابر حمله‌های XSS آسیب پذیر هستند. بهترین روش فعلی، ذخیره آن‌ها در یک کوکی به همراه فلگ‌های <code>httpOnly</code> و <code>Secure</code>  می‌باشد. 
</div>
<br>

بیایید به یک نسخه ساده از پیام‌های HTTP واقعی در این جریان چالش/پاسخ نگاه کنیم. توجه داشته باشید که هدر `WWW-Authenticate` استفاده از `Token` را مشخص کرده است، که این توکن نیز در هدر `Authorization` درخواست پاسخ در نظر گرفته شده است. 

نمودار
```code
Client                                                                                                                  Server
------                                                                                                                  ------
  
--------------------------------------->
GET / HTTP/1.1
  
                                                                                        <-------------------------------------
                                                                                                     HTTP/1.1 401 Unauthorized
                                                                                                     WWW-Authenticate: Token
                                                                                                                               
                                                                                                                               
--------------------------------------->
GET / HTTP/1.1
Authorization: Token 401f7ac837da42b97f613d789819ff93537bee6a
                                                                                          
                                                                                          
                                                                                         <-------------------------------------
                                                                                                                HTTP/1.1 200 OK
```

در این دیدگاه چندین مزیت وجود دارد از آنجایی که توکن‌ها در کلاینت ذخیره می‌شوند، دیگر مقیاس سرورها به منظور به‌ روز‌ نگه داشتن شئ جلسه، یک مشکل نخواهد بود. و توکن‌ها می‌توانند بین چندین فرانت اند به اشتراک گذاشته شوند. همان توکن می‌تواند نمایانگر یک کاربر روی وب سایت و همان کاربر روی اپلیکیشن موبایل باشد. اما همان شناسه جلسه نمی‌تواند بین فرانت اند‌های مختلف به اشتراک گذاشته شود که این محدودیت عمده این روش است.

یک نقطه ضعف  بالقوه در این روش این است که یک توکن ممکن است خیلی بزرگ شود. یک توکن شامل تمام اطلاعات یک کاربر است نه فقط شناسه به عنوان شناسه جلسه یا شی جلسه تنظیم شده. از آنجایی که توکن‌ها در هر درخواست ارسال می شوند مدیریت اندازه آنها می‌تواند به یک مسئله کارایی تبدیل شود.

نحوه پیاده سازی دقیق توکن به طور قابل توجهی می‌تواند متفاوت باشد. [احراز هویت مبتنی بر توکن](http://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication) پیش ساخته فریمورک رست جنگو عمداً اساسی طراحی شده است. به این معنا که تنظیماتی برای منقضی کردن توکن پشتیبانی نمی‌شود که یک بهبود امنیتی است، که می‌تواند اضافه شود. همچنین فقط یک توکن برای هر کاربر تولید می‌کند بنابراین یک کاربر در وبسایت و در اپلیکیشن موبایل از همان یک توکن استفاده می‌کند . از آن جایی که اطلاعات مربوط به کاربر به صورت محلی ذخیره می‌شود، این خود می‌تواند سبب مشکلی برای نگهداری و به روز رسانی دو مجموعه از اطلاعات کلاینت شود.

جیسون وب توکن‌ها(JWT) جدید هستند در واقع نسخه بهبود یافته توکن‌ها هستند که می‌توانند به فریمورک رست جنگو از طریق پکیج‌های واسط اضافه شوند. JWT ها چندین مزیت دارد از جمله اینکه می‌توانند  توکن‌های یکتا برای کلاینت تولید کنند که دارای تایم انقضا باشد. آنها همچنین می‌توانند روی سرور و یا با سرویس‌های واسط مانند [Auth0](https://auth0.com/) تولید شوند و JWT ها می توانند رمز گذاری شوند تا به صورت ایمن بر بستر ارتباطات ناامن HTTP ارسال شوند.

در نهایت مطمئن‌ترین شرط برای اکثر APIهای وب استفاده از طرح احراز هویت مبتنی بر توکن است. JWTها یک قابلیت اضافه هستند اگرچه نیاز به پیکربندی اضافه‌تری دارند. در نتیجه، در این کتاب از `TokenAuthentication` پیش ساخته استفاده خواهیم کرد.

 
## احراز هویت پیش فرض

اولین قدم این است که تنظیمات احراز هویت جدیدمان را پیکربندی کنیم. فریمورک رست جنگو با [تعدادی تنظیمات](http://www.django-rest-framework.org/api-guide/settings/) ارائه می شود که به طور ضمنی تنظیم شده اند .برای مثال قبل از تغییر `DEFAULT_PERMISSION_CLASSES` به  `IsAuthenticate`  به صورت پیش فرض بر روی `AllowAny` تنظیم شده است.

<div style="dir:rtl;">
<code>DEFAULT_AUTHENTICATION_CLASSES</code> به طور پیش‌فرض تنظیم شده‌ است، بنابراین بیایید صراحتاً <code>SessionAuthentication</code> و <code>BasicAuthentication</code> را به فایل <code>config/settings.py</code> خود اضافه کنیم.
</div>
<br>

کد
```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [ # new
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication'
    ],
}
```

چرا از هر **دو** روش استفاده کنیم؟ پاسخ این است که آن‌ها اهداف مختلفی را دنبال می‌کنند. Sessions برای تقویت API قابل مرور استفاده می‌شوند و توانایی وارد شدن و خارج شدن از آن. BasicAuthentication برای ارسال شناسه جلسه در هدرهای HTTP برای خود API استفاده می‌شود.

اگر دوباره به API قابل مرور در <http://127.0.0.1:8000/api/v1/> مراجعه کنید، مانند قبل کار خواهد کرد. از نظر فنی، هیچ چیز تغییر نکرده است، ما فقط تنظیمات پیش فرض را به صراحت بیان کرده‌ایم.  
                                                                                          
                                                                                          
## پیاده سازی احراز هویت مبتنی بر توکن

اکنون نیاز داریم تا سیستم احراز هویت خود را بروز رسانی کنیم تا بتوانیم از توکن‌ها استفاده کنیم. قدم اول به روز رسانی  `DEFAULT_AUTHENTICATION_CLASS` به مقدار `TokenAuthentication` مانند زیر می‌باشد:

کد
```python
# config/settings.py
REST_FRAMEWORK = {
'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication', # new
    ],
}

```

ما `SessionAuthentication` را حذف نکرده زیرا برای APIهای قابل مرور به آن‌ها نیاز داریم، اما اکنون از توکن‌ها برای ارسال و دریافت اعتبارنامه احراز هویت در هدرهای HTTPمان استفاده می‌کنیم. همچنان ما نیاز داریم که اپ `authtoken` را برای تولید توکن‌ها روی سرور اضافه کنیم. آن اپ به همراه فریمورک رست جنگو است اما باید در بخش `INSTALLED_APPS` افزوده شود:

کد
```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
  
    # 3rd-party apps
    'rest_framework',
    'rest_framework.authtoken', # new
  
    # Local
    'posts',
]
```

از آنجایی که در INSTALLED_APPS تغییراتی ایجاد کرده‌ایم نیاز داریم پایگاه داده خود را همگام سازی کنیم. سرور را با کلید ترکیبی `Control + c` متوقف کرده و سپس کامند زیر را اجرا کنید.

کامند لاین
```shell
(blogapi) $ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, authtoken, contenttypes, posts, sessions
Running migrations:
  Applying authtoken.0001_initial... OK
  Applying authtoken.0002_auto_20160226_1747... OK
```

سپس سرور را مجدداً راه اندازی کنید.

کامند لاین
```shell
(blogapi) $ python manage.py runserver
```

اگر به آدرس پنل مدیریت جنگو در <http://127.0.0.1:8000/admin/> رجوع کنید شما بخش `Tokens` را در بالا مشاهده خواهید کرد. پیش از آن اطمینان حاصل کنید که با کاربر superuser وارد شده‌اید که دسترسی داشته باشید.  

|![صفحه خانه پنل مدیریت جنگو به همراه توکن‌ها](images/1.jpg)|
|:--:|
|صفحه خانه پنل مدیریت جنگو به همراه توکن‌ها|

بر روی لینک `Tokens` کلیک کنید.در حال حاضر هیچ توکنی وجود ندارد که ممکن است برای شما سوپرایز کننده باشد.

|![صفحه مدیریت توکن‌ها](images/2.jpg)|
|:--:|
|صفحه مدیریت توکن‌ها|

در صورتی که ما همه کابران موجود را داریم. توکن‌ها *بعد از* اینکه کاربری برای وارد شدن به حسابش، API را صدا بزند ساخته می‌شوند، که این بخش را ما اکنون انجام ندادیم در نتیجه توکنی برای مشاهده وجود ندارد. اما به زودی این کار را انجام خواهیم داد.
                                                                                          
  
## نقاط پایانی(Endpoints)

ما همچنین نیاز داریم نقاط پایانی‌ای ایجاد کنیم که کاربران بتوانند از طریق آن وارد یا خارج شوند. ما *می‌توانیم* یک اپ اختصاصی `users`برای این منظور ایجاد کنیم و سپس urlها، ویوها و سریالایزرهای خود را به آن اضافه کنیم. با این وجود احراز هویت کاربر بخشی است که واقعا نمی‌خواهیم اشتباهی در آن رخ دهد. و از آن جایی ک همه API ها به این قابلیت نیاز دارند، منطقی است که چندین پکیج واسط عالی و تست شده وجود دارند که می‌توانیم از آنها استفاده کنیم. 

به ویژه ما از [dj-rest-auth](https://github.com/jazzband/dj-rest-auth) در ترکیب با [django-allauth](https://github.com/pennersr/django-allauth) برای ساده‌تر شدن کارها استفاده خواهیم کرد. هیچ گونه احساس بدی در مورد استفاده از پکیج‌های واسط نداشته باشید. آن‌ها به دلیلی وجود دارند و حتی حرفه‌ای‌های جنگو هم همیشه به آن‌ها تکیه می‌کنند. اگر مجبور نباشید، هیچ فایده‌ای برای اختراع مجدد چرخ از اول وجود ندارد.
                                                                                          
                                                                                          
## پکیج dj-rest-auth 

اول ما نقاط‌های پایانی APIهای وارد شدن، خارج شدن و بازنشانی رمز عبور را اضافه خواهیم کرد. این بلافاصله با پکیج محبوب `dj-rest-auth` می‌آید. سرور را با `Control + c` متوقف کنید و سپس آن را نصب کنید.

کامند لاین
```shell
(blogapi) $ pipenv install dj-rest-auth==1.1.0
```

اپ جدید را به پیکربندی `INSTALLED_APPS` در فایل `config/settings.py` اضافه کنید.

کد
```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
  
    # 3rd-party apps
    'rest_framework',
    'rest_framework.authtoken',
    'dj_rest_auth', # new
  
    # Local
    'posts',
]

```

فایل `config/urls.py` را با پکیج `dj-rest-auth` بروز رسانی کنید. ما url را به `api/v1/dj-rest-auth` مسیر دهی می‌کنیم. اطمینان حاصل کنید که URL تیره - از هم جدا شوند و نه خط فاصله _ . این یک خطای آسان است.

کد
```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path
  
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('posts.urls')),
    path('api-auth/', include('rest_framework.urls')),
    path('api/v1/dj-rest-auth/', include('dj_rest_auth.urls')), # new
]
```

و تمام. اگر تا به حال تلاش کرده باشید که نقطه پایانی احراز هویت کاربر را خودتان پیاده سازی کنید متوجه خواهید شد که `dj-rest-auth` واقعا فوق العاده است که چگونه چقدر از اتلاف وقت و سردرد ما کم می‌کند. اکنون می‌توانیم سرور را مجدداً راه اندازی کنیم تا ببینیم `dj-rest-auth` چه چیزی را برای ما فراهم کرده است.

کامند لاین
```shell
(blogapi) $ python manage.py runserver
```

ما یک نقطه پایانی برای وارد شدن در <http://127.0.0.1:8000/api/v1/dj-rest-auth/login/> داریم

|![نقطه پایانی API ورود کاربر](images/3.jpg)|
|:--:|
|نقطه پایانی API ورود کاربر|

و نقطه پایانی برای خارج شدن در <http://127.0.0.1:8000/api/v1/dj-rest-auth/logout> داریم

|![نقطه پایانی API خارج شدن کاربر](images/4.jpg)|
|:--:|
|نقطه پایانی API ورود کاربر|

و همچنین نقاط پایانی بازنشانی رمز عبور نیز به صورت زیر در نظر گرفته شده است:

<http://127.0.0.1:8000/api/v1/dj-rest-auth/password/reset/>

|![API بازنشانی رمز عبور](images/5.jpg)|
|:--:|
|API بازنشانی رمز عبور|

و برای تایید بازنشانی رمز عبور: 

<http://127.0.0.1:8000/api/v1/dj-rest-auth/password/reset/confirm>

|![API تایید بازنشانی رمز عبور](images/6.jpg)|
|:--:|
|API تایید بازنشانی رمز عبور|                                                                                      
                                                                                          
 
## ثبت نام کاربر

مرحله بعدی طراحی و پیاده سازی نقطه پایانی  ثبت نام کاربر است. جنگو سنتی یا حتی فریمورک رست جنگو، ویوها یا URLهایی را به صورت پیش ساخته برای ثبت نام کاربر طراحی نکرده است؛ به این معنا که نیاز داریم برای ثبت نام از صفر خودمان کد بنویسیم. این روش قدری با توجه به جدی بودن اشتباه و پیامدهای امنیتی آن قدری خطرناک است.

یک روش محبوب استفاده از پکیج واسط [django-allauth](https://github.com/pennersr/django-allauth) است که ویژگی ثبت نام کاربر را به همراه ویژگی‌های اضافه‌تری برای سیستم احراز هویت جنگو مانند احراز هویت از طریق فیس بوک، گوگل، توییتر و ... را دارد. اگر ما `dj-rest-auth.registration` را از پکیج `dj-rest-auth` اضافه کنیم در نتیجه نقاط پایانی  ثبت نام کاربر را نیز خواهیم داشت. 

سرور محلی را با `Control + c` متوقف کرده و پکیج `django-allauth` را نصب کنید.

کامند لاین
```shell
(blogapi) $ pipenv install django-allauth~=0.42.0
```

سپس بخش INSTALLED_APPS را بروز رسانی می‌کنیم. ما باید بعد از آن پیکربندی‌های جدیدی را اضافه کنیم:

- django.contrib.sites
- allauth
- allauth.account
- allauth.socialaccount
- dj_rest_auth.registration

اطمینان حاصل کنید که ` EMAIL_BACKEND` و ` SITE_ID` را اضافه کرده باشید. از نظر فنی مهم نیست این تنظیمات را کجای فایل `config/settings.py` قرار داده باشید اما معمولاً پیکربندی‌های اضافه را مانند قسمت زیر به انتهای  فایل اضافه می‌کنند. 

کد
```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django.contrib.sites', # new
  
    # 3rd-party apps
    'rest_framework',
    'rest_framework.authtoken',
    'allauth', # new
    'allauth.account', # new
    'allauth.socialaccount', # new
    'dj_rest_auth',
    'dj_rest_auth.registration', # new
  
    # Local
    'posts',
]
  
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend' # new
  
SITE_ID = 1 # new
```

پیکربندی email backend مورد نیاز است زیرا به صورت پیش فرض یک ایمیل  زمانی که یک کاربر جدید ثبت نام می‌کند ارسال خواهد شد، و از آن‌ها می‌خواهد که حساب کاربری خود را تایید کنند. *همچنین* بجای راه اندازی کردن یک سرور ایمیل، ایمیل‌ها را با تنظیم کردن `console.EmailBackend` درون کنسول نمایش خواهیم داد. 

پیکربندی `SITE_ID` یک بخش از [فریمورک «sites»](https://docs.djangoproject.com/en/3.1/ref/contrib/sites/) جنگو است که یک راهی برای میزبانی چندین وب سایت از یک پروژه جنگو یکسان است. در اینجا ما فقط یک وب سایت داریم که روی آن کار می‌کنیم اما `django-allauth` از فریمورک سایت استفاده می‌کند، پس ما باید تنظیمات پیش فرض را مشخص کنیم. 

حال که اپ‌های جدیدی را افزودیم، لازم است که پایگاه داده را بروزرسانی کنیم.

کامند لاین
```shell
(blogapi) $ python manage.py migrate
```

سپس مسیر URL جدیدی را برای ثبت نام اضافه می‌کنیم.

کد
```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path
  
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('posts.urls')),
    path('api-auth/', include('rest_framework.urls')),
    path('api/v1/dj-rest-auth/', include('dj_rest_auth.urls')),
    path('api/v1/dj-rest-auth/registration/', # new
    include('dj_rest_auth.registration.urls')),
]
```

و تمام است. می‌توانیم سرور محلی را اجرا کنیم.

کامند لاین
```shell
(blogapi) $ python manage.py runserver
```

اکنون نقطه جدیدی برای ثبت نام کاربر در<http://127.0.0.1:8000/api/v1/dj-rest-auth/registration/> وجود دارد.

|![API ثبت نام کاربر](images/7.jpg)|
|:--:|
|API ثبت نام کاربر|                                                                                       
                                                                                          
                                                                                          
### Tokens                                                                                        
      
                                                                                          
To make sure everything works, create a third user account via the browsable API endpoint. I’ve
called my user testuser2. Then click on the “POST” button.                                                                                        
                                                                                          
                                                                                          
![API Register New User](images/8.jpg)                                                                                        
                                                                                          
The next screen shows the HTTP response from the server. Our user registration POST was
successful, hence the status code HTTP 201 Created at the top. The return value key is the
auth token for this new user.                                                                                         
                                                                                          
                                                                                          
![API Auth Key](images/9.jpg)                                                                                         
                                                                                          
                                                                                          
If you look at the command line console, an email has been automatically generated by django-allauth.
This default text can be updated and an email SMTP server added with additional configuration
that is covered in the book [Django for Beginners](https://djangoforbeginners.com/).                                                                                         
                                                                                          
   
<div dir="ltr">
  
Command Line
```shell
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Subject: [example.com] Please Confirm Your E-mail Address
From: webmaster@localhost
To: testuser2@email.com
Date: Wed, 29 Jul 2020 20:54:26 -0000
Message-ID:
  <159605606600.8206.5520712009851546888@1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0\
.0.0.0.0.0.0.0.ip6.arpa>
    
Hello from example.com!
    
You're receiving this e-mail because user testuser2 has given yours as an e-mail address \
to connect their account.
    
To confirm this is correct, go to http://127.0.0.1:8000/api/v1/dj-rest-auth/\
registration/account-confirm-email/MQ:1k0t5m:6l0l09er1p_cbxgkJWDuSw2j00M/
    
Thank you from example.com!
example.com
```
  
</div>                                                                                         
                                                                                          
                                                                                          
Switch over to the Django admin in your web browser at http://127.0.0.1:8000/admin/. You
will need to use your superuser account for this. Then click on the link for Tokens at the top of
the page. You will be redirected to the Tokens page.
                                                                                        
                                                                                          
                                                                                          
![Admin Tokens](images/10.jpg)                                                                                        
                                                                                          
                                                                                          
A single token has been generated by Django REST Framework for the testuser2 user. As
additional users are created via the API, their tokens will appear here, too.
  
  
A logical question is, Why are there are no tokens for our superuser account or testuser? The
answer is that we created those accounts before token authentication was added. But no worries,
once we log in with either account via the API a token will automatically be added and available.
  
  
Moving on, let’s log in with our new testuser2 account. In your web browser, navigate to
http://127.0.0.1:8000/api/v1/dj-rest-auth/login/. Enter the information for our testuser2
account. Click on the “POST” button.                                                                                         
                                                                                          
                                                                                          
                                                                                          
![API Log In testuser2](images/11.jpg)                                                                                          
         
  
Two things have happened. In the upper righthand corner, our user account testuser2 is visible,
confirming that we are now logged in. Also the server has sent back an HTTP response with the
token.                                                                                       

  
![API Log In Token](images/12.jpg)
  
  
In our front-end framework, we would need to capture and store this token. Traditionally this
happens on the client, either in [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) or as a cookie, and then all future requests include
the token in the header as a way to authenticate the user. Note that there are additional security
concerns on this topic so you should take care to implement the best practices of your front-end
framework of choice.

  
### Conclusion  
  
  
User authentication is one of the hardest areas to grasp when first working with web APIs.
Without the benefit of a monolithic structure, we as developers have to deeply understand and
configure our HTTP request/response cycles appropriately. 
  
  
  
Django REST Framework comes with a lot of built-in support for this process, including
built-in TokenAuthentication. However developers must configure additional areas like user
registration and dedicated urls/views themselves. As a result, a popular, powerful, and secure
approach is to rely on the third-party packages dj-rest-auth and django-allauth to minimize
the amount of code we have to write from scratch.