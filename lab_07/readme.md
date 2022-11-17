# Aplikacje WWW, semestr 2022Z

## Lab 7 - Uprawnienia w aplikacji Django oraz dla Django Rest Framework.

> **Materiały uzupełniające:**
> * Ciekawy rtykuł o uprawnieniach dla Django: https://realpython.com/manage-users-in-django-admin/ (UWAGA: linki odnoszą do dokumentacji dla wersi 2.x Django)
> * Django i uprawnienia: https://docs.djangoproject.com/pl/4.1/topics/auth/default/#permissions-and-authorization


## 1. Uprawnienia a panel administracyjny Django.

Django posiada wbudowany moduł `auth`, dzięki któremu dostarczany jest mechanizm uwierzytelniania i autoryzacji dla użytkowników aplikacji. Jeżeli aplikacje `django.contrib.admin` oraz `django.contrib.auth`  są zainstalowane to w ramach panelu administracyjnego możliwe jest tworzenie użytkowników, grup i przypisywanie im uprawnień.

Ogólna zasada zarządzania uprawnieniami jest taka, iż powinniśmy uprawnienia przypisywać do grup, a nie bezpośrednio do użytkowników. To pozwala na tworzenie różnych poziomów uprawnień (możemy przypisać użytkownika do wielu grup) i zdecydowanie łatwiejszego nimi zarządzania (pojawienie się nowego użytkownika danego działu nie wymaga kopiowania pojedynczych uprawnień tylko przypisania go do odpowiedniej grupy).

Domyślne uprawnienia dla modeli w panelu administracyjnym, będą kontrolowane przez ten panel dla użytkowników w zespole (atrybut użytkownika), którym ten atrybut umożliwia zalogowanie się w panelu. Pozostałe, stworzone przez nas uprawnienia można przypisywać poprzez panel, ale nie zadziałają z automatu i ich reguły muszą być określone na poziomie np. modeli administracyjnych (`ModelAdmin`). Właściwe wykorzystanie tych właściwości i wykorzystanie własnej implementacji modelu użytkownika (przesłonięcie wbudowanego modelu `User`) pozwala zbudować całkiem funkcjonalną aplikację opartą o Django admin.

## 2. Uprawnienia a modele Django.

Django posiada wbudowany mechanizm podstawowych uprawnień dla każdego modelu, który w aplikacji zostanie utworzony. Dla każdego modelu zostaną utworzone cztery prawa:

* `add` - pozwala na stworzenie nowej instancji modelu,
* `delete` - pozwala na usunięcie instancji modelu,
* `change` - pozwala na aktualizację instancji modelu,
* `view` - pozwala na wyświetlenie instancji obiektu.

Aby całość funkcjonowała poprawnie w ekosystemie projektu nazwy uprawnień nadawane są wg. konwencji:
`<nazwa_aplikacji>.<akcja>_<nazwa_modelu>` np. `ankiety.change_person`.

> **Każdy superuser posiada wszystkie prawa, chociaż bliższe prawdy jest stwierdzenie, że tak na prawdę te uprawnienia nie są dla niego sprawdzane w momencie użycia metody `has_perm(uprawnienie)`, które dla tekiego uzytkownika zawsze zwraca wartość `True`.**


Samo istnienie uprawnień nie powoduje, że w każdym miejscu aplikacji działa mechanizm ich weryfikacji, jeżeli na danym modelu odbywa się jedna z operacji CRUD. Jedynym miejscem, w którym się to odbywa jest panel administracyjny.

W każdym innym miejscu musimy zaimplementować metody sprawdzające posiadanie przez uzytkownika wykonującego daną akcję posiadanie przez niego stosownych uprawnień.

**_Listing 1_**  
```python
from django.core.exceptions import PermissionDenied

def person_view(request, pk):
    if not request.user.has_perm('ankiety.view_person'):
        raise PermissionDenied()
    try:
        person = Person.objects.get(pk=pk)
        return HttpResponse(f"Ten użytkownik nazywa się {person.name}")
    except Person.DoesNotExist:
        return HttpResponse(f"W bazie nie ma użytkownika o id={pk}.")
```

Można wykorzystać również dekoratory na wzór:

**_Listing 2_**

```python
from django.contrib.auth.decorators import permission_required

@permission_required('ankiety.view_person')
def person_view(request, pk):
    try:
        person = Person.objects.get(pk=pk)
        return HttpResponse(f"Ten użytkownik nazywa się {person.name}")
    except Person.DoesNotExist:
        return HttpResponse(f"W bazie nie ma użytkownika o id={pk}.")
```

Oprócz wbudowanych uprawnień dla modeli możliwe jest również zdefiniowanie dodatkowych uprawnień w klasie wewnętrznej `Meta` każdego modelu.

**_Listing 3_**

