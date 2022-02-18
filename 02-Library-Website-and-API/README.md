<div style='text-align: justify;' dir="rtl">
Django REST Framework در کنار Django web Framework برای ایجاد رابط های برنامه های کاربردی وب(web APIs) ها کار می کند. ما نمی توانیم تنها با استفاده از Django REST Framework برای ساخت یک رابط برنامه کاربردی وب استفاده کنیم بلکه همواره باید آن را بعد از نصب و پیکربندی Django به پروژه اضافه کنیم.
در این فصل، ما شباهت ها و تفاوت های بین جنگو مرسوم(Traditional Django یا Django web Framework) و Django REST Framework می پردازیم. مهمترین تفاوت این است که جنگو وبسایت هایی محتوی صفحات وب را ایجاد می کند درحالیکه Django REST Framework به ایجاد رابط برنامه های کاربردی می پردازد که مجموعه ای از نقاط پایانی(endpoint) را که شامل متد های HTTP در دسترس که پاسخی در قالب JSON باز میگردانند، می باشند. برای نشان دادن این مفاهیم، ما یک وبسایت کتابخانه با جنگوی مرسوم می سازیم و سپس آن را با کمک Django REST Framework به یک web API تبدیل می کنیم.

مطمئن شوید که Python 3 و [Pipenv](https://pypi.org/project/pipenv/) بر روی کامپیوتر شما نصب شده باشد. اگر به راهنمایی نیاز دارید، دستورالعمل کامل در [اینجا](https://djangoforbeginners.com/initial-setup/) می باشد.


## جنگو مرسوم

ابتدا ما نیاز داریم که محلی را برای ذخیره کد بر روی کامپیوترمان اختصاص دهیم. این محل میتواند هر جایی در سیستم شما باشد اما اگر از مک(macOS) استفاده می کنید برای راحتی آن را در پوشه Desktop قرار دهید. محل قرارگیری واقعا اهمیتی ندارد و فقط باید به راحتی قابل دسترسی باشد.
</div>

```powershell
$ cd ~/Desktop
$ mkdir code && cd code 
```
<div style='text-align: justify;' dir="rtl">
	
این پوشه code محل قرارگیری تمام کدهای داخل این کتاب خواهد بود. قدم بعدی ایجاد یک محل اختصاصی برای وبسایت کتابخانه مان، نصب جنگو با Pipenv، و بعد ورود به محیط مجازی با استفاده از دستور shell است. شما همیشه باید یک محیط مجازی اختصاصی برای هر پروژه جدید پایتون استفاده کنید.
</div>

```powershell
$ mkdir library && cd library
$ pipenv install django==2.2.6
$ pipenv shell
(library) $
```
<div style='text-align: justify;' dir="rtl">
	Pipenv درون مسیر و محل فعلی یک Pipfile و Pipfile.lock میسازد. کلمه library(اسم محیط مجازی) داخل پرانتز نشان دهنده این است که محیط مجازی ما فعال است.

یک وبسایت جنگویی مرسوم از یک پروژه تنها و یک یا بیشتر از یک برنامه(app) که اعمال جداگانه ای را برعهده دارند، تشکیل شده اند. بیاید با دستور startproject یک پروژه جدید ایجاد کنیم. گذاشتن علامت نقظه را در آخر دستور برای نصب کدها در محل فعلی فراموش نکنید. اگر نقطه را نگذارید، جنگو یک پوشه اضافی بصورت پیش فرض میسازد.

</div>

```powershell
(library) $ django-admin startproject library_project .
```
<div style='text-align: justify;' dir="rtl">
	
جنگو بصورت خودکار یک پروژه جدید برای ما تولید میکند که میتوانیم با دستور tree آن را مشاهده کنیم.(توجه: اگر tree برای شما در Mac کار نمیکند، آن را با [Homebrew](https://brew.sh/) نصب کنید: brew install tree).
</div>

```powershell
(library) $ tree
.
├── Pipfile
├── Pipfile.lock
├── library_project
│ ├── __init__.py
│ ├── settings.py
│ ├── urls.py
│ └── wsgi.py
└── manage.py
```

<div style='text-align: justify;' dir="rtl">
فایل ها وظایف زیر را برعهده دارند:

* فایل __init____.py__ یک روش پایتونی است برای رفتار کردن با یک محل یا دایرکتوری مانند یک پکیج؛ این فایل خالی است.
* فایل settings.py تمامی تنظیمات و پیکربندی های پروژه ما را شامل میشود.
* فایل urls.py مسیرهای URL سطح بالا را کنترل میکند.
* فایل wsgi.py مخفف web server gateway interface است و به جنگو کمک میکند که صفحات وب نهایی را سرویس دهی کند.
* فایل manage.py دستورات مختلف جنگو را از قبیل اجرا کردن وب سرور محلی(local web server ) یا ایجاد یک برنامه جدید را اجرا میکند.

دستور migrate را برای همگام سازی پایگاه داده با تنظیمات پیش فرض جنگو اجرا کنید و وب سرور محلی جنگو را بالا بیاورید.
</div>

```powershell
(library) $ python manage.py migrate
(library) $ python manage.py runserver
```

یک مرورگر وب باز کنید و به آدرس http://127.0.0.1:8000 بروید تا از نصب درست پروژه اطمینان حاصل کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/1.png?raw=true)





## اولین برنامه
قدم بعدی اضافه کردن برنامه هایی است که مسئول محدوده های مجزایی از عملکرد پروژه هستند. یک پروژه جنگو میتواند دارای چندین برنامه باشد.

سرور محلی را با کلیدهای ترکیبی Control+c متوقف کنید و سپس یک برنامه بنام books ایجاد کنید.

```powershell
(library) $ python manage.py startapp books
```



حالا بیاید ببینیم که جنگو چه فایلهایی تولید کرده است.

```powershell
(library) $ tree
.
├── Pipfile
├── Pipfile.lock
├── books
│ ├── __init__.py
│ ├── admin.py
│ ├── apps.py
| ├── migrations
│ │ └── __init__.py
│ ├── models.py
│ ├── tests.py
│ └── views.py
├── library_project
│ ├── __init__.py
│ ├── settings.py
│ ├── urls.py
│ └── wsgi.py
└── manage.py
```

هر برنامه یک فایل __init____.py__ دارد که آن را به عنوان یک پکیج پایتون معرفی می کند. 6 فایل جدید ایجاد شده اند:

* فایل admin.py یک فایل پیکربندی برای برنامه داخلی مدیر جنگو.
* فایل apps.py یک فایل پیکربندی برای خود برنامه.
* دایرکتوری migrations فایلهای migrations را برای تغییرات پایگاه داده ذخیره می کند.
* فایل models.py جائیکه ما مدل های پایگاه داده را تعریف می کنیم.
* فایل tests.py برای تست های خاص برنامه می باشد.
* فایل views.py جایی است که ما منطق درخواست و پاسخ(request/response) برنامه وب(web app) خود را پیاده می کنیم.

معمولا توسعه دهندگان هم در هر برنامه یک فایل urls.py هم برای مسیردهی ایجاد می کنند.



بیایید فایل ها را بسازیم تا پروژه کتابخانه ما همه کتاب ها را در صفحه اصلی فهرست کند. 

فایل settings.py را با ویرایشگر متن باز کنید. اولین قدم اضافه کردن برنامه های جدید به تنظیمات INSTALLED_APPS می باشد. ما همیشه برنامه های جدید را به آخر این لیست اضافه می کنیم چون جنگو آن ها را به ترتیب می خواند و ما می خواهیم که برنامه های هسته داخلی جنگو مانند admin و auth قبل از برنامه های ما بارگیری شوند.

```python
# library_project/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Local
    'books.apps.BooksConfig', # new
]
```

سپس دستور migrate را برای همگام سازی پایگاه داده با تغییرات، اجرا می کنیم.

```powershell
(library) $ python manage.py migrate
```

هر صفحه وب در جنگو مرسوم به چندین فایل نیاز دارد: یک view، یک url و یک template. اما قبل از همه ما به یک مدل پایگاه داده نیاز داریم بنابراین بیاید از آن شروع کنیم.





## مدل ها
فایل books/models.py را در ویرایشگر متن(text editor) خود باز کنید و آن را بصورت زیر بروزرسانی کنید:

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

این یک مدل ساده جنگو است که در خط بالا آن ما models را از Django وارد می کنیم و سپس یک کلاس book ایجاد کرده ایم که آن را توسعه می دهد. درون این کلاس چهار فیلد وجود دارد: author، subtitle، title و isbn. ما همچنین یک متد __str____.py__ ساخته ایم تا عنوان کتاب در قسمت مدیریت نمایش داده شود.

توجه داشته باشید که ISBN یک شناسه 13 حرفی یکتا میباشد که به هر کتاب منتشر شده ای اختصاص داده می شود.

چون ما یک مدل پایگاه داده جدید ساخته ایم نیاز به ساخت یک فایل مهاجرت(migration) داریم تا تغییر ایجاد شده در پایگاه داده هم اعمال شود. مشخص کردن نام برنامه اختیاری است اما اینجا توصیه می شود. می توانیم تنها تایپ کنیم python manage.py makemigrations اما اگر چندین برنامه که تغییرات پایگاه داده ای داشته اند، وجود داشته باشد، همه آنها به فایل migrations اضافه میشوند که خطایابی و رفع اشکال را در آینده تبدیل به یک چالش میکند. فایل های migrations خود را تا جای ممکن مجزا نگه دارید.

سپس دستور migrate را برای بروزرسانی پایگاه داده اجرا کنید.

```powershell
(library) $ python manage.py makemigrations books
(library) $ python manage.py migrate
```



اگر هرکدام از مطالبی که تا اینجا بررسی کردیم برای شما جدید بنظر می آید، به شما پیشنهاد می کنم مطالعه این کتاب را همینجا خاتمه دهید و برای تشریح جزئی تر جنگو مرسوم، کتاب [Django for Beginners](https://djangoforbeginners.com/) را مرور کنید.





## مدیر
ما می توانیم وارد کردن داده به مدل جدیدمان را با استفاده از برنامه داخلی جنگو انجام دهیم. اما اول باید دو کار را انجام دهیم: ایجاد یک حساب کاربری superuser و بروزرسانی فایل admin.py تا برنامه books نمایش داده شود.

با اکانت superuser شروع می کنیم. در ترمینال دستور زیر را اجرا کنید.

```powershell
(library) $ python manage.py createsuperuser
```

مطابق درخواست ها یک email، username و password وارد کنید. توجه داشته باشید که به دلایل امنیتی، متن در هنگام وارد کردن password بر روی صفحه نمایش داده نمی شود.

حالا فایل admin.py را بروزرسانی کنید.

```python
# books/admin.py
from django.contrib import admin
from .models import Book

admin.site.register(Book)
```

این تمام کاری بود که باید انجام می دادیم! سرور محلی را دوباره بالا بیاورید.

```powershell
(library) $ python manage.py runserver
```

به آدرس http://127.0.0.1:8000/admin بروید و وارد شوید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/2.png?raw=true)



شما به صفحه خانه admin منتقل خواهید شد.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/3.png?raw=true)



روی لینک Books کلیک کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/4.png?raw=true)



سپس روی دکمه + Add Book در گوشه بالا سمت راست کلیک کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/5.png?raw=true)


من جزئیات را برای کتاب Django for Beginners خودم وارد کرده ام. شما می توانید هر متنی که بخواهید اینجا وارد کنید. این اطلاعات صرفا برای اهداف نمایشی است. بعد از کلیک کردن روی دکمه Save به صفحه Books که در آن تمام اطلاعات ثبت شده فعلی لیست شده اند، منتقل می شویم.

![books_list_page](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/6.png?raw=true)


پروژه جنگو مرسوم ما الآن داده دارد اما ما یک راه برای نمایش آن به عنوان یک صفحه وب نیاز داریم. این یعنی ایجاد فایل های URLs، views و templates. بیاید اینکار را الآن انجام دهیم.





## نما ها(views)
فایل views.py نحوه نمایش محتوای مدل های پایگاه داده را کنترل می کند. از آنجاییکه ما میخواهیم تمام کتابها را لیست کنیم می توانیم از کلاس عمومی داخلی ListView استفاده کنیم.

فایل books/views.py را بروزرسانی کنید.

```python
# books/views.py
from django.views.generic import ListView

from .models import Book


class BookListView(ListView):
    model = Book
    template_name = 'book_list.html'
```

در خطوط بالا ما ListView و مدل Book خودمان را وارد کرده ایم. سپس یک کلاس BookListView که مشخص میکند از چه مدل و قالبی(template) باید استفاده شود، ایجاد می کنیم.(توجه داشته باشید که قالب را هنوز نساخته ایم.)

دو گام دیگر برای اینکه یک صفحه پیج آماده داشته باشیم، باقیمانده است: درست کردن قالب و پیکربندی URL ها. بیاید با URL ها شروع کنیم.





## مکان یاب های منبع یکسان(URLs)
نیاز است که هم فایل urls.py اصلی که مربوط به پروژه و هم فایلی که داخل برنامه books داریم را تنظیم کنیم. وقتی یک کاربر از سایت ما بازدید می کند ابتدا با فایل library_project/urls.py تعامل خواهد داشت بنابراین ابتدا پیکربندیهای مربوط به این فایل را انجام می دهیم.

```python
# library_project/urls.py
from django.contrib import admin
from django.urls import path, include # new

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('books.urls')), # new
]
```

دو خط بالا برنامه داخلی admin، متد path برای مسیرهای ما و متد include که با برنامه books ما استفاده می شود را وارد می کنند. اگر کاربر به /admin/ برود به برنامه admin منتقل می شود. ما برای مسیر برنامه books از رشته خالی '' استفاده کردیم که این معنی را می دهد که یک کاربر در صفحه اصلی بطور مستقیم به برنامه books منتقل می شود.

حالا ما می توانیم فایل books/urls.p را تنظیم کنیم. اما جنگو به دلایلی بطور پیش فرض شامل فایل urls.py در برنامه ها نمی باشد بنابراین خود ما باید آن را ایجاد کنیم.

```powershell
(library) $ touch books/urls.py
```

حالا با استفاده از یک ویرایشگر متن فایل جدید را بروزرسانی کنید.

```python
# books/urls.py
from django.urls import path

from .views import BookListView

urlpatterns = [
    path('', BookListView.as_view(), name='home'),
]
```

<div style='text-align: justify;' dir="rtl">
	
ما فایل views.py خودمان را وارد می کنیم، BookListView را در آدرس رشته خالی '' تنظیم می کنیم و یک [named URL](https://docs.djangoproject.com/en/2.1/topics/http/urls/#naming-url-patterns) با مقدار home به عنوان یک best practice اضافه می کنیم.

روشی که جنگو کار می کند، حالا وقتی یک کاربر به صفحه خانه وب سایت ما می رود، آنها نخست به فایل  library_project/urls.py می رسند سپس به books/urls.py منتفل می شوند که مشخص می کند باید از اBookListView استفاده شود. در این فایل view از مدل Book به همراه ListView برای فهرست کردن همه کتاب ها استفاده شده است.

قدم نهایی ایجاد فایل قالب است که چیدمان در صفحه حقیقی وب را کنترل می کند. ما قبلا نام آن(book_list.html) را در فایل view مشخص کرده ایم. دو گزینه برای مکان آن وجود دارد: بطور پیش فرض بارکننده قالب جنگو داخل برنامه books ما در مسیر books/templates/books/book_list.html بدنبال قالب ها می گردد. همچنین ما می توانیم یک دایرکتوری templates جدا هم سطح پروژه ایجاد کنیم و فایل settings.py را برای رفتن به آن مکان برای یافتن قالب ها بروزرسانی کنیم.

اینکه شما در پروژ های خود از کدام روش استفاده کنید یک ترجیح شخصی است. ما در اینجا از ساختار پیشفرض استفاده می کنیم. اگر درباره روش دوم کنجکاو هستید، کتاب [Django for Beinners](https://djangoforbeginners.com/) را بررسی کنید.

با ساخت یک پوشه templates در برنامه books شروع می کنیم، سپس داخل آن یک پوشه books و در آخر یک فایل book_list.html در آن می سازیم.
</div>

```powershell
(library) $ mkdir books/templates
(library) $ mkdir books/templates/books
(library) $ touch books/templates/books/book_list.html
```

حالا فایل template را بروزرسانی کنید.

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

<div style='text-align: justify;' dir="rtl">
	
جنگو با [زبان قالبی](https://docs.djangoproject.com/en/2.1/ref/templates/language/) عرضه می شود که اجازه اعمال عملیات های منطقی ساده را به ما می دهد. در اینجا ما از تگ [for](https://docs.djangoproject.com/en/2.1/ref/templates/builtins/#std:templatetag-for) برای حلقه زدن بر روی تمام کتاب های در دسترس استفاده می کنیم. تگ های قالب باید همیشه باید شامل براکت های باز و بسته باشند. بنابراین فرمت همیشه بصورت {% ... for %} می باشد و سپس ما باید حلقه را با {% endfor %} خاتمه دهیم.
	
چیزی که ما درحال حقه زدن بر روی آن هستیم، شئ ای است که شامل تمامی کتابهای در دسترس در مدل ما می باشد. نام این شئ object_list است. بنابراین برای حلقه زدن بر روی هر کتاب ما می نویسیم{% for book in object_list %}. و سپس هر فیلد از مدلمان را نمایش می دهیم.
</div>





## صفحه وب
حالا میتوانیم سرور محلی جنگو را بالا بیاوریم و صفحه وب خود را مشاهده کنیم.

```powershell
(library) $ python manage.py runserver
```

به آدرس صفحه خانه که http://127.0.0.1:8000/ میباشد بروید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/7.png?raw=true)



اگر ما کتاب های دیگری را در admin اضافه کنیم، آنها نیز در اینجا به نمایش درخواهند آمد.

این یک مرور سریع از وبسایت جنگویی مرسوم بود. حالا بیایید به آن رابط برنامه کاربردی هم اضافه کنیم!





## رست فریموورک جنگو(Django REST Framework)
رست فریمورک جنگو هم مانند بقیه برنامه های شخص ثالث نصب می شود. با فشردن کلیدهای Control+c سرور محلی را اگر در حال اجرا است، متوقف کنید سپس در خط فرمان دستور زیر را تایپ کنید.

```powershell
(library) $ pipenv install djangorestframework==3.10.3
```

عبارت rest_framework را به تنظیمات INSTALLED_APPS در فایل settings.py اضافه کنید. من عادت دارم که یک فاصله بین برنامه های شخص ثالث و محلی قرار دهم به این دلیل که تعداد برنامه ها در بیشتر پروژه ها به سرعت افزایش پیدا می کنند.

```python
# library_project/settings.py
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
    'books.apps.BooksConfig',
]
```

در نهایت رابط برنامه کاربردی ما یک نقطه پایانی که تمام کتاب ها را در فرمت JSON لیست می کند، نمایش می دهد. بنابراین ما یک مسیر URL جدید، یک view جدید و یک فایل serializer جدید(درباره این فایل بزودی بیشتر می خوانید) نیاز خواهیم داشت.
راه های زیادی وجود دارد که می توانیم این فایل ها را سازمان دهیم اگرچه ترجیح من این است که یک برنامه مختص به رابط های برنامه کاربردی ایجاد کنیم. در این روش حتی اگر در آینده هم برنامه های بیشتری اضافه کنیم، هر برنامه میتواند شامل model ها، view ها، template ها و url های مورد نیاز برای صفحات وب باشد اما تمام فایل های مختص به رابط های برنامه کاربردی مربوط به کل پروژه درون یک برنامه اختصاص یافته به رابط های برنامه کاربردی قرار می گیرند.

بیایید ابتدا یک برنامه جدید بنام api بسازیم.

```powershell
(library) $ python manage.py startapp api
```

سپس آن را به INSTALLED_APPS اضافه کنید.

```python
# library_project/settings.py
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

برنامه api مدل پایگاه داده ای مختص به خود ندارد بنابراین نیازی به ایجاد یک فایل مهاجرت و بروزرسانی پایگاه داده طبق روال معمول نیست.





## مسیرها(URLs)
بیایید با تنظیم URL ها شروع کنیم. اضافه کردن یک نقطه پایانی درست مشابه پیکربندی مسیرهای یک برنامه جنگوی مرسوم می باشد. ابتدا نیاز است که برنامه api و مسیرهای آن را در فایل urls.py اصلی پروژه و در مسیر /api اضافه و تنظیم کنیم.

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

سپس یک فایل urls.py درون برنامه api می سازیم.

```powershell
(library) $ touch api/urls.py
```

و آن را به شکل زیر بروزرسانی می کنیم:

```python
# api/urls.py
from django.urls import path

from .views import BookAPIView

urlpatterns = [
	path('', BookAPIView.as_view()),
]
```

همه چیز آماده است.





## نما ها
مرحله بعد فایل views.py ما است که متکی به نماهای کلاسی عمومی داخلی(built-in generics class views) رست فریمورک جنگو می باشد. این نماها در ظاهر از نماهای کلاسی عمومی جنگوی مرسوم تقلید می کند اما این دو گروه از نماها یکی نیستند.
برای جلوگیری از سردرگمی، بعضی از توسعه دهندگان یک فایل API views را apiviews.py یا api.py می نامند. شخصا، وقتی که در یک برنامه مختص به رابط های برنامه کاربردی کار می کنم دچار سردرگمی نمی شوم اگر که نما های مربوط به Django REST Framework را views.py بنامم اما نظر ها در اینباره متفاوت هستند.

محتوای فایل views.py را بصورت زیر بروزرسانی کنید:

```python
# api/views.py
from rest_framework import generics

from books.models import Book
from .serializers import BookSerializer

class BookAPIView(generics.ListAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

در خطوط بالا نماهای کلاسی عمومی(generics class of views) رست فریمورک جنگو، مدل ها را از برنامه books و فایل serializers را از برنامه api (ما serializers را بعدا ایجاد می کنیم) وارد کرده ایم.

سپس یک BookAPIView که از ListAPIVIew برای ایجاد یک نقطه پایانی فقط خواندنی برای تمام نمونه(instance) های کتاب استفاده می کند، می سازیم. چندین نمای عمومی در دسترس وجود دارند که جلوتر در فصل های بعد آنها را بررسی خواهیم کرد.
دو قدمی که در نمای ما نیاز است، مشخص کردن queryset که تمام کتاب های در دسترس می باشد و سپس serializer_class که BookSrializer می باشد، است.





## سریالایزرها(Serializers)
یک سریالایزر داده را به شکلی که استفاده از آنها در اینترنت راحت باشد، معمولا JSON، و در یک نقطه پایان رابط برنامه تعاملی به نمایش درمی آید، درمی آورد. در فصل های آینده سریالایزر ها و JSON را جزئی تر پوشش خواهیم داد. برای الآن من میخواهم نشان دهم که ساخت یک سریالایزر با استفاده از رست فریمورک جنگو برای تبدیل مدل های جنگو به JSON چقدر آسان است.

یک فایل serializer.py درون برنامه api درست کنید.

```powershell
(library) $ touch api/serializers.py
```

سپس آن را بصورت زیر در ویرایشگر متن بروزرسانی کنید.

```python
# api/serializers.py
from rest_framework import serializers

from books.models import Book


class BookSerializer(serializers.ModelSerializer):
    class Meta:
    model = Book
    fields = ('title', 'subtitle', 'author', 'isbn')
```

در خطوط بالا ما کلاس serializer رست فریمورک جنگو و مدل Book را از فایل مدل برنامه books وارد می کنیم. ما ModelSerializer رست فریمورک جنگو را در کلاس BookSerializer که مدل پایگاه داده Book و فیلد های پایگاه داده ای که می خواهیم نمایش بدهیم(author، subtitle، title و isbn) را مشخص می کند، توسعه می دهیم.





## کِرل(cURL)
ما میخواهیم ببینیم نقطه پایانی رابط برنامه کاربردی ما به چه شکلی است. می دانیم که باید در مسیر http://127.0.0.1:8000/api یک JSON برگرداند. بیایید مطمئن شویم که سرور محلی جنگو در حال اجرا است:

```powershell
(library) $ python manage.py runserver
```

حالا یک خط فرمان دوم جدید باز کنید. از این خط فرمان برای رفتن به آدرس رابط برنامه کاربردی که در خط فرمان موجود در حال اجرا است، استفاده می کنیم.

ما می توانیم از برنامه محبوب [cURL](https://en.wikipedia.org/wiki/CURL) برای اجرا درخواست های HTTP در خط فرمان استفاده کنیم. تمام چیزی که برای یک درخواست GET ساده نیاز داریم این است که curl را بنویسیم و URL ای که می خواهیم صدا بزنیم را برای آن مشخص کنیم.

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

تمام داده ها به شکل JSON در آنجا هستند اما فالب بندی ضعیفی دارند و درک آن ها سخت است. خوشبختانه Django REST Framework یک شگفتی دیگر هم برای ما دارد: یک حالت بصری قدرتمند برای نقاط پایانی رابط های برنامه کاربردی ما.





## رابط برنامه کاربردی تحت مرورگر(Browsable API)
در حالیکه سرور محلی همچنان در اولین خط فرمان در حال اجرا است، در مرورگر به آدرس http://127.0.0.1:8000/api بروید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/8.png?raw=true)

 

وای به آن نگاه کنید! Django REST Framework این تجسم را برای ما بطور پیشفرض فراهم کرده است. و امکانات زیادی در این صفحه تعبیه وجود دارد که در طول کتاب آن ها را بررسی می کنیم. حالا میخواهیم این صفحه را با نقطه پایانی JSON خام مقایسه کنیم. بر روی دکمه GET کلیک کنید و از منو گزینه json را انتخاب کنید.

![](https://github.com/Seyyed-Mahdi-Sepahbodi/dfa-persian/blob/main/02-Library-Website-and-API/images/9.png?raw=true)



این شکل JSON خام نقطه پایانی رابط برنامه کاربردی ما است. فکر می کنم میتوانیم توافق کنیم که نسخه Django REST Framework، جذاب تر است.





## نتیجه گیری
<div dir="rtl">
	ما در این فصل مباحث زیادی را پوشش دادیم پس اگر موضوعات الآن کمی گیج کننده بنظر می رسند، نگران نباشید. ابتدا یک وبسایت کتابخانه جنگویی مرسوم ساختیم. سپس Django REST Framework را اضافه کردیم و یک با تعداد کمی کد یک نقطه پایانی رابط برنامه کاربردی ساختیم.
در دو فصل بعدی ما بک اند رابط برنامه کاربری خود را می سازیم و آن را به یک فرانت اند ریکتی متصل می کنیم تا یک مثال کامل را نشان دهیم که به درک اینکه در عمل تمام این اصول چگونه به یکدیگر مرتبط می شوند کمک می کند.  
</div>
