<div style='text-align: justify;' dir="rtl">
	
# کتابخانه سایت و API

جنگو رست Django REST Framework در کنار Django web Framework برای ایجاد رابط‌های برنامه‌های کاربردی وب (web APIs) کار می‌کند. ما نمی‌توانیم تنها با استفاده از Django REST Framework برای ساخت یک رابط برنامه کاربردی وب استفاده کنیم بلکه همواره باید آن را بعد از نصب و پیکربندی Django به پروژه اضافه کنیم.
در این فصل، ما شباهت‌ها و تفاوت‌های بین جنگو مرسوم (Traditional Django یا Django web Framework) و Django REST Framework می‌پردازیم. مهمترین تفاوت این است که جنگو وب‌سایت‌هایی محتوی صفحات وب را ایجاد می‌کند، درحالی که Django REST Framework به ایجاد رابط برنامه‌های کاربردی می پردازد که مجموعه‌ای از نقاط پایانی (endpoint) را که شامل متدهای HTTP در دسترس که پاسخی در قالب JSON باز می‌گردانند، می‌باشند. برای نشان دادن این مفاهیم، ما یک وب‌سایت کتابخانه با جنگوی مرسوم می‌سازیم و سپس آن را با کمک Django REST Framework به یک web API تبدیل می‌کنیم.

