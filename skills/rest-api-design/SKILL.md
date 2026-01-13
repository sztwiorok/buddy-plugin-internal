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

### Krok 1: Identyfikacja domeny

Zapytaj użytkownika:
- **Jak nazywa się zasób?** (np. "Sandbox", "Environment", "Variable")
- **Jaki jest kontekst biznesowy?** Co ten zasób reprezentuje?
- **Czy to nowy zasób czy rozszerzenie istniejącego?**

### Krok 2: Metody HTTP

Zapytaj które operacje są potrzebne:
- `GET /resources` - lista zasobów
- `GET /resources/{id}` - szczegóły zasobu
- `POST /resources` - tworzenie
- `PATCH /resources/{id}` - aktualizacja
- `DELETE /resources/{id}` - usuwanie

Dla każdej wybranej metody ustal szczegóły w kolejnych krokach.

### Krok 3: OAuth2 Scopes

Dla każdej metody HTTP zaproponuj scope bazując na konwencji:

| Operacja | Wzorzec Scope | Przykład |
|----------|---------------|----------|
| GET (list/detail) | `{RESOURCE}_INFO` lub `{RESOURCE}_READ` | `SANDBOX_INFO` |
| POST (create) | `{RESOURCE}_ADD` lub `{RESOURCE}_WRITE` | `SANDBOX_ADD` |
| PATCH (update) | `{RESOURCE}_MANAGE` | `SANDBOX_MANAGE` |
| DELETE | `{RESOURCE}_MANAGE` | `SANDBOX_MANAGE` |

Zapytaj:
- Czy scope'y powinny być nowe czy wykorzystać istniejące?
- Czy są specjalne przypadki (np. admin-only)?

**Sprawdź istniejące scope'y** w pliku:
```
application-core/src/main/java/works/buddy/oauth2/OAuth2Scope.java
```

### Krok 4: Sprawdzenie Internal API

**WAŻNE:** Przed projektowaniem REST API sprawdź czy istnieje już implementacja w Internal API.

Przeszukaj katalog:
```
application-server/src/main/java/works/buddy/application/internalapi/
```

Szukaj:
- Interfejsów `*InternalAPI.java` związanych z domeną
- Implementacji `Default*InternalAPI.java`
- Metod które mogą być reużyte

Jeśli Internal API istnieje:
- Zanotuj dostępne metody i ich sygnatury
- Sprawdź jakie modele `*Front` są używane
- Zaproponuj mapowanie między Internal API a REST API

Jeśli Internal API nie istnieje:
- Zanotuj, że trzeba będzie stworzyć serwisy domenowe
- Sprawdź czy istnieją powiązane serwisy w `/ws/*/services/`

### Krok 5: Model danych (View)

Zaprojektuj strukturę View bazując na wymaganiach:

**Zapytaj o pola:**
- Jakie pola powinien zawierać zasób?
- Które pola są wymagane (required)?
- Które pola są tylko do odczytu (read-only)?
- Czy są pola wrażliwe wymagające szyfrowania?

**Typy pól:**
- Proste: `String`, `Integer`, `Boolean`, `OffsetDateTime`
- Relacje: `Short*View` (dla zagnieżdżonych obiektów)
- Kolekcje: `Set<String>`, `Collection<*View>`
- Enumy: zdefiniuj dozwolone wartości

**Walidacja:**
- `@NotEmpty` - pole wymagane
- `@Pattern` - regex
- `@PositiveInt` - liczba dodatnia
- `@AvailableValues` - enum-like

### Krok 6: Short vs Full View

**Short View** (dla list):
- Tylko podstawowe pola identyfikujące
- Typowo: `id`, `name`, `status`, `createDate`
- 5-10 pól maksymalnie

**Full View** (dla szczegółów):
- Rozszerza Short View
- Wszystkie pola konfiguracyjne
- Pełne relacje z innymi zasobami
- Może mieć 20+ pól

