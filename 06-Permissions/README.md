<div dir="ltr">
  
# مجوزها    
  
امنیت بخش مهمی از هر وب‌سایت است اما با API ها دو چندان می‌شود.در حال حاضر API ما اجازه دسترسی کامل به هر شخصی را می‌دهد. هیچ محدودیتی وجود ندارد و هر کاربری هر کاری می‌تواند انجام دهد که بسیار خطرناک است.به عنوان مثال، یک کاربر ناشناس می‌تواند پستی را ایجاد کند، بخواند، تغییر دهد یا حذف کند. حتی آن پستی که خودش ایجاد نکرده‌است.مشخصاً ما چنین چیزی را نمی‌خواهیم.


البته `Django REST Framework` با تنظیمات ساده‌ي مجوزها همراه می‌باشد که ما می‌توانیم از آن‌ها برای ایمن کردن API استفاده کنیم.
این تنظیمات را می‌توان در سطح پروژه، در سطح نما یا در سطح هر مدل اعمال کرد.
  
در این فصل ابتدا یک کاربر جدید اضافه می‌کنیم و تنظیمات چندین مجوز را آزمایش می‌کنیم. سپس مجوز شخصی‌سازی شده‌ی خودمان را طوری ایجاد می‌کنیم که تنها نویسنده آن پست بتواند آن‌را بروزرسانی یا حذف کند.
  
 ### ایجاد کاربر جدید
 بیایید با ایجاد کاربر دوم شروع کنیم. با این کار می‌توانیم بین اکانت‌های این دو کاربر جابجا شویم تا تنظیمات مجوزها را آزمایش کنیم.
 
به پنل ادمین در `http://127.0.0.1:8000/admin/` بروید.سپس روی “+ Add” کنار Users کلیک کنید. نام‌کاربری و پسورد را برای کاربر جدید وارد کنید و سپس بر روی دکمه‌ی 'Save' کلیک کنید. من در اینجا نام‌کاربری را testuser انتخاب کرده‌ام.
  
![Admin Add User Page](images/1.jpg)

تصویر بعدی صفحه‌ی ادمین برای تغییر کاربرها می‌باشد. من کاربر خود را testuser  انتخاب کرده‌ام و در اینجا می‌توانم اطلاعات بیشتری را نظیر نام، نام‌خانوادگی، آدرس ایمیل، آدرس و ...به مدل کاربر اضافه کنم. اما هیچکدام از آن‌ها برای مقصود ما مهم نیستند: ما فقط به نام‌کاربری و پسورد برای آزمایش نیاز داریم. 
  
![Admin User Change](images/2.jpg)

به پایین این صفحه رفته و روی دکمه‌ی 'Save' کلیک کنید. که به صفحه اصلی کاربران در .
http://127.0.0.1:8000/admin/auth/user/
هدایت می‌کند.
  
 ![Admin All Users](images/3.jpg)
  
 همانطور که می‌بینید دو کاربر ما حضور دارند.
  
 ### اضافه کردن قابلیت ورود به API قابل مرور
 
 هر بار که بخواهیم بین دو کاربر جابجا شویم باید به پنل ادمین جنگو رفته و از آن حساب خارج شده و به دیگری لاگین کنیم. سپس به API موردنظر برویم.
 
این اتفاق معمولی هست که`Django REST Framework`  تنظیمی یک خطی برای اضافه  کردن قابلیت وارد و خارج شدن کاربر به صورت مستقیم به API قابل جستجو دارد. که ما آن‌ را پیاده‌سازی می‌کنیم.
 
در مسیر پروژه داخل فایل `urls.py`، یک مسیر که شامل `rest_framework.urls` می‌باشد را به URLها اضافه کنید. تا حدودی گیج کننده‌است، مسیرواقعی مشخص شده می‌تواند هر چیزی که ما می‌خواهیم باشد. چیزی که مهم است `rest_framework.urls` یک جایی قرار دارد. ما از مسیر   api-auth بخاطر این که که با مستندات مطابقت دارد استفاده می‌کنیم، اما ما هر چیزی را می‌توانستیم استفاده کنیم و همه آن‌ها نیز شبیه به هم عمل می‌کردند.  
   
 
کد
  
 ```
  # config/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('posts.urls')),
    path('api-auth/', include('rest_framework.urls')), # new
]
```
حال به مسیر API قابل مرور در http://127.0.0.1:8000/api/v1/، بروید.  یک تغییر کوچک بوجود آمده‌است: که یک فلش رو به پایین در کنار نام‌کاربری در گوشه بالا سمت راست ظاهر شده‌است.روی آن کلیک کنید. 
  