مطمئن شوید که Python 3 و [Pipenv](https://pypi.org/project/pipenv/) بر روی کامپیوتر شما نصب شده باشد. اگر به راهنمایی نیاز دارید، دستورالعمل کامل در [اینجا](https://djangoforbeginners.com/initial-setup/) است.


## جنگو مرسوم

ابتدا ما نیاز داریم که محلی را برای ذخیره کد بر روی کامپیوترمان اختصاص دهیم. این محل میتواند هر جایی در سیستم شما باشد اما اگر از مک (macOS) استفاده می‌کنید برای راحتی آن را در پوشه Desktop قرار دهید. محل قرارگیری واقعا اهمیتی ندارد و فقط باید به راحتی قابل دسترسی باشد.

<div dir="ltr"> 

```powershell
$ cd ~/Desktop
$ mkdir code && cd code 
```

</div>

این پوشه code محل قرارگیری تمام کدهای داخل این کتاب خواهد بود. قدم بعدی ایجاد یک محل اختصاصی برای وب‌سایت کتابخانه‌مان، نصب جنگو با Pipenv و بعد ورود به محیط مجازی با استفاده از دستور shell است. شما همیشه باید یک محیط مجازی اختصاصی برای هر پروژه جدید پایتون استفاده کنید.

<div dir="ltr">

```powershell
$ mkdir library && cd library
$ pipenv install django~=3.1.0
$ pipenv shell
(library) $
```

</div>

Pipenv درون مسیر و محل فعلی یک Pipfile و Pipfile.lock می‌سازد. کلمه library (اسم محیط مجازی) داخل پرانتز نشان دهنده این است که محیط مجازی ما فعال است.

یک وب‌سایت جنگویی مرسوم از یک پروژه تنها و یک یا بیشتر از یک برنامه (app) که اعمال جداگانه‌ای را برعهده دارند، تشکیل شده‌اند. بیاید با دستور startproject یک پروژه جدید ایجاد کنیم. گذاشتن علامت نقطه را در آخر دستور برای نصب کدها در محل فعلی فراموش نکنید. اگر نقطه را نگذارید، جنگو یک پوشه اضافی بصورت پیش‌فرض می‌سازد.

<div dir="ltr">

```powershell
(library) $ django-admin startproject config .
```

</div>

جنگو بصورت خودکار یک پروژه جدید برای ما تولید می‌کند که می‌توانیم با دستور tree آن را مشاهده کنیم. (توجه: اگر tree برای شما در Mac کار نمی‌کند، آن را با [Homebrew](https://brew.sh/) نصب کنید: brew install tree).

<div dir="ltr">

```powershell
(library) $ tree
.
├── Pipfile
├── Pipfile.lock
├── config
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

1 directory, 8 files
```

</div>

فایل‌ها وظایف زیر را برعهده دارند:

* فایل __init____.py__ یک روش پایتونی است برای رفتار کردن با یک محل یا دایرکتوری مانند یک پکیج؛ این فایل خالی است.
* فایل settings.py تمامی تنظیمات و پیکربندی‌های پروژه ما را شامل می‌شود.
* فایل urls.py مسیرهای URL سطح بالا را کنترل می‌کند.
* فایل wsgi.py مخفف web server gateway interface است و به جنگو کمک می‌کند که صفحات وب نهایی را سرویس دهی کند.
* فایل manage.py دستورات مختلف جنگو را از قبیل اجرا کردن وب سرور محلی (local web server) یا ایجاد یک برنامه جدید را اجرا می‌کند.

دستور migrate را برای همگام سازی پایگاه داده با تنظیمات پیش‌فرض جنگو اجرا کنید و وب سرور محلی جنگو را بالا بیاورید.

<div dir="ltr">

```powershell
(library) $ python manage.py migrate
(library) $ python manage.py runserver
```

</div>

یک مرورگر وب باز کنید و به آدرس http://127.0.0.1:8000 بروید تا از نصب درست پروژه اطمینان حاصل کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/1.png?raw=true)


## اولین برنامه
قدم بعدی اضافه کردن برنامه‌هایی است که مسئول محدوده‌های مجزایی از عملکرد پروژه هستند. یک پروژه جنگو میتواند دارای چندین برنامه باشد.

سرور محلی را با کلیدهای ترکیبی `Ctrl+c` متوقف کنید و سپس یک برنامه بنام books ایجاد کنید.

<div dir="ltr">

```powershell
(library) $ python manage.py startapp books
```

</div>

حالا بیاید ببینیم که جنگو چه فایل‌هایی تولید کرده است.

<div dir="ltr">

```powershell
(library) $ tree
.
├── Pipfile
├── Pipfile.lock
├── books
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
|   ├── migrations
│   │ └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── library_project
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```

</div>

هر برنامه یک فایل __init____.py__ دارد که آن را به عنوان یک پکیج پایتون معرفی می‌کند. 6 فایل جدید ایجاد شده اند:

* فایل admin.py یک فایل پیکربندی برای برنامه داخلی مدیر جنگو.
* فایل apps.py یک فایل پیکربندی برای خود برنامه.
* دایرکتوری migrations فایل‌های migrations را برای تغییرات پایگاه داده ذخیره می‌کند.
* فایل models.py جایی که ما مدل‌های پایگاه داده را تعریف می‌کنیم.
* فایل tests.py برای تست‌های خاص برنامه است.
* فایل views.py جایی است که ما منطق درخواست و پاسخ (request/response) برنامه وب (web app) خود را پیاده می‌کنیم.

معمولاً توسعه‌دهندگان هم در هر برنامه یک فایل urls.py هم برای مسیردهی ایجاد می‌کنند.

بیایید فایل‌ها را بسازیم تا پروژه کتابخانه ما همه کتاب‌ها را در صفحه اصلی فهرست کند. 

فایل settings.py را با ویرایشگر متن باز کنید. اولین قدم اضافه کردن برنامه‌های جدید به تنظیمات INSTALLED_APPS است. ما همیشه برنامه‌های جدید را به آخر این لیست اضافه می‌کنیم چون جنگو آن‌ها را به ترتیب می خواند و ما می‌خواهیم که برنامه‌های هسته داخلی جنگو مانند admin و auth قبل از برنامه‌های ما بارگیری شوند.

<div dir="ltr">

```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Local
    'books', # new
]
```

</div>

سپس دستور migrate را برای همگام سازی پایگاه داده با تغییرات، اجرا می‌کنیم.

<div dir="ltr">

```powershell
(library) $ python manage.py migrate
```

</div>

هر صفحه وب در جنگو مرسوم به چندین فایل نیاز دارد: یک view، یک url و یک template. اما قبل از همه ما به یک مدل پایگاه داده نیاز داریم بنابراین بیاید از آن شروع کنیم.


## مدل‌ها
فایل books/models.py را در ویرایشگر متن (text editor) خود باز کنید و آن را بصورت زیر بروزرسانی کنید:

<div dir="ltr">

```python
# books/models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=250)
    subtitle = models.CharField(max_length=250)
    author = models.CharField(max_length=100)
    isbn = models.CharField(max_length=13)
    
    def __str__(self):
        return self.title
```

</div>

این یک مدل ساده جنگو است که در خط بالا آن ما models را از Django وارد می‌کنیم و سپس یک کلاس book ایجاد کرده‌ایم که آن را توسعه می‌دهد. درون این کلاس چهار فیلد وجود دارد: author، subtitle، title و isbn. ما همچنین یک متد __str____.py__ ساخته ایم تا عنوان کتاب در قسمت مدیریت نمایش داده شود.

توجه داشته باشید که [ISBN](https://www.isbn-international.org/content/what-isbn) یک شناسه 13 حرفی یکتا می‌باشد که به هر کتاب منتشر شده‌ای اختصاص داده می‌شود.

چون ما یک مدل پایگاه داده جدید ساخته ایم نیاز به ساخت یک فایل مهاجرت (migration) داریم تا تغییر ایجاد شده در پایگاه داده هم اعمال شود. مشخص کردن نام برنامه اختیاری است اما اینجا توصیه می‌شود. می‌توانیم تنها تایپ کنیم `python manage.py makemigrations` اما اگر چندین برنامه که تغییرات پایگاه داده‌ای داشته‌اند، وجود داشته باشد، همه آن‌ها به فایل migrations اضافه می‌شوند که خطایابی و رفع اشکال را در آینده تبدیل به یک چالش می‌کند. فایل‌های migrations خود را تا جای ممکن مجزا نگه دارید.

سپس دستور migrate را برای بروزرسانی پایگاه داده اجرا کنید.

<div dir="ltr">

```powershell
(library) $ python manage.py makemigrations books
Migrations for 'books':
  books/migrations/0001_initial.py
    - Create model Book
(library) $ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, books, contenttypes, sessions
Running migrations:
  Applying books.0001_initial... OK
```

</div>

اگر هرکدام از مطالبی که تا اینجا بررسی کردیم برای شما جدید به نظر می‌آید، به شما پیشنهاد می‌کنم مطالعه این کتاب را همین جا خاتمه دهید و برای تشریح جزئی‌تر جنگو مرسوم، کتاب [Django for Beginners](https://djangoforbeginners.com/) را مرور کنید.


## مدیر
ما می‌توانیم وارد کردن داده به مدل جدیدمان را با استفاده از برنامه داخلی جنگو انجام دهیم. اما اول باید دو کار را انجام دهیم: ایجاد یک حساب کاربری superuser و بروزرسانی فایل admin.py تا برنامه books نمایش داده شود.

با اکانت superuser شروع می‌کنیم. در ترمینال دستور زیر را اجرا کنید.

<div dir="ltr">

```powershell
(library) $ python manage.py createsuperuser
```

</div>

مطابق درخواست‌ها یک email، username و password وارد کنید. توجه داشته باشید که به دلایل امنیتی، متن در هنگام وارد کردن password بر روی صفحه نمایش داده نمی‌شود.

حالا فایل admin.py را بروزرسانی کنید.

<div dir="ltr">

```python
# books/admin.py
from django.contrib import admin
from .models import Book

admin.site.register(Book)
```

</div>

این تمام کاری بود که باید انجام می‌دادیم! سرور محلی را دوباره بالا بیاورید.

<div dir="ltr">

```powershell
(library) $ python manage.py runserver
```

</div>

به آدرس http://127.0.0.1:8000/admin بروید و وارد شوید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/2.png?raw=true)



شما به صفحه خانه admin منتقل خواهید شد.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/3.png?raw=true)



روی لینک Books کلیک کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/4.png?raw=true)



سپس روی دکمه + Add Book در گوشه بالا سمت راست کلیک کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/5.png?raw=true)


من جزئیات را برای کتاب Django for Beginners خودم وارد کرده‌ام. شما می‌توانید هر متنی که بخواهید اینجا وارد کنید. این اطلاعات صرفاً برای اهداف نمایشی است. بعد از کلیک کردن روی دکمه Save به صفحه Books که در آن تمام اطلاعات ثبت شده فعلی لیست شده اند، منتقل می‌شویم.

![books_list_page](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/6.png?raw=true)


پروژه جنگو مرسوم ما الآن داده دارد اما ما یک راه برای نمایش آن به عنوان یک صفحه وب نیاز داریم. این یعنی ایجاد فایل‌های URLs، views و templates. بیاید اینکار را الآن انجام دهیم.


## نماها (views)
فایل views.py نحوه نمایش محتوای مدل‌های پایگاه داده را کنترل می‌کند. از آنجایی که ما می‌خواهیم تمام کتاب‌ها را لیست کنیم می‌توانیم از کلاس عمومی داخلی [ListView](https://docs.djangoproject.com/en/3.1/ref/class-based-views/generic-display/#django.views.generic.list.ListView) استفاده کنیم.

فایل books/views.py را بروزرسانی کنید.

<div dir="ltr">

```python
# books/views.py
from django.views.generic import ListView
from .models import Book


class BookListView(ListView):
    model = Book
    template_name = 'book_list.html'
```

</div>

در خطوط بالا ما ListView و مدل Book خودمان را وارد کرده‌ایم. سپس یک کلاس BookListView که مشخص می‌کند از چه مدل و قالبی (template) باید استفاده شود، ایجاد می‌کنیم. (توجه داشته باشید که قالب را هنوز نساخته‌ایم.)

دو گام دیگر برای اینکه یک صفحه پیج آماده داشته باشیم، باقیمانده است: درست کردن قالب و پیکربندی URL ها. بیاید با URL ها شروع کنیم.



## مکان‌یاب‌های منبع یکسان (URLs)
نیاز است که هم فایل urls.py اصلی که مربوط به پروژه و هم فایلی که داخل برنامه books داریم را تنظیم کنیم. وقتی یک کاربر از سایت ما بازدید می‌کند ابتدا با فایل library_project/urls.py تعامل خواهد داشت بنابراین ابتدا پیکربندی‌های مربوط به این فایل را انجام می‌دهیم.

<div dir="ltr">

```python
# library_project/urls.py
from django.contrib import admin
from django.urls import path, include # new

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('books.urls')), # new
]
```

</div>

دو خط بالا برنامه داخلی admin، متد path برای مسیرهای ما و متد include که با برنامه books ما استفاده می‌شود را وارد می‌کنند. اگر کاربر به /admin/ برود به برنامه admin منتقل می‌شود. ما برای مسیر برنامه books از رشته خالی '' استفاده کردیم که این معنی را می‌دهد که یک کاربر در صفحه اصلی به طور مستقیم به برنامه books منتقل می‌شود.

حالا ما می‌توانیم فایل books/urls.p را تنظیم کنیم. اما جنگو به دلایلی به طور پیش‌فرض شامل فایل urls.py در برنامه‌ها نمی‌باشد بنابراین خود ما باید آن را ایجاد کنیم.

<div dir="ltr">

```powershell
(library) $ touch books/urls.py
```

</div>

حالا با استفاده از یک ویرایشگر متن فایل جدید را بروزرسانی کنید.

<div dir="ltr">

```python
# books/urls.py
from django.urls import path
from .views import BookListView

urlpatterns = [
    path('', BookListView.as_view(), name='home'),
]
```

</div>

ما فایل views.py خودمان را وارد می‌کنیم، BookListView را در آدرس رشته خالی '' تنظیم می‌کنیم و یک [named URL](https://docs.djangoproject.com/en/2.1/topics/http/urls/#naming-url-patterns) با مقدار home به عنوان یک best practice اضافه می‌کنیم.

روشی که جنگو کار می‌کند، حالا وقتی یک کاربر به صفحه خانه وب‌سایت ما می‌رود، آن‌ها نخست به فایل  library_project/urls.py می‌رسند سپس به books/urls.py منتقل می‌شوند که مشخص می‌کند باید از BookListView استفاده شود. در این فایل view از مدل Book به همراه ListView برای فهرست کردن همه کتاب‌ها استفاده شده است.

قدم نهایی ایجاد فایل قالب است که چیدمان در صفحه حقیقی وب را کنترل می‌کند. ما قبلا نام آن (book_list.html) را در فایل view مشخص کرده‌ایم. دو گزینه برای مکان آن وجود دارد: به طور پیش‌فرض بارکننده قالب جنگو داخل برنامه books ما در مسیر books/templates/books/book_list.html به دنبال قالب‌ها می‌گردد. همچنین ما می‌توانیم یک دایرکتوری templates جدا هم سطح پروژه ایجاد کنیم و فایل settings.py را برای رفتن به آن مکان برای یافتن قالب ها بروزرسانی کنیم.

اینکه شما در پروژه‌های خود از کدام روش استفاده کنید یک ترجیح شخصی است. ما در اینجا از ساختار پیشفرض استفاده می‌کنیم. اگر درباره روش دوم کنجکاو هستید، کتاب [Django for Beinners](https://djangoforbeginners.com/) را بررسی کنید.

با ساخت یک پوشه templates در برنامه books شروع می‌کنیم، سپس داخل آن یک پوشه books و در آخر یک فایل book_list.html در آن می‌سازیم. 

<div dir="ltr">

```powershell
(library) $ mkdir books/templates
(library) $ mkdir books/templates/books
(library) $ touch books/templates/books/book_list.html
```

</div>

حالا فایل template را بروزرسانی کنید.

<div dir="ltr">

```html
<!-- books/templates/books/book_list.html -->
<h1>All books</h1>
{% for book in object_list %}
    <ul>
        <li>Title: {{ book.title }}</li>
	<li>Subtitle: {{ book.subtitle }}</li>
	<li>Author: {{ book.author }}</li>
	<li>ISBN: {{ book.isbn }}</li>
    </ul>
{% endfor %}
```

</div>

جنگو با [زبان قالبی](https://docs.djangoproject.com/en/2.1/ref/templates/language/) عرضه می‌شود که اجازه اعمال عملیات منطقی ساده را به ما می‌دهد. در اینجا ما از تگ [for](https://docs.djangoproject.com/en/2.1/ref/templates/builtins/#std:templatetag-for) برای حلقه زدن بر روی تمام کتاب‌های در دسترس استفاده می‌کنیم. تگ‌های قالب باید همیشه باید شامل براکت‌های باز و بسته باشند. بنابراین فرمت همیشه بصورت `{% ... for %}` است و سپس ما باید حلقه را با `{% endfor %}` خاتمه دهیم.
	
چیزی که ما درحال حلقه زدن بر روی آن هستیم، شئ‌ای است که شامل تمامی کتاب‌های در دسترس در مدل ما می‌باشد. نام این شئ object_list است. بنابراین برای حلقه زدن بر روی هر کتاب ما می‌نویسیم `{% for book in object_list %}` و سپس هر فیلد از مدلمان را نمایش می‌دهیم.


## صفحه وب
حالا می‌توانیم سرور محلی جنگو را بالا بیاوریم و صفحه وب خود را مشاهده کنیم.

<div dir="ltr">

```powershell
(library) $ python manage.py runserver
```

</div>

به آدرس صفحه خانه که http://127.0.0.1:8000/ میباشد بروید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/7.png?raw=true)



اگر ما کتاب‌های دیگری را در admin اضافه کنیم، آن‌ها نیز در اینجا به نمایش درخواهند آمد.

این یک مرور سریع از وب‌سایت جنگویی مرسوم بود. حالا بیایید به آن رابط برنامه کاربردی هم اضافه کنیم!



## رست فریمورک جنگو (Django REST Framework)
رست فریمورک جنگو هم مانند بقیه برنامه‌های شخص ثالث نصب می‌شود. با فشردن کلیدهای `Ctrl+c` سرور محلی را اگر در حال اجرا است، متوقف کنید. سپس در خط فرمان دستور زیر را تایپ کنید.

<div dir="ltr">

```powershell
(library) $ pipenv install djangorestframework==3.10.3
```

</div>

عبارت rest_framework را به تنظیمات INSTALLED_APPS در فایل settings.py اضافه کنید. من عادت دارم که یک فاصله بین برنامه‌های شخص ثالث و محلی قرار دهم به این دلیل که تعداد برنامه‌ها در بیشتر پروژه‌ها به سرعت افزایش پیدا می‌کنند.

<div dir="ltr">

```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 3rd party
    'rest_framework', # new

    # Local
    'books',
]
```

</div>

در نهایت رابط برنامه کاربردی ما یک نقطه پایانی که تمام کتاب‌ها را در فرمت JSON لیست می‌کند، نمایش می‌دهد. بنابراین ما یک مسیر URL جدید، یک view جدید و یک فایل serializer جدید (درباره این فایل بزودی بیشتر می‌خوانید) نیاز خواهیم داشت.
راه‌های زیادی وجود دارد که می‌توانیم این فایل‌ها را سازمان دهیم. اگرچه ترجیح من این است که یک برنامه مختص به رابط‌های برنامه کاربردی ایجاد کنیم. در این روش حتی اگر در آینده هم برنامه‌های بیشتری اضافه کنیم، هر برنامه می‌تواند شامل modelها، viewها، templateها و urlهای مورد نیاز برای صفحات وب باشد اما تمام فایل‌های مختص به رابط‌های برنامه کاربردی مربوط به کل پروژه درون یک برنامه اختصاص یافته به رابط‌های برنامه کاربردی قرار می‌گیرند.

بیایید ابتدا یک برنامه جدید بنام api بسازیم.

<div dir="ltr">

```powershell
(library) $ python manage.py startapp api
```

</div>

سپس آن را به INSTALLED_APPS اضافه کنید.

<div dir="ltr">

```python
# config/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 3rd party
    'rest_framework',

    # Local
    'books.apps.BooksConfig',
    'api.apps.ApiConfig', # new
]

```

</div>

برنامه api مدل پایگاه داده‌ای مختص به خود ندارد بنابراین نیازی به ایجاد یک فایل مهاجرت و بروزرسانی پایگاه داده طبق روال معمول نیست.



## مسیرها (URLs)
بیایید با تنظیم URL ها شروع کنیم. اضافه کردن یک نقطه پایانی درست مشابه پیکربندی مسیرهای یک برنامه جنگوی مرسوم می‌باشد. ابتدا نیاز است که برنامه api و مسیرهای آن را در فایل urls.py اصلی پروژه و در مسیر /api اضافه و تنظیم کنیم.

<div dir="ltr">

```python
# library_project/urls.py
from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('books.urls')),
    path('api/', include('api.urls')), # new
]
```

</div>

سپس یک فایل urls.py درون برنامه api می‌سازیم.

<div dir="ltr">

```powershell
(library) $ touch api/urls.py
```

</div>

و آن را به شکل زیر بروزرسانی می‌کنیم:

<div dir="ltr">

```python
# api/urls.py
from django.urls import path
from .views import BookAPIView

urlpatterns = [
    path('', BookAPIView.as_view()),
]
```

</div>

همه چیز آماده است.


## نماها
مرحله بعد فایل views.py ما است که متکی به نماهای کلاسی عمومی داخلی (built-in generics class views) رست فریمورک جنگو می‌باشد. این نماها در ظاهر از نماهای کلاسی عمومی جنگوی مرسوم تقلید می‌کند اما این دو گروه از نماها یکی نیستند.
برای جلوگیری از سردرگمی، بعضی از توسعه‌دهندگان یک فایل API views را apiviews.py یا api.py می نامند. شخصا، وقتی که در یک برنامه مختص به رابط‌های برنامه کاربردی کار می‌کنم دچار سردرگمی نمی‌شوم اگر که نماهای مربوط به Django REST Framework را views.py بنامم اما نظر ها در اینباره متفاوت هستند.

محتوای فایل views.py را بصورت زیر بروزرسانی کنید:

<div dir="ltr">

```python
# api/views.py
from rest_framework import generics
from books.models import Book
from .serializers import BookSerializer


class BookAPIView(generics.ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

</div>

در خطوط بالا نماهای کلاسی عمومی (generics class of views) رست فریمورک جنگو، مدل‌ها را از برنامه books و فایل serializers را از برنامه api (ما serializers را بعدا ایجاد می‌کنیم) وارد کرده‌ایم.

سپس یک BookAPIView که از ListAPIView برای ایجاد یک نقطه پایانی فقط خواندنی برای تمام نمونه (instance) های کتاب استفاده می‌کند، می‌سازیم. چندین نمای عمومی در دسترس وجود دارند که جلوتر در فصل های بعد آن‌ها را بررسی خواهیم کرد.
دو قدمی که در نمای ما نیاز است، مشخص کردن queryset که تمام کتاب‌های در دسترس است و سپس serializer_class که BookSrializer می‌باشد، است.


## سریالایزرها (Serializers)
یک سریالایزر داده را به شکلی که استفاده از آن‌ها در اینترنت راحت باشد، معمولاً JSON، و در یک نقطه پایان رابط برنامه تعاملی به نمایش درمی‌آید، در‌می‌آورد. در فصل‌های آینده سریالایزرها و JSON را جزئی‌تر پوشش خواهیم داد. برای الآن من می‌خواهم نشان دهم که ساخت یک سریالایزر با استفاده از رست فریمورک جنگو برای تبدیل مدل‌های جنگو به JSON چقدر آسان است.

یک فایل serializer.py درون برنامه api درست کنید.

<div dir="ltr">

```powershell
(library) $ touch api/serializers.py
```

</div>

سپس آن را بصورت زیر در ویرایشگر متن بروزرسانی کنید.

<div dir="ltr">

```python
# api/serializers.py
from rest_framework import serializers
from books.models import Book


class BookSerializer(serializers.ModelSerializer):
    class Meta:
    	model = Book
    	fields = ('title', 'subtitle', 'author', 'isbn')
```

</div>

در خطوط بالا ما کلاس serializer رست فریمورک جنگو و مدل Book را از فایل مدل برنامه books وارد می‌کنیم. ما ModelSerializer رست فریمورک جنگو را در کلاس BookSerializer که مدل پایگاه داده Book و فیلدهای پایگاه داده‌ای که می‌خواهیم نمایش بدهیم (author، subtitle، title و isbn) را مشخص می‌کند، توسعه می‌دهیم.



## کِرل (cURL)
ما می‌خواهیم ببینیم نقطه پایانی رابط برنامه کاربردی ما به چه شکلی است. می‌دانیم که باید در مسیر http://127.0.0.1:8000/api یک JSON برگرداند. بیایید مطمئن شویم که سرور محلی جنگو در حال اجرا است:

<div dir="ltr">

```powershell
(library) $ python manage.py runserver
```

</div>

حالا یک خط فرمان دوم جدید باز کنید. از این خط فرمان برای رفتن به آدرس رابط برنامه کاربردی که در خط فرمان موجود در حال اجرا است، استفاده می‌کنیم.

ما می‌توانیم از برنامه محبوب [cURL](https://en.wikipedia.org/wiki/CURL) برای اجرا درخواست‌های HTTP در خط فرمان استفاده کنیم. تمام چیزی که برای یک درخواست GET ساده نیاز داریم این است که curl را بنویسیم و URLای که می‌خواهیم صدا بزنیم را برای آن مشخص کنیم.

<div dir="ltr">

```powershell
$ curl http://127.0.0.1:8000/api/
[
    {
        "title":"Django for Beginners",
        "subtitle":"Build websites with Python and Django",
        "author":"William S. Vincent",
        "isbn":"978-198317266"
    }
]
```

</div>

تمام داده‌ها به شکل JSON در آنجا هستند اما قالب‌بندی ضعیفی دارند و درک آن‌ها سخت است. خوشبختانه Django REST Framework یک شگفتی دیگر هم برای ما دارد: یک حالت بصری قدرتمند برای نقاط پایانی رابط‌های برنامه کاربردی ما.



## رابط برنامه کاربردی تحت مرورگر (Browsable API)
در حالی که سرور محلی همچنان در اولین خط فرمان در حال اجرا است، در مرورگر به آدرس http://127.0.0.1:8000/api بروید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/8.png?raw=true)

 

وای به آن نگاه کنید! Django REST Framework این تجسم را برای ما به طور پیشفرض فراهم کرده است. و امکانات زیادی در این صفحه تعبیه وجود دارد که در طول کتاب آن‌ها را بررسی می‌کنیم. حالا می‌خواهیم این صفحه را با نقطه پایانی JSON خام مقایسه کنیم. بر روی دکمه GET کلیک کنید و از منو گزینه json را انتخاب کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/9.png?raw=true)



این شکل JSON خام نقطه پایانی رابط برنامه کاربردی ما است. فکر می‌کنم می‌توانیم توافق کنیم که نسخه Django REST Framework، جذاب‌تر است.


## نتیجه گیری
ما در این فصل مباحث زیادی را پوشش دادیم پس اگر موضوعات الآن کمی گیج کننده به نظر می‌رسند، نگران نباشید. ابتدا یک وب‌سایت کتابخانه جنگویی مرسوم ساختیم. سپس Django REST Framework را اضافه کردیم و یک با تعداد کمی کد یک نقطه پایانی رابط برنامه کاربردی ساختیم.
در دو فصل بعدی ما بک‌اند رابط برنامه کاربری خود را می‌سازیم و آن را به یک فرانت‌اند ری‌اکتی متصل می‌کنیم تا یک مثال کامل را نشان دهیم که به درک اینکه در عمل تمام این اصول چگونه به یکدیگر مرتبط می‌شوند، کمک می‌کند.  
</div> 
