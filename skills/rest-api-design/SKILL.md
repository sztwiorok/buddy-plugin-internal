---
name: rest-api-design
description: This skill should be used when the user asks to "design REST API", "create new API endpoint", "add REST endpoint", "plan API implementation", "new REST feature", or mentions designing, planning, or creating new REST API functionality for the Buddy backend.
---

# REST API Design Assistant

Jesteś asystentem pomagającym zaprojektować nową funkcjonalność REST API w backendzie Buddy. Twoim zadaniem jest przeprowadzenie użytkownika przez proces projektowania API poprzez zadawanie pytań i generowanie planu implementacji.

## Twoja rola

Działasz jako architekt API, który:
1. Zadaje pytania, aby zrozumieć wymagania
2. Sprawdza istniejący kod w repozytorium
3. Sugeruje rozwiązania bazując na wzorcach używanych w projekcie
4. Generuje szczegółowy plan implementacji

## Proces projektowania

### Krok 1: Identyfikacja zasobu

Zapytaj użytkownika:
- **Czy to nowy zasób czy rozszerzenie istniejącego?**
- **Podaj nazwę zasobu** (użytkownik wpisuje ręcznie, np. "Distribution", "Routing")

**NIE listuj przykładowych nazw** - użytkownik wie co chce stworzyć.

Po uzyskaniu nazwy **natychmiast** wykonaj Krok 2 (sprawdzenie Internal API).

### Krok 2: Automatyczne sprawdzenie Internal API

**WAŻNE:** Zaraz po poznaniu nazwy zasobu, automatycznie sprawdź istniejący kod:

1. **Przeszukaj Internal API:**
```
application-server/src/main/java/works/buddy/application/internalapi/
```

- Szukaj `*{ResourceName}InternalApi.java` i `Default*{ResourceName}InternalApi.java`
- Zanotuj dostępne metody i ich sygnatury

2. **Przeszukaj modele Front:**
```
application-core/src/main/java/works/buddy/**/
```

- Szukaj `{ResourceName}Front.java`, `{ResourceName}sFront.java`
- Zanotuj strukturę pól

3. **Przedstaw użytkownikowi wyniki:**
- "Znalazłem Internal API: [lista metod]"
- "Znalezione modele Front: [lista]"
- Lub: "Nie znalazłem Internal API dla tego zasobu"

Internal API na tym etapie zawsze powinno być zaimplementowane - skill służy do tworzenia dokumentacji REST API dla klienta.

### Krok 3: OAuth2 Scopes

1. **Automatycznie sprawdź istniejące scope'y** w pliku:
```
application-core/src/main/java/works/buddy/ws/api/authentication/entities/OAuth2Scope.java
```

2. **Zapytaj użytkownika:**
- Czy endpoint będzie używał **istniejących scope'ów**? (podaj listę pasujących, np. `SANDBOX_INFO`, `SANDBOX_MANAGE`)
- Czy będziemy **tworzyć nowe scope'y**? (zaproponuj nazwy wg konwencji: `{RESOURCE}_INFO`, `{RESOURCE}_ADD`, `{RESOURCE}_MANAGE`)

3. **Konwencja dla nowych scope'ów:**

| Operacja | Wzorzec Scope | Higher Level |
|----------|---------------|--------------|
| GET (list/detail) | `{RESOURCE}_INFO` | `{RESOURCE}_ADD` |
| POST (create) | `{RESOURCE}_ADD` | `{RESOURCE}_MANAGE` |
| PATCH (update) | `{RESOURCE}_MANAGE` | `null` |
| DELETE | `{RESOURCE}_MANAGE` | `null` |

### Krok 4: Struktura URL i metody HTTP

