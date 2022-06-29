
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
  
در ادامه یک تست ابتدایی برای مدل پست وبلاگ می نویسیم. در این تست اطمینان حاصل می شود که کاربر وارد شده به سیستم بتواند یک پست با عنوان و بدنه ایجاد کند.
  
  
<div dir="ltr">
    
Code
```python
# posts/tests.py
from django.test import TestCase
from django.contrib.auth.models import User
from .models import Post
    
    
class BlogTests(TestCase):
    
    @classmethod
    def setUpTestData(cls):
        # Create a user
        testuser1 = User.objects.create_user(
            username='testuser1', password='abc123')
        testuser1.save()

        # Create a blog post
        test_post = Post.objects.create(
            author=testuser1, title='Blog title', body='Body content...')
        test_post.save()

    def test_blog_content(self):
        post = Post.objects.get(id=1)
        author = f'{post.author}'
        title = f'{post.title}'
        body = f'{post.body}'
        self.assertEqual(author, 'testuser1')
        self.assertEqual(title, 'Blog title')
        self.assertEqual(body, 'Body content...')
```

</div>
  
برای اطمینان از کارکرد صحیح تست با فشردن Control+c سرور لوکال را بسته و سپس تست ها را اجرا می کنیم.
    

<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py test
```

</div>
  
پس از اجرا باید خروجی های مانند زیر مشاهده شود که نشان دهنده کارکرد صحیح همه قسمت های برنامه است.
   
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py test
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.105s
    
OK
Destroying test database for alias 'default'...
```

</div>  
    
    
در این زمان قسمت معمول ای پی ای جنگو به پایان رسیده است و زمان آن است که مدل و دیتاهایی را به دیتابیس اضافه کنیم. بنابراین زمان آن است که از جنگو رست فریمورک برای انتقال دیتاهای مدل به ای پی آی خود بهره ببریم.
  
    
### جنگو رست فریمورک 
    

همانطور که قبلاً دیده‌ایم، `Django REST Framework` وظیفه تبدیل مدل‌های پایگاه داده ما به یک API RESTful را بر عهده دارد. سه مرحله اصلی برای این فرآیند وجود دارد:
- `urls.py` فایل مشخص کننده مسیر url ها
- `serializers.py` فایل برای تبدیل دیتا به جیسون  
- `views.py` فایل برای پیاده سازی منطق هر کدام از اندپوینت ها

    
در کامندلاین با استفاده از pipenv جنگو رست فریمورک را به شکل زیر نصب می کنیم.
    
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ pipenv install djangorestframework~=3.11.0
```

</div>     

سپس آن را در قسمت INSTALLED_APPS در فایل config/settings.py اضافه می کنیم. همچنین بهتر است برای جنگو رست فریمورک در قسمت پرمیشن ها در ستینگ مجوز مناسب را که به صورت پیشفرض AllowAny تعریف شده است را مشخص کنیم. این قسمت در فصل بعدی آپدیت خواهد شد.    
    
  
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
    
    # 3rd-party apps
    'rest_framework', # new
    
    # Local
    'posts.apps.PostsConfig',
]
    
# new
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ]
}
```

</div>    
    
=اکنون ما باید url ها ، View ها و سریالایزرهای برنامه را ایجاد کنیم.
  
    
### URL ها   

بیایید با مسیرهای URL برای مکان های واقعی endpoint ها شروع کنیم. ابتدا فایل `url.py` را با وارد کردن در خط دوم در آدرس api/v1/ برای پست های برنامه آپدیت می کنیم.
 
    
<div dir="ltr">
    
Code
```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path # new
    
    
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('posts.urls')), # new
]
```

</div>    

