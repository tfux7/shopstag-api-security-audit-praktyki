#  Pełny Raport z Audytu Bezpieczeństwa i Jakości API ShopStag

**Data przeprowadzenia audytu:** Październik 2026  
**System poddany testom:** ShopStag REST API (`v1.0.0-ci`)  
**Środowisko:** Staging / Testowe (`https://shopstag-ci.fullstag.pl`)  
**Technologia:** FastAPI / Uvicorn (Backend), Cloudflare (WAF/CDN)  
**Standard odniesienia:** OWASP API Security Top 10  

---

## 1. Podsumowanie Menedżerskie (Executive Summary)

W ramach prac audytowych przeprowadzono kompleksową weryfikację bezpieczeństwa oraz spójności biznesowej interfejsu REST API aplikacji e-commerce **ShopStag**. Prace obejmowały testy penetracyjne, weryfikację walidacji danych wejściowych, obsługę sesji oraz weryfikację mechanizmów kontroli dostępu.

**Główne wnioski:**
* Ogólny poziom zabezpieczeń backendu oceniono jako **wysoki**.
* Zidentyfikowano **2 defekty** (1x High, 1x Medium) związane z nieobsłużonymi wyjątkami serwera (kod HTTP `500`).
* Backend wykazuje bardzo dobrą odporność na luki klasy **BOLA/IDOR**, prawidłowo zarządza tokenami JWT, unieważnia sesje po wylogowaniu oraz skutecznie stosuje ograniczenia liczby zapytań (Rate Limiting).

---

## 2. Statystyki i Wyniki Testów

| Metryka | Liczba | Procentowy udział |
| :--- | :---: | :---: |
| **Łącznie przetestowanych scenariuszy** | **11** | **100%** |
| **Testy zaliczone (Pass)** | **9** | **81.8%** |
| **Wykryte defekty (Fail)** | **2** | **18.2%** |

### Podział błędów według krytyczności:
* 🔴 **Krytyczność Wysoka (High):** 1 (Błąd `500` przy logowaniu na konto administratora)
* 🟡 **Krytyczność Średnia (Medium):** 1 (Błąd `500` przy braku dolnej walidacji parametru `limit=-1`)
* 🟢 **Krytyczność Niska / Informacyjna:** 0

---

## 3. Szczegółowa Macierz Wykonanych Scenariuszy Testowych

### 3.1. Moduł Uwierzytelniania i Autoryzacji (Auth)
* **TC-AUTH-01 [FAIL]:** Logowanie na konto administratora (`POST /auth/login`). Serwer zwrócił kod `500 Internal Server Error` zamiast prawidłowej walidacji lub wydania tokenu.
* **TC-AUTH-03 [PASS]:** Dostęp do chronionego endpointu (`GET /users/me`) bez nagłówka `Authorization`. Serwer prawidłowo odrzucił żądanie kodem `401 Unauthorized`.
* **TC-SEC-04 [PASS]:** Weryfikacja unieważniania tokenu JWT (`POST /auth/logout`). Po wywołaniu logoutu token przestał być aktywny, zwracając status `401 Unauthorized`.

### 3.2. Moduł Produktów i Katalogu (Products)
* **TC-LOG-02 [FAIL]:** Walidacja wartości skrajnych parametrów (`GET /products?limit=-1`). Przekazanie ujemnej limitacji wywołało nieobsłużony błąd serwera `500`.
* **TC-SEC-01 [PASS]:** Sanityzacja znaku specjalnego (`GET /products?search=' OR '1'='1`). Serwer poprawnie zneutralizował zapytanie, zwracając status `200 OK` bez podatności na wstrzykiwanie.

### 3.3. Kontrola Dostępu i Zasoby (Orders / IDOR)
* **TC-IDOR-01 [PASS]:** Próba odczytu zamówienia innego użytkownika (`GET /orders/{id}`). Serwer zwrócił status `404 Not Found`, zapobiegając enumeracji identyfikatorów zasobów.

### 3.4. Bezpieczeństwo Sieciowe i Protokołu (Headers / Security)
* **TC-SEC-02 [PASS]:** Analiza nagłówków odpowiedzi i konfiguracji CORS. Brak podwójnych/sprzecznych deklaracji, poprawna konfiguracja nagłówków bezpieczeństwa Cloudflare.
* **TC-SEC-03 [PASS]:** Ochrona przed atakami typu Brute-Force (`POST /auth/login`). Serwer po serii szybkich zapytań aktywował limit żądań ze statusem `429 Too Many Requests`.
* **TC-METH-01 [PASS]:** Testowanie zamiany metod HTTP (`PUT /auth/login`). Serwer prawidłowo odrzucił żądanie kodem `405 Method Not Allowed`.

### 3.5. Walidacja Wejścia (Validation / Input)
* **TC-VAL-03 [PASS]:** Niezgodność typów danych w ciele żądania JSON. Wsunięcie ciągu znaków w pole liczbowe spowodowało wywołanie walidatora Pydantic i odpowiedź `422 Unprocessable Entity`.
* **TC-HDR-01 [PASS]:** Weryfikacja akceptowanego nagłówka `Content-Type`. Przesłanie typu `text/plain` zamiast `application/json` zostało prawidłowo zablokowane ze statusem `422 Unprocessable Entity`.

---

## 4. Wykryte Defekty i Rekomendacje Naprawcze

### 4.1. BUG-01 (TC-AUTH-01): Błąd serwera przy logowaniu administratora
* **Problem:** Nieobsłużony wyjątek po stronie backendu podczas przetwarzania danych logowania dla konta z rolą `admin`.
* **Rekomendacja:** Zastosowanie bloku `try-except` w serwisie autoryzacji oraz weryfikacja spójności schematu Pydantic z obiektem bazy danych dla użytkowników o podwyższonych uprawnieniach.

### 4.2. BUG-02 (TC-LOG-02): Ujemne wartości w parametrach zapytania
* **Problem:** Brak dolnego ograniczenia numerycznego dla parametru `limit` w `GET /products` wywołuje awarię zapytania SQL/ORM.
* **Rekomendacja:** Doprecyzowanie walidacji parametrów w FastAPI za pomocą klasy `Query`:
  ```python
  limit: int = Query(default=10, ge=1, le=100)
  ```

---

## 5. Podsumowanie i Dalsze Krok

Aplikacja ShopStag API posiada solidne fundamenty bezpieczeństwa. Wdrożenie poprawek dla defektów **BUG-01** oraz **BUG-02** wyeliminuje podatności związane z niestabilnością serwera (`500 Internal Server Error`) i pozwoli na pełne bezpieczne przekazanie API do eksploatacji produkcyjnej.