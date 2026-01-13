# Wzorce kodu REST API

## Struktura katalogów

```
application-core/src/main/java/works/buddy/view/
├── {feature}/
│   └── model/
│       ├── {Feature}View.java           # Pełny model
│       ├── Short{Feature}View.java      # Model skrócony (dla list)
│       ├── {Feature}sView.java          # Kolekcja z paginacją
│       ├── Add{Feature}Request.java     # Request dla POST (opcjonalnie)
│       └── Update{Feature}Request.java  # Request dla PATCH (opcjonalnie)
├── common/
│   └── model/
│       ├── ResourceView.java            # Bazowa klasa dla wszystkich View
│       └── PageableResourceView.java    # Bazowa dla paginowanych kolekcji
└── openapi/
    ├── {Feature}ApiDescriptions.java    # Opisy pól dla OpenAPI
    └── {Feature}ApiExamples.java        # Przykłady JSON

application-server/src/main/java/works/buddy/application/api/view/
├── {feature}/
│   ├── rest/
│   │   ├── {Feature}Api.java            # Interfejs JAX-RS
│   │   └── Default{Feature}Api.java     # Implementacja
│   └── services/
│       ├── {Feature}ViewService.java    # Interfejs serwisu
│       └── Default{Feature}ViewService.java  # Implementacja
```

## Wzorzec interfejsu API

```java
@CrossOriginResourceSharing(allowAllOrigins = true)
@Produces(ApiConstants.APPLICATION_JSON)
@Consumes(ApiConstants.APPLICATION_JSON)
@Path("/workspaces/{workspace_domain}/resources")
@Tag(name = "Resource API")
public interface ResourceApi {

    // Preflight dla CORS
    @OPTIONS
    @Path("/")
    @LocalPreflight
    Response preflightResources();

    @OPTIONS
    @Path("/{id}")
    @LocalPreflight
    Response preflightResource();

    // Lista zasobów
    @GET
    @Path("/")
    @Operation(
        summary = GET_RESOURCES,
        security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_INFO"}),
        responses = {
            @ApiResponse(responseCode = "200", description = "Success",
                content = @Content(schema = @Schema(implementation = ResourcesView.class)))
        }
    )
    Response getResources(
        @HeaderParam(CustomHeaders.USER) Integer invokerId,
        @HeaderParam(CustomHeaders.TOKEN) String token,
        @PathParam("workspace_domain") String domain,
        @QueryParam("page") Integer page,
        @QueryParam("per_page") Integer perPage
    ) throws CommonException;

    // Szczegóły zasobu
    @GET
    @Path("/{id}")
    @Operation(
        summary = GET_RESOURCE,
        security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_INFO"}),
        responses = {
            @ApiResponse(responseCode = "200", description = "Success",
                content = @Content(schema = @Schema(implementation = ResourceView.class))),
            @ApiResponse(responseCode = "404", description = "Not found",
                content = @Content(schema = @Schema(implementation = Error.class)))
        }
    )
    Response getResource(
        @HeaderParam(CustomHeaders.USER) Integer invokerId,
        @HeaderParam(CustomHeaders.TOKEN) String token,
        @PathParam("workspace_domain") String domain,
        @PathParam("id") String id
    ) throws CommonException;

    // Tworzenie zasobu
    @POST
    @Path("/")
    @Operation(
        summary = ADD_RESOURCE,
        security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_ADD"}),
        responses = {
            @ApiResponse(responseCode = "201", description = "Created",
                content = @Content(schema = @Schema(implementation = ResourceView.class))),
            @ApiResponse(responseCode = "400", description = "Bad request",
                content = @Content(schema = @Schema(implementation = Errors.class)))
        }
    )
    Response addResource(
        @HeaderParam(CustomHeaders.USER) Integer invokerId,
        @HeaderParam(CustomHeaders.TOKEN) String token,
        @PathParam("workspace_domain") String domain,
        @Valid AddResourceRequest request
    ) throws CommonException, ViolationException;

    // Aktualizacja zasobu
    @PATCH
    @Path("/{id}")
    @Operation(
        summary = UPDATE_RESOURCE,
        security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_MANAGE"}),
        responses = {
            @ApiResponse(responseCode = "200", description = "Success",
                content = @Content(schema = @Schema(implementation = ResourceView.class))),
            @ApiResponse(responseCode = "404", description = "Not found",
                content = @Content(schema = @Schema(implementation = Error.class)))
        }
    )
    Response updateResource(
        @HeaderParam(CustomHeaders.USER) Integer invokerId,
        @HeaderParam(CustomHeaders.TOKEN) String token,
        @PathParam("workspace_domain") String domain,
        @PathParam("id") String id,
        @Valid UpdateResourceRequest request
    ) throws CommonException, ViolationException;

    // Usuwanie zasobu
    @DELETE
    @Path("/{id}")
    @Operation(
        summary = DELETE_RESOURCE,
        security = @SecurityRequirement(name = "oauth2", scopes = {"RESOURCE_MANAGE"}),
        responses = {
            @ApiResponse(responseCode = "204", description = "Deleted"),
            @ApiResponse(responseCode = "404", description = "Not found",
                content = @Content(schema = @Schema(implementation = Error.class)))
        }
    )
    Response deleteResource(
        @HeaderParam(CustomHeaders.USER) Integer invokerId,
        @HeaderParam(CustomHeaders.TOKEN) String token,
        @PathParam("workspace_domain") String domain,
        @PathParam("id") String id
    ) throws CommonException;
}
```