این مسئله تمرین مناسبی است برای اینکه همیشه ورژن API ها را به صورت v1/ ، v2/ ، ... نگه داریم. به دلیل اینکه ممکن است زمانی که تغییرات حجیمی بر روی API ها اعمال میکنیم باعث باگ یا لگ شود و نیاز به ورژن های قبلی داشته باشیم. به این ترتیب میتوانیم در زمان لانچ ورژن جدید API از ورژن های قبلی نیز ساپورت کنیم. زیرا ممکن است سرویس برخی از مشتریان بر روی ورژن های قبلی لانچ شده باشد.


    
توجه داشته باشید که از آنجایی که تنها برنامه ما در این مرحله پست است، می توانیم آن را مستقیماً در اینجا قرار دهیم. اگر ما چندین برنامه در یک پروژه داشتیم، ممکن است منطقی تر باشد که یک برنامه اختصاصی api ایجاد کنیم و سپس همه مسیرهای آدرس API دیگر را در آن قرار دهیم. اما برای پروژه‌های اساسی مانند این، ترجیح می‌دهم از یک برنامه api که فقط برای مسیریابی استفاده می‌شود اجتناب کنم. در صورت نیاز همیشه می‌توانیم یکی را بعداً اضافه کنیم.
    
در مرحله بعد فایل url.py برنامه را ایجاد میکنیم.
    
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ touch posts/urls.py
```

</div>   
    
سپس کد زیر را در آن فایل وارد میکنیم. 
    
    
<div dir="ltr">
    
Code
```python
# posts/urls.py
from django.urls import path
from .views import PostList, PostDetail
    
    
urlpatterns = [
    path('<int:pk>/', PostDetail.as_view()),
    path('', PostList.as_view()),
]

```

</div>     
    
    
همه مسیرهای وبلاگ در api/v1/ خواهند بود، بنابراین نمای PostList ما (که به زودی خواهیم نوشت) دارای رشته خالی '' در api/v1/ خواهد بود و نمای PostDetail (همچنین باید نوشته شود) در api/v1/ # که در آن # کلید اصلی ورودی را نشان می دهد. به عنوان مثال، اولین پست وبلاگ دارای شناسه اولیه 1 است، بنابراین در مسیر api/v1/1، پست دوم در api/v1/2 و غیره خواهد بود.   
    
    
### سریالایزرها    
    
اکنون زمان سریالایز کردن است. یک فایل serializers.py جدید در برنامه پست های خود ایجاد کنید.    
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ touch posts/serializers.py    
```

</div>     
    
    
سریالایز نه تنها داده ها را به JSON تبدیل می کند، بلکه می تواند تعیین کند که کدام فیلدها را شامل یا حذف کند. در مورد ما، فیلد شناسه‌ای را که Django به‌طور خودکار به مدل‌های پایگاه داده اضافه می‌کند، اضافه می‌کنیم، اما فیلد «updated_at» را با درج نکردن آن در فیلدهای خود حذف می‌کنیم.
    
توانایی گنجاندن/حذف فیلدها در API ما یک ویژگی قابل توجه است. اغلب اوقات، یک مدل پایگاه داده زیربنایی، فیلد‌های بسیار بیشتری نسبت به آنچه باید در معرض نمایش قرار گیرد، خواهد داشت. کلاس سریالایز قدرتمند Django REST Framework کنترل این مورد را بسیار ساده می کند.   
    
    
<div dir="ltr">
    
Code
```python
# posts/serializers.py
from rest_framework import serializers
from .models import Post
    
    
class PostSerializer(serializers.ModelSerializer):
    
    class Meta:
        fields = ('id', 'author', 'title', 'body', 'created_at',)
        model = Post
```

</div>      
    
در بالای فایل، کلاس سریالایزرهای Django REST Framework و مدل های خودمان را وارد کرده ایم. سپس یک PostSerializer ایجاد کردیم و یک کلاس Meta اضافه کردیم که در آن مشخص کردیم کدام فیلدها را شامل شود و به صراحت مدل را برای استفاده تنظیم کردیم. راه‌های زیادی برای سفارشی‌سازی سریالایزرها وجود دارد، اما برای موارد استفاده رایج، مانند وبلاگ اولیه، این روش پیاده سازی شده کفایت می کند.
    
    
### ویوها
  
