
<div dir="rtl">

# فصل 5: ای پی آی وبلاگ
  پروژه بعدی ما یک `Blog API` با استفاده از مجوعه `Django REST Framework` است که شامل کاربران، مجوزها، و امکان استفاده از عملکرد کامل CRUD(ایجاد، خواندن، به روزرسانی، حذف) است. ما همچنین ویوست ها، روترها و اسناد را بررسی خواهیم کرد.
  
  در این بخش ما بخش اصلی API را می سازیم. دقیقا مانند پروژه کتابخانه و TODO API ساخت پروژه را ابتدا با جنگو شروع کرده و سپس آن را به چهارچوب Django Rest Framework اضافه می کنیم.تفاوت اصلی این است که نقاط پایانی API ما از ابتدا از CRUD پشتیبانی می کنند؛ و همانطور که خواهید دید فریم ورک REST جنگو نیز کاملا مشابه همین کار را بدون مشکل انجام میدهد.

### تنظیمات اولیه

تنظیمات اولیه مانند قبل است. در ابتدا به دایرکتوری اصلی پروژه رفته و برنامه ای با نام blogapi می سازیم. سپس جنگو را در محیط مجازی خود نصب میکنیم و یک پیکربندی برای پروژه جدید جنگو خود و برنامه بلاگ برای پست ها انجام می دهیم.

<div dir="ltr">
  
Command Line

```shell
$ cd ~/Desktop && cd code
$ mkdir blogapi && cd blogapi
$ pipenv install django~=3.1.0
$ pipenv shell
(blogapi) $ django-admin startproject config .
(blogapi) $ python manage.py startapp posts
```
  
</div>

هنگامی که برنامه بلاگ را در پروژه ایجاد کردیم باید به جنگو نیز حضور این برنامه را اعلام کنیم. به این منظور برنامه را در قسمت `INSTALLED_APPS` در فایل `config/settings.py` اضافه می کنیم.

<div dir="ltr">
  
Code

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
    'posts', # new
]

```
  
</div>

اکنون migrate را برای اولین بار اجرا کنید تا پایگاه داده ما با تنظیمات پیش فرض جنگو و برنامه جدید همگام شود.

<div dir="ltr">

Command Line
```shell
(blogapi) $ python manage.py migrate
```
  
</div>

### مدل
مدل پایگاه داده ما دارای پنج فیلد است: نویسنده، عنوان، بدنه، create_at و updated_at. ما می‌توانیم از مدل کاربر داخلی جنگو به‌عنوان کاربر اصلی استفاده کنیم، مشروط بر اینکه آن را در خط دوم کد خود ایمپورت کنیم.

<div dir="ltr">

Code
```python
# posts/models.py
from django.db import models
from django.contrib.auth.models import User
  
  
class Post(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=50)
    body = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
  
    def __str__(self):
        return self.title
```  
  
</div>  

توجه داشته باشید که ما همچنین در حال تعریف این هستیم که نمایش str مدل باید چگونه باشد که بهترین روش جنگو برای نمایش تیتر پست ها در صفحه ادمین جنگو است.

اکنون پایگاه داده خود را با ایجاد یک فایل migrate جدید و سپس اجرای migrate برای همگام سازی پایگاه داده با تغییرات مدل خود، به روزرسانی کنید.

<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py makemigrations posts
Migrations for 'posts':
  posts/migrations/0001_initial.py
    - Create model Post
(blogapi) $ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, posts, sessions
Running migrations:
  Applying posts.0001_initial... OK
```

</div>

اکنون میخواهیم داده های خود را در برنامه داخلی ادمین جنگو مشاهده کنیم. برای این منظور ویو را به شکل زیر به فایل posts/admin.py اضافه میکنیم.

<div dir="ltr">

Code
```python
# posts/admin.py
from django.contrib import admin
from .models import Post
  
admin.site.register(Post)
```
  
</div>

در ادامه برای ایجاد یک ابر کاربر برای دسترسی به پنل ادمین دستور زیر را تایپ کرده و ورودی های مورد نظر را به آن می دهیم.

<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py createsuperuser
```

</div>

حال وب سرور لوکال را با دستور زیر اجرا می کنیم.

<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py runserver
```

</div>

سپس به آدرس http://127.0.0.1:8000/admin/  رفته و با اطلاعات ابرکاربر خود به داشبورد ادمین وارد می شویم. روی دکمه "+ افزودن" در کنار پست ها کلیک کنید و یک پست وبلاگ جدید ایجاد کنید. در کنار "نویسنده" یک منوی کشویی وجود خواهد داشت که دارای حساب کاربری ابرکاربر شما است (نام کاربر من wsv است). مطمئن شوید که یک نویسنده انتخاب شده باشد.سپس عنوان و محتوای متن را اضافه کنید سپس روی دکمه "ذخیره" کلیک کنید.

![Admin add blog post](images/1.png)

در ادامه شما به صفحه ای هدایت می شوید که همه پست های موجود در وبلاگ را نمایش می دهد.

![Admin blog posts](images/2.png)

### تست ها

</div>