![API Log In](images/4.jpg)
  
 از آنجایی که ما به عنوان کاربر اصلی وارد شده‌ایم در اینجا برای من WSV نمایان شده‌است. بر روی لینک کلیک کنید و یک منو کشویی برای خارج شدن(Logout)ظاهر می‌شود. روی آن کلیک کنید.   

لینک بالا در سمت راست حالا به لاگین(login) تغییر پیدا کرده‌است. روی آن کلیک کنید. به صفحه لاگین `Django Rest Framework` هدایت می‌شویم. در اینجا از اکانت آزمایشی استفاده می‌کنیم. در نهایت به صفحه اصلی API جایی که کاربر آزمایشی در سمت راست و بالا نمایان می‌باشد، هدایت می‌شویم.
  
![API Log In Testuser](images/5.jpg)
  
به عنوان قدم آخر از حساب آزمایشی خود خارج شوید.

  
![API Log In Link](images/6.jpg)

باید لینک لاگین را دوباره در سمت راست بالا  ببینید.  
 
### مجوز برای همه
  
در حال حاضر، هر کاربر ناشناس احراز هویت نشده می‌تواند به لیست پست‌ها دسترسی داشته باشد. ما این را می‌دانیم زیرا اگرچه لاگین نکرده‌ایم، اما می‌توانیم تنها پستمان را مشاهده کنیم. حتی بدتر، هر کسی می‌تواند دسترسی کامل برای ایجاد پست، بروزرسانی و یا حذف آن‌ها داشته باشد.
  
 صفحه جزئیات در آدرس /http://127.0.0.1:8000/api/v1/1 اطلاعات قابل مشاهده می‌باشد و هر کاربر تصادفی می‌تواند پستی را در صورت وجود بروزرسانی یا حذف کند. که خوب نیست.
  
![API Detail Logged Out](images/7.jpg)
  
دلیلی که ما هنوز می‌توانیم پست‌ها و جزئیات آن‌ها را مشاهده کنیم این است که ما تنظیمات مجوزها را در فایل `config/settings.py` روی AllowAny قرار داده‌ایم.
به عنوان یک یادآوری کوچک، به شکل زیر می‌باشد:  
<div dir="ltr">
  
کد

```python
# config/settings.py
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ]
}
```
</div>

### مجوز در سطح نما
  
آنچه اکنون می‌خواهیم این است که دسترسی کاربران را به API محدود کنیم. چندین روش برای این منظور وجود دارد :در سطح پروژه، در سطح نما(view) یا در سطح شیء اما از آنجایی که ما فقط دو نما داریم پس بیایید از آنجا شروع کنیم و برای هر کدام مجوز تعیین کنیم.
  
در فایل `posts/views.py`،ماژول permissions را وارد کنید و سپس به هر فیلد `permisson_classes` را اضافه کنید.
  
کد

```
# posts/views.py
from rest_framework import generics, permissions # new
from .models import Post
from .serializers import PostSerializer
  
  
class PostList(generics.ListCreateAPIView):
    permission_classes = (permissions.IsAuthenticated,) # new
    queryset = Post.objects.all()
    serializer_class = PostSerializer
  
  
class PostDetail(generics.RetrieveUpdateDestroyAPIView):
    permission_classes = (permissions.IsAuthenticated,) # new
    queryset = Post.objects.all()
    serializer_class = PostSerializer  
```
این تمام چیزی بود که نیاز داشتیم. صفحه را در http://127.0.0.1:8000/api/v1/ رفرش کنید. ببینید چه اتفاقی افتاد!

![API Post List Logged Out](images/8.jpg)
  
ما دیگر نمی‌توانیم لیست پست‌ها را ببینیم. در عوض با پیام HTTP 403 که کد وضعیت ممنوع می‌باشد زیرا ما لاگین نشده‌ایم. و از آنجایی که مجوز نداریم هیچ فرمی در API برای ویرایش داده‌ها وجود ندارد.
  
اگر به مسیر جزئیات پست درhttp://127.0.0.1:8000/api/v1/ بروید پیامی مشابه را خواهید دید و همچنین  فرمی برای ویرایش وجود ندارد.
  
![API Detail Logged Out](images/9.jpg)  
  
بنابراین از این لحظه تنها کاربران لاگین شده می‌توانند صفحه API را ببینند. اگر با حساب testuser‌ یا superuser خود وارد شوید این صفحات در دسترس هستند.
  
اما به این فکر کنید که اگر API از نظر پیچیدگی گسترده‌تر شود. در واقع ما نماها و صفحات بیشتری را در آینده خواهیم داشت. که اضافه کردن `permission_classes`اختصاصی برای هر نما اگر بخواهیم از تنظیمات مجوز مشابه در تمام API استفاده کنیم کاری تکراری  به نظر می‌آید.
  
