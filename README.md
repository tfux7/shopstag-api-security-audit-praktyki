# ShopStag API - Security & QA Audit Portfolio Project

## Opis projektu
Projekt przedstawia pełny audyt bezpieczeństwa i jakości interfejsu REST API aplikacji e-commerce **ShopStag** (FastAPI / Uvicorn / Cloudflare).

Celem audytu była weryfikacja odporności backendu na powszechne podatności (zgodnie ze standardem **OWASP API Security Top 10**), kontrola dostępu (IDOR/BOLA), obsługa tokenów JWT, walidacja wejścia oraz obsługa błędów.



## Zakres testów i kluczowe wnioski
 **Przetestowane scenariusze:** 11
 **Zaliczone (Pass):** 9
 **Wykryte defekty:** 2 (1x High, 1x Medium)

### Zidentyfikowane błędy (Defekty):
1. **[BUG-01] TC-AUTH-01 (High):** Błąd serwera `500 Internal Server Error` podczas próby logowania na konto administratora przez API.
2. **[BUG-02] TC-LOG-02 (Medium):** Nieobsłużony wyjątek `500` dla ujemnej wartości parametru `limit=-1` w endpointcie `GET /products`.



## Podsumowanie wyników testów

| Kod testu | Obszar / Endpoint | Opis | Status |
| :--- | :--- | :--- | :--- |
| **TC-AUTH-01** | `POST /auth/login` | Logowanie na konto administratora |  **Defekt (High)** |
| **TC-LOG-02** | `GET /products` | Walidacja ujemnych parametrów `limit` i `price_max` |  **Defekt (Medium)** |
| **TC-SEC-01** | `GET /products` | Odporność na wstrzykiwanie znaków w `search` |  **Pass** |
| **TC-SEC-02** | Nagłówki / CORS | Weryfikacja polityki CORS i X-Frame-Options |  **Pass** |
| **TC-SEC-03** | `POST /auth/login` | Ochrona przed Brute-Force (Rate Limiting 429) |  **Pass** |
| **TC-IDOR-01** | `GET /orders/{id}` | Ochrona dostępu horyzontalnego (IDOR 404) |  **Pass** |
| **TC-SEC-04** | `POST /auth/logout` | Unieważnianie tokenu JWT po wylogowaniu |  **Pass** |
| **TC-AUTH-03** | `GET /users/me` | Odrzucanie zapytań bez autoryzacji (401) |  **Pass** |
| **TC-METH-01** | `POST /auth/login` | Walidacja niedozwolonych metod HTTP (405) |  **Pass** |
| **TC-VAL-03** | `POST /cart/items` | Walidacja typów danych w ciele JSON (422) |  **Pass** |
| **TC-HDR-01** | `POST /auth/login` | Weryfikacja nagłówka Content-Type (422) |  **Pass** |



##  Użyte narzędzia
 **Postman** (Pakiety zapytań, zmienne środowiskowe, automatyzacja nagłówków)
 **DevTools / cURL** (Analiza struktury żądań sieciowych)
 **Markdown / JIRA** (Dokumentacja i raportowanie błędów)



##  Struktura repozytorium
 `/postman` - Wyeksportowana kolekcja Postman gotowa do zaimportowania i uruchomienia.
 `/reports` - Szczegółowy raport audytowy oraz plik CSV z macierzą testów.
 `/jira-defects` - Szablony zgłoszeń błędów dla zespołu deweloperskiego.