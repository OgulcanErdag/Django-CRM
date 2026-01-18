DJANGO CRM with MySQL:

mkdir /c/dcrm
cd /c/dcrm
ls
python -m venv virt
ls
virt/
source virt/Scripts/activate
pip install django
pip install mysql

Django uygulamamizi MySQL uygulamamiza baglamak icin :

pip install mysql-connector-python
pip install mysql-connector

simdi mysql indirip kurmamiz gerekiyor ;

- https://dev.mysql.com/downloads/installer/

cd dcrm/ ==> User@anonymous-win MINGW64 /c/dcrm/dcrm
python manage.py startapp website

dcrm dosyasini bul projeyi ac ve burayi bu sekilde degistir ==>

DATABASES = {
'default': {
'ENGINE': 'django.db.backends.mysql',
'NAME': 'elderco',
'USER': 'root',
'PASSWORD': 'password123',
'HOST': 'localhost',
'PORT': '3306'
}
}

CTRL + S yaptiktan sonra tekrar terminale dön..

User@anonymous-win MINGW64 /c/dcrm/dcrm
bu dizinde ve virt acik oldugundan emin ol.

touch mydb.py

Simdi mysql connector u ice aktarmamiz gerekiyor ;

bu yuzden mydb.py in icine ;

import mysql.connector

dataBase = mysql.connector.connect(
host = 'localhost',
user = 'root',
passwd = 'password123'
)
#prepare a cursor object
cursorObject = dataBase.cursor()

# create a database

cursorObject.execute("CREATE DATABASE elderco")

print("All Done!")

ardindan python mydb.py komutunu calisitirizoruz database olusuturuluyor. MySQL workbench i acip database i kontrol edebiliriz.

Daha sonra ;

python manage.py migrate komutunu terminale yazmamiz gerekiyor cunku, butun verileri bu komut sayesinde database e aktarma islemi gerceklesiyor.

MySQL Workbench e gidip migrate sayesinde tables icine bütün Django ogelerini gorüntüleyebiliriz.

Simdi bir kullanici olusturalim ;

winpty python manage.py createsuperuser

ardindan Server i calistiralim

python manage.py runserver

ve bu asamada projemizi kurduk. Baslangic ekrani geliyor ve MySQL veritabanimiz da kurduk ve bagladik.

Simdi gelistirmeye baslamaya haziriz!!

Django Version Control!!!!

Baslamadan önce Git ve Github ile cok hizli bir sekilde sürüm kontrolünü kuralim, böylece kod ilerledikce kontrol edilebilir.

Terminale geri dönüyoruz;
git ayarlarini yaptiktan sonra Github baglantisi yapiyoruz.

DJANGO Build APP

ilk olarak dcrm klasörü altinda URLs.py dosyasi icerigi ile biraz oynamamiz gerekiyor..

Standard hali Boyle ;;

from django.contrib import admin
from django.urls import path

urlpatterns = [
path('admin/', admin.site.urls),
]

DEGISIKLIGI SU SEKILDE YAPACAGIZ ;

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
path('admin/', admin.site.urls),
path('', include(website.urls)),
]

daha sonra website kalsörü altina urls.py dosyasi olusuturuyoruz. Yeni bir ana sayfa olusturacagiz.
Django da bir web sayfasi olsuturmak istediginizde her zaman bir ana sayfa olusturulur.
3 adimli surec de
öncelikle, sablon dosyasini, yani HTML sayfasini olustururuz daha sonra URL olusutururuz ve son olarak View olustururuz.

Simdi öncelike URL olusturalim.
urls.py icine de ;

from django.urls import path
from . import views

urlpatterns = [
path('', views.home, name='home'),
]

yazalim.

simdi views.py olusturalim ;

from django.shortcuts import render

# bu koddaki amac request i döndürmek istiyoruz ve

# bunu da home.html ile yönlendirmek istiyoruz

def home(request): - return render(request, 'home.html', {})

simdi bir websitesi klasörü altinda bir templates klasörü olusturalim ve icine home.html dosyasi acalim. Django html leri templates klasörü icinde arayacagini biliyor.

home.html icine bir deneme yapalim <h1>Hello world!</h1> ve ardindan browser da görecegiz ki kodumuz calisiyor.

ardindan base.html dosyamizi olusturalim. base.html dosyasi sitemizdeki her web sayafasinin referans alacagi ve basliklari altbilgileri cekecegi dosyadir.

CSS icin bootstrap kullanacagiz

https://getbootstrap.com/

ve daha sonra bu kodu base.html icine ekleyecegiz ;

<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Django CRM</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-sRIl4kxILFvY47J16cr9ZwB07vP4J8+LH7qKQnuqkuIAvNWLzeN8tE5YBujZqJLB" crossorigin="anonymous">
  </head>
  <body>
    {% block content %}
    {% endblock %}
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js" integrity="sha384-FKyoEForCGlyvwx9Hj09JcYn3nv7wiPVlz7YYwJrWVcXK/BmnVDxM+D2scQbITxI" crossorigin="anonymous"></script>
  </body>
