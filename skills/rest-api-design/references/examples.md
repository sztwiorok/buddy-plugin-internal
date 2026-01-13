# Przykłady implementacji REST API

## Przykład 1: GroupApi (prosty CRUD)

Dobry przykład dla podstawowego API z nested resources (grupy i członkowie grup).

### Lokalizacja plików

```
application-server/src/main/java/works/buddy/application/api/view/group/
├── rest/
│   ├── GroupApi.java
│   └── DefaultGroupApi.java
└── services/
    ├── GroupViewService.java
    └── DefaultGroupViewService.java

application-core/src/main/java/works/buddy/view/group/
└── model/
    ├── GroupView.java
    ├── GroupsView.java
    ├── AddGroupRequest.java
    └── UpdateGroupRequest.java
```

### Charakterystyka

- **Scope**: `WORKSPACE` dla wszystkich operacji
- **Nested resources**: `/groups/{id}/members`
- **Request classes**: Osobne `AddGroupRequest` i `UpdateGroupRequest` (record)
- **Walidacja**: `@NotEmpty` z grupami walidacyjnymi

### Kluczowe fragmenty

```java
// AddGroupRequest.java - użycie record
public record AddGroupRequest(
    @NotEmpty(viewNameField = "name", groups = {AddGroupGroup.class})
    @Schema(description = GROUP_NAME)
    String name,

    @Schema(description = GROUP_DESCRIPTION)
    String description,

    @Schema(description = GROUP_AUTO_ASSIGN_TO_NEW_PROJECTS)
    Boolean autoAssignToNewProjects
) {}
```

---

## Przykład 2: VariableApi (różne scope'y)

Dobry przykład dla API z różnymi scope'ami dla różnych operacji.

### Lokalizacja plików

```
application-server/src/main/java/works/buddy/application/api/view/variable/
├── rest/
│   ├── VariableApi.java
│   └── DefaultVariableApi.java
└── services/
    └── ...

application-core/src/main/java/works/buddy/view/variable/
└── model/
    ├── VariableView.java
    ├── VariablesView.java
    └── ...
```

### Charakterystyka

- **Scope'y**:
  - `VARIABLE_INFO` - GET
  - `VARIABLE_ADD` - POST
  - `VARIABLE_MANAGE` - PATCH, DELETE
- **Query params**: Wiele filtrów (`project_name`, `pipeline_id`, `environment_id`, `action_id`)
- **Szyfrowanie**: Wrażliwe wartości zmiennych są szyfrowane

### Kluczowe fragmenty

```java
// Różne scope'y per operacja
@GET
@Operation(security = @SecurityRequirement(name = "oauth2", scopes = {"VARIABLE_INFO"}))
Response getVariables(...);

@POST
@Operation(security = @SecurityRequirement(name = "oauth2", scopes = {"VARIABLE_ADD"}))
Response addVariable(...);

@PATCH
@Operation(security = @SecurityRequirement(name = "oauth2", scopes = {"VARIABLE_MANAGE"}))
Response updateVariable(...);

// Wiele filtrów query
Response getVariables(
    @QueryParam("project_name") String projectName,
    @QueryParam("pipeline_id") Integer pipelineId,
    @HashId(HashIdType.ENVIRONMENT) @QueryParam("environment_id") Integer environmentId,
    @QueryParam("action_id") Integer actionId,
    @QueryParam("sandbox_id") String sandboxId
);
```

---

## Przykład 3: TargetApi (polimorfizm)

Dobry przykład dla API z wieloma typami zasobu (różne typy targetów).

### Lokalizacja plików

```
application-server/src/main/java/works/buddy/application/api/view/target/
├── rest/
│   ├── TargetApi.java
│   └── DefaultTargetApi.java
└── services/
    └── ...

application-core/src/main/java/works/buddy/view/target/
└── model/
    ├── TargetView.java (abstract)
    ├── TargetSSHView.java
    ├── TargetDockerHubView.java
    ├── TargetKubernetesView.java
    └── ... (więcej typów)
```

