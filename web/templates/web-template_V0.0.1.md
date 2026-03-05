---
name: web-documentation
description: >
Generate or update Writerside wiki documentation for REST API microservices
with hexagonal architecture (WEB pattern). Use when documenting a microservice
that exposes REST endpoints and connects to external APIs and PostgreSQL.
No Kafka consumers — only REST input (driving/api-rest) and optional Kafka
producer as output. This is the V0.0.1 version for web repositories.
Covers all topic sections: overview, architecture, structure, dependencies,
particularities, design-tradeoffs, tests, and infrastructure.
---

# Project Documentation Guide (Writerside) — WEB Microservices

## Wiki Structure

This project uses JetBrains Writerside v2.0 for documentation. All documentation lives under the `wiki/` folder.

```
wiki/
├── writerside.cfg          # Main Writerside config (points to topics/, resources/, wiki.tree)
├── wiki.tree               # Navigation tree (TOC) — defines page hierarchy
├── c.list                  # Category definitions
├── v.list                  # Reusable variables (%var_name% syntax)
├── resources/              # Diagrams (Mermaid .mmd files, PNGs, etc.)
└── topics/                 # All documentation pages
    ├── overview/
    ├── architecture/
    ├── structure/
    ├── dependencies/
    ├── particularities/
    ├── design-tradeoffs/
    ├── tests/
    └── infrastructure/
```

## Documentation Conventions

- **Language**: Write documentation in Spanish.
- **Format**: Markdown files (`.md`) inside `wiki/topics/<topic-name>/`.
- **One topic per folder**: Each topic gets its own subdirectory under `topics/`.
- **Variables**: Define reusable values in `v.list` and reference them as `%variable_name%` in markdown.
- **Diagrams**: Place Mermaid (`.mmd`) or image files in `wiki/resources/` and embed with:
  ```xml
  <code-block lang="Mermaid" src="../../resources/diagram.mmd"/>
  ```
- **Code blocks**: Use standard markdown fenced blocks or Writerside `<code-block>` tags when lang-specific rendering is needed.
- **Horizontal rules**: Use `---` to separate major sections within a page.
- **Section titles**: Do NOT use numbering prefixes (e.g. `## Estilo Arquitectónico`, NOT `## 1. Estilo Arquitectónico`).
- **Cross-references**: Use the full path style `wiki/topics/<file>.md` when referencing other wiki pages from within a document.

## DOCTYPE Rules — ALL XML files MUST include DOCTYPE

Every XML file in the wiki MUST include the correct DOCTYPE declaration. This is required by Writerside for correct processing. The exact strings for each file are:

**writerside.cfg:**
```xml
<!DOCTYPE ihp SYSTEM "https://resources.jetbrains.com/writerside/1.0/ihp.dtd">
```

**wiki.tree:**
```xml
<!DOCTYPE instance-profile
        SYSTEM "https://resources.jetbrains.com/writerside/1.0/product-profile.dtd">
```

**c.list:**
```xml
<!DOCTYPE categories
        SYSTEM "https://resources.jetbrains.com/writerside/1.0/categories.dtd">
```

**v.list:**
```xml
<!DOCTYPE vars SYSTEM "https://resources.jetbrains.com/writerside/1.0/vars.dtd">
```

## Adding a New Topic

1. Create a folder: `wiki/topics/<topic-name>/`
2. Create the markdown file: `wiki/topics/<topic-name>/<topic-name>.md`
3. Register the page in `wiki/wiki.tree` inside the `<instance-profile>` block:
   ```xml
   <toc-element topic="<topic-name>.md"/>
   ```
4. If the topic introduces reusable values, add them to `wiki/v.list`.

---

## writerside.cfg Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE ihp SYSTEM "https://resources.jetbrains.com/writerside/1.0/ihp.dtd">

<ihp version="2.0">
    <topics dir="topics" web-path="topics"/>
    <images dir="resources" web-path="resources"/>
    <instance src="wiki.tree"/>
</ihp>
```

Key points:
- DOCTYPE is mandatory.
- Both `<topics>` and `<images>` elements MUST include `web-path` attributes matching their `dir` values.
- The images `dir` is `resources` (not `images`) — this is the convention in web repos.

---

## wiki.tree Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE instance-profile
        SYSTEM "https://resources.jetbrains.com/writerside/1.0/product-profile.dtd">

<instance-profile id="wiki"
                 name="web-movements-wiki"
                 start-page="overview.md">

    <toc-element topic="overview.md"/>
    <toc-element topic="architecture.md"/>
    <toc-element topic="structure.md"/>
    <toc-element topic="dependencies.md"/>
    <toc-element topic="particularities.md"/>
    <toc-element topic="design-tradeoffs.md"/>
    <toc-element topic="tests.md"/>
    <toc-element topic="infrastructure.md"/>
</instance-profile>
```

