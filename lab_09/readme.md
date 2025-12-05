# Aplikacje WWW, semestr 2025Z

## 1. Szablony stron w Django.

Framework Django dostarcza mechanizmu szablonów, dzięki któremu tworzenie modułowych stron opartych o HTML i CSS jest dużo prostsze niż tworzenie ich oddzielnie dla każdego z widoków w aplikacji. Wyobraź to sobie jako szablon, który definiuje różne obszary na naszej stronie np. nagłówek, gdzie znajduje się logo, może nawigacja. Poniżej możemy mieć dodatkowe obszary np. w układzie kolumnowym albo kafelki. Odwiedzając kolejne podstrony cała strona zazwyczaj nie zmienia się diametralnie, ale jest podmieniana tylko treść w niektórych obszarach. To znacznie ułatwia zarządzanie samym szablonem, gdyż zmniejsza ilość pracy w celu zmiany głównego szablonu dla całego serwisu, ale również ułatwia definicję kolejnych widoków i treści, która ma zostać podmieniona tylko w wybranych obszarach szablonu.

Cały proces od przygotowania szablonu (-ów) do finalnego wyświetlenia jednego z widoków w ramach tego szablonu będzie wymagał sporo pracy, więc zostanie opisany w kolejnych krokach.

**Krok 1 - Przygotowanie odpowiedniej struktury folderów i plików w projekcie.**

Najpierw przygotujemy strukturę katalogów oraz kilka pustych plików HTML, w których umieścimy szablon oraz póki co dwie podstrony. Struktur została zaprezentowana poniżej.


```console

blog
    - posts
        - templates
            - posts
                - category
                    - detail.html
                    - list.html 
                - base.html
```

**Krok 2 - stworzenie szablonu bazowego.**

W pliku `base.html` umieszczamy poniższą zawartość.

_**Listing 1**_
```html
<!DOCTYPE html>
<html>
    <head>
        <title>{% block title %}{% endblock %}</title>
    </head>
    <body>
        <div id="header" >
            Aplikacja blog/posts
        </div>
        <div id="content">
            {% block content %}
            {% endblock %}
        </div>
        <div id="sidebar">
            <p>Panel boczny</p>
        </div>
    </body>
</html>
```

Znaczniki `{}`, które znajdują się w treści szablonu są specjalnymi znacznikami silnika szablonów Django i oznaczają:
* `{% znacznik %}` - znacznik szablonu
* `{{ zmienna }}` - zmienna szablonu
* `{{ zmienna|filtr }}` - filtr zmiennej

W szablonie powyżej zdefiniowane zostały bloki `title` oraz `content`, których zawartość będziemy mogli podmieniać w poszczególnych podwidokach nie zmieniając całej reszty szablonu. Aktualnie nasz szablon nie korzysta z żadnego pliku ze stylami, zajmiemy się tym później.

**Krok 3 - przygotowanie widoku dla listy obiektów `Category`.**

Teraz umieszczamy poniższą treść w pliku `templates\posts\category\list.html`.

_**Listing 2**_
```html
{% extends "posts\base.html" %}

{% block title %}Lista obiektów Category{% endblock %}

{% block content %}
Lista kategorii:
{% for category in categories %}
<p>Nazwa: {{ category.name }}</p>
<p>Opis: {{ category.description }}</p>
<hr>
{% endfor %}
{% endblock %}
```

Kilka słów wyjaśnień do powyższego pliku.  
Znacznik `{% extends "posts\base.html" %}` informuje silnik szablonów o tym, że ten plik rozszerza szablon wskazany w ścieżce, tzn. że jeżeli zdefiniujemy w nim znaczniki bloków o tej samej nazwie co w szablonie, zostaną one podmienione na zawartość bloków zdefiniowanych tutaj jeszcze przed wyświetleniem w przeglądarce. Dzięki temu możemy zamieniać w poszczególnych widokach tylko wybrane fragmenty.  
Znacznik `{% for category in categories %}` oznacza uruchomienie pętli `for` przed wyświetleniem szablonu, która oczekuje istnienia w przestrzeni tego szablonu (przekażemy ją w definicji widoku) zmiennej `categories`, którą można iterować (przechodzić po jej elementach), gdzie przy każdym przejściu pobierany będzie jeden obiekt typu `Category` i zapisywany do zmiennej lokalnej `category` tej pętli i powtarzane będą kolejne linie, aż do znacznika `{% endfor %}`.  
Zmienne takie jak `{{ category.name }}` odwołują się do własności pojedynczego obiektu `category` i w szablonie zostaną zamienione na ich postać łańcuchową (w uproszczeniu tekstową, tak jakbyśmy na każdej z nich wywołali funkję `print()`, a właściwie `__str()__`). Wszystko co zdefiniowane w pętli, zostanie wywołane i powielone tyle razy ile obiektów znajdzie się w zmiennej `categories`.

**Krok 4 - zmiana definicji widoków.**

Teraz musimy zmienić nieco definicję widoków, tak aby korzystały z odpowiednich szablonów, które stworzyliśmy. W pliku `posts\views.py` zmieniamy definicję widoku funkcyjnego `category_view` na poniższy.

