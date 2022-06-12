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


**Cookies vs localStorage**
  
Cookies are used for reading **server-side** information. They are smaller (4KB) in size and automatically sent with each HTTP request.
LocalStorage is designed for **client-side** information.
It is much larger (5120KB) and its contents are not sent by default with each HTTP request.
Tokens stored in both cookies and localStorage are vulnerable to XSS attacks. The current
best practice is to store tokens in a cookie with the httpOnly and Secure cookie flags.

  
Let’s look at a simple version of actual HTTP messages in this challenge/response flow. Note that
the HTTP header WWW-Authenticate specifies the use of a Token which is used in the response
Authorization header request.

  
  
<div dir="ltr">
  
Diagram
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
  
</div>  
  
 
                                                                                         
There are multiple benefits to this approach. Since tokens are stored on the client, scaling the
servers to maintain up-to-date session objects is no longer an issue. And tokens can be shared
amongst multiple front-ends: the same token can represent a user on the website and the same
user on a mobile app. The same session ID can not be shared amongst different front-ends, a
major limitation.
                                                                                          
A potential downside is that tokens can grow quite large. A token contains all user information,
not just an id as with a session id/session object set up. Since the token is sent on every request,
managing its size can become a performance issue.                                                                                         
                                                                                         
   
Exactly how the token is implemented can also vary substantially. Django REST Frameworks’
built-in [TokenAuthentication](http://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication) 
is deliberately quite basic. As a result, it does not support setting
tokens to expire, which is a security improvement that can be added. It also only generates one
token per user, so a user on a website and then later a mobile app will use the same token.
Since information about the user is stored locally, this can cause problems with maintaining and
updating two sets of client information.
                                                                                          
                                                                                          
JSON Web Tokens (JWTs) are a new, enhanced version of tokens that can be added to Django
REST Framework via several third-party packages. JWTs have several benefits including the
ability to generate unique client tokens and token expiration. They can either be generated on
the server or with a third-party service like [Auth0](https://auth0.com/). And JWTs can be encrypted which makes
them safer to send over unsecured HTTP connections.
                                                                                          
 
Ultimately the safest bet for most web APIs is to use a token-based authentication scheme. JWTs
are a nice, modern addition though they require additional configuration. As a result, in this book
we will use the built-in `TokenAuthentication`.
  
 
### Default Authentication

  
 
The first step is to configure our new authentication settings. Django REST Framework comes
with a [number of settings](https://www.django-rest-framework.org/api-guide/settings/) that are implicitly set. For example, `DEFAULT_PERMISSION_CLASSES`
was set to AllowAny before we updated it to `IsAuthenticated`.
                                                                                          
The `DEFAULT_AUTHENTICATION_CLASSES` are set by default so let’s explicitly add both SessionAuthentication
and BasicAuthentication to our `config/settings.py` file.
                                                                                          
                                                                                          
                                                                                          
<div dir="ltr">
  
Code
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
  
</div>                                                                                         
                                                                                          
                                                                                          
Why use **both** methods? The answer is they serve different purposes. Sessions are used to power
the Browsable API and the ability to log in and log out of it. BasicAuthentication is used to pass
the session ID in the HTTP headers for the API itself.
                                                                                          
If you revisit the browsable API at http://127.0.0.1:8000/api/v1/ it will work just as before.
Technically, nothing has changed, we’ve just made the default settings explicit.                                                                                        
                                                                                          
                                                                                          
### Implementing token authentication                                                                                    
                                                                                          

Now we need to update our authentication system to use tokens. The first step is to update our
`DEFAULT_AUTHENTICATION_CLASSES` setting to use TokenAuthentication as follows:
                                                                                          
  
<div dir="ltr">
  
Code
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
  
</div> 
                                                                                          
  
We keep `SessionAuthentication` since we still need it for our Browsable API, but now use tokens
to pass authentication credentials back and forth in our HTTP headers. We also need to add the
authtoken app which generates the tokens on the server. It comes included with Django REST
Framework but must be added to our INSTALLED_APPS setting:

                                                                                          
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
    'rest_framework',
    'rest_framework.authtoken', # new
  
    # Local
    'posts',
]
```
  
</div>                                                                                          
                                                                                          
Since we have made changes to our `INSTALLED_APPS` we need to sync our database. Stop the
server with `Control+c`. Then run the following command.                                                                                         
                                                                                          

<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, authtoken, contenttypes, posts, sessions
Running migrations:
  Applying authtoken.0001_initial... OK
  Applying authtoken.0002_auto_20160226_1747... OK
```
  
</div> 
     
                                                                                          
Now start up the server again .                                                                                     
                                                                                          
   
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py runserver
```
  
</div> 
                                                                                          
                                                                                          
If you navigate to the Django admin at http://127.0.0.1:8000/admin/ you’ll see there is now
a Tokens section at the top. Make sure you’re logged in with your superuser account to have
access.                                                                                          
   
![Admin Homepage with Tokens](images/1.jpg)
  
Click on the link for Tokens. Currently there are no tokens which might be surprising.

                                                                                          
![Admin Tokens Page](images/2.jpg)                                                                                          
                                    
                                                                                          
After all we have existing users. However, the tokens are only generated after there is an API call
for a user to log in. We have not done that yet so there are no tokens. We will shortly.
                                                                                          
  
### Endpoints                                                                                        
                                                                                          
                                                                                          
We also need to create endpoints so users can log in and log out. We could create a dedicated
users app for this purpose and then add our own urls, views, and serializers. However user
authentication is an area where we really do not want to make a mistake. And since almost all APIs
require this functionality, it makes sense that there are several excellent and tested third-party
packages we can use it instead.
                                                                                          
Notably we will use [dj-rest-auth](https://github.com/jazzband/dj-rest-auth) in combination with [django-allauth](https://github.com/pennersr/django-allauth)
to simplify things. Don’t feel bad about using third-party packages. They exist for a reason and even the best Django
professionals rely on them all the time. There is no point in reinventing the wheel if you don’t
have to!
                                                                                          
                                                                                          
### dj-rest-auth                                                                                        
                                                                                          
First we will add log in, log out, and password reset API endpoints. These come out-of-the-box
with the popular `dj-rest-auth package`. Stop the server with `Control+c` and then install it.                                                                                       
                                                                                          
                                                                                          
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ pipenv install dj-rest-auth==1.1.0
```
  
</div>                                                                                         
                                                                                          
    
Add the new app to the INSTALLED_APPS config in our `config/settings.py` file.                                                                                          

                                                                                          
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
    'rest_framework',
    'rest_framework.authtoken',
    'dj_rest_auth', # new
  
    # Local
    'posts',
]

```
  
</div>                       

                                                                                          
                                                                                          
Update our `config/urls.py` file with the `dj_rest_auth` package. We’re setting the URL routes
to api/v1/dj-rest-auth. Make sure to note that URLs should have a dash - not an underscore
_, which is an easy mistake to make.
                                                                                          
                                                                                          
<div dir="ltr">
  
Code
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
  
</div>                                                                                       
                                                                                          
                                                                                          
And we’re done! If you have ever tried to implement your own user authentication endpoints, it
is truly amazing how much time—and headache—dj-rest-auth saves for us. Now we can spin up
the server to see what dj-rest-auth has provided.                                                                                      
                                                                                          

<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py runserver
```
  
</div>
                                                                                          

We have a working log in endpoint at http://127.0.0.1:8000/api/v1/dj-rest-auth/login/.
                                                                                          
                                                                                          
![API Log In Endpoint](images/3.jpg)                                                                                          
             
                                                                                          
And a log out endpoint at http://127.0.0.1:8000/api/v1/dj-rest-auth/logout/.                                                                                         
              
                                                                                          
                                                                                          
![API Log Out Endpoint](images/4.jpg)                                                                                      
                                                                                          
There are also endpoints for password reset, which is located at:

                                                                                          
http://127.0.0.1:8000/api/v1/dj-rest-auth/password/reset                                                                               
   

![API Password Reset](images/5.jpg)
                                                                                          
                                                                                          
                                                                                          
And for password reset confirmed:
                                                                                          
http://127.0.0.1:8000/api/v1/dj-rest-auth/password/reset/confirm                                                                                         
                                                                                          
                                                                                          
![API Password Reset Confirm](images/6.jpg)                                                                                      
                                                                                          
 
### User Registration
                                                                                         
                                                                                          
Next up is our user registration, or sign up, endpoint. Traditional Django does not ship with builtin views or 
URLs for user registration and neither does Django REST Framework. Which means
we need to write our own code from scratch; a somewhat risky approach given the seriousness–
and security implications–of getting this wrong                                                                                          
                                                                                          
                                                                                          
A popular approach is to use the third-party package [django-allauth](https://github.com/pennersr/django-allauth) which comes with user
registration as well as a number of additional features to the Django auth system such as social authentication via Facebook, Google, Twitter, etc.
If we add dj_rest_auth.registration from the dj-rest-auth package then we have user registration endpoints too!

                                                                                        
Stop the local server with `Control+c` and install django-allauth.                                                                                         
                                                                                          
 
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ pipenv install django-allauth~=0.42.0
```
  
</div>
                                                                                          
                                                                                          
Then update our `INSTALLED_APPS` setting. We must add several new configs:                                                                                        
                                                                                          
                                                                                          
- django.contrib.sites
- allauth
- allauth.account
- allauth.socialaccount
- dj_rest_auth.registration                                                                                          
                                                                                          
   
Make sure to also include EMAIL_BACKEND and SITE_ID. Technically it does not matter where in
the `config/settings.py` file they are placed, but it’s common to add additional configs like that
at the bottom.

                                                                                          
                                                                                          
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
  
</div>                                                                                          
                                                                                          
                                                                                        
The email back-end config is needed since by default an email will be sent when a new user is
registered, asking them to confirm their account. Rather than also set up an email server, we will
output the emails to the console with the console.EmailBackend setting.
                                                                                          
                                                                                          
`SITE_ID` is part of the built-in Django [“sites” framework](https://docs.djangoproject.com/en/3.1/ref/contrib/sites/), which is a way to host multiple
websites from the same Django project. We obviously only have one site we are working on here
but django-allauth uses the sites framework, so we must specify a default setting.
                                                                                          
                                                                                          
Ok. We’ve added new apps so it’s time to update the database.                                                                                          
                                                                                          
                                                                                          
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py migrate
```
  
</div>                                                                                          
                                                                                          
                                                                                          
Then add a new URL route for registration.                                                                                       
                                                                                          
                                                                                          
<div dir="ltr">
  
Code
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
  
</div>                                                                                         
                                                                                          
                                                                                          
And we’re done. We can run the local server.                                                                                          
                                                                                          
                                                                                          
<div dir="ltr">
  
Command Line
```shell
(blogapi) $ python manage.py runserver
```
  
</div>                                                                                          
                                                                                          
                                                                                          
                                                                                          
There is now a user registration endpoint at http://127.0.0.1:8000/api/v1/dj-rest-auth/registration/.
                                                                                          
                                                                                          
                                                                                          
                                                                                          
![API Register](images/7.jpg)                                                                                         
                                                                                          
                                                                                          
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