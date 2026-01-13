# OAuth2 Scopes

## Lokalizacja definicji

```
application-core/src/main/java/works/buddy/oauth2/OAuth2Scope.java
```

## Hierarchia scope'ów

Scope'y są zorganizowane hierarchicznie - wyższy poziom zawiera uprawnienia niższego:

```
INFO/READ (odczyt)
    ↓
ADD/WRITE (tworzenie)
    ↓
MANAGE (pełna kontrola)
```

## Lista istniejących scope'ów

### Workspace

| Scope | Poziom | Opis |
|-------|--------|------|
| `WORKSPACE` | read | Podstawowy dostęp do workspace |

### Repository

| Scope | Poziom | Opis |
|-------|--------|------|
| `REPOSITORY_READ` | read | Odczyt repozytorium |
| `REPOSITORY_WRITE` | write | Zapis do repozytorium |

### Execution (Pipeline runs)

| Scope | Poziom | Opis |
|-------|--------|------|
| `EXECUTION_INFO` | read | Informacje o wykonaniach |
| `EXECUTION_RUN` | write | Uruchamianie pipeline'ów |
| `EXECUTION_MANAGE` | manage | Zarządzanie wykonaniami |

### Variable

| Scope | Poziom | Opis |
|-------|--------|------|
| `VARIABLE_INFO` | read | Odczyt zmiennych |
| `VARIABLE_ADD` | write | Dodawanie zmiennych |
| `VARIABLE_MANAGE` | manage | Zarządzanie zmiennymi |

### Webhook

| Scope | Poziom | Opis |
|-------|--------|------|
| `WEBHOOK_INFO` | read | Odczyt webhooków |
| `WEBHOOK_ADD` | write | Dodawanie webhooków |
| `WEBHOOK_MANAGE` | manage | Zarządzanie webhookami |

### Integration

| Scope | Poziom | Opis |
|-------|--------|------|
| `INTEGRATION_INFO` | read | Odczyt integracji |
| `INTEGRATION_ADD` | write | Dodawanie integracji |
| `INTEGRATION_MANAGE` | manage | Zarządzanie integracjami |

### Token

| Scope | Poziom | Opis |
|-------|--------|------|
| `TOKEN_INFO` | read | Odczyt tokenów |
| `TOKEN_MANAGE` | manage | Zarządzanie tokenami |

### Target

| Scope | Poziom | Opis |
|-------|--------|------|
| `TARGET_INFO` | read | Odczyt targetów |
| `TARGET_ADD` | write | Dodawanie targetów |
| `TARGET_MANAGE` | manage | Zarządzanie targetami |

### Environment

| Scope | Poziom | Opis |
|-------|--------|------|
| `ENVIRONMENT_INFO` | read | Odczyt środowisk |
| `ENVIRONMENT_ADD` | write | Dodawanie środowisk |
| `ENVIRONMENT_MANAGE` | manage | Zarządzanie środowiskami |

### Package

| Scope | Poziom | Opis |
|-------|--------|------|
| `PACKAGE_READ` | read | Odczyt pakietów |
| `PACKAGE_WRITE` | write | Zapis pakietów |
| `PACKAGE_MANAGE` | manage | Zarządzanie pakietami |

### Zone

| Scope | Poziom | Opis |
|-------|--------|------|
| `ZONE_READ` | read | Odczyt stref |
| `ZONE_WRITE` | write | Zapis stref |
| `ZONE_MANAGE` | manage | Zarządzanie strefami |

### Sandbox

| Scope | Poziom | Opis |
|-------|--------|------|
| `SANDBOX_INFO` | read | Odczyt sandboxów |
| `SANDBOX_MANAGE` | manage | Zarządzanie sandboxami |

### Member

| Scope | Poziom | Opis |
|-------|--------|------|
| `MEMBER_EMAIL` | read | Dostęp do emaili członków |

### User

| Scope | Poziom | Opis |
|-------|--------|------|
| `USER_INFO` | read | Informacje o użytkowniku |
| `USER_KEY` | read | Klucze użytkownika |
| `USER_EMAIL` | read | Email użytkownika |
| `MANAGE_EMAILS` | manage | Zarządzanie emailami |

### Project

| Scope | Poziom | Opis |
|-------|--------|------|
| `PROJECT_DELETE` | manage | Usuwanie projektów |

## Wzorzec nazewnictwa dla nowych scope'ów

```
{RESOURCE}_{LEVEL}
```

Gdzie:
- `{RESOURCE}` - nazwa zasobu WIELKIMI LITERAMI (np. `SANDBOX`, `TARGET`)
- `{LEVEL}` - poziom dostępu:
  - `INFO` lub `READ` - odczyt
  - `ADD` lub `WRITE` - tworzenie
  - `MANAGE` - pełna kontrola

## Przykład dodawania nowego scope'a

```java
// W OAuth2Scope.java

// Dla nowego zasobu "DEPLOYMENT"
DEPLOYMENT_INFO("deployment:info", DEPLOYMENT_READ),
DEPLOYMENT_ADD("deployment:add", DEPLOYMENT_MANAGE),
DEPLOYMENT_MANAGE("deployment:manage", null),

// Konstruktor: OAuth2Scope(String value, OAuth2Scope parent)
// parent = scope wyższego poziomu (null dla najwyższego)
```

## Mapowanie na operacje HTTP

| HTTP Method | Operacja | Zalecany poziom scope |
|-------------|----------|----------------------|
| GET (list) | Odczyt listy | `*_INFO` / `*_READ` |
| GET (detail) | Odczyt szczegółów | `*_INFO` / `*_READ` |
| POST | Tworzenie | `*_ADD` / `*_WRITE` |
| PATCH / PUT | Aktualizacja | `*_MANAGE` |
| DELETE | Usuwanie | `*_MANAGE` |

## Użycie w API

```java
// Odczyt
@Operation(security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_INFO"}))
Response getResources(...);

// Tworzenie
@Operation(security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_ADD"}))
Response addResource(...);

// Aktualizacja/Usuwanie
@Operation(security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_MANAGE"}))
Response updateResource(...);
```

## Specjalne przypadki

### Scope dla workspace-level operations

Niektóre operacje używają ogólnego scope `WORKSPACE`:
- Zarządzanie grupami
- Zarządzanie członkami workspace
- Podstawowe operacje administracyjne

### Scope dla admin-only operations

Operacje tylko dla adminów nie używają OAuth2 scope, ale `@Access` annotation:
```java
@Access(level = AdminAccessLevel.class)
```

### Wiele scope'ów

Czasem operacja wymaga więcej niż jednego scope:
```java
@Operation(security = @SecurityRequirement(name = "oauth2",
    scopes = {"REPOSITORY_READ", "EXECUTION_INFO"}))
```