آیا بهتر نیست که مجوزها را یکبار برای همیشه، برای سطح پروژه   انجام دهیم، تا اینکه به ازای هر نما اینکار را انجام دهیم؟
  
### مجوز در سطح پروژه
 
در این مرحله باید سر خود را به نشانه موافقت تکان دهید. این یک روش ساده‌تر و امن‌تر برای تنظیم سیاست محدودیت‌‌های دسترسی در سطح پروژه و در حد نیاز اعمال آن در سطح نما می‌باشد. این کاری است که انجام می‌دهیم.
  
خوشبختانه `Django REST Framework` با تعدادی از تنظیمات مجوزها  در سطح پروژه همراه می‌باشد که می‌توانیم استفاده کنیم، شامل:
  
- [AllowAny](http://www.django-rest-framework.org/api-guide/permissions/#allowany)  - هر کاربری، احراز هویت شده باشد یا نه، اجازه دسترسی کامل را دارد  
- [IsAuthenticated](http://www.django-rest-framework.org/api-guide/permissions/#isauthenticated) - کاربران ثبت‌نام شده و احراز هویت شده فقط اجازه دسترسی دارند
- [IsAdminUser](http://www.django-rest-framework.org/api-guide/permissions/#isadminuser) - تنها ادمین یا کاربران سوپر اجازه دسترسی خواهند داشت
- [IsAuthenticatedOrReadOnly](http://www.django-rest-framework.org/api-guide/permissions/#isauthenticatedorreadonly) - تمام کاربران می‌توانند هر صفحه‌ای را ببینند، اما تنها کاربران احراز هویت شده اجازه نوشتن، ویرایش، یا حذف را خواهند داشت
  
پیاده‌سازی هر کدام از این چهار مورد مستلزم به بروزرسانی  `DEFAULT_PERMISSION_CLASSES` و رفرش صفحه‌ی مرورگر می‌باشد. همین!
  
بیایید تنظیمات را روی IsAuthenticated  قرار دهیم تا تنها کاربران احراز هویت و لاگین شده بتوانند صفحه API را ببینند.
فایل config/settings.py را به صورت زیر آپدیت کنید. 
  
<div dir="ltr">
  
Code
  
```python
# config/settings.py
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated', # new
    ]
}
```
  
</div>  

حال به فایل posts/views.py بروید و تغییرات مجوزهایی که همین الان انجام دادیم را حذف کنید.  
  
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
  
اگر صفحات لیست پست‌ها و جزئیات  APIرا  رفرش کنید، هنوز کد وضعیت ۴۰۳ را خواهید دید.    
حال همه‌ي کاربران برای دسترسی به API نیاز به احراز هویت دارند، اما همیشه می‌توانیم تغییراتی اضافی در سطح نما در صورت نیاز ایجاد کنیم.
  
### مجوزهای سفارشی
  
نوبت به اولین مجوز سفارشی ما می‌باشد. به عنوان خلاصه‌ای از جایی که الان هستیم: دو کاربر داریم، testuser و دیگری superuser. یک پست وبلاگ در پایگاه داده ما وجود دارد، که توسط کاربر superuser ساخته شده‌است.
  
می‌خواهیم تنها نویسنده آن پست مشخص اجازه دسترسی به ویرایش و یا حذف آن را داشته باشد، در غیر اینصورت پست وبلاگ باید از نوع فقط  خواندنی باشد. بنابراین حساب superuser باید دسترسی کامل به عملیات CRUD برای هر پست داشته باشد، اما کاربر معمولی یا همان testuser نباید این مجوزها را داشته‌باشد.   
  
سرور محلی را با دکمه‌های ترکیبی Control+c متوقف کرده و یک فایل جدید در مسیر اپلیکیشن posts به نام permissions.py ایجاد کنید.  
  
خط فرمان 
  
``` (blogapi) $ touch posts/permissions.py ```

به صورت پیش‌فرض `Django REST Framework` متکی بر کلاس BasePermission می‌باشد که تمام دیگر کلاس‌های مربوط به مجوزها از آن ارث‌بری می‌کنند. این به این معنی است که تنظیمات مجوزهای از پیش ساخته شده‌ای مثل `AllowAny` و `IsAuthenticated` و سایر تنظیمات آن را گسترش می‌دهند. سورس کد اصلی آن 
[در گیت‌هاب در دسترس است ](https://github.com/encode/django-rest-framework). 
  
<div dir="ltr">
  
کد
  
```python
class BasePermission(object):
    """
    A base class from which all permission classes should inherit.
    """
  
    def has_permission(self, request, view):
        """
        Return `True` if permission is granted, `False` otherwise.
        """
        return True
  
    def has_object_permission(self, request, view, obj):
        """
        Return `True` if permission is granted, `False` otherwise.
        """
        return True
```
  
</div>
  
برای ساخت مجوز سفارشی شده خود، ما تابع `has_object_permission` را اورراید(یا باطل) می‌کنیم
به طور مشخص می‌خواهیم مجوز فقط خواندنی را به همه درخواست‌ها بدهیم اما برای هر درخواست نوشتن، مانند ویرایش یا حذف، نویسنده باید همانی باشد که به سایت وارد شده و لاگین کرده‌است.  
در اینجا فایل `posts/permissions.py` ما به صورت زیر می‌باشد.  
  
<div dir="ltr">
  
کد
  
```python
# posts/permissions.py
from rest_framework import permissions
  
  
class IsAuthorOrReadOnly(permissions.BasePermission):
  
def has_object_permission(self, request, view, obj):
    # Read-only permissions are allowed for any request
    if request.method in permissions.SAFE_METHODS:
        return True
  
    # Write permissions are only allowed to the author of a post
    return obj.author == request.user
```
 
</div>  

ابتدا کلاس `permissions` را وارد کرده‌ایم و کلاس خود را `IsAuthorOrReadOnly` که کلاس BasePermissions را گسترش می‌دهد و از آن ارث‌بری می‌کند را ساخته‌ایم. سپس تابع has_object_permission را اورراید کرده‌ایم. اگر در خواست شامل افعال HTTP شامل متدهای امن که یک تاپل شامل GET و OPTIONS و HEAD، باشد پس از نوع درخواست فقط خواندنی است و مجوز مربوط داده می‌شود.
  
در غیراینصورت درخواست برای نوشتن از هر نوعی می‌باشد، که به معنی بروزرسانی منابع API بنابراین ایجاد، حذف و یا ویرایش می‌باشد. در این مورد، بررسی می‌کنیم که نویسنده‌ای که در شیء(آبجکت) درخواست وجود دارد، که همان `obj.author` پست وبلاگ می‌باشد، با همان کاربری که درخواست را ارسال کرده مطابقت دارد.
  
در فایل `views.py`‌باید `IsAuthorOrReadOnly` را وارد کنیم و سپس می‌توانیم برای نمای جزئیات پست(PostDetail) permission_classes را اضافه کنیم.  
  
<div dir="ltr">
  
کد
  
```python
# posts/views.py
from rest_framework import generics
from .models import Post
from .permissions import IsAuthorOrReadOnly # new
from .serializers import PostSerializer
  
  
class PostList(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
  
  
class PostDetail(generics.RetrieveUpdateDestroyAPIView):
    permission_classes = (IsAuthorOrReadOnly,) # new
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```
  
</div>  

و کار ما به پایان رسید، حالا بیایید مجوزها را آزمایش کنیم. به  صفحه جزئیات پست که در آدرس http://127.0.0.1:8000/api/v1/1/ می‌باشد، بروید. مطمئن شوید که به عنوان کاربر superuser که نویسنده پست می‌باشد،  وارد شده‌باشید. نام‌کاربری باید در قسمت سمت راست بالای صفحه مشخص باشد

![API Detail Superuser](images/10.jpg)

اگرچه، اگر خارج شوید و با کاربر testuser وارد شوید، صفحه تغییر می‌کند.

![API Detail Testuser](images/11.jpg)

ما ***می‌توانیم*** این صفحه را ببینیم زیرا دسترسی‌های فقط خواندنی مجاز هستند. اگرچه به دلیل کلاس مجوز `IsAuthorOrReadOnly ` که ساختیم  ***نمی‌توانیم*** درخواست‌هایی نظیر PUT , DELETE بفرستیم.
  
دقت کنید که نماهای عمومی تنها مجوزهای در سطح آبجکت را برای نماهایی که تنها یک مدل نمونه را بازمی‌گردانند،بررسی می‌کنند. اگر به فیلتر سطح آبجکت برای نماهایی که لیستی از نمونه‌ها را بازمی‌گردانند نیاز دارید، نیاز به فیلتر با [اووراید کردن کوئری‌ست اولیه ](https://www.django-rest-framework.org/api-guide/filtering/#overriding-the-initial-queryset) دارید.   

### نتیجه‌گیری

اعمال تنظیمات مناسب بخش مهمی از هر API می‌باشد. به عنوان یک استراتژی عمومی، ایده‌ی خوبی است که از مجوزهای سختگیرانه سطح  پروژه استفاده کنید تا فقط کاربران احراز هویت شده دسترسی به API داشته باشند. سپس در صورت نیاز برای نماهای مختلف API مجوزهای در سطح نما یا مجوزهای سفارشی شده خود را ایجاد کنید.
  
  
</div> 
