# Aplikacje WWW, semestr 2024Z

## Lab 1
---


# 1. Warto zacząć od...

1.1 Zanim rozpoczniesz przygotowanie swojego pierwszego projektu z użyciem Django 4.2 rzuć okiem na release notes dla tej wersji: https://docs.djangoproject.com/en/5.2/releases/5.2/

1.2 Zachęcam również do zaglądnięcia do kilku ciekawych tutoriali (poza oficjalnym oczywiście) wprowadzających do tworzenia aplikacji z użyciem Django:
* https://realpython.com/get-started-with-django-1/ (i więcej kursów z tej strony w tematyce Django: https://realpython.com/tutorials/django/ )
* https://www.w3schools.com/django/



# Zadania

1. Jeżeli korzystasz na zajęciach ze swojego sprzętu to zainstaluj niezbędne oprogramowanie.
2. Stwórz środowisko wirtualne z interpreterem python na potrzeby swojego projektu.
3. Zainstaluj niezbędne paczki (aktualnie pakiet o nazwie `django` powinien wystarczyć, wskaż tu wersję 5.2). W razie problemów z instalacją rzuć okiem na oficjalną dokumentację pod adresem https://docs.djangoproject.com/pl/5.2/intro/install/
4. Przygotuj swój pierwszy projekt oraz aplikację Django (warto znać różnicę między tymi pojęciami) bazując na oficjalnym tutorialu znajdującym się pod adresem https://docs.djangoproject.com/pl/5.2/intro/tutorial01/. Uruchom wbudowany serwer www (polecenie znajdziesz w tutorialu lub w przykładzie poniżej zadań) i sprawdź czy projekt się uruchamia.
5. Skonfiguruj lokalne i zdalne repozytorium git. Przygotuj odpowiedni plik `.gitignore` dla projektu Django i wykonaj inicjalny commit oraz push do zdalnego repo. Przetestuj czy konfiguracja jest odpowiednia.
6. Zastanów się nad tematem Twojego projektu i schematem bazy danych, który będzie potrzebny do jej działania. Możesz również wybrać na tym etapie silnik bazy danych, na którym będziesz chciał osadzić swoje dane.

## Przydatne instrukcje do zadań

### Zadanie 1: Instalacja niezbędnego oprogramowania

1. **Zainstaluj Python**: 
   - Sprawdź, czy Python jest zainstalowany na twoim komputerze. Możesz to zrobić, uruchamiając polecenie w terminalu:
     ```bash
     python --version
     ```
   - Jeśli nie masz Pythona zainstalowanego, pobierz go ze strony [oficjalnej Pythona](https://www.python.org/downloads/) i zainstaluj.

2. **Zainstaluj pip**: `pip` to menedżer pakietów Pythona. W większości instalacji Pythona `pip` jest domyślnie instalowany. Możesz to sprawdzić poleceniem:
   ```bash
   pip --version
   ```

## Zadanie 2: Stworzenie środowiska wirtualnego

1. **Utwórz folder projektu**:
   - Otwórz terminal i przejdź do folderu, w którym chcesz utworzyć projekt.
   - Utwórz nowy folder:
     ```bash
     mkdir myproject
     cd myproject
     ```

2. **Stwórz środowisko wirtualne**:

   - Uruchom polecenie, aby stworzyć wirtualne środowisko:
     ```bash
     python -m venv .venv
     # lub wykorzystując pakiet virtualenv
     python -m virtualenv .venv
     ```
  
   - Aktywuj środowisko wirtualne (korzystając z narzędzi takich jak PyCharm lub VSC ta operacja powinna się wykonać automatycznie w momencie uruchomienia okna terminala wewnątrz w/w narzędzi):
     - Na Windows:
       ```bash
       .venv\\Scripts\\activate
       ```
     - Na macOS/Linux:
       ```bash
       source .venv/bin/activate
       ```
    Utworzenie środowiska wirtualnego jak już rozumiemy cały proces może być wykonane również za pomocą GUI narzędzi PyCharm lub VSC. W przypadku tego pierwszego możliwe jest to na etapie tworzenia nowego projektu lub w dowolnym momencie pracy nad nim. W przypadku VSC mamy możliwość wykonać to po zainstalowaniu pluginu do obsługi Pythona i skorzystaniu z **Command palette**, które będzie zawierało odpowiednie opcje. Proces tez zostanie zaprezentowany przez prowadzącego w trakcie zajęć.


## Zadanie 3: Instalacja niezbędnych paczek

1. **Zainstaluj Django**:
   - W aktywowanym środowisku wirtualnym, upewnij się, że masz zainstalowany pakiet `django`:
     ```bash
     # wyświetlenie listy zainstalowanych paczek
     pip list
     # instalacja Django (najnowsza wersja major 5, minor 5.2)
     pip install django==5.2.*
     # lub konkretnie (w momencie pisania dokumentacji najnowsza wersja release)
     pip install django==5.2.7
     ```

2. **Rozwiązywanie problemów z instalacją**:
   - W razie problemów z instalacją, sprawdź oficjalną dokumentację [Django](https://docs.djangoproject.com/pl/5.2/intro/install/) w celu uzyskania wskazówek dotyczących instalacji.

## Zadanie 4: Przygotowanie pierwszego projektu i aplikacji Django

1. **Stwórz nowy projekt Django**:
   - W terminalu, upewnij się, że jesteś w katalogu głównym swojego projektu (blog). Następnie użyj polecenia:
     ```bash
     django-admin startproject blog
     ```
   - Przejdź do folderu projektu:
     ```bash
     cd blog
     ```

2. **Stwórz aplikację Django**:
   - W folderze projektu utwórz nową aplikację:
     ```bash
     python manage.py startapp posts
     ```
   - **Rozróżnienie**: Projekt Django to kontener dla aplikacji, podczas gdy aplikacja to konkretna część projektu, która wykonuje określone zadania (np. obsługa użytkowników, bloga itp.).

3. **Uruchom wbudowany serwer**:
   - Aby uruchomić serwer, użyj polecenia:
     ```bash
     python manage.py runserver
     ```
   - Sprawdź, czy aplikacja działa, otwierając przeglądarkę i wpisując adres `http://127.0.0.1:8000/`.

## Zadanie 5: Konfiguracja repozytorium Git

1. **Zainicjalizuj lokalne repozytorium**:
   - W folderze projektu, zainicjalizuj repozytorium Git:
     ```bash
     git init
     ```

2. **Przygotuj plik `.gitignore`**:
   - Utwórz plik `.gitignore` w katalogu głównym projektu. Dodaj następujące linie, aby zignorować pliki środowiska wirtualnego i inne pliki, które nie powinny być wersjonowane:
     ```
     .venv/
     *.pyc
     __pycache__/
     db.sqlite3
     ```

3. **Wykonaj inicjalny commit**:
   - Dodaj wszystkie pliki do repozytorium:
     ```bash
     git add .
     ```
   - Wykonaj commit:
     ```bash
     git commit -m "Initial commit"
     ```

4. **Skonfiguruj zdalne repozytorium**:
   - Utwórz zdalne repozytorium na platformie, takiej jak GitHub, GitLab lub Bitbucket.
   - Po utworzeniu zdalnego repozytorium, połącz je z lokalnym repozytorium:
     ```bash
     git remote add origin https://github.com/username/repo.git
     ```

5. **Wykonaj push do zdalnego repo**:
   - Prześlij zmiany do zdalnego repozytorium:
     ```bash
     git push -u origin main
     ```
   - Sprawdź, czy konfiguracja jest odpowiednia, przeglądając zdalne repozytorium.