### Charakterystyka

- **Polimorfizm**: Abstrakcyjna klasa bazowa `TargetView` z wieloma implementacjami
- **JSON Type Info**: Automatyczne mapowanie na podstawie pola `type`
- **Scope'y**: `TARGET_INFO`, `TARGET_ADD`, `TARGET_MANAGE`
- **Szyfrowanie**: Hasła i klucze są szyfrowane

### Kluczowe fragmenty

```java
// TargetView.java - polimorfizm
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.EXISTING_PROPERTY, property = "type")
@JsonSubTypes({
    @JsonSubTypes.Type(value = TargetSSHView.class, name = TargetSSHView.JSON_TYPE),
    @JsonSubTypes.Type(value = TargetDockerHubView.class, name = TargetDockerHubView.JSON_TYPE),
    @JsonSubTypes.Type(value = TargetKubernetesView.class, name = TargetKubernetesView.JSON_TYPE),
    // ...
})
public abstract class TargetView extends ResourceView {
    public String type;
    public String identifier;
    public String name;
    // wspólne pola...
}

// TargetSSHView.java - konkretna implementacja
public class TargetSSHView extends TargetView {
    public static final String JSON_TYPE = "SSH";

    public String host;
    public String port;
    public String login;

    @Encrypted
    public String password;

    @Encrypted
    public String privateKey;
}
```

---

## Przykład 4: ProjectApi (złożone nested resources)

Dobry przykład dla API z wieloma poziomami zagnieżdżenia i różnymi typami requestów.

### Lokalizacja plików

```
application-server/src/main/java/works/buddy/application/api/view/project/
├── rest/
│   ├── ProjectApi.java
│   └── DefaultProjectApi.java
└── services/
    └── ...

application-core/src/main/java/works/buddy/view/project/
└── model/
    ├── ProjectView.java
    ├── ShortProjectView.java
    ├── ProjectsView.java
    ├── CreateBuddyProjectRequest.java
    ├── CreateIntegratedProjectRequest.java
    ├── CreateCustomRepoProjectRequest.java
    └── ...
```

### Charakterystyka

- **Wiele typów requestów**: 3 różne klasy dla POST (różne sposoby tworzenia projektu)
- **Short vs Full View**: `ShortProjectView` dla list, `ProjectView` dla szczegółów
- **Nested resources**: members, groups, repository, pipelines, environments
- **Paginacja + sortowanie + filtry**: Pełne wsparcie
- **Różne scope'y**: `WORKSPACE`, `REPOSITORY_READ`, `REPOSITORY_WRITE`, `PROJECT_DELETE`

### Kluczowe fragmenty

```java
// Wiele typów requestów dla POST
@POST
@Path("/")
@Operation(
    summary = ADD_PROJECT,
    requestBody = @RequestBody(content = @Content(
        schema = @Schema(oneOf = {
            CreateBuddyProjectRequest.class,
            CreateIntegratedProjectRequest.class,
            CreateCustomRepoProjectRequest.class
        }),
        examples = {
            @ExampleObject(name = "Buddy project", value = BUDDY_PROJECT_REQUEST),
            @ExampleObject(name = "GitHub project", value = GITHUB_PROJECT_REQUEST),
            @ExampleObject(name = "Custom repo", value = CUSTOM_REPO_PROJECT_REQUEST)
        }
    ))
)
Response addProject(...);

// Paginacja + sortowanie + filtry
@GET
@Path("/")
Response getProjects(
    @QueryParam("page") Integer page,
    @QueryParam("per_page") Integer perPage,
    @QueryParam("status") String status,
    @QueryParam("sort_by") String sortBy,
    @QueryParam("sort_direction") String sortDirection,
    @QueryParam("membership") String membership
);

// Nested resources z własnymi filtrami
@GET
@Path("/{project_name}/repository/commits")
Response getCommits(
    @QueryParam("page") Integer page,
    @QueryParam("per_page") Integer perPage,
    @QueryParam("path") String path,
    @QueryParam("branch") String branch,
    @QueryParam("author") String author,
    @QueryParam("since") String since,
    @QueryParam("until") String until
);
```

