# Aplikacje WWW, semestr 2022Z

## Lab 2
---

### **1. Narzędzia administracyjne frameworka Django.**

Django posiada narzędzie administracyjne w postaci skryptu `manage.py` oraz polecenia `django-admin` dzięki którym możemy wykonać wiele czynności administracyjnych, takich jak dodanie użytkownika administracyjnego, wykonanie migracji bazy danych, uruchomienie serwera i inne. Pełną listę poleceń można znaleźć w oficjalnej dokumentacji pod adresem: https://docs.djangoproject.com/pl/4.1/ref/django-admin/

Dodatkowe i przydatne narzędzia można dodać poprzez instalację zewnętrznych modułów, np. [django-admin-tools](https://github.com/django-admin-tools/django-admin-tools) (**aktualnie istnieje problem z instalacją z Django 4.1**),[django-debug-toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/) lub [django-extensions](https://pypi.org/project/django-extensions/).

Jeżeli w trakcie poprzednich zajęć wykonana została tylko pierwsza część oficjalnego tutoriala, to aby móc wykonać ćwiczenia i zadania dla bieżącego laboratorium należy wykonać poniższe czynności.

**Stworzenie schematu bazy danych**

>**Uwaga** Jak zostało już wspomniane na pierwszych zajęciach praca w trakcie zajęć powinn iść dwutorowo, tj. potrzebne są dwa projekty Django, jeden do wykonania zadań w trakcie zadań oraz jako środowisko eksperymentalne, a drugi projekt to projekt za zaliczenie. Od studenta zależy czy będzie to jedno dobrze zorganizowane repozytorium czy dwa oddzielne. Będzie to również miało wpływ na przygotowanie odpowiedniej konfiguracji połączenia z bazą danych przed wykonaniem operacji migrate (patrz dokumentacja: https://docs.djangoproject.com/en/4.1/ref/databases/).
Istnieje również dość ryzykowna opcja polegająca na stworzeniu oddzielnych aplikacji wewnątrz projektu, z której jedna będzie aplikacją testową a druga docelowym projektem, którymi dodatkowo można zarządzać z poziomu oddzielnych gałęzi repozytorium. Jest to jednak rozwiązanie dość problematyczne, a w szczególności wymagające dużej uwagi w trakcie przełączania między nimi.

Poniższa operacja stworzy schematy dla wszystkich zainstalowanych aplikacji (zmienna `INSTALLED_APPS` w pliku `settings.py`). Jeżeli nie są niezbędne to można wybrane linie zakomentować lub dodać na końcu polecenia nazwy aplikacji, dla których chcemy stworzyć schemat.

```console
python manage.py migrate
```

Warto przeczytać dokumentację dotyczącą tego polecenia, gdyż posiada ciekawe opcje, dość często potrzebne w trakcie początkowej pracy z modelem bazy danych i częstymi migracjami: https://docs.djangoproject.com/pl/4.1/ref/django-admin/#migrate


**Stworzenie superużytkownika**

```console
python manage.py createsuperuser
```

Ten użytkownik jest niezbędny, aby możliwe było zalogowanie się do panelu administracyjnego i wykonywanie w nim czynności administracyjnych.

Zaloguj się do panelu administracyjnego i sprawdź dostępne w nim opcje na tym etapie tworzenia aplikacji.


### 2. Schemat bazy danych.

Każdy projekt potrzebuje jakiejś formy trwałego przechowywania choćby niewielkiej ilości danych, ale większość z nich przewiduje również powiększanie tych zbiorów przez aktywność użytkowników i rozwój aplikacji. Jeżeli nie dokonałeś jeszcze wyboru to zastanów się nad docelowym silnikiem bazy danych. Poinformuj prowadzącego jeżeli zamierzasz korzystać z bazy MySQL hostowanej na serwerze uczelnianym.

Tworzenie schematu bazy na potrzeby projektu Django można przeprowadzić na dwa sposoby.

**Sposób 1.**
Tworzymy odpowiednie implementacje klasy `model` z API Django a następnie poprzez migrację tworzymy schemat w docelowej bazie.

**Sposób 2**
Iżynieria wsteczna (ang. reverse engineering). Schemat bazy danych możemy przygotować bezpośrednio poprzez polecenia SQL lub narzędzia graficzne, a następnie poleceniem `inspectdb` wygenerować kod w języku Python zgodny z deklaracją, którą należy przygotować postępując sposobem pierwszym.

**ZADANIA**

1. Zastanów się nad sposobem organizacji swojego kodu dla aplikacji testowej (zajęcia) oraz projektu docelowego i przygotuj środowisko w odpowiedni sposób.
2. Korzystając z odpowiedniego oprogramowania (MySQl Workbench, pgAdmin, inne) przygotuj pierwszą wersję schematu bazy danych dla swojego projektu i utwórz ten schemat bezpośrednio w bazie danych.
3. Za pomocą polecenia `inspectdb` narzędzia administracyjnego Django stwórz plik z modelami z bazy i umieść kod w pliku models.py projektu. Sprawdź czy powinieneś wykonać dodatkowe czynności (edycja, pliki migracji?) po tej operacji.
4. Przeanalizuj plik z kodem modeli i dodaj/zmodyfikuj jedno z pól modelu. Przygotuj plik migracji (`makemigrations`) a następnie wykonaj migrację i sprawdź czy zmiany zostały propagowane na bazę. Przeglądnij zawartość pliku migracji.
5. Za pomocą polecenia `showmigrations` wylistuj wszystkie migracje dla danej aplikacji a następnie sprawdzając w dokumentacji działanie polecenia `migrate` wycofaj ostatnią migrację.
6. Zainstaluj pakiet `django-debug-toolbar` i skonfiguruj go.
7. Posługując się przykładem z [oficjalnego tutoriala numer 2](https://docs.djangoproject.com/pl/4.1/intro/tutorial02/) dodaj jeden model do panelu administracyjnego i przetestuj dodanie, modyfikację i usunięcie instancji tego modelu. Sprawdzaj zawartość w panelu django debug toolbar za każdym razem.
8. Ponownie w tutorialu https://docs.djangoproject.com/pl/4.1/intro/tutorial02/ przeanalizuj przykłady z wykorzystaniem django shell i wykonaj analogiczny przykład (import modułów, modelu, utworzenie instancji, itd.) wykorzystując jeden z utworzonych modeli.
9. Zapisz zmiany w repozytorium i jeżeli jest to wymagane wyślij informacje o dodaktowym repozytorium dla prowadzącego zajęcia.


### Dodatkowe narzędzia i materiały

* Do obsługi wielu źródeł danych może przydać się darmowe narzędzie o nazwie DBeaver, które posiada również możliwość tworzenia schematów bazy danych a następnie generowania poleceń SQL z takiego schematu. Więcej: https://dbeaver.com/2022/07/07/forward-engineering-with-erd/