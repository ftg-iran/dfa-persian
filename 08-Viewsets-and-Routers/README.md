<div dir="rtl">


# ویوست و روتر ها Viewsets and Routers 

[ویوست‌ها (Viewsets)](https://www.django-rest-framework.org/api-guide/viewsets/) و [روتر ها routers](https://www.django-rest-framework.org/api-guide/routers/) 
ابزارهایی در Django REST Framework هستند که می‌توانند سرعت توسعه API را افزایش دهند.
آن‌ها یک لایه اضافی از انتزاع در بالای ویوها و URL ها هستند. مزیت اصلی, این است که یک viewset می‌تواند جایگزین چندین view مرتبط شود. 
و یک router میتواند به طور خودکار url برای تویعه‌دهنده‌ایجاد کند. در پروژه‌های بزرگتر با endpoint زیاد، این بدان معناست که یک توسعه‌دهنده باید کد کمتری بنویسد.
همچنین, مسلماً برای یک توسعه‌دهنده با تجربه, درک و استدلال در مورد تعداد کمی از viewset و ترکیبات router آسان‌تر از یک لیست طولانی از view ها و URL ها است.  

 
در این فصل ما دو API endpoint جدید را به پروژه فعلی خود اضافه خواهیم کرد و خواهیم دید که چگونه تغییر از
view ها و URL ها به viewset ها و router ها می‌تواند به همان عملکرد با کد بسیار کمتر دست یافت.
  
  

### اندپوینت‌های کاربر User endpoints
  
در حال حاضر ما API endpoit های زیر را در پروژه خود داریم. همه آن‌ها با پیشوند `api/v1/` هستند که برای اختصار نشان داده نشده است:

<div dir="ltr">
    
Diagram
```code
|Endpoint                              |HTTP Verb|
|--------------------------------------|---------|
|/                                     |GET      |
|/:pk/                                 |GET      |
|/rest-auth/registration               |POST     |
|/rest-auth/login                      |POST     |
|/rest-auth/logout                     |GET      |
|/rest-auth/password/reset             |POST     |
|/rest-auth/password/reset/confirm     |POST     |

```

</div>


دو endpoint توسط ما ایجاد شد در حالی که dj-rest-auth پنج تای دیگر را ارائه می‌دهد . بیاید اکنون
دو endpoint دیگر برای فهرست کردن همه و تک تک کاربران اضافه کنیم . این یک ویژگی مشترک در بسیاری
از api ها است که واضح تر می‌کند view ها و URL ها به viewset ها و router ها می‌تواند منطقی باشد.

جنگو یک مدل کلاس کاربر داخلی (User) مرسوم دارد که در فصل قبل برای احراز هویت از آن استفاده کرده‌ایم. 
بنابراین نیازی به ایجاد مدل جدید در دیتابیس نداریم.
در عوض ما فقط باید endpoint های جدید را سیم کشی کنیم . این فرآیند همیشه شامل سه مرحله زیر است:

- کلاس serializer جدید برای مدل
- view های جدید برای هر endpoint
- مسیرهای URL جدید برای هر endpoint

با serializer شروع کنید. ما نیاز داریم مدل User را import کنیم و یک کلاس UserSerializer
ایجاد کنیم که از آن استفاده کند. سپس آن را به فایل `posts/serializers.py` موجود خود اضافه می‌کنیم.

    
<div dir="ltr">
    
code
```python
# posts/serializers.py
from django.contrib.auth import get_user_model # new
from rest_framework import serializers
from .models import Post


class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ('id', 'author', 'title', 'body', 'created_at',)


class UserSerializer(serializers.ModelSerializer): # new
    class Meta:
      model = get_user_model()
      fields = ('id', 'username',)
```

</div>
    
 
شایان ذکر است که در حالی که ما از get_user_model برای ارجاع به مدل User در اینجا استفاده کرده‌ایم ,
در واقع [سه راه مختلف برای ارجاع](https://docs.djangoproject.com/en/3.1/topics/auth/customizing/#referencing-the-user-model) 
به مدل User در جنگو وجود دارد.

با استفاده از get_user_model اطمینان حاصل می‌کنیم که به مدل User صحیح اشاره می‌کنیم,
چه User پیش‌فرض باشد یا یک مدل [User سفارشی](https://docs.djangoproject.com/en/3.1/topics/auth/customizing/#specifying-a-custom-user-model)
همانطور که اغلب در پروژه‌های جنگو جدید تعریف می‌شود.
 
 

 
 
در ادامه باید برای هر endpoint نماها(views) را تعریف کنیم. ابتدا `UserSerializer` را به لیست import ها اضافه کنید.
سپس هم یک کلاس UserList ایجاد کنید که همه
کاربران را فهرست می‌کند و هم یک کلاس UserDetails که نمای جزئیات یک کاربر را ارائه می‌دهد.
درست مانند view های پست خود، می‌توانیم از `ListCreateAPIView` و `RetrieveUpdateDestroyAPIView` در اینجا استفاده کنیم. ما همچنین نیاز به ارجاع به
مدل کاربران از طریق get_user_model داریم بنابراین در خط بالا import می‌شود.

 
<div dir="ltr">
    
    
code
```python
# posts/views.py
from django.contrib.auth import get_user_model # new
from rest_framework import generics
from .models import Post
from .permissions import IsAuthorOrReadOnly
from .serializers import PostSerializer, UserSerializer # new



class PostList(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer


class PostDetail(generics.RetrieveUpdateDestroyAPIView):
    permission_classes = (IsAuthorOrReadOnly,)
    queryset = Post.objects.all()
    serializer_class = PostSerializer


class UserList(generics.ListCreateAPIView): # new
    queryset = get_user_model().objects.all()
    serializer_class = UserSerializer


class UserDetail(generics.RetrieveUpdateDestroyAPIView): # new
    queryset = get_user_model().objects.all()
    serializer_class = UserSerializer

```

</div>

اگر متوجه شده باشید، در اینجا کمی تکرار وجود دارد. هم view های پست و هم view های کاربر queryset و serializer_class 
یکسانی دارند. شاید بتوان این را به نوعی برای ذخیره کد ترکیب کرد؟

در نهایت ما مسیرهای URL خود را داریم. اطمینان حاصل کنید که view های UserList و UserDetail را import کرده‌اید.
سپس می‌توانیم از پیشوند users/ برای هر کدام استفاده کنیم.


<div dir="ltr">
    
code
```python
# posts/urls.py
from django.urls import path
from .views import UserList, UserDetail, PostList, PostDetail # new


urlpatterns = [
    path('users/', UserList.as_view()), # new
    path('users/<int:pk>/', UserDetail.as_view()), # new
    path('', PostList.as_view()),
    path('<int:pk>/', PostDetail.as_view()),
]

```

</div>

و ما تمام کردیم. اطمینان حاصل کنید که سرور محلی همچنان در حال اجرا است و به 
API قابل مرور (browsable API) بروید تا تأیید کنید همه چیز همانطور که انتظار می رود کار می‌کند.

لیست endpoint کاربران ما در http://127.0.0.1:8000/api/v1/users/ قرار دارد.

![API Users List](images/1.png)

کد وضعیت (status code) برابر 200 OK است که به این معنی است که همه چیز کار می‌کند. ما می‌توانیم سه کاربر موجود خود را ببینیم.
 
نقطه پایانی (endpint) جزئیات کاربر در کلید اصلی برای هر کاربر در دسترس است. بنابراین حساب کاربری superuser
ما در آدرس زیر قرار دارد: http://127.0.0.1:8000/api/v1/users/1/.
 
![API User Instance](images/2.png)


### ویوست‌ها Viewsets

ویوست‌ها راهی برای ترکیب منطق برای چندین view مرتبط در یک کلاس واحد است. به عبارت دیگر،
یک ویوست می‌تواند جایگزین چندین ویو شود. در حال حاضر ما چهار ویو (view) داریم: دو مورد برای پست‌های وبلاگ و دو مورد برای کاربران. 
در عوض می‌توانیم عملکرد یکسانی را با دو ویوست تقلید کنیم: یکی برای پست‌های وبلاگ و دیگری برای کاربران.

معامله این است که کمبود خوانایی برای اشخاص توسعه‌دهنده که با ویوست‌ها آشنایی ندارند وجود دارد.  بنابراین این یک معامله است.
 
 
اینجاست که کد در فایل آپدیت شده ی ما `posts/views.py` ظاهر می‌شود وقتی که ویو ست هارا با هم عوض می‌کنیم.
 

<div dir="ltr">
    

code
```python
# posts/views.py
from django.contrib.auth import get_user_model
from rest_framework import viewsets # new
from .models import Post
from .permissions import IsAuthorOrReadOnly
from .serializers import PostSerializer, UserSerializer


class PostViewSet(viewsets.ModelViewSet): # new
    permission_classes = (IsAuthorOrReadOnly,)
    queryset = Post.objects.all()
    serializer_class = PostSerializer


class UserViewSet(viewsets.ModelViewSet): # new
    queryset = get_user_model().objects.all()
    serializer_class = UserSerializer
```

</div>
    
    
در بالا به جای import کردن generics از rest_framework، اکنون در حال import کردن viewsets در خط دوم هستیم.
سپس ما از [ModelViewSet](http://www.django-rest-framework.org/api-guide/viewsets/#modelviewset) استفاده می‌کنیم  که هم ویو لیست و هم ویو جزئیات
را برای ما فراهم می‌کند . و دیگر مجبور نیستیم همان `queryset` و `serializer_class` را برای هر view تکرار کنیم.

در این مرحله، وب سرور محلی متوقف می‌شود زیرا جنگو از عدم وجود مسیرهای URL مربوطه شکایت می‌کند. بیایید آن‌هارا در کار بعدی تنظیم کنیم.



### روترها
[روترها](https://www.django-rest-framework.org/api-guide/routers/) مستقیما با ویوست‌ها کار می‌کنند که به صورت اتوماتیک url ها را برای ما تولید کنند. فایل posts/urls.py حال حاضر ما چهار الگوی url دارد: دو تا از urlها برای بلاگ پست‌ها و دوتای دیگر به یوزرها اختصاص دارد. در عوض ما می‌توانیم برای هر viewset یک مسیر(route) داشته باشیم. بنابراین به جای چهار مسیر(route) ما دو مسیر(route) داریم. به نظر بهتر میاد. درسته؟

جنگو رست فریمورک دو روتر دیفالت و پیشفرض دارد: [SimpleRouter](https://www.django-rest-framework.org/api-guide/routers/#simplerouter) و [DefaultRouter](https://www.django-rest-framework.org/api-guide/routers/#defaultrouter). ما در این کتاب از روتر ساده(SimpleRouter) استفاده خواهیم کرد همچنین امکان ساخت روترهای شخصی‌سازی شده برای عملکرد پیشرفته‌تر وجود دارد.
چیزی که قرار است در کد قدیمی آپدیت بشود چیزی شبیه به این است:    

<div dir="ltr">

    

```python
# posts/urls.py
from django.urls import path
from rest_framework.routers import SimpleRouter
from .views import UserViewSet, PostViewSet

router = SimpleRouter()
router.register('users', UserViewSet, basename='users')
router.register('', PostViewSet, basename='posts')

urlpatterns = router.urls
```
    
</div>
    
در خط بالا SimpleRouter همراه با ویوها ایمپورت شده است . روتر بر روی حالت SimpleRouter ست شده است و ما هر کدام از viewsetها برای یوزر و پست‌ها را ثبت(register) می‌کنیم. در نهایت ما URL های خود را برای استفاده از روترهای جدید تنظیم می‌کنیم. پیش میرویم و چهار اندپوینت خود را با دستور python manage.py runserver بررسی می‌کنیم.


![Image 3](images/3.png)

توجه داشته باشید که لیست کاربران یکسان است، اما view جزئیات کمی متفاوت است. این مورد به عنوان "User Instance"  به جای "User Detail" شناخته می‌شود و یک آپشن اضافه "delete" وجود دارد که در [ModelViewSet](https://www.django-rest-framework.org/api-guide/viewsets/#modelviewset) تعبیه شده است.


![Image 4](images/4.png)
امکان شخصی‌سازی viewsetها وجود دارد اما یک  بده و بستان (tradeoff) مهم در ازای نوشتن کد کمتر با viewset می‌باشد که تنظیمات پیش‌فرض ممکن است نیازمند یک سری پیکربندی اضافی برای رفع آنچیزی که می‌خواهید باشد. 

با رفتن به اندپویت لیست پست‌ها به آدرس http://127.0.0.1:8000/api/v1/ می‌توانیم ببینیم که با حالت قبل (بدون viewset) یکسان است. 

![Image 5](images/5.png)

و مورد مهم این که همچنان دسترسی ها و مجوزها کار می‌کند. وقتی که به عنوان کاربر testuser2 لاگین کنید آبجکت پست در آدرس http://127.0.0.1:8000/api/v1/1/ همچنان در حالت read-only می‌باشد.

![Image 6](images/6.png)

به هر حال اگر به عنوان کاربر superuser ما لاگین کنیم که تنها نویسنده بلاگ پست نیز هست ما تمام دسترسی ها از جمله خواندن نوشتن ویرایش کردن و حذف پست را داریم.  

![Image 7](images/7.png)


### نتیجه گیری
<span dir="ltr">Viewset</span> ها و روترها یک انتزاع قدرتمند هستند که مقدار کدهایی را که ما به عنوان توسعه‌دهندگان باید بنویسیم را کاهش می‌دهند به هر حال این مختصر بودن به قیمت یادگیری اولیه یک سری موارد تمام می‌شود اولین باری که از viewset ها و روترها به جای نماها و الگوهای URL استفاده می‌کنید، احساس عجیبی خواهد داشت.

در نهایت تصمیم گیری در مورد اینکه چه زمانی مجموعه ها و روترها را به پروژه خود اضافه کنید کاملاًً درونی است. یک قانون خوب این است که با نماها و URL ها شروع کنید. با افزایش پیچیدگی API شما، اگر متوجه شدید که اندپوینت یکسانی را بارها و بارها تکرار می‌کنید، به Viewset‌ها و روترها نگاه کنید. تا در آن زمان، همه چیز را ساده نگه دارید.

    

</div>
 