Key points:
- DOCTYPE is mandatory.
- `name` attribute uses the short form: `web-<service>-wiki` (e.g. `web-movements-wiki`).
- Canonical TOC order: **overview → architecture → structure → dependencies → particularities → design-tradeoffs → tests → infrastructure**.
- No child pages for tests (unlike SNK pattern). All test cases go in `tests.md` directly.

---

## c.list Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE categories
        SYSTEM "https://resources.jetbrains.com/writerside/1.0/categories.dtd">
<categories>
    <category id="wrs" name="Writerside documentation" order="1"/>
</categories>
```

Key points:
- DOCTYPE is mandatory.
- Use `id="wrs"`, `name="Writerside documentation"`, `order="1"` exactly as shown.
- Do NOT customise to project name.

---

## v.list Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE vars SYSTEM "https://resources.jetbrains.com/writerside/1.0/vars.dtd">
<vars>
    <var name="product" value="Writerside"/>
    <var name="latest_version" value="0.1.0"/>
    <var name="java-version" value="21"/>
    <var name="app_name" value="<project-name>"/>
    <var name="instancio-version" value="<version>"/>
    <var name="commons-collections4.version" value="<version>"/>
    <var name="testcontainers-version" value="<version>"/>
    <var name="spring-boot-version" value="<version>"/>
    <var name="hibernate.version" value="<version>"/>
    <var name="resilience4j-version" value="<version>"/>
</vars>
```

Key points:
- DOCTYPE is mandatory.
- Canonical variable set for web repos: `product`, `latest_version`, `java-version`, `app_name`, `instancio-version`, `commons-collections4.version`, `testcontainers-version`, `spring-boot-version`, `hibernate.version`, `resilience4j-version`.
- No Avro version variables — those are SNK-only.
- Add `<var name="<client>-client-version" value="<version>"/>` for each generated API client (e.g. `bff-client-version`, `product-client-version`).

---

## Required Sections and Writing Patterns per Topic Type

### Overview (`overview.md`)

**Title:** `# Descripción`

**Required sections in order:**
1. Header block with `**bold label:**` fields
2. `## ¿Cómo funciona a alto nivel?` — brief hexagonal architecture reference
3. `## Flujo típico (creación de un movimiento)` — numbered steps for the primary use case
4. `## Módulos del repositorio` — cross-reference to architecture and structure
5. `## Integraciones y otros temas` — links to other wiki pages
6. `## Tecnologías (resumen)` — brief bullet list of key technologies

**Intro header block pattern (use `---` after this block):**
```markdown
# Descripción

**Nombre del Microservicio:** %app_name% - %latest_version%.

**¿Qué es?**
Microservicio responsable de <descripción principal del dominio>.

**¿Para qué sirve? (Propósito)**
Proveer una API robusta para orquestar reglas de negocio y mantener la consistencia de datos, integrándose con servicios corporativos como <servicios principales>.

**Responsabilidades clave (resumen):**
- Crear y gestionar <entidades principales>.
- Validar reglas de negocio y transiciones de estado.
- Integrarse con sistemas externos y exponer endpoints REST.

---
```

**`## ¿Cómo funciona a alto nivel?` pattern:**
```markdown
## ¿Cómo funciona a alto nivel?
Sigue el enfoque de Arquitectura Hexagonal (Ports & Adapters). Para el detalle de capas, puertos, adaptadores, límites transaccionales e idempotencia, consulta el documento de Arquitectura.

- Ver Arquitectura completa: ../architecture/architecture.md

---
```

**`## Flujo típico` pattern:**
```markdown
## Flujo típico (<operación principal>)
- El cliente llama al endpoint <método> con los datos del <recurso>.
- La aplicación valida reglas y persiste el <recurso>.
- Cuando aplica, se sincronizan estados y se interactúa con servicios externos o buckets.

Detalles paso a paso y variantes: ../architecture/architecture.md#flujo-típico-de-una-petición

---
```

**`## Módulos del repositorio` pattern:**
```markdown
## Módulos del repositorio
Para la descripción detallada de módulos y su relación con las capas, consulta:
- Arquitectura: ../architecture/architecture.md#estructura-de-módulos-y-capas
- Estructura y listado de adapters: ../structure/structure.md

---
```

