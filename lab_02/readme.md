# Aplikacje WWW, semestr 2025Z

## Lab 2
---

**UWAGA!** Pamiętaj, aby na początku zajęć rozpocząć pracę na nowym branchu repozytorium!

### **1. Narzędzia administracyjne frameworka Django.**

Django posiada narzędzie administracyjne w postaci skryptu `manage.py` oraz polecenia `django-admin` dzięki którym możemy wykonać wiele czynności administracyjnych, takich jak dodanie użytkownika administracyjnego, wykonanie migracji bazy danych, uruchomienie serwera i inne. Pełną listę poleceń można znaleźć w oficjalnej dokumentacji pod adresem: https://docs.djangoproject.com/pl/5.2/ref/django-admin/ oraz https://docs.djangoproject.com/en/5.2/ref/contrib/admin/

Dodatkowe i przydatne narzędzia można dodać poprzez instalację zewnętrznych modułów, np. [django-admin-tools](https://github.com/django-admin-tools/django-admin-tools) (**archiwalnie istniał problem z instalacją z Django 4.2**),[django-debug-toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/index.html) lub [django-extensions](https://pypi.org/project/django-extensions/).

Jeżeli w trakcie poprzednich zajęć wykonana została tylko pierwsza część oficjalnego tutoriala, to aby móc wykonać ćwiczenia i zadania dla bieżącego laboratorium należy wykonać poniższe czynności.

**Stworzenie schematu bazy danych**

>**Uwaga** Jak zostało już wspomniane na pierwszych zajęciach praca w trakcie zajęć powinn iść dwutorowo, tj. potrzebne są dwa projekty Django, jeden do wykonania zadań w trakcie zajęć oraz jako środowisko eksperymentalne, a drugi projekt Django to projekt na zaliczenie. Od studenta zależy czy będzie to jedno dobrze zorganizowane repozytorium czy dwa oddzielne, jednak zalecane są oddzielne. Będzie to również miało wpływ na przygotowanie odpowiedniej konfiguracji połączenia z bazą danych przed wykonaniem operacji `migrate` (patrz dokumentacja: https://docs.djangoproject.com/en/5.2/ref/databases/).
Istnieje również dość ryzykowna opcja polegająca na stworzeniu oddzielnych aplikacji wewnątrz projektu, z której jedna będzie aplikacją testową, a druga docelowym projektem, którymi dodatkowo można zarządzać z poziomu oddzielnych gałęzi repozytorium. Jest to jednak rozwiązanie dość problematyczne, a w szczególności wymagające dużej uwagi w trakcie przełączania między nimi.

Aby podłączyć bazę danych MySQL do projektu Django, należy wykonać kilka kroków, w tym instalację odpowiedniego sterownika, konfigurację projektu Django oraz utworzenie bazy danych.

Zalecane jest korzystać z bazy danych SQLite, gdyż nie będzie sprawiać to problemów w trakcie prezentacji projektu na zaliczenie.

Poniższa operacja stworzy schematy dla wszystkich zainstalowanych aplikacji (zmienna `INSTALLED_APPS` w pliku `settings.py`). Jeżeli nie są niezbędne to można wybrane linie zakomentować lub dodać na końcu polecenia nazwy aplikacji, dla których chcemy stworzyć schemat.

```console
# przygotowanie plików migracji
python manage.py makemigrations

# wykonanie migracji
python manage.py migrate
```

Warto przeczytać dokumentację dotyczącą tego polecenia, gdyż posiada ciekawe opcje, dość często potrzebne w trakcie początkowej pracy z modelem bazy danych i częstymi migracjami: https://docs.djangoproject.com/pl/5.2/ref/django-admin/#migrate


**Stworzenie superużytkownika**

```console
python manage.py createsuperuser
```

Ten użytkownik jest niezbędny, aby możliwe było zalogowanie się do panelu administracyjnego i wykonywanie w nim czynności administracyjnych.

Zaloguj się do panelu administracyjnego i sprawdź dostępne w nim opcje na tym etapie tworzenia aplikacji.
Adres panelu przy domyślnych ustawieniach to: http://127.0.0.1:8000/admin

### 2. Schemat bazy danych.

Każdy projekt potrzebuje jakiejś formy trwałego przechowywania choćby niewielkiej ilości danych, ale większość z nich przewiduje również powiększanie tych zbiorów przez aktywność użytkowników i rozwój aplikacji. Jeżeli nie dokonałeś jeszcze wyboru to zastanów się nad docelowym silnikiem bazy danych.

Tworzenie schematu bazy na potrzeby projektu Django można przeprowadzić na dwa sposoby.

**Sposób 1.**
Tworzymy odpowiednie implementacje klasy `django.db.models.Model` z API Django (patrz **listing 1**), a następnie poprzez migrację tworzymy schemat w docelowej bazie.

**Sposób 2**
Iżynieria wsteczna (ang. reverse engineering). Schemat bazy danych możemy przygotować bezpośrednio poprzez polecenia SQL lub narzędzia graficzne, a następnie poleceniem `manage.py inspectdb` wygenerować kod w języku Python zgodny z deklaracją, którą należy przygotować postępując sposobem pierwszym. Ten kod zostanie wyświetlony w konsoli i można strumień przekierować do pliku. Przykład poniżej.

```console
# dopisanie wyjścia polecenia po lewej stronie znaków >> do pliku
manage.py inspectdb >> models.py 
```

### 3. Modele w Django.

**UWAGA!**
Tworzymy w projekcie z labu numer 1 nową aplikację o nazwie `blog`, w której implementujemy kolejne przykłady zaprezentowane w materiałach.

> Dokumentacja: https://docs.djangoproject.com/pl/5.2/topics/db/models/

Definicje modeli dodajemy w pliku `blog/posts/models.py`.
Poniżej zaprezentowane zostaną dwa przykładowe modele, przygotowanie migracji, propagacje modeli na bazę danych oraz zarządzanie nimi z poziomu panelu administracyjnego.

__*Listing 1:*__
```python

class Category(models.Model):
    name = models.CharField(max_length=60)


class Topic(models.Model):
    name = models.CharField(max_length=60)
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    created = models.DateTimeField(auto_now_add=True)

```

Jeżeli chcemy aby mechanizm migracji wykrywał stworzone modele w naszej aplikacji i migrował je na bazę danych to należy dodać naszą aplikację do zmiennej `INSTALLED_APPS` w pliku `settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 'posts.apps.PostsConfig',
    # lub
    'posts'
]
```

Następnie trzeba przygotować plik migracji poleceniem `manage.py makemigrations`, który przeanalizuje kod modeli i pozwoli nam na tym etapie rozwiązać ewentualne problemy ze spójnością danych, błędami w kodzie, wartościami null i innymi problemami, którymi nie będziemy musieli zajmować się na etapie propagacji modeli na schemat bazy danych. 

Ostatni krok to wykonanie migracji poprzez polecenie `manage.py migrate`, które stworzy odpowiednie tabele w domyślnej bazie danych (zdefiniowana w pliku `settings.py`).

#### 3.1 Rejestracja modeli w panelu administracyjnym Django.

Po wykonaniu migracji modele nie będą domyślnie widoczne w panelu administracyjnym Django.
Należy je najpierw zarejestrować w pliku `admin.py` znajdującym się w folderze naszej aplikacji:

```python
# modele musimy zaimportować
from .models import Category, Topic

# a następnie zarejestrować (pokazano najprostszy przypadek)
admin.site.register(Category)
admin.site.register(Topic)
```

Odświeżając teraz widok w penelu administracyjnym powinniśmy mieć możliwość zarządzania obiektami zarejestrowanych modeli.


**ZADANIA**

1. Dodaj pole `description` do modelu `Category` o typie `TextField`.  Przygotuj plik migracji (`makemigrations`), a następnie wykonaj migrację i sprawdź czy zmiany zostały propagowane na bazę (np. poprzez panel administracyjny lub przeglądając strukturę bazy danych). Przejrzyj zawartość pliku migracji.
2. Za pomocą polecenia `showmigrations` wylistuj wszystkie migracje dla danej aplikacji, a następnie sprawdzając w dokumentacji działanie polecenia `migrate` wycofaj ostatnią migrację.
3. Zainstaluj pakiet `django-debug-toolbar` i skonfiguruj go (link na początku tego pliku z materiałami).
4. Poprzez panel administracyjny przetestuj dodanie, modyfikację i usunięcie instancji modeli `Category` oraz `Topic`. Przeanalizuj co się dzieje w aplikacji poprzez `django debug toolbar`.
5. Zatwierdź niezbędne zmiany w repozytorium i wypchnij je do zdalnego repozytorium. Jeżeli nie zostało to jeszcze wykonane, należy dodać prowadzącego jako kolaboratora repozytorium w celu łatwiejszego dostepu do kodu projektu rozwijanego na zajęciach w celach archiwizacji oraz ewentualnej pomocy w rozwiązywaniu problemów w kodzie.


### Dodatkowe narzędzia i materiały.

* Do obsługi wielu źródeł baz danych może przydać się darmowe narzędzie o nazwie DBeaver, które posiada również możliwość tworzenia schematów bazy danych a następnie generowania poleceń SQL z takiego schematu. Więcej: https://dbeaver.com/2022/07/07/forward-engineering-with-erd/
