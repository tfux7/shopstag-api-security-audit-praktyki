# [BUG-01] TC-AUTH-01: Błąd serwera 500 Internal Server Error podczas logowania na konto administratora

##  Metadane zgłoszenia
* **Typ zgłoszenia (Issue Type):** Bug (Defekt)
* **Projekt:** ShopStag API
* **Komponent:** `Authentication` / `POST /auth/login`
* **Priorytet (Priority):** High
* **Ważność (Severity):** High
* **Środowisko:** Staging / Testowe (`https://shopstag-ci.fullstag.pl`)
* **Wykryte w wersji:** `v1.0.0-ci`
* **Powiązane scenariusze testowe:** `TC-AUTH-01`

---

##  Opis usterki
Podczas próby uwierzytelnienia na poprawne dane logowania konta administracyjnego (rola `admin`) za pośrednictwem bezpośredniego zapytania HTTP `POST /auth/login`, backend odrzuca połączenie i zwraca kod odpowiedzi **`500 Internal Server Error`** zamiast poprawnego tokenu lub dedykowanego komunikatu walidacji.

Błąd ten uniemożliwia automatyzację zadań administracyjnych poprzez API oraz może świadczyć o nieobsłużonym wyjątku w logice mapowania ról lub schematach Pydantic/ORM.

---

##  Kroki do odtworzenia (Steps to Reproduce)
1. Otwórz klienta API (np. Postman).
2. Skonfiguruj żądanie HTTP:
   * **Metoda:** `POST`
   * **URL:** `{{base_url}}/auth/login`
   * **Header:** `Content-Type: application/json`
3. W zakładce **Body** (`raw` -> `JSON`) wprowadź dane konta administratora:
   ```json
   {
     "username": "admin@shopstag.pl",
     "password": "CorrectAdminPassword123!"
   }
   ```
4. Wyślij zapytanie i przeanalizuj odpowiedź serwera.

---

##  Oczekiwany rezultat (Expected Result)
Serwer przetwarza żądanie i zwraca kod **`200 OK`** wraz z poprawnym tokenem dostępowym JWT w obiekcie JSON:
```json
{
  "access_token": "eyJhbGciOi...",
  "token_type": "bearer"
}
```

---

##  Rzeczywisty rezultat (Actual Result)
Serwer odpowiada kodem **`500 Internal Server Error`**:
```json
{
  "detail": "Internal Server Error"
}
```

---

##  Wpływ na bezpieczeństwo i jakość (Impact)
* **Obsługa błędów:** Wyciek nieobsłużonych wyjątków po stronie backendu świadczy o braku odpowiedniej obsługi błędów (Exception Handling).
* **Automatyzacja i Integracja:** Uniemożliwienie integracji zewnętrznym systemom z rolą administracyjną.

---

##  Sugerowane rozwiązanie (Recommendation)
1. Sprawdzić logi aplikacji (application logs) w momencie wykonywania żądania w celu zidentyfikowania dokładnego wyjątku (Stack Trace).
2. Upewnić się, że model użytkownika z rolą `admin` jest poprawnie serializowany przez Pydantic przed zwróceniem odpowiedzi.
3. Obsłużyć potencjalne braki w polach profilu administracyjnego za pomocą bloku `try-except`, zwracając w razie potrzeby czytelny błąd z grupy `4xx`.