---

## Przykład 5: WebhookApi (z HashId)

Dobry przykład dla API używającego HashId do kodowania/dekodowania identyfikatorów.

### Lokalizacja plików

```
application-server/src/main/java/works/buddy/application/api/view/webhook/
├── rest/
│   ├── WebhookApi.java
│   └── DefaultWebhookApi.java
└── services/
    └── ...
```

### Charakterystyka

- **HashId**: Identyfikatory są kodowane/dekodowane automatycznie
- **Scope'y**: `WEBHOOK_INFO`, `WEBHOOK_ADD`, `WEBHOOK_MANAGE`
- **Short View**: `ShortWebhookView` dla list

### Kluczowe fragmenty

```java
// Użycie HashId
@GET
@Path("/{hash_id}")
Response getWebhook(
    @HashId(HashIdType.WEBHOOK) @PathParam("hash_id") Integer webhookId,
    ...
);

@PATCH
@Path("/{hash_id}")
Response updateWebhook(
    @HashId(HashIdType.WEBHOOK) @PathParam("hash_id") Integer webhookId,
    ...
);
```

---

## Przykład 6: PermissionSetApi (grupy walidacyjne)

Dobry przykład dla API z różnymi grupami walidacyjnymi dla różnych operacji.

### Lokalizacja plików

```
application-server/src/main/java/works/buddy/application/api/view/permission/
├── rest/
│   ├── PermissionSetApi.java
│   └── DefaultPermissionSetApi.java
└── services/
    └── ...

application-core/src/main/java/works/buddy/view/permission/
└── model/
    ├── BasePermissionSetView.java
    ├── AddPermissionSetRequest.java
    ├── UpdatePermissionSetRequest.java
    └── ...
```

### Charakterystyka

- **Grupy walidacyjne**: Różna walidacja dla POST vs PATCH
- **Dziedziczenie**: `AddPermissionSetRequest` i `UpdatePermissionSetRequest` dziedziczą z `BasePermissionSetView`
- **Walidacja z grupami**: `@ValidWithGroups` na poziomie metody

### Kluczowe fragmenty

```java
// BasePermissionSetView.java - pola z grupami walidacyjnymi
public abstract class BasePermissionSetView extends ResourceView {
    @NotEmpty(viewNameField = "name", groups = {AddPermissionSet.class})
    @Schema(description = PERMISSION_SET_NAME, requiredMode = Schema.RequiredMode.REQUIRED)
    public String name;

    @Pattern(viewNameField = "pipeline_access_level",
        regex = "DENIED|READ_ONLY|RUN_ONLY|READ_WRITE",
        groups = {AddPermissionSet.class, UpdatePermissionSet.class})
    public String pipelineAccessLevel;
}

// API z walidacją grup
@POST
@Path("/")
@ValidWithGroups(AddPermissionSet.class)
@Operation(summary = CREATE_PERMISSION_SET, ...)
Response addPermissionSet(..., @Valid AddPermissionSetRequest request);

@PATCH
@Path("/{id}")
@ValidWithGroups(UpdatePermissionSet.class)
@Operation(summary = UPDATE_PERMISSION_SET, ...)
Response updatePermissionSet(..., @Valid UpdatePermissionSetRequest request);
```

---

## Podsumowanie - kiedy używać którego przykładu

| Przypadek | Przykład do wzorowania |
|-----------|------------------------|
| Prosty CRUD | GroupApi |
| Różne scope'y per operacja | VariableApi |
| Polimorficzne typy zasobu | TargetApi |
| Nested resources, wiele filtrów | ProjectApi |
| HashId dla identyfikatorów | WebhookApi |
| Grupy walidacyjne | PermissionSetApi |
