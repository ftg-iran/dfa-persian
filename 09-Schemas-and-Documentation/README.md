<div dir="rtl" >
  
# الگو ها (schemas) و مستندات 

اکنون که API خود را کامل کرده ایم، به راهی نیاز داریم تا عملکرد آن را به سرعت و با دقت برای دیگران مستند کنیم. 
به هر حال، در اکثر شرکت ها و تیم ها، توسعه دهنده ای که از API استفاده می کند،
همان توسعه دهنده ای نیست که در ابتدا آن را ساخته است.
خوشبختانه، ابزارهای خودکاری برای مدیریت این موضوع برای ما وجود دارد.


[schema](https://www.django-rest-framework.org/api-guide/schemas/#schema) 
یک سند قابل خواندن توسط ماشین هست که چارچوب کلی یک
API endpoint 
های موجود
URLs و
HTTP methods (GET, POST, PUT, DELETE, etc.)
هایی که پشتیبانی میکنند را مشخص میکند.
مستندات چیزی است که به یک schema اضافه شده که خواندن و استفاده آن را برای انسان آسان‌تر می‌کند.
در این فصل ما یک schema به پروژه وبلاگ خود اضافه می کنیم و سپس به دو روش مختلف مستند سازی آن را انجام میدهیم.
در پایان، ما یک روش خودکار را برای ثبت هرگونه تغییر  (چه در حال چه در آینده ) در API خود پیاده سازی کرده ایم.




به عنوان یادآوری ، در اینجا لیست کاملی از endpoint های API های فعلی را آمده است :
<div dir="ltr">
  
Diagram
```code
|Endpoint                              |HTTP Verb|
|--------------------------------------|---------|
|/                                     |GET      |
|/:pk/                                 |GET      |
|users/                                |GET      |
|users/:pk/                            |GET      |
|/rest-auth/registration               |POST     |
|/rest-auth/login                      |POST     |
|/rest-auth/logout                     |GET      |
|/rest-auth/password/reset             |POST     |
|/rest-auth/password/reset/confirm     |POST     |
```
  
</div>


### Schemas
  
  
قبل از 
[نسخه 3.9](https://www.django-rest-framework.org/community/3.9-announcement/)
پایتون
Django REST Framework
برای schema های خود به
[Core API](http://www.coreapi.org/) 
وابسته بود اما حالا کاملا به schema
[OpenAPI](https://www.openapis.org/)
(که قبلا Swagger نامیده میشد)
تغیر کرده است.
  
در قدم اول باید
[PyYAML](https://pyyaml.org/) 
و
[uritemplate](https://github.com/python-hyper/uritemplate)
نصب کنیم.

PyYAML
schema 
های ما را به فرمت OpenAPI مبتنی بر YAML تبدیل می کند،
در حالی که uritemplate پارامترهایی را به مسیرهای URL اضافه می کند.
  
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ pipenv install pyyaml==5.3.1 uritemplate==3.0.1
```
  
</div>
  

در مرحله بعد، یک انتخاب به ما ارائه می شود: ایجاد یک schema ایستا یا یک schema پویا.
اگر API شما
اغلب تغییر نمی کند، طرحواره ایستا می تواند به صورت دوره ای تولید شود و از فایل های استاتیک برای عملکرد قوی استفاده شود.
با این حال، اگر API شما اغلب تغییر می کند، ممکن است گزینه پویا را در نظر بگیرید.
ما هر دو را در اینجا اجرا خواهیم کرد.

در مرحله اول،
سراغ schema ایستا میرویم که از دستور مدیریتی
generateschema
برای تولید schema استفاده میکند.
ما می توانیم نتیجه را در فایلی به نام
openapi-schema.yml
قرار دهیم.

<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py generateschema > openapi-schema.yml
```
  
</div>

اگر آن فایل را باز کنید، بسیار طولانی است و خیلی کاربر پسند نیست. اما برای کامپیوتر کاملا فرمت شده است.
 
برای رویکرد پویا،
`config/urls.py` 
را با وارد کردن
`get_schema_view`
به روزرسانی کنید و سپس یک مسیر اختصاصی در openapi ایجاد کنید.
عنوان، توضیحات و نسخه را می توان در صورت نیاز سفارشی کرد.

عنوان، توضیحات و نسخه را می توان در صورت نیاز سفارشی کرد.

<div dir="ltr">
  
Code
```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path
from rest_framework.schemas import get_schema_view # new
  
  
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('posts.urls')),
    path('api-auth/', include('rest_framework.urls')),
    path('api/v1/dj-rest-auth/', include('dj_rest_auth.urls')),
    path('api/v1/dj-rest-auth/registration/',
          include('dj_rest_auth.registration.urls')),
    path('openapi', get_schema_view( # new
        title="Blog API",
        description="A sample API for learning DRF",
        version="1.0.0"
    ), name='openapi-schema'),
]

```
  
</div>
  
اگر پروژه شما در بستر سرور محلی قرار دارد با دستور
<div dir="ltr">

```shell
(blogapi) $ python manage.py runserver 
```

</div>
مجددا آن را اجرا کنید، مشاهده میکنید زمانی که به آدرس
http://127.0.0.1:8000/openapi
مراجعه کردید schema به صورت خودکار ساخته شده و دردسترس شما قرار میگیرد.


![API Schema](https://github.com/ftg-iran/dfa-persian/blob/main/09-Schemas-and-Documentation/images/1.jpg?raw=true)
  
من شخصاً رویکرد پویا را در پروژه ها ترجیح می دهم.



### مستندات  


Django REST Framework 
همچنین دارای ویژگی
[مستندسازی](https://www.django-rest-framework.org/topics/documenting-your-api/)
API
داخلی است که schema را به فرمت خواناتر برای توسعه‌دهندگان دیگر تبدیل می‌کند.


در حال حاضر، سه رویکرد محبوب در اینجا وجود دارد:
استفاده از
[SwaggerUI](https://swagger.io/tools/swagger-ui/)،
[ReDoc](https://github.com/Rebilly/ReDoc) و
یا بسته شخص ثالث
[drf-yasg](https://drf-yasg.readthedocs.io/en/stable/)
 از آنجایی که drf-yasg بسیار محبوب است و دارای بسیاری از ویژگی های داخلی است، ما در اینجا از آن استفاده خواهیم کرد.

مرحله اول نصب آخرین نسخه drf-yasg است.
<div dir="ltr">

```shell
(blogapi) $ pipenv install drf-yasg==1.17.1
```

</div>
  
مرحله دوم، آن را به پیکربندی `INSTALLED_APPS` خود در `config/settings.py` اضافه کنید.


  
<div dir="ltr">
  
Code
```python
# config/settings.py
INSTALLED_APPS = [
    ...
    # 3rd-party apps
    'rest_framework',
    'rest_framework.authtoken',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'rest_auth',
    'rest_auth.registration',
    'drf_yasg', # new
  
    # Local
    'posts.apps.PostsConfig',
]
```
  
</div>
  

مرحله سوم، فایل `urls.py` در سطح پروژه را به‌روزرسانی کنید.
در بالای فایل می‌توانیم
DRF’s `get_schema_view` 
را با `drf_yasg` جایگزین کنیم
و همچنین openapi را وارد کنیم.
همچنین مجوز DRF را برای گزینه‌های دیگر اضافه می‌کنیم.


متغیر`schema_view`
به‌روزرسانی می‌شود
و شامل فیلدهای اضافی مانند
`terms_of_service` 
، contact
و license است.
سپس تحت الگوهای url خود مسیرهایی را برای Swagger و ReDoc اضافه می کنیم.
 
<div dir="ltr">
  
Code
```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path
from rest_framework import permissions # new
from drf_yasg.views import get_schema_view # new
from drf_yasg import openapi # new
  
schema_view = get_schema_view( # new
    openapi.Info(
        title="Blog API",
        default_version="v1",
        description="A sample API for learning DRF",
        terms_of_service="https://www.google.com/policies/terms/",
        contact=openapi.Contact(email="hello@example.com"),
        license=openapi.License(name="BSD License"),
    ),
    public=True,
    permission_classes=(permissions.AllowAny,),
)
  
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('posts.urls')),
    path('api-auth/', include('rest_framework.urls')),
    path('api/v1/dj-rest-auth/', include('dj_rest_auth.urls')),
    path('api/v1/dj-rest-auth/registration/',
          include('dj_rest_auth.registration.urls')),
    path('swagger/', schema_view.with_ui( # new
      'swagger', cache_timeout=0), name='schema-swagger-ui'),
    path('redoc/', schema_view.with_ui( # new
       'redoc', cache_timeout=0), name='schema-redoc'),
]
```
  
</div>

مطمئن شوید که سرور شما در حال اجرا است. آدرس Swagger اکنون در آدرس زیر در دسترس است:

http://127.0.0.1:8000/swagger/
  
![Swagger view](https://github.com/ftg-iran/dfa-persian/blob/main/09-Schemas-and-Documentation/images/2.jpg?raw=true)
  

سپس تأیید کنید که نمای
ReDoc نیز در
http://127.0.0.1:8000/redoc/
کار می کند.
  
  
![ReDoc view](https://github.com/ftg-iran/dfa-persian/blob/main/09-Schemas-and-Documentation/images/3.jpg?raw=true)
  
اسناد drf-yasg کاملاً جامع هستند و بسته به نیاز API های شما سفارشی سازی میشوند .

### نتیجه 

افزودن مستندات بخش مهمی از هر API است.
این معمولاً اولین چیزی است که یک توسعه دهنده همکار به آن نگاه می کند،
چه در یک تیم و چه در پروژه های منبع باز.
با تشکر از ابزارهای خودکار پوشش داده شده در این فصل،
اطمینان از اینکه API شما دارای اسناد دقیق و به روز است، فقط به مقدار کمی پیکربندی نیاز دارد.


</div>



  
  