**`## Tecnologías (resumen)` pattern:**
```markdown
## Tecnologías (resumen)
- Java %java-version% / Spring Boot %spring-boot-version%
- Spring Web (REST), JPA/Hibernate %hibernate.version%
- Resilience4j, Testcontainers, JUnit/Mockito
```

---

### Architecture (`architecture.md`)

**Title:** `# Arquitectura`

**Opening paragraph (MANDATORY — place immediately after title, before any section heading):**
```markdown
# Arquitectura

Este documento describe la arquitectura del microservicio **%app_name%**, incluyendo su estilo arquitectónico, estructura de módulos, conectividad, decisiones transversales y principios de diseño. Se mantiene el enfoque Hexagonal y la imagen de referencia.

---
```

**Required sections in order (NO numbering prefixes):**
1. `## Estilo Arquitectónico`
2. `## Estructura de Módulos y Capas`
3. `## Puertos y Adaptadores`
4. `## Flujo típico de una petición`
5. `## Persistencia (PostgreSQL)`
6. `## Mensajería y Eventos` *(solo si hay producers Kafka)*
7. `## Integraciones Externas`
8. `## Resiliencia y Errores`
9. `## Seguridad`
10. `## Observabilidad`
11. `## Configuración`
12. `## Estrategia de Tests`
13. `## Decisiones de Diseño relevantes`
14. `## Dependencias y Versionado`
15. `## Recursos relacionados`

**Module entry pattern** — REST-driven modules follow this exact one-liner format:
```markdown
- `driving/api-rest/`: adaptadores de entrada (controladores REST) y contratos OpenAPI.
```

Driven modules (one-liner each):
```markdown
- `driven/postgres-repository/`: adaptadores de salida hacia PostgreSQL (repositorios, mappers, migraciones SQL).
- `driven/bff-datasource/`: cliente REST hacia la API BFF corporativa.
- `driven/product-api-datasource/`: cliente REST hacia la API de Producto (client-credentials).
- `driven/email-producer/`: productor de eventos para notificaciones por email (Kafka).
- `driven/bucket-datasource/`: acceso a almacenamiento de ficheros (S3/GCS).
- `driven/masterdata-datasource/`: cliente de datos maestros corporativos.
- `boot/`: empaquetado, configuración de arranque y charts específicos.
```