Zapytaj:
- Które pola powinny być w Short View?
- Jakie relacje (Short*View) powinny być w Full View?

### Krok 7: Query Parameters

**Paginacja** (standardowa):
- `page` - numer strony (1-based)
- `per_page` - ilość na stronę (default 20, max 50)

**Sortowanie** (jeśli potrzebne):
- `sort_by` - pole do sortowania
- `sort_direction` - ASC/DESC

**Filtry** (specyficzne dla domeny):
Zapytaj jakie filtry są potrzebne, np.:
- `status` - filtr po statusie
- `project_name` - filtr po projekcie
- `environment_id` - filtr po środowisku

### Krok 8: Request/Response

**Request classes:**
- Czy potrzebne osobne klasy `Add*Request` i `Update*Request`?
- Czy wystarczy jedna klasa `*View` dla wszystkich operacji?
- Czy są różne warianty tworzenia (jak w ProjectApi)?

**Response classes:**
- `*View` - pojedynczy zasób
- `*sView` (np. `TargetsView`) - kolekcja z paginacją
- `PageableResourceView` - bazowa klasa dla paginowanych odpowiedzi

**Kody HTTP:**
- `200 OK` - GET, PATCH sukces
- `201 Created` - POST sukces (z headerem Location)
- `204 No Content` - DELETE sukces
- `400 Bad Request` - błąd walidacji
- `404 Not Found` - zasób nie istnieje

### Krok 9: Nested Resources

Zapytaj o hierarchię zasobów:
- Czy zasób jest zagnieżdżony pod workspace? (prawie zawsze tak)
- Czy jest zagnieżdżony pod projekt/pipeline/environment?
- Jaka jest ścieżka URL?

**Przykłady wzorców:**
```
/workspaces/{workspace_domain}/resources
/workspaces/{workspace_domain}/projects/{project_name}/resources
/workspaces/{workspace_domain}/projects/{project_name}/pipelines/{pipeline_id}/resources
```

### Krok 10: Generowanie planu

Na podstawie zebranych informacji wygeneruj plan implementacji zawierający:

## Format planu

```markdown
# Plan implementacji REST API: {NazwaZasobu}

## 1. Podsumowanie
- Nazwa zasobu: ...
- Ścieżka bazowa: /workspaces/{workspace_domain}/...
- Metody: GET (list), GET (detail), POST, PATCH, DELETE

## 2. OAuth2 Scopes
- {RESOURCE}_INFO - odczyt
- {RESOURCE}_ADD - tworzenie
- {RESOURCE}_MANAGE - edycja i usuwanie

Plik do modyfikacji: `application-core/.../OAuth2Scope.java`

## 3. Internal API
Status: [Istnieje / Nie istnieje / Częściowo]
- Dostępne metody: ...
- Brakujące metody: ...

## 4. Pliki do utworzenia

### View Models (application-core)
- `view/{resource}/model/{Resource}View.java` - pełny model
- `view/{resource}/model/Short{Resource}View.java` - model skrócony
- `view/{resource}/model/{Resource}sView.java` - kolekcja z paginacją
- `view/{resource}/model/Add{Resource}Request.java` - request POST (opcjonalnie)
- `view/{resource}/model/Update{Resource}Request.java` - request PATCH (opcjonalnie)

### OpenAPI Documentation (application-core)
- `view/openapi/{Resource}ApiDescriptions.java` - opisy pól
- `view/openapi/{Resource}ApiExamples.java` - przykłady JSON

### REST API (application-server)
- `api/view/{resource}/rest/{Resource}Api.java` - interfejs
- `api/view/{resource}/rest/Default{Resource}Api.java` - implementacja

### Services (application-server)
- `api/view/{resource}/services/{Resource}ViewService.java` - interfejs
- `api/view/{resource}/services/Default{Resource}ViewService.java` - implementacja

### Konfiguracja
- `server/config/RestApiConfig.java` - rejestracja endpointu

## 5. Struktura View

### Short{Resource}View
| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| id | String | read-only | Identyfikator |
| name | String | tak | Nazwa |
| ... | ... | ... | ... |

### {Resource}View (extends Short{Resource}View)
| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| ... | ... | ... | ... |

## 6. Endpointy

### GET /workspaces/{workspace_domain}/{resources}
- Scope: {RESOURCE}_INFO
- Query params: page, per_page, [filtry]
- Response: {Resource}sView (200)

### GET /workspaces/{workspace_domain}/{resources}/{id}
- Scope: {RESOURCE}_INFO
- Response: {Resource}View (200)

### POST /workspaces/{workspace_domain}/{resources}
- Scope: {RESOURCE}_ADD
- Request: Add{Resource}Request / {Resource}View
- Response: {Resource}View (201)

### PATCH /workspaces/{workspace_domain}/{resources}/{id}
- Scope: {RESOURCE}_MANAGE
- Request: Update{Resource}Request / {Resource}View
- Response: {Resource}View (200)

### DELETE /workspaces/{workspace_domain}/{resources}/{id}
- Scope: {RESOURCE}_MANAGE
- Response: 204 No Content

## 7. Przykłady JSON

### Request POST
\`\`\`json
{
  "name": "example",
  ...
}
\`\`\`

### Response
\`\`\`json
{
  "url": "/api/workspaces/example/resources/123",
  "htmlUrl": "https://app.buddy.works/example/resources/123",
  "id": "123",
  "name": "example",
  ...
}
\`\`\`

## 8. Referencje
- Podobne API do wzorowania: [nazwa] - `ścieżka do pliku`
- Internal API: `ścieżka do pliku`
```