</html>

!!!! bura da dikkat etmemiz gerek kisim DJango etiketlerini kullanmak {% block content %} acilis etiketi ve kapanis etiketi olarak da {% endblock %}. Bu etiketleri web sayfamizda ki her seyi bu durumda sadece bu seyleri cekecek ve Django"nun icine yerlestirecektir.

Bu sebepten dolayi home.html e de ki kod yapisi su sekilde degistirilmelidir ;

{% extends 'base.html' %}

{% block content %}

<h1>Hello World!</h1>
{% endblock %}

navbar.html olusturuyoruz ve kodumuz su sekilde oluyor ;

<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container-fluid">
        <a class="navbar-brand" href="{%url 'home'%}">Django CRM</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent"
            aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">

                <li class="nav-item">
                    <a class="nav-link" href="#">Link</a>
                </li>

            </ul>
            </li>

            </ul>

        </div>
    </div>

</nav>

DJANGO LOGIN USERS

webiste klasörü icindeki views.py a geliyoruz. Django kimlik dogrulama sistemini uygulamamiza aktarmamiz gerekiyor. Bu yüzden views.py da en üste from django.contrib.auth import authenticate, login, logout yaziyoruz. Ve Django nun ekranda kucuk mesajlar gostermesini de istiyoruz. Ornegin biri giris yaptiginda "Giris yaptiniz" diyen kucuk bir mesah, cikis yaptiginda "Cikis yaptiniz, gorusuruz" diyen bir mesaj istiyoruz. bu Yuzden views.py da yine su kodu da from django.contrib import messages
ekliyoruz. ve fonksiyonumuzu yaziyoruz.

def home(request):
return render(request, 'home.html', {})

def login_user(request):
pass

def logout_user(request):
pass

daha soran urls.py a path('login/', views.login_user, name='login'),
path('logout/', views.logout_user, name='logout'), ekliyoruz.

home.html icerisinde ki degisikilerimiz soyle olacaktir;

{% extends 'base.html' %}

{% block content %}

<div class="col-md-6 offset-md-3">
    <h1>Login</h1>

    <form method="POST" action="{%url 'home' %}">
        {% csrf_token %}

    </form>

</div>
{% endblock %}

<h1> tag i Login olarak degistirdik ve form tagi icinde method a POST verdik cünkü birisi formu doldurup buttona tikladiginda bunu sunucuya gönderir. Ayrica buna bir eylem de verelim. Bu bir Django URL si olacak. Django ile bir form olusturdugunuzda bir CSRF belirtecine de ihtiyaciniz vardir. Bu siteler arasi istektir. Sahtecilik belirteci, formunuzun bilgisayar korsanlari tarafindan ee gecirilmesini önlemeye yardimci olur. Bu neden bu bir gereklilikdir.

Simdi Bootstrap sayfasindan Overview sekmesine gidelim ve ;

<form>
  <div class="mb-3">
    <label for="exampleInputEmail1" class="form-label">Email address</label>
    <input type="email" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
    <div id="emailHelp" class="form-text">We'll never share your email with anyone else.</div>
  </div>
  <div class="mb-3">
    <label for="exampleInputPassword1" class="form-label">Password</label>
    <input type="password" class="form-control" id="exampleInputPassword1">
  </div>
  <div class="mb-3 form-check">
    <input type="checkbox" class="form-check-input" id="exampleCheck1">
    <label class="form-check-label" for="exampleCheck1">Check me out</label>
  </div>
  <button type="submit" class="btn btn-primary">Submit</button>
</form> bu kodu CSRF kodumuzun altina yapistiralim.

home.html kodumuzu su sekilde gelistirilmelidir ;

{% extends 'base.html' %}

{% block content %}

<div class="col-md-6 offset-md-3">
    {%if user.is_authenticated %}
    <h1>Hello World</h1>

    {% else %}
    <h1>Login</h1>
    <br>
    <form method="POST" action="{%url 'home' %}">
        {% csrf_token %}
        <form>
            <div class="mb-3">

                <input type="text" class="form-control" name="username" placeholder="User Name" required>

            </div><br>
            <div class="mb-3">

                <input type="password" class="form-control" name="password" , placeholder="Password" required>
            </div>

            <button type="submit" class="btn btn-secondary">Login</button>
        </form>
    </form>

</div>
{% endif %}
{% endblock %}

daha sonra views.py a gidip def home fonsiyonumuzu gelistirelim
def home(request):

# Check to see if logging in

if request.method == 'POST':
username = request.POST['username']
password = request.POST['password']

# Authenticate

user = authenticate(request, username=username, password=password)
if user is not None:
login(request, user)
messages.success(request, "You have been logged in")
return redirect('home')
else:
messages.success(request, "There was an error logging in, please try again...")
return redirect('home')
return render(request, 'home.html', {})