## Wzorzec implementacji API

```java
@Service
public class DefaultResourceApi implements ResourceApi {

    private final ResourceViewService resourceViewService;

    public DefaultResourceApi(ResourceViewService resourceViewService) {
        this.resourceViewService = resourceViewService;
    }

    @Override
    public Response preflightResources() {
        return handleOptionsRequest();
    }

    @Override
    public Response preflightResource() {
        return handleOptionsRequest();
    }

    @Override
    @Access(level = MemberAccessLevel.class)
    public Response getResources(Integer invokerId, String token, String domain,
            Integer page, Integer perPage) throws CommonException {
        ResourcesView resources = resourceViewService.getResources(domain, invokerId, page, perPage);
        return Response.ok(resources).build();
    }

    @Override
    @Access(level = MemberAccessLevel.class)
    public Response getResource(Integer invokerId, String token, String domain,
            String id) throws CommonException {
        ResourceView resource = resourceViewService.getResource(id, domain, invokerId);
        return Response.ok(resource).build();
    }

    @Override
    @Access(level = MemberAccessLevel.class)
    public Response addResource(Integer invokerId, String token, String domain,
            AddResourceRequest request) throws CommonException {
        ResourceView resource = resourceViewService.addResource(request, domain, invokerId);
        return Response.created(URI.create(resource.url)).entity(resource).build();
    }

    @Override
    @Access(level = MemberAccessLevel.class)
    public Response updateResource(Integer invokerId, String token, String domain,
            String id, UpdateResourceRequest request) throws CommonException {
        ResourceView resource = resourceViewService.updateResource(id, request, domain, invokerId);
        return Response.ok(resource).build();
    }

    @Override
    @Access(level = MemberAccessLevel.class)
    public Response deleteResource(Integer invokerId, String token, String domain,
            String id) throws CommonException {
        resourceViewService.deleteResource(id, domain, invokerId);
        return Response.noContent().build();
    }

    private Response handleOptionsRequest() {
        return Response.ok().build();
    }
}
```

## Wzorzec View

### Bazowy ResourceView

```java
public abstract class ResourceView {
    @Schema(description = URL)
    @JsonProperty(access = JsonProperty.Access.READ_ONLY)
    public String url;

    @Schema(description = HTML_URL)
    @JsonProperty(access = JsonProperty.Access.READ_ONLY)
    public String htmlUrl;

    protected ResourceView() {}

    protected ResourceView(String url, String htmlUrl) {
        this.url = url;
        this.htmlUrl = htmlUrl;
    }

    // Dla szyfrowania wrażliwych pól
    public void encryptValues(Encryptor encryptor) {}
    public void decryptValues(Decryptor decryptor) {}
}
```

### Short View (dla list)

```java
public class ShortResourceView extends ResourceView {

    @Schema(description = RESOURCE_ID)
    public String id;

    @Schema(description = RESOURCE_NAME, requiredMode = Schema.RequiredMode.REQUIRED)
    public String name;

    @Schema(description = RESOURCE_STATUS)
    public String status;

    @Schema(description = RESOURCE_CREATE_DATE)
    public OffsetDateTime createDate;

    public ShortResourceView() {}

    public ShortResourceView(String baseApiUrl, String htmlUrl, ResourceFront resource) {
        super(
            PathHelper.join(baseApiUrl, "resources", resource.id.toString()),
            PathHelper.join(htmlUrl, "resources", resource.id.toString())
        );
        this.id = resource.id.toString();
        this.name = resource.name;
        this.status = resource.status != null ? resource.status.name() : null;
        this.createDate = resource.createDate;
    }
}
```

### Full View (dla szczegółów)