After the module list, always add the layer-to-module mapping paragraph:
```markdown
Relación capas ↔ módulos:
- Entrada (Ports In) → `driving/api-rest` expone endpoints y traduce DTOs a comandos/queries de aplicación.
- Dominio/Aplicación → `application` contiene casos de uso, servicios y puertos.
- Salida (Ports Out) → `driven/*` implementan los puertos para persistencia, mensajería y servicios externos.

Para un inventario detallado de adapters y métodos, ver wiki/topics/structure.md.
```

**`## Puertos y Adaptadores` section pattern:**
```markdown
## Puertos y Adaptadores

- Entradas (Driving):
  - REST Controllers (Spring MVC) en `driving/api-rest`.
- Salidas (Driven):
  - Repositorios PostgreSQL en `driven/postgres-repository`.
  - Buckets S3/GCS en `driven/bucket-datasource`.
  - Productor Kafka (email) en `driven/email-producer`.
  - APIs externas en `driven/bff-datasource` y `driven/product-api-datasource`.
  - Masterdata API en `driven/masterdata-datasource`.

Los puertos definen interfaces dentro de `application`, y los adapters las implementan en los módulos `driven`.

---
```

**`## Flujo típico de una petición` section pattern:**
```markdown
## Flujo típico de una petición

1. Cliente llama a un endpoint REST (controller en `driving/api-rest`).
2. El controller valida y transforma DTOs a comandos/queries de aplicación.
3. El caso de uso (en `application`) ejecuta la lógica de negocio y coordina los puertos de salida necesarios.
4. Los adapters (`driven/*`) persisten/leen datos de PostgreSQL, consultan APIs externas, interactúan con Buckets o publican eventos en Kafka.
5. La respuesta se mapea a DTOs y se devuelve al cliente.

Límites transaccionales:
- Operaciones de escritura en DB se encapsulan en transacciones a nivel de caso de uso.
- Publicaciones a Kafka suelen realizarse tras persistencia exitosa.

Idempotencia y consistencia:
- Endpoints de creación soportan idempotencia basada en identificadores de negocio cuando aplica.
- Se favorece consistencia eventual para integraciones externas.

---
```

**Cross-reference style:** Use the full path `wiki/topics/<file>.md` when referencing other wiki documents:
- `wiki/topics/infrastructure.md`
- `wiki/topics/design-tradeoffs.md`
- `wiki/topics/dependencies.md`
- `wiki/topics/tests.md`
- `wiki/topics/structure.md`
- `wiki/topics/particularities.md`

---

### Structure (`structure.md`)

**Title:** `# Estructura del proyecto` *(can use emoji: `# 🧱 Estructura del proyecto`)*

**Purpose:** Lists all REST controller adapters and database/driven adapters with their methods. This is the adapter inventory page — no equivalent in SNK template.

**Required sections in order:**
1. `## Controladores REST` — one subsection per controller adapter
2. `## Base de datos` — one subsection per datasource/persistence adapter
3. Additional driven adapters (bucket, email, masterdata) as needed

**Controller entry pattern:**
```markdown
### <ControllerAdapterClassName>
**Descripción:**
Controlador REST para la gestión de <dominio breve>.

**Métodos:**
- `ResponseEntity<<ResponseDto>> <methodName>(<params>)`: <descripción breve del método>.
- `ResponseEntity<Void> <methodName>(<params>)`: <descripción breve>.

---
```

**Datasource adapter entry pattern:**
```markdown
### <AdapterClassName>
**Descripción:**
Adaptador para la información de <dominio>.

**Métodos:**
- `<ReturnType> <methodName>(<params>)`: <descripción breve>.

---
```

Key points:
- This page is unique to web repos. SNK repos don't have a `structure.md`.
- Keep method descriptions to one line each — this is a reference page, not a detailed guide.
- Use "Pendiente de implementación (no hay métodos públicos definidos por el momento)." for empty adapters.

---

### Dependencies (`dependencies.md`)

**Title:** `# Dependencias Ad-hoc`

**Intro paragraph:**
```markdown
Este documento describe las dependencias utilizadas en el microservicio **%app_name%**, incluyendo sus versiones y propósito principal.

---
```

**Per-dependency format** — Use `###` sub-sections. Each dependency MUST use exactly these bullet fields:
```markdown
## Dependencias Externas (otros servicios y librerías)

### <dependency-name>
- **GroupId:** <groupId>
- **ArtifactId:** <artifactId>
- **Version:** %<version-var>%
- **Scope:** <scope>  ← only include this field if scope is not default (e.g. `test`)
- **Descripción:** <description>

---
```

Key points:
- Use `GroupId`, `ArtifactId`, `Version`, `Scope` (if non-default), `Descripción`.
- Document only project-specific or notable dependencies — not every transitive one.
- Use `%variable%` for versions wherever a matching `v.list` variable exists.

**External API Clients table — add this section instead of Avro Schemas table:**

```markdown
## Clientes de APIs Externas

| Módulo | Artefacto cliente | Tipo de auth | Endpoints principales |
|--------|-------------------|--------------|-----------------------|
| bff-datasource | `bff-api-client-webclient-cna-21` | propagate (user token) | `/product/brands`, `/product/tariffs` |
| product-api-datasource | `product-client-webclient-cna-21` | client-credentials (ADFS) | `/products/brands`, `/products/formats/tariffs` |
```

Key points for the API clients table:
- One row per driven API client module.
- `Artefacto cliente` = the exact Maven `artifactId` of the generated WebClient jar.
- `Tipo de auth` = `propagate` (user context forwarding) or `client-credentials` (service-to-service).
- `Endpoints principales` = the main endpoints consumed.
- No Avro schema table — that's SNK-only.

---

### Particularities (`particularities.md`)

**Title:** `# Particularidades`

**Purpose:** Documents special behaviors, edge cases, or non-obvious design decisions not covered in architecture or design-tradeoffs. Unique to web repos — not present in SNK template.

**Intro paragraph template:**
```markdown
Este documento recoge las particularidades y comportamientos especiales del microservicio **%app_name%** que no quedan completamente cubiertas en los documentos de arquitectura o concesiones de diseño.

---
```

Key points:
- Start as a placeholder and fill incrementally.
- Use `##` sections per peculiarity, without numbering.
- Examples of what to document here: custom auth filter order in WebClient, special block validation rules, idempotency behavior in specific endpoints, context propagation details.

---

### Design Tradeoffs (`design-tradeoffs.md`)

**Title:** `# Concesiones de diseño`

**Intro paragraph template:**
```markdown
Este documento describe las peculiaridades y concesiones de diseño adoptadas en el microservicio **%app_name%**, incluyendo las razones detrás de estas decisiones y sus implicaciones.

---
```

Key points:
- The title is `# Concesiones de diseño` (NOT `# Decisiones de Diseño`).
- May start as a placeholder (just title + intro + `---`) and be filled incrementally.
- When adding entries, use `##` sections with the decision name. Do NOT use numbered prefixes.

---

### Tests (`tests.md`)

**Title:** `# ✅ Tests` *(emoji is optional but matches existing convention)*

**Purpose:** Documents all test cases for use cases and controller adapters. All test cases live in this single page (no child pages, unlike SNK integration_tests pattern).

**Required structure:** One collapsible `##` section per test class, with method subsections and a case table per method.

**Test class section pattern:**
```markdown
## <TestClassName> {collapsible="true"}

### <methodName>

<Descripción de una línea de qué hace este método>

| ID | Condición Principal | Acciones Principales |
|----|---------------------|----------------------|
| <PREFIX>_<METHOD>-1 | <condición> | <resultado esperado> |
| <PREFIX>_<METHOD>-2 | <condición> | <resultado esperado> |
```

**ID naming convention:**
- Format: `<CLASS_ABBREV>_<METHOD_ABBREV>-<N>`
- Class abbreviation: 2-3 uppercase letters from the class name (e.g. `PU` for ProductUseCase, `PMC` for ProductMovementController, `LC` for LifetimeController)
- Method abbreviation: 2-4 uppercase letters from the method name (e.g. `GBD` for getBasicData, `CM` for createMovement)
- Example: `PMC_GBF-1`, `STC_FCG-1`, `PU_FBC-3`

**Result language conventions:**
- HTTP responses: use `**HTTP 200 OK**`, `**HTTP 400 Bad Request**`, `**HTTP 401 Unauthorized**`, `**HTTP 404 Not Found**`, `**HTTP 204 No Content**`
- Exceptions: use `<ExceptionClassName>` in backticks (e.g. `` `UnauthorizedException` ``)
- Empty lists: `lista vacía de <entidades>`
- Objects: `<ClassName>` not null, or with specific populated fields

**Test file location in web repos:** `boot/src/test/java/...`

**Naming pattern in web repos:** `*ITTest.java` (e.g. `ProductMovementRepositoryITTest.java`) or `*Test.java` for application layer tests.

**Base test class:** `DbSpringbootITTest` (PostgreSQL TestContainer, `@SpringBootTest`).

---

### Infrastructure (`infrastructure.md`)

**Title:** `# Infraestructura`

**Intro paragraph template:**
```markdown
Este documento resume la configuración de infraestructura necesaria para ejecutar el microservicio **%app_name%** tanto en local como en entornos de despliegue.
```

**Required sections in order:**
1. `## Resumen rápido` — bullet list of key infrastructure components
2. `## Buckets de Almacenamiento` *(if applicable)*
3. `## Mensajería (Kafka)` *(only for email producer — not consumers)*
4. `## Base de datos (PostgreSQL)`
5. `## Despliegue`

**`## Resumen rápido` pattern (web version):**
```markdown
## Resumen rápido
- Base de datos: PostgreSQL (variables: LOCAL_DB_URL, LOCAL_DB_USER, LOCAL_DB_PASSWORD; migraciones: FLYWAY_DATABASE_USERNAME, FLYWAY_DATABASE_PASSWORD).
- Almacenamiento de ficheros: Bucket S3/GCS (variables: BUCKET_URL, STORAGE_BUCKET_NAME, BUCKET_ACCESS_KEY, BUCKET_SECRET_KEY).
- Mensajería: Kafka (variables: BOOTSTRAP_SERVERS, SCHEMA_REGISTRY_URL, KAFKA_USER, KAFKA_PASSWORD). Solo productor de email — no consumers.
- Servicios externos: API_MASTERDATA_URL, API_BFF_URL, API_PRODUCT_URL.
- Auth: ADFS (variables: ADFS_AUTHORITY, ADFS_APP_ID, ADFS_APP_PASS).
- Certificados/TrustStore (si aplica): TRUSTORE_LOCATION.
- Despliegue: Docker/Helm. Charts en devops/charts/<chart-name> y boot/charts/<chart-name>.

---
```

**Kafka config snippet style (producer only — no consumer topics):**
```yaml
fwkcna:
  kafka:
    producer:
      topics:
        output-0: internalservices.itoperation.emailmessage.event.restrictedin.v0.canonical.cpd
```

**Environment variables format — use `properties` inline code blocks:**
```properties
BOOTSTRAP_SERVERS=bootstrap.kafka.pre.cm.mercadona.com:443
SCHEMA_REGISTRY_URL=https://schemaregistry.pre.cm.mercadona.com
KAFKA_USER=***
```

**Database variables format:**
```properties
LOCAL_DB_URL=jdbc:postgresql://localhost:65432/<db-name>
LOCAL_DB_USER=<db-user>
LOCAL_DB_PASSWORD=********
FLYWAY_DATABASE_USERNAME=<db-user>
```

Key points:
- Infrastructure page is a **simplified reference** with example values.
- Do NOT paste exhaustive YAML configs or full env listings.
- In web repos, Kafka is only for the email producer (output). There are no consumer topic variables here.
- External API URLs go in the Resumen rápido as variable names only.

---

## Key Differences vs SNK Template

| Aspect | SNK (consumer) | WEB (REST) |
|--------|---------------|------------|
| Driving adapter | `driving/*-consumer` (Kafka) | `driving/api-rest` (REST) |
| Kafka usage | Consumers (multiple topics) | Producer only (email) |
| External integrations | Avro schemas | REST API clients (WebClient) |
| Dependencies table | Avro Schemas table | External API Clients table |
| v.list avro vars | Yes (one per consumer) | No |
| TOC order | overview→architecture→dependencies→design-tradeoffs→integration_tests→business_logic→infrastructure | overview→architecture→**structure**→dependencies→**particularities**→design-tradeoffs→**tests**→infrastructure |
| Topics unique to web | — | `structure.md`, `particularities.md` |
| Topics unique to SNK | `business_logic.md`, `integration_tests.md` (with child pages) | — |
| Test doc pattern | Child pages per IT class (`IT.md` suffix) | Single `tests.md` with collapsible sections |
| Test naming pattern | `*IT.java` | `*ITTest.java` or `*Test.java` |
| Base test class | (Kafka+PostgreSQL TestContainer) | `DbSpringbootITTest` |
| Diagram folder | `wiki/images/` | `wiki/resources/` |
| Infrastructure Kafka | Consumer + Producer topics | Producer only (email) |
| Architecture Ports In | Kafka listeners | REST Controllers |

---

## Rules for Claude

- When asked to document a feature or component, follow the structure and patterns above.
- Always register new pages in `wiki.tree`.
- Always use `%variable_name%` for versions and project name instead of hardcoding.
- Keep documentation concise and technical — no filler text.
- Place diagrams in `wiki/resources/` and reference them, never inline large diagrams.
- Match the existing naming conventions: lowercase with hyphens for folders, underscores or hyphens for filenames matching the topic slug.
- **When adding a new REST controller:** add an entry in `structure.md` (following the controller entry pattern).
- **When adding a new driven adapter:** add an entry in `structure.md` (following the datasource adapter entry pattern).
- **When adding a new external API client module:** add a row to the External API Clients table in `dependencies.md` and a module entry in `architecture.md`.
- **Read the source code first:** Before documenting a controller or adapter, read its class to understand methods, parameters, and response types.
- **Match the prose style:** Use established Spanish phrases. Keep the same level of detail as existing sections.
- **Do NOT number section headings** in any wiki file. Sections use plain `##` or `###` headings without numeric prefixes.
- **`wiki/resources/`** is the diagram folder in web repos (NOT `wiki/images/`).
- **infrastructure.md** is a simplified reference page — do not paste exhaustive topic tables or full YAML configs.
- **DOCTYPE declarations are mandatory** in all four XML files (`writerside.cfg`, `wiki.tree`, `c.list`, `v.list`). Never omit them.
- **`web-path` attributes are mandatory** on both `<topics>` and `<images>` elements in `writerside.cfg`.
- **`c.list`** always uses `id="wrs"`, `name="Writerside documentation"`, `order="1"` — do not customise these to the project name.
- **Cross-references** inside wiki pages use the full `wiki/topics/<file>.md` path style, not bare filenames.
- **No Avro schemas table** — that section is SNK-only. Use External API Clients table instead.
- **No `business_logic.md`** — web repos don't have batch consumer logic. Business logic is embedded in use case documentation.
- **No child pages for tests** — all test cases go in a single `tests.md` with collapsible `## <TestClass>` sections.
- **`particularities.md`** is the place for non-obvious behaviors like WebClient filter order, auth edge cases, or custom context propagation.
