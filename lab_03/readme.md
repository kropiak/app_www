# Aplikacje WWW, semestr 2022Z

## Lab 3
---

### **1. Więcej możliwości modeli w ekosystemie Django.**


Oprócz typowego mapowania obiektowo-relacyjnego klasy modeli frameworka Django oferują dużo więcej możliwości, m. in. implementację funkcji pomocniczych, określenie domyślnego porządku sortowania zwracanych krotek, określenie parametrów walidacji i inne. Poniżej zostaną zaprezentowane niektóre z możliwości wraz z przykładami użycia.

Przydatne linki do dokumentacji, które mogą się przydać w trakcie rozwiązywania zadań:
1. Lista dostępnych pól klas modeli: https://docs.djangoproject.com/pl/4.1/ref/models/fields/#model-field-types
2. Atrybuty modeli: https://docs.djangoproject.com/pl/4.1/topics/db/models/#field-options
3. Klasa QuerySet i dostępne metody: https://docs.djangoproject.com/pl/4.1/ref/models/querysets/

#### **1.1 Statyczna lista wartości pola wyboru.** 

Bardzo często w wielu aplikacjach potrzebujemy przechować dane w postaci tabel słownikowych, czyli takich, które są statycznymi skończonymi zbiorami wartości definiowanymi za zwyczaj w momencie wdrażania aplikacji, a potrzebnymi w procesie jej eksploatacji. Takim przykładem może być lista miesięcy do wyboru, nazw województw itp. Często informacje te są przechowywane w bazie danych co ułatwia zarządzanie (nie są rozproszone w wielu miejscach systemu), ale istnieją również możliwości określenia takich list na poziomie modelu Django. Przykład poniżej.

**_Listing 1_**
```python
# kod z oficjalnej dokumentacji Django
class Person(models.Model):
    # lista wartości do wyboru w formie krotek
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    # wskazanie listy poprzez przypisanie do parametru choices
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```

Dodanie klasy do pliku `models.py` a następnie przygotowanie, wykonanie migracji oraz dodanie modelu do panelu administracyjnego pozwoli na przetestowanie działania takiego podejścia.

**Zadanie 1**  
Pracę rozpocznij od utworzenia nowego brancha o nazwie `lab_3` w swoim repozytorium. Następnie stwórz nowy model o nazwie `Osoba` z polami:
* `imie` - pole tekstowe, wymagane, niepuste (sprawdź dokumentację z pkt. 2)
* `nazwisko` - pole tekstowe, wymagane, niepuste
* `miesiac_urodzenia` - (pole wyboru z zakodowanymi numerami i nazwami miesięcy)

Przygotuj i wykonaj migrację. Dodaj model do panelu administracyjnego i zapisz do bazy 3 instancje obiektu Osoba.
Sprawdź w bazie w jaki sposób zapisywane są dane dla pola `miesiac_urodzenia` oraz w jaki sposób wygląda deklaracja HTML pola w źródle strony oraz sposób ich wyświetlania w panelu administracyjnym. Sprawdź w bazie jak przechowywane są dane. Czy pojawiły się tam kolumny, które nie zostały zdefiniowane w pliku modelu?

**Zadanie 2**  
Dodaj definicję wartości domyślnej dla pola `miesiac_urodzenia` modelu `Osoba`, tak aby wybierana była pierwsza wartość z listy wyboru (aktualnie nie jest).

**Zadanie 3**  
W dokumentacji pola typu `DateField` znajdź informacje o sposobie automatycznego wstawiania wartości znacznika czasu (timestamp) w momencie utworzenia obiektu. Ustaw taką własność dla nowego pola `data_dodania` modelu `Osoba`.

**Zadania 4**  
Z dokumentacji typów wyliczeniowych https://docs.djangoproject.com/en/4.1/ref/models/fields/#enumeration-types wykorzystaj klasę `models.IntegerChoices` i zmień deklarację pola `miesiac_urodzenia`, tak żeby ją wykorzystać.

#### **1.2 Przesłanianie metod oraz właściwości META**  

**Zadanie 5**  
Przesłoń metodę `__str()__` modelu `Osoba` tak aby zamiast `Person object #` wyświetlane było imię i nazwisko osoby.

