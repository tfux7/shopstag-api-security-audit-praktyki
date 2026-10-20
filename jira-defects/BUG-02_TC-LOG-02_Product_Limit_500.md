# \[BUG-02\] TC-LOG-02: Nieobsłużony wyjątek 500 przy ujemnym parametrze limit w GET /products

##  Metadane zgłoszenia

* **Typ zgłoszenia (Issue Type):** Bug (Defekt)

* **Projekt:** ShopStag API

* **Komponent:** `Product Catalog` / `GET /products`

* **Priorytet (Priority):** Medium

* **Ważność (Severity):** Medium

* **Środowisko:** Staging / Testowe (`https://shopstag-ci.fullstag.pl`)

* **Wykryte w wersji:** `v1.0.0-ci`

* **Powiązane scenariusze testowe:** `TC-LOG-02`

##  Opis usterki

Endpoint `GET /products` służący do pobierania katalogu produktów nie posiada prawidłowej walidacji dolnej granicy parametrów zapytania (Query Parameters).

Przekazanie wartości ujemnej w parametrze `limit` (np. `limit=-1`) powoduje wystąpienie nieobsłużonego wyjątku po stronie bazy danych / ORM i zwrot kodu **`500 Internal Server Error`**.

Dodatkowo, parametr `price_max` przyjmuje ujemne wartości (np. `price_max=-1`) i zwraca status `200 OK` z pustą listą zamiast walidować logiczny zakres cen.

##  Kroki do odtworzenia (Steps to Reproduce)

1. Wyślij zapytanie HTTP GET na adres:
   `GET {{base_url}}/products?limit=-1`

2. Zaobserwuj odpowiedź serwera.

3. Następnie wyślij zapytanie:
   `GET {{base_url}}/products?price_max=-1`

4. Zaobserwuj odpowiedź serwera.

##  Oczekiwany rezultat (Expected Result)

Dla obu zapytań serwer powinien odrzucić błędne parametry na poziomie walidacji wejścia i zwrócić kod **`422 Unprocessable Entity`** z czytelnym komunikatem walidacji, np.:

```
{
  "detail": [
    {
      "loc": ["query", "limit"],
      "msg": "Input should be greater than or equal to 1",
      "type": "greater_than_equal"
    }
  ]
}

```

##  Rzeczywisty rezultat (Actual Result)

* Dla `GET /products?limit=-1`: Serwer zwraca **`500 Internal Server Error`**.

* Dla `GET /products?price_max=-1`: Serwer zwraca **`200 OK`** z wartością `[]` (brak walidacji wartości ujemnej).

##  Sugerowane rozwiązanie (Recommendation)

Dodaj walidatory skrajnych wartości parametrów w definicji trasy FastAPI za pomocą obiektów `Query` z Pydantic:

```
from fastapi import Query, APIRouter
from typing import Optional

router = APIRouter()

@router.get("/products")
async def get_products(
    limit: int = Query(default=10, ge=1, le=100, description="Liczba produktów na stronę"),
    price_max: Optional[float] = Query(default=None, ge=0.0, description="Maksymalna cena")
):
    ...

```