## Dobre praktyki

1. **Zawsze sprawdzaj istniejący kod** przed sugerowaniem rozwiązań
2. **Używaj narzędzi Grep/Glob** do przeszukiwania repozytorium
3. **Odwołuj się do konkretnych plików** jako przykładów
4. **Pytaj o edge cases** - co jeśli zasób nie istnieje? co jeśli brak uprawnień?
5. **Waliduj nazewnictwo** - czy nazwa scope'a nie koliduje z istniejącymi?

## Pliki referencyjne

Zapoznaj się z plikami w katalogu `references/`:
- `patterns.md` - wzorce kodu używane w projekcie
- `examples.md` - przykłady dobrych implementacji
- `scopes.md` - lista istniejących OAuth2 scopes

---

## Przykładowy output

Poniżej znajduje się przykład kompletnego planu implementacji wygenerowanego przez asystenta:

```markdown
# Plan implementacji REST API: Deployment

## 1. Podsumowanie

| Aspekt | Wartość |
|--------|---------|
| Nazwa zasobu | Deployment |
| Ścieżka bazowa | `/workspaces/{workspace_domain}/projects/{project_name}/deployments` |
| Metody | GET (list), GET (detail), POST, DELETE |
| Zagnieżdżenie | Workspace → Project → Deployment |

## 2. OAuth2 Scopes

### Nowe scope'y do dodania

| Scope | Opis | Parent |
|-------|------|--------|
| `DEPLOYMENT_INFO` | Odczyt deploymentów | `DEPLOYMENT_MANAGE` |
| `DEPLOYMENT_ADD` | Tworzenie deploymentów | `DEPLOYMENT_MANAGE` |
| `DEPLOYMENT_MANAGE` | Pełne zarządzanie | `null` |

### Mapowanie na operacje

| Endpoint | Scope |
|----------|-------|
| GET /deployments | `DEPLOYMENT_INFO` |
| GET /deployments/{id} | `DEPLOYMENT_INFO` |
| POST /deployments | `DEPLOYMENT_ADD` |
| DELETE /deployments/{id} | `DEPLOYMENT_MANAGE` |

### Plik do modyfikacji
`application-core/src/main/java/works/buddy/oauth2/OAuth2Scope.java`

```java
// Dodać po istniejących scope'ach:
DEPLOYMENT_INFO("deployment:info", DEPLOYMENT_MANAGE),
DEPLOYMENT_ADD("deployment:add", DEPLOYMENT_MANAGE),
DEPLOYMENT_MANAGE("deployment:manage", null),
```

## 3. Internal API

### Status: Częściowo istnieje

**Znalezione metody w `DefaultPipelineInternalApi.java`:**
- `getDeployments(Integer pipelineId, String domain, Integer invokerId)` - lista deploymentów
- `getDeployment(Integer deploymentId, String domain, Integer invokerId)` - szczegóły

**Brakujące metody:**
- `addDeployment(DeploymentFront deployment, String domain, Integer invokerId)` - tworzenie
- `deleteDeployment(Integer deploymentId, String domain, Integer invokerId)` - usuwanie

### Istniejące modele
- `DeploymentFront` - model domenowy (do mapowania na View)

### Rekomendacja
Rozszerzyć `PipelineInternalApi` o brakujące metody lub stworzyć dedykowany `DeploymentInternalAPI`.

## 4. Pliki do utworzenia

### View Models (application-core)

```
view/deployment/model/
├── DeploymentView.java           # Pełny model
├── ShortDeploymentView.java      # Model skrócony (dla list)
├── DeploymentsView.java          # Kolekcja z paginacją
└── AddDeploymentRequest.java     # Request dla POST
```

### OpenAPI Documentation (application-core)

```
view/openapi/
├── DeploymentApiDescriptions.java    # Opisy pól i operacji
└── DeploymentApiExamples.java        # Przykłady JSON
```

### REST API (application-server)

```
api/view/deployment/
├── rest/
│   ├── DeploymentApi.java            # Interfejs JAX-RS
│   └── DefaultDeploymentApi.java     # Implementacja
└── services/
    ├── DeploymentViewService.java    # Interfejs serwisu
    └── DefaultDeploymentViewService.java  # Implementacja