```java
public class ResourceView extends ShortResourceView {

    @Schema(description = RESOURCE_DESCRIPTION)
    public String description;

    @Schema(description = RESOURCE_CONFIG)
    public Map<String, String> config;

    // Relacje jako Short views
    public ShortProjectView project;
    public ShortEnvironmentView environment;

    @Schema(description = RESOURCE_TAGS)
    public Set<String> tags;

    @Schema(description = RESOURCE_CREATED_BY)
    public MemberView createdBy;

    @Schema(description = RESOURCE_MODIFIED_DATE)
    public OffsetDateTime modifiedDate;

    public ResourceView() {}

    public ResourceView(String baseApiUrl, String htmlUrl, ResourceFront resource) {
        super(baseApiUrl, htmlUrl, resource);
        this.description = resource.description;
        this.config = resource.config;
        this.tags = resource.tags;
        this.modifiedDate = resource.modifiedDate;
        // ... mapowanie relacji
    }
}
```

### Kolekcja z paginacją

```java
public class ResourcesView extends PageableResourceView {

    @Schema(description = RESOURCES)
    public Collection<ShortResourceView> resources;

    public ResourcesView() {}

    public ResourcesView(Collection<ShortResourceView> resources,
            Integer page, Integer pageSize, Integer totalCount) {
        this.resources = resources;
        this.page = page;
        this.pageSize = pageSize;
        this.totalElementCount = totalCount;
        this.totalPageCount = (int) Math.ceil((double) totalCount / pageSize);
        this.elementCount = resources.size();
    }
}
```

## Wzorzec Request

### Add Request

```java
public record AddResourceRequest(
    @NotEmpty(viewNameField = "name", groups = {AddResourceGroup.class})
    @Schema(description = RESOURCE_NAME, requiredMode = Schema.RequiredMode.REQUIRED)
    String name,

    @Schema(description = RESOURCE_DESCRIPTION)
    String description,

    @Schema(description = RESOURCE_TAGS)
    Set<String> tags
) {}
```

### Update Request

```java
public record UpdateResourceRequest(
    @Schema(description = RESOURCE_NAME)
    String name,

    @Schema(description = RESOURCE_DESCRIPTION)
    String description,

    @Schema(description = RESOURCE_TAGS)
    Set<String> tags
) {}
```

## Wzorzec OpenAPI Descriptions

```java
public class ResourceApiDescriptions {
    // Opisy pól
    public static final String RESOURCE_ID = "Unikalny identyfikator zasobu";
    public static final String RESOURCE_NAME = "Nazwa zasobu";
    public static final String RESOURCE_DESCRIPTION = "Opis zasobu";
    public static final String RESOURCE_STATUS = "Status zasobu (ACTIVE, DISABLED)";
    public static final String RESOURCE_CREATE_DATE = "Data utworzenia";
    public static final String RESOURCE_TAGS = "Tagi przypisane do zasobu";

    // Opisy operacji
    public static final String GET_RESOURCES = "Pobierz listę zasobów";
    public static final String GET_RESOURCE = "Pobierz szczegóły zasobu";
    public static final String ADD_RESOURCE = "Utwórz nowy zasób";
    public static final String UPDATE_RESOURCE = "Zaktualizuj zasób";
    public static final String DELETE_RESOURCE = "Usuń zasób";
}
```

## Wzorzec OpenAPI Examples

```java
public final class ResourceApiExamples {

    public static final String ADD_RESOURCE_REQUEST = """
        {
            "name": "my-resource",
            "description": "Example resource",
            "tags": ["production", "critical"]
        }
        """;

    public static final String RESOURCE_RESPONSE = """
        {
            "url": "/api/workspaces/my-workspace/resources/abc123",
            "htmlUrl": "https://app.buddy.works/my-workspace/resources/abc123",
            "id": "abc123",
            "name": "my-resource",
            "description": "Example resource",
            "status": "ACTIVE",
            "createDate": "2024-01-15T10:30:00Z",
            "tags": ["production", "critical"]
        }
        """;
}
```

## Adnotacje walidacyjne

```java
// Pole wymagane
@NotEmpty(viewNameField = "name", groups = {AddResourceGroup.class})

// Regex pattern
@Pattern(viewNameField = "status", regex = "ACTIVE|DISABLED")

// Dozwolone wartości (enum-like)
@AvailableValues(viewNameField = "type", values = {"TYPE_A", "TYPE_B", "TYPE_C"})

// Liczba dodatnia
@PositiveInt(viewNameField = "count")

// Niestandardowa walidacja
@ForbiddenResourceName(viewNameField = "name")

// Filtrowanie pól na podstawie scope
@ScopeRestrictions(scopes = OAuth2Scope.RESOURCE_MANAGE)
```

## Kody odpowiedzi HTTP

```java
// 200 OK - GET sukces, PATCH sukces
return Response.ok(resource).build();

// 201 Created - POST sukces
return Response.created(URI.create(resource.url)).entity(resource).build();

// 204 No Content - DELETE sukces
return Response.noContent().build();

// Błędy obsługiwane przez exception mappery
throw new CommonException(Messages._msg_resource_not_found());
```