_**Listing 3**_
```python
# dodajemy brakujący import
from django.shortcuts import render

def category_list(request):
    # pobieramy wszystkie obiekty Person z bazy poprzez QuerySet
    categories = Category.objects.all()

    return render(request,
                  "posts/category/list.html",
                  {'categories': categories})
```


Widok na pewno sam w sobie nie jest imponujący pod względem wizualnym, ale wiemy już jak tworzyć proste szablony i przekazywać do nich dane pobrane z naszej bazy. 

Na koniec dokonamy jeszcze modyfikacji pliku z szablonem, aby możliwe było dodanie styli CSS i zmiana wyglądu strony.

Modyfikacja pliku `base.html`.

_**Listing 4**_
```html
{% load static %}
<!DOCTYPE html>
<html>
    <head>
        <title>{% block title %}{% endblock %}</title>
        <link href="{% static "css/posts.css" %}" rel="stylesheet">
    </head>
    <body>
        <div id="header" >
            Aplikacja blog/posts
        </div>
        <div id="content">
            {% block content %}
            {% endblock %}
        </div>
        <div id="sidebar">
            <p>Panel boczny</p>
        </div>
    </body>
</html>
```

Dodaliśmy mechanizm wczytywania plików statycznych przez Django, które to mogą być plikami ze stylami, obrazami, dołączanymi bibliotekami JavaScript.

Teraz stworzymy jeszcze wymieniony w nagłówku szablonu plik `posts.css`. Obsługa plików statycznych przez Django odbywa się w specyficzny sposób i domyślnie powinny być umieszczane w folderze `static` w każdej aplikacji z osobna. Tworzymy więc folder `static` a w nim folder `css`, w którym umieszczamy plik `posts.css`.  
Uruchom ponownie serwer i przejdź na widok listy wszystkich obiektów `Category` i sprawdź w konsoli, gdzie uruchomiony jest serwer czy nie pojawia się błąd 404 protokołu HTTP (NOT FOUND) przy ścieżce do pliku css np. tak 
```console
[28/Nov/2025 21:16:47] "GET /static/css/post.css HTTP/1.1" 404 1798
```

Może to oznaczać błąd w zdefiniowanej ścieżce.

Teraz przygotujemy widok dla pojedynczego obiektu typu `Category`, którego defininicja będzie wymagała przekazywania przez adres URL wartości parametru `id`, dla którego konkretny obiekt `Category` zostanie pobrany z bazy danych.

_**Listing 5**_  

Zawartość pliku `template\category\detail.html`
```html
{% extends "posts\base.html" %}

{% block title %}{{ category.name }}{% endblock %}

{% block content %}
<p>Nazwa: {{ category.name }}</p>
<p>Opis: {{ category.description }}</p>
{% endblock %}
```

Teraz dodanie widoku w pliku `posts\views.py`

_**Listing 6**_
```python
def category_detail(request, id):
    # pobieramy konkretny obiekt Category
    category = Category.objects.get(id=id)

    return render(request,
                  "posts/category/detail.html",
                  {'category': category})
```

I dodajemy mapowanie URL na nowy widok w pliku `posts\urls.py`

_**Listing 7**_

```python
from django.urls import path

from . import views

urlpatterns = [
    path("welcome", views.welcome_view),
    path("categories", views.category_list),
    path("category/<int:id>", views.category_detail),
]
```

Definicja `"category/<int:id>"` oznacza, że adres pasujący do schematu `.../category/liczba_całkowita` będzie mapowany na widok `category_detail`. Jest to jeden ze sposobów przekazywania wartości parametrów do naszej aplikacji.

Teraz w przeglądarce po wpisaniu adresu http://127.0.0.1:8000/posts/category/1 powinniśmy zobaczyć widok dla obiektu typu `Category` o id=1 o ile istnieje w bazie. Jeżeli obiekt o podanym id nie znajduje się w bazie zgłoszony zostanie wyjątek `models.ModelNotFound`, który nieobsłużony da nam dość jasną odpowiedź co jest nie tak (o ile pracujemy z włączonym trybem debug).

Możemy temu zaradzić dodając obsługę takiej ewentualności. Zmodyfikowana postać widoku `category_detail`:

_**Listing 14**_

```python
# dodajemy brakujący import na początku pliku (modyfikacja)
from django.http import Http404, HttpResponse

def category_detail(request, id):
    # pobieramy konkretny obiekt Category
    try:
        category = Category.objects.get(id=id)
    except Category.DoesNotExist:
        raise Http404("Obiekt Category o podanym id nie istnieje")

    return render(request,
                  "posts/category/detail.html",
                  {'category': category})
```

**Ćwiczenia do samodzielnego wykonania**

**Zadanie 1**

W ramach tego zadania stwórz:
* widok listy oraz detali dla modelu `Topic` oraz `Post`

**Zadanie 2**

Stwórz widok, który będzie wyświetlał wszystkie posty dla danego tematu (`Topic`) w kolejności od najnowszych do najstarszych (pomińmy fakt, że tu mogłaby się przydać paginacja).

**Zadanie 3**

Dodatkowo w pliku `posts.css` zdefiniuj style, które nadadzą stronie trochę bardziej atrkacyjny widok.