```python
class Person(models.Model):
    ...
    class Meta:
        permissions = [
            ("change_person_owner", "Pozwala przypisać inną osobę do obiektu Person."),
            ("change_assign_to_team", "Pozwala przypisać osobę do innej drużyny."),
        ]
```
Aby takie uprawnienia pojawiły się w panelu administracyjnym i było możliwe ich wykorzystanie w projekcie należy wykonać proces migracji, gdyż funckja tworząca uprawnienia jest powiązana z sygnałem `post_migrate`.
Implementacja logiki tych uprawnień spoczywa w całości na programiście.

Możliwe jest również weryfikowanie uprawnień na poziomie szablonów widoków Django (jeżeli to jest nasza docelowa technologia wizualizacji dla projektu) odwołując się do zmiennej `perms` w szablonie.

**_Listing 4_**
```python
{% if perms.ankiety.delete_person %}
<button>Delete</button>
{% endif %}
```

## 3.Uprawnienia a Django Rest Framework

Django Rest Framework dostarcza kilka wbudowanych uprawnień, z których część została przedstawiona na poprzednich zajęciach (np. IsAuthenticated).

Pozostałe zostały opisane tutaj: https://www.django-rest-framework.org/api-guide/permissions/#api-reference

**Wykorzystanie istniejących uprawnień modeli w DRF.**

Dokumentacja: https://www.django-rest-framework.org/api-guide/permissions/#djangomodelpermissions

Z racji tego, że aktualnie nie ma oficjalnego rozwiązania pozwalającego na wykorzystanie `DjangoModelPermissions` w przypadku wykorzystania widoków funkcyjnych (to te, gdzie używamy dekoratora `@api_view`) należy niezbędną logikę widoków dla DRF przepisać na class based views. Przykład dla modelu team poniżej.

**_Listing 4_**

```python
class TeamDetail(APIView):
    # ustalamy dostępne dla widoków metody uwierzytelniania
    authentication_classes = [TokenAuthentication]
    # określamy zbiór uprawnień - tutaj propagacja uprawnień na modelach, zdefiniowanych
    # dla grupy lub użytkownika przypisanego do tokena
    permission_classes = [DjangoModelPermissions]

    def get_object(self, pk):
        try:
            return Team.objects.get(pk=pk)
        except Team.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        team = self.get_object(pk)
        serializer = TeamSerializer(team)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        team = self.get_object(pk)
        serializer = TeamSerializer(team, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        team = self.get_object(pk)
        team.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

W dokumentacji znajdziemy informacje o mapowaniu żądań na uprawnienia:
* żądania `POST` wymagają posiadania prawa `add` na modelu,
* żądania `PUT` i `PATCH` wymagają posiadania prawa `change` na modelu,
* żądania `DELETE` wymagają posiadania prawa `delete` na modelu.

A gdzie żądanie `GET`? Otóż chociaż mogłoby się wydawać logicznym, że powinno być mapowanie na uprawnienie `view` dla modelu, to tak nie jest. Można jednak rozszerzyć bazową klasę `DjangoModelPermissions` i dodać tę funkcjonalność.

**_Listing 5_**
Kod został umieszczony w pliku `permissions.py` w głównym folderze projektu.
```python
import copy

from rest_framework import permissions


class CustomDjangoModelPermission(permissions.DjangoModelPermissions):

    def __init__(self):
        self.perms_map = copy.deepcopy(self.perms_map)
        self.perms_map['GET'] = ['%(app_label)s.view_%(model_name)s']
```



## **Zadania**

**Zadanie 1**  
Dodaj nową grupę w panelu administracyjnym. Dodaj do tej grupy jedno wybrane uprawnienie domyślne dla modelu, który został dodany do zarządzania w panelu administracyjnym. Stwórz nowego użytkownika, który będzie "w zespole", ale nie będzie superużytkownikiem i przypisz go do tej grupy. Zaloguj się na konto stworzonego użytkownika i sprawdź czy kontrola tego uprawnienia działa poprawnie.

**Zadanie 2**  
Korzystając z przykładów z listingów 1 oraz 2 dodaj prosty widok, w logice którego sprawdź czy user posiada uprawnienie `osoba_view` i wyświetlaj odpowiednią treść.

**Zadanie 3**  
Do modelu `Osoba` dodaj własne uprawnienie o nazwie `can_view_other_persons`, które dodaj do logiki z zadania 2 i jeżeli jest ono przypisane to pozwalaj wyświetlać obiekty modelu `Osoba`, których zalogowany użytkownik nie jest właścicielem. W przeciwnym wypadku nie daj takiej możliwości.

**Zadanie 4**  
Przetestuj działanie klasy `DjangoModelPermissions` z DRF z różnymi prawami dostępu (GET, PUT, POST, DELETE).