**Domyślna konwencja** (jeśli zasób ma scope'y WORKSPACE/PROJECT/ENVIRONMENT):
```
/workspaces/{workspace_domain}/{resources}
```

Scope jest obsługiwany przez **query parameters**:
- `scope` - WORKSPACE, PROJECT, ENVIRONMENT
- `project_name` - filtr po projekcie (dla scope PROJECT)
- `environment_id` - filtr po środowisku (dla scope ENVIRONMENT)

**Wzorzec:** Zobacz `PackageApi` jako referencję.

**Zaproponuj użytkownikowi tę strukturę** i pozwól mu ją zaakceptować lub podać alternatywną ścieżkę jeśli ma specyficzne wymagania.

**Zapytaj które operacje są potrzebne:**
- `GET /resources` - lista zasobów
- `GET /resources/{id}` - szczegóły zasobu
- `POST /resources` - tworzenie
- `PATCH /resources/{id}` - aktualizacja
- `DELETE /resources/{id}` - usuwanie

### Krok 5: Model danych

**Zapytaj o pola (wszystko w jednym kroku):**

1. **Short Objects dla list:**
   - **ZAWSZE zapytaj użytkownika:** "Czy dla list używamy Short Objects?"
   - Jeśli **TAK**: `GET /resources` zwraca tablicę `Short{Resource}View`, a `GET /resources/{id}` zwraca pełny `{Resource}View`
   - Jeśli **NIE**: wszystkie endpointy zwracają ten sam `{Resource}View`

2. **Pola podstawowe:**
   - Jakie pola powinien zawierać zasób?
   - Które pola są wymagane (required)?
   - Które pola są tylko do odczytu (read-only)?

3. **Request vs Response View:**
   - **Request View** (dla POST/PATCH): pola które klient wysyła
   - **Response View** (dla GET): pola które API zwraca
   - Typowe różnice:
     - Response ma pola read-only: `id`, `url`, `html_url`, `createDate`, `updateDate`
     - Request może mieć pola które nie są zwracane (np. `password`)

4. **Standardowe pola Response (ZAWSZE dodawaj):**
   - `url` - URL do zasobu w API (read-only), np. `/api/workspaces/{domain}/{resources}/{id}`
   - `html_url` - URL do zasobu w serwisie WWW (read-only)

5. **Walidacja (dla Request View):**
   - `@NotEmpty` - pole wymagane
   - `@AvailableValues` - enum-like

6. **Typy pól:**
   - Proste: `String`, `Integer`, `Boolean`, `OffsetDateTime`
   - Relacje: `Short*View` (dla zagnieżdżonych obiektów)
   - Kolekcje: `Set<String>`, `Collection<*View>`
   - Enumy: zdefiniuj dozwolone wartości

### Krok 6: Generowanie planu

Na podstawie zebranych informacji wygeneruj plan implementacji w formacie poniżej.
**WAZNE: planie nie dajemy listy plikow, ktore trzeba stworzyc**

## Format planu

```markdown
# Plan implementacji REST API: {NazwaZasobu}

## Podsumowanie
| Aspekt | Wartość |
|--------|---------|
| Zasób | {Resource} |
| Ścieżka | `/workspaces/{workspace_domain}/{resources}` |
| Metody | GET, POST, PATCH, DELETE |
| Scopes | {RESOURCE}_INFO, {RESOURCE}_ADD, {RESOURCE}_MANAGE |
| Short Objects | Tak/Nie |


## Endpointy

| Metoda | Ścieżka | Scope | Request | Response |
|--------|---------|-------|---------|----------|
| GET | `/workspaces/{domain}/{resources}` | {RESOURCE}_INFO | - | 200 {Resource}sView (Short*) |
| GET | `/workspaces/{domain}/{resources}/{id}` | {RESOURCE}_INFO | - | 200 {Resource}View |
| POST | `/workspaces/{domain}/{resources}` | {RESOURCE}_ADD | {Resource}Request | 201 {Resource}View |
| PATCH | `/workspaces/{domain}/{resources}/{id}` | {RESOURCE}_MANAGE | {Resource}Request | 200 {Resource}View |
| DELETE | `/workspaces/{domain}/{resources}/{id}` | {RESOURCE}_MANAGE | - | 204 No Content |

## Query Parameters (dla GET list)
| Param | Typ | Default | Opis |
|-------|-----|---------|------|
| page | Integer | 1 | Numer strony |
| per_page | Integer | 20 | Ilość na stronę (max 50) |
| project_name | String | - | Filtr po projekcie |

## Modele danych

### Request: {Resource}Request (POST/PATCH)
| Pole | Typ | Wymagane | Walidacja | Opis |
|------|-----|----------|-----------|------|
| name | String | tak | @NotEmpty | Nazwa zasobu |
| ... | ... | ... | ... | ... |

### Response: Short{Resource}View (dla list - jeśli Short Objects = Tak)
| Pole | Typ | Opis |
|------|-----|------|
| url | String | URL do zasobu w API |
| html_url | String | URL do zasobu w serwisie WWW |
| id | String | Identyfikator |
| name | String | Nazwa |
| ... | ... | ... |

### Response: {Resource}View (dla szczegółów, extends Short jeśli Short Objects = Tak)
| Pole | Typ | Opis |
|------|-----|------|
| url | String | URL do zasobu w API |
| html_url | String | URL do zasobu w serwisie WWW |
| id | String | Identyfikator |
| name | String | Nazwa |
| ... pola specyficzne ... | ... | ... |

## Referencje
- Wzorzec: `TargetApi.java`
- Internal API: `{Resource}InternalApi.java`
```

## Dobre praktyki

1. **Zawsze sprawdzaj istniejący kod** przed sugerowaniem rozwiązań
2. **Używaj narzędzi Grep/Glob** do przeszukiwania repozytorium
4. **Pytaj o edge cases** - co jeśli zasób nie istnieje? co jeśli brak uprawnień?
5. **Waliduj nazewnictwo** - czy nazwa scope'a nie koliduje z istniejącymi?

## Pliki referencyjne

Zapoznaj się z plikami w katalogu `references/`:
- `patterns.md` - wzorce kodu używane w projekcie
- `examples.md` - przykłady dobrych implementacji
- `scopes.md` - lista istniejących OAuth2 scopes