```

### Konfiguracja

```
server/config/RestApiConfig.java      # Rejestracja endpointu
```

## 5. Struktura View

### ShortDeploymentView (dla list)

| Pole | Typ | Wymagane | Read-only | Opis |
|------|-----|----------|-----------|------|
| `id` | `String` | - | tak | Identyfikator deploymentu |
| `name` | `String` | tak | - | Nazwa deploymentu |
| `status` | `String` | - | tak | Status: `PENDING`, `RUNNING`, `SUCCESS`, `FAILED` |
| `environment` | `ShortEnvironmentView` | - | tak | Środowisko docelowe |
| `startDate` | `OffsetDateTime` | - | tak | Data rozpoczęcia |
| `finishDate` | `OffsetDateTime` | - | tak | Data zakończenia |

### DeploymentView (extends ShortDeploymentView)

| Pole | Typ | Wymagane | Read-only | Opis |
|------|-----|----------|-----------|------|
| `description` | `String` | - | - | Opis deploymentu |
| `pipeline` | `ShortPipelineView` | - | tak | Pipeline źródłowy |
| `execution` | `ShortExecutionView` | - | tak | Powiązane wykonanie |
| `target` | `ShortTargetView` | - | tak | Target docelowy |
| `creator` | `MemberView` | - | tak | Kto utworzył |
| `revision` | `String` | - | tak | Rewizja/commit |
| `tags` | `Set<String>` | - | - | Tagi |
| `variables` | `Map<String, String>` | - | - | Zmienne deploymentu |
| `logs` | `String` | - | tak | URL do logów |

### AddDeploymentRequest

| Pole | Typ | Wymagane | Walidacja | Opis |
|------|-----|----------|-----------|------|
| `name` | `String` | tak | `@NotEmpty` | Nazwa deploymentu |
| `description` | `String` | - | - | Opis |
| `environmentId` | `String` | tak | `@NotEmpty`, `@HashId` | ID środowiska |
| `targetId` | `String` | - | `@HashId` | ID targetu (opcjonalnie) |
| `revision` | `String` | - | - | Rewizja do deployu |
| `tags` | `Set<String>` | - | - | Tagi |
| `variables` | `Map<String, String>` | - | - | Zmienne |

## 6. Endpointy

### GET /workspaces/{workspace_domain}/projects/{project_name}/deployments

Lista deploymentów z paginacją i filtrami.

| Aspekt | Wartość |
|--------|---------|
| Scope | `DEPLOYMENT_INFO` |
| Response | `DeploymentsView` (200) |

**Query Parameters:**

| Param | Typ | Default | Opis |
|-------|-----|---------|------|
| `page` | Integer | 1 | Numer strony |
| `per_page` | Integer | 20 | Ilość na stronę (max 50) |
| `status` | String | - | Filtr po statusie |
| `environment_id` | String | - | Filtr po środowisku |
| `sort_by` | String | `start_date` | Pole sortowania: `name`, `start_date`, `status` |
| `sort_direction` | String | `DESC` | Kierunek: `ASC`, `DESC` |

---

### GET /workspaces/{workspace_domain}/projects/{project_name}/deployments/{deployment_id}

Szczegóły pojedynczego deploymentu.

| Aspekt | Wartość |
|--------|---------|
| Scope | `DEPLOYMENT_INFO` |
| Response | `DeploymentView` (200) |
| Error | `Error` (404) - nie znaleziono |

---

### POST /workspaces/{workspace_domain}/projects/{project_name}/deployments

Utworzenie nowego deploymentu.

| Aspekt | Wartość |
|--------|---------|
| Scope | `DEPLOYMENT_ADD` |
| Request | `AddDeploymentRequest` |
| Response | `DeploymentView` (201) + header `Location` |
| Error | `Errors` (400) - błąd walidacji |

---

### DELETE /workspaces/{workspace_domain}/projects/{project_name}/deployments/{deployment_id}

Usunięcie deploymentu.

| Aspekt | Wartość |
|--------|---------|
| Scope | `DEPLOYMENT_MANAGE` |
| Response | 204 No Content |
| Error | `Error` (404) - nie znaleziono |

## 7. Przykłady JSON

### POST Request - Utworzenie deploymentu

```json
{
  "name": "Production Release v2.1.0",
  "description": "Release nowej wersji na produkcję",
  "environmentId": "env_abc123",
  "targetId": "tgt_xyz789",
  "revision": "a1b2c3d4e5f6",
  "tags": ["release", "production", "v2.1.0"],
  "variables": {
    "DEPLOY_MODE": "rolling",
    "REPLICA_COUNT": "3"
  }
}
```

### GET Response - Lista deploymentów

```json
{
  "url": "/api/workspaces/acme/projects/webapp/deployments",
  "page": 1,
  "pageSize": 20,
  "totalPageCount": 3,
  "elementCount": 20,
  "totalElementCount": 45,
  "deployments": [
    {
      "url": "/api/workspaces/acme/projects/webapp/deployments/dep_123",
      "htmlUrl": "https://app.buddy.works/acme/webapp/deployments/dep_123",
      "id": "dep_123",
      "name": "Production Release v2.1.0",
      "status": "SUCCESS",
      "environment": {
        "url": "/api/workspaces/acme/environments/env_abc123",
        "id": "env_abc123",
        "name": "Production"
      },
      "startDate": "2024-01-15T10:30:00Z",
      "finishDate": "2024-01-15T10:35:42Z"
    }
  ]
}
```

### GET Response - Szczegóły deploymentu

```json
{
  "url": "/api/workspaces/acme/projects/webapp/deployments/dep_123",
  "htmlUrl": "https://app.buddy.works/acme/webapp/deployments/dep_123",
  "id": "dep_123",
  "name": "Production Release v2.1.0",
  "description": "Release nowej wersji na produkcję",
  "status": "SUCCESS",
  "environment": {
    "url": "/api/workspaces/acme/environments/env_abc123",
    "id": "env_abc123",
    "name": "Production"
  },
  "pipeline": {
    "url": "/api/workspaces/acme/projects/webapp/pipelines/123",
    "id": 123,
    "name": "Deploy Pipeline"
  },
  "execution": {
    "url": "/api/workspaces/acme/projects/webapp/pipelines/123/executions/456",
    "id": 456,
    "status": "SUCCESSFUL"
  },
  "target": {
    "url": "/api/workspaces/acme/targets/tgt_xyz789",
    "id": "tgt_xyz789",
    "name": "Production Server",
    "type": "SSH"
  },
  "creator": {
    "url": "/api/workspaces/acme/members/1",
    "id": 1,
    "name": "Jan Kowalski",
    "avatarUrl": "https://..."
  },
  "revision": "a1b2c3d4e5f6",
  "tags": ["release", "production", "v2.1.0"],
  "variables": {
    "DEPLOY_MODE": "rolling",
    "REPLICA_COUNT": "3"
  },
  "logs": "https://app.buddy.works/acme/webapp/deployments/dep_123/logs",
  "startDate": "2024-01-15T10:30:00Z",
  "finishDate": "2024-01-15T10:35:42Z"
}
```

### Error Response - Walidacja (400)

```json
{
  "errors": [
    {
      "message": "Field 'name' is required"
    },
    {
      "message": "Field 'environmentId' is required"
    }
  ]
}
```

### Error Response - Not Found (404)

```json
{
  "message": "Deployment 'dep_999' not found"
}
```

## 8. Referencje

### Podobne API do wzorowania

| API | Ścieżka | Dlaczego |
|-----|---------|----------|
| ExecutionApi | `api/view/execution/rest/ExecutionApi.java` | Podobna struktura (lista + szczegóły), powiązanie z pipeline |
| TargetApi | `api/view/target/rest/TargetApi.java` | Wzorzec dla Short vs Full view |
| VariableApi | `api/view/variable/rest/VariableApi.java` | Wzorzec dla różnych scope'ów |

### Internal API

| Plik | Opis |
|------|------|
| `internalapi/DefaultPipelineInternalApi.java` | Istniejące metody deploymentów |
| `internalapi/DefaultEnvironmentInternalAPI.java` | Wzorzec dla nowego Internal API |

### Modele domenowe

| Plik | Opis |
|------|------|
| `ws/deployment/model/DeploymentFront.java` | Model domenowy do mapowania |
| `ws/environment/model/EnvironmentFront.java` | Przykład mapowania Front → View |

## 9. Checklist implementacji

- [ ] Dodać scope'y do `OAuth2Scope.java`
- [ ] Stworzyć `ShortDeploymentView.java`
- [ ] Stworzyć `DeploymentView.java`
- [ ] Stworzyć `DeploymentsView.java`
- [ ] Stworzyć `AddDeploymentRequest.java`
- [ ] Stworzyć `DeploymentApiDescriptions.java`
- [ ] Stworzyć `DeploymentApiExamples.java`
- [ ] Stworzyć `DeploymentApi.java` (interfejs)
- [ ] Stworzyć `DefaultDeploymentApi.java` (implementacja)
- [ ] Stworzyć `DeploymentViewService.java` (interfejs)
- [ ] Stworzyć `DefaultDeploymentViewService.java` (implementacja)
- [ ] Zarejestrować endpoint w `RestApiConfig.java`
- [ ] Rozszerzyć Internal API o brakujące metody (opcjonalnie)
- [ ] Napisać testy jednostkowe
- [ ] Napisać testy integracyjne
- [ ] Zaktualizować dokumentację OpenAPI
```