**Zadanie 6**  
Bazując na przykładzie z dokumentacji https://docs.djangoproject.com/pl/4.1/topics/db/models/#meta-options dodaj właściwość `META` sortując domyślnie po nazwisku rosnąco (alfabatycznie).

#### **1.3 Modyfikacja listy pól w widoku listy i filtry w module admin**

W panelu administracyjnym dla obiektów `Osoba` nie są widoczne wszystkie pola na liście wszystkich obiektów lecz tylko jedno z nich. Aby to zrobić należy najpierw zdefiniować w pliku `admin.py` klasę admin dla danego modelu. Przykład poniżej:

**_Listing 2_**
```python
class PersonAdmin(admin.ModelAdmin):
    list_display = ['name', 'shirt_size']

# ten obiekt też trzeba zarejestrować w module admin
admin.site.register(Person, PersonAdmin)
```

**Zadanie 7**  
Dodaj klasę admin dla modelu `Osoba` i zmień listę wyświetlanych kolumn w panelu administracyjnym.

**Zadanie 8**  
Stwórz nowy model o nazwie `Druzyna` i dodaj do niego pola:
* nazwa - pole tekstowe
* kraj - pole tekstowe (docelowo tylko dwa znaki wielkimi literami np. PL, DE, US, itd.)
Dodaj ten model jako klucz obcy w klasie `Osoba` (pole nie jest wymagane, ustaw NULL w razie usunięcia). Zarejestruj model w panelu administracyjnym i dodaj model administracyjny (`DruzynaAdmin`) tak jak w przykładzie z listingu 2. Stwórz kilka instancji modelu `Druzyna` i przypisz niektóre z nich do obiektów 'Osoba`.

**Zadanie 9**  
Zmodyfikuj kod aplikacji tak, żeby na liście modeli `Osoba` w panelu administracyjnym wyświetlana została również kolumna `Drużyna` (z polskim ogonkiem) o postaci 'nazwa (kraj)' np. The Tigers (PL).

W panelu administracyjnym możliwe jest również dodanie filtrów do widoków. Cały proces polega na dodaniu pola filter_list w klasie admin danego modułu:

**_Listing 3_**
```python
class PersonAdmin(admin.ModelAdmin):
    list_filter = ('team')
```
![filtry](filters.png)


W zależności od typu pola zostanie wyświetlony odpowiedni filtr. W przypadku niektórych rodzajów i dużej liczby unikalnych wartości używanie filtrów może być niepraktyczne ze względów wydajnościowych i wizualnych.

**Zadanie 10**  
Dodaj filtr dla drużyn oraz daty utworzenia dla klasy `Osoba` oraz dla krajów dla klasy `Druzyna`. Przetestuj działanie filtrów.

**Zadanie 11**  
Korzystając z dokumentacji API klasy QuerySet z pkt. 3 wykonaj następujące zapytania za pomocą shella Django (kod Pythona z zapytaniami umieść w pliku lab_3_zadanie_11.md w swoim repozytorium):
* wyświetl wszystkie obiekty modelu `Osoba`,
* wyświetl obiekt modelu `Osoba` z id = 3,
* wyświetl obiekty modelu `Osoba`, których nazwa rozpoczyna się na wybraną przez Ciebie literę alfabetu (tak, żeby był co najmniej jeden wynik),
* wyświetl unikalną listę drużyn przypisanych dla modeli `Osoba`,
* wyświetl nazwy drużyn posortowane alfabetycznie malejąco,
* dodaj nową instancję obiektu klasy `Osoba`

**Zadanie 12**  
Jeżeli nie robiłeś/-aś tego wcześniej to zatwierdź wszystkie zmiany w danym branchu i spróbuj wykonać merge z główną gałęzią. Proponuję wykonać tę operację przy pomocy interfejsu IDE PyCharm, aby przetestować wbudowane narzędzie do wspomagania procesu merge (o ile wystąpią konflikty).

**Dla chętnych**  
Podobne zagadnienia zostały przedstawione w oficjalnym tutorialu numer 7: https://docs.djangoproject.com/pl/4.1/intro/tutorial07/. Przeglądnij je i spróbuj wybrane rozwiązania zastosować w kodzie z zajęć.