مرحله نهایی ساخت ویوهای برنامه است. Django Rest Framework چندین ویو عمومی مناسب و کمک کننده برای این کار در اختیار دارد. در حال حاضر ما از [ListAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#listapiview) در کتابخانه و برنامه todolist برای ساخت اندپوینت های فقط قابل خواندن جهت گرفتن لیست مدل های ساخته شده در برنامه استفاده می کنیم. در برنامه Todos همچنین از [RetrieveAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#retrieveapiview) جهت ساخت اندپوینت تکی فقط خواندنی که مشابه نمای جزئیات در جنگو سنتی است نیز استفاده میکنیم.       
    
   
برای API وبلاگ ما میخواهیم لیستی از همه پست های بلاگ را به صورت اندپوینت خواندنی-نوشتنی در اختیار داشته باشیم که به این منظور از [ListCreateAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#listcreateapiview) استفاده می نماییم که بسیار شبیه بهListAPIView است که قبلا از آن استفاده میکردیم با این تفاوت که به کاربر امکان نوشتن نیز میدهد. ما همچنین نیاز داریم که برای هر پست به صورت جدا امکانات شامل خواندن، ادیت کردن و حذف پست را نیز داشته باشیم. که به این منظوری کتابخونه عمومی ای برای این هدف با نام  [RetrieveUpdateDestroyAPIView](https://www.django-rest-framework.org/api-guide/generic-views/#retrieveupdatedestroyapiview) در Django REST Framework وجود دارد که ما در این برنامه از آن استفاده میکنیم. 
    
  
اکنون فایل views.py را به شکل زیر آپدیت میکنیم.
    
    
<div dir="ltr">
    
Code
```python
# posts/views.py
from rest_framework import generics
from .models import Post
from .serializers import PostSerializer
    
    
class PostList(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    
    
class PostDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

</div>   
    
در بالاب فایل کتابخانه generic را از Django REST Framework همانند فایل مدل هاو سریالایزر خود وارد میکنیم. لیست پست ها از کتابخانه عمومی  `ListCreateAPIView` و هر پست از `RetrieveUpdateDestroyAPIView` استفاده میکند.       

    
بسیار شگفت انگیز است که تنها کاری که باید انجام دهیم این است که نمای کلی خود را به روز کنیم تا رفتار یک نقطه پایانی API را به طور اساسی تغییر دهیم. این مزیت استفاده از یک فریم ورک با ویژگی‌های کامل مانند Django REST Framework است. همه این قابلیت‌ها در دسترس هستند، آزمایش شده‌اند و فقط کار می‌کنند. به عنوان توسعه دهندگان، مجبور نیستیم چرخ را در اینجا دوباره اختراع کنیم.
    
    
### ای پی آی قابل مرور   
   
سرور محلی را برای برقراری ارتباط با ای پی آی خود اجرا میکنیم.
  
    
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py runserver
```

</div> 
    
برای دیدن لیست پست های ایجاد شده به آدرس  http://127.0.0.1:8000/api/v1/  میرویم.
  

![API Post List](images/3.png)
    

این صفحه یک لیست از پست های موجود در ای پی آی را با فرمت جیسون نشان می دهد. توجه داشته باشید که هر دو متد GET و POST قابل اجرا هستند.
  
    

اکنون اجازه دهید تأیید کنیم که نقطه پایانی نمونه مدل ما - که به جای یک لیست از همه پست ها به یک پست مربوط می شود - وجود دارد.
 
به آدرس http://127.0.0.1:8000/api/v1/1/ میرویم.

![API Post Detail](images/4.png)   
    
   

در هدر مشاهده میشود که متدهای GET,PUT,PATCH و DELETE ساپورت میشوند ولی نمیتوان از متد POST در اینجا استفاده کرد. در حقیقت شما میتوانید از فرم HTML موجود برای ایجاد تغییرات در فرم و از کلید DELETE برای حذف آن استفاده کنید.
    
Let’s try things out. Update our title with the additional text (edited) at the end. Then click
on the “PUT” button.
    
![API Post Detail edited](images/5.png)  
    
Go back to the Post List view by clicking on the link for it at the top of the page or navigating
directly to http://127.0.0.1:8000/api/v1/ and you can see the updated text there as well.   
    
![API Post List edited](images/6.png)  
    
 
### نتیجه گیری
    
ای پی آی وبلاگ ما در این مرحله به درستی کار میکند. اگرچه یک مشکل بزرگ در این برنامه وجود دارد: هر کسی می تواند پست دلخواهی را ادیت و یا حذف کند. به بیان دیگر، مجوزی برای این کار برای کاربران وجود ندارد. در فصل بعدی با هم یاد میگیریم که چگونه مجوزهای لازم را برای تامین امنیت برنامه خود به کار ببریم.
    
</div>
