
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


</div>
