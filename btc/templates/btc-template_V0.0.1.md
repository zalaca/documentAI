# Project Documentation Guide — BFF (Writerside)

> **Tipo de microservicio:** Back for Front (BFF).
> Este template es para micros que exponen una API REST al frontend y agregan llamadas a servicios downstream mediante WebClient. **Sin consumidores Kafka, sin base de datos propia.**

## Wiki Structure

This project uses JetBrains Writerside v2.0 for documentation. All documentation lives under the `wiki/` folder.

```
wiki/
├── writerside.cfg          # Main Writerside config (points to topics/, images/, wiki.tree)
├── wiki.tree               # Navigation tree (TOC) — defines page hierarchy
├── c.list                  # Category definitions
├── v.list                  # Reusable variables (%var_name% syntax)
├── images/                 # Diagrams (Mermaid .mmd files, PNGs, etc.)
└── topics/                 # All documentation pages
    ├── overview/
    ├── architecture/
    ├── dependencies/
    ├── design-tradeoffs/
    ├── business-logic/
    ├── infrastructure/
    └── integration_tests/
```

## Documentation Conventions

- **Language**: Write documentation in Spanish.
- **Format**: Markdown files (`.md`) inside `wiki/topics/<topic-name>/`.
- **One topic per folder**: Each topic gets its own subdirectory under `topics/`.
- **Variables**: Define reusable values in `v.list` and reference them as `%variable_name%` in markdown.
- **Diagrams**: Place Mermaid (`.mmd`) or image files in `wiki/images/` and embed with:
  ```xml
  <code-block lang="Mermaid" src="../../images/diagram.mmd"/>
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
   For nested pages:
   ```xml
   <toc-element topic="parent.md">
       <toc-element topic="child.md"/>
   </toc-element>
   ```
4. If the topic introduces new version numbers or reusable values, add them to `wiki/v.list`.

---

## writerside.cfg Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE ihp SYSTEM "https://resources.jetbrains.com/writerside/1.0/ihp.dtd">

<ihp version="2.0">
    <topics dir="topics" web-path="topics"/>
    <images dir="images" web-path="images"/>
    <instance src="wiki.tree"/>
</ihp>
```

Key points:
- DOCTYPE is mandatory.
- Both `<topics>` and `<images>` elements MUST include `web-path` attributes matching their `dir` values.

---

## wiki.tree Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE instance-profile
        SYSTEM "https://resources.jetbrains.com/writerside/1.0/product-profile.dtd">

<instance-profile id="wiki"
                 name="<project-short-name>-wiki"
                 start-page="overview.md">

    <toc-element topic="overview.md"/>
    <toc-element topic="architecture.md"/>
    <toc-element topic="dependencies.md"/>
    <toc-element topic="design-tradeoffs.md"/>
    <toc-element topic="integration_tests.md"/>
    <toc-element topic="business_logic.md">

    </toc-element>
    <toc-element topic="infrastructure.md"/>
</instance-profile>
```

Key points:
- DOCTYPE is mandatory.
- `name` attribute uses the short form: `<project-short-name>-wiki` (NOT the full repo name).
- `business_logic.md` element has an intentional blank inner element (not self-closing).
- En un BFF no hay IT tests con contenedores, por lo que `integration_tests.md` NO tiene páginas hijo. Es un elemento simple (no anidado).
- The canonical TOC order is: overview → architecture → dependencies → design-tradeoffs → integration_tests → business_logic → infrastructure.

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
- Do NOT use a custom project-specific id or name.

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
    <var name="spring-boot-version" value="<fwkcna-parent-version>"/>
    <!-- One var per downstream API client: -->
    <var name="<service>-client-version" value="<version>"/>
    <!-- Add one entry per driven/*-datasource module -->
</vars>
```

Key points:
- DOCTYPE is mandatory.
- El conjunto canónico de variables es: `product`, `latest_version`, `java-version`, `app_name`, `spring-boot-version` y una entrada `<service>-client-version` por cada cliente WebClient de un servicio downstream.
- No se añaden variables de Avro, Hibernate, Instancio ni TestContainers — no aplican a un BFF.
- Reference variables in markdown as `%variable_name%`.

---

## Required Sections and Writing Patterns per Topic Type

### Overview (`overview.md`)

**Title:** `# Descripción`

**Required sections in order:**
1. Header block with `**bold label:**` fields
2. `## ¿Cómo funciona?` — numbered steps explaining the hexagonal flow (REST → UseCase → WebClient)
3. `## Módulos del repositorio y su rol` — bulleted list of modules and their roles
4. `## Errores, resiliencia y pruebas` — brief paragraph on error handling and test approach
5. `## Integraciones y datos` — brief paragraph on downstream APIs (no DB)

**Intro header block pattern (use `---` after this block):**
```markdown
# Descripción

**Nombre del Microservicio:** %app_name%.

**Descripción:**
Microservicio Back for Front (BFF) responsable de agregar y orquestar llamadas a múltiples servicios internos para exponer una API REST adaptada a las necesidades del frontend.

**Propósito:**
Proveer un punto de entrada único para el frontend, simplificando las interacciones con los microservicios de dominio y adaptando los datos al contrato que necesita la aplicación cliente.

**Responsabilidades Principales:**
- Exponer una API REST unificada para el frontend.
- Agregar y orquestar llamadas a los servicios downstream.
- Transformar y adaptar las respuestas al contrato del frontend.
- Gestionar la autenticación OAuth2 (ADFS) hacia los servicios downstream.

**Componentes Principales:**
- **Controladores REST / Driving Adapters:** Capa que expone los endpoints REST y delega en los casos de uso.
- **Casos de Uso / Servicios:** Lógica de orquestación para cada área funcional.
- **Adaptadores de salida / Driven Adapters:** Clientes WebClient que invocan cada uno de los servicios downstream.

**Tecnologías:**
- Java %java-version% / Spring Boot (fwkcna-parent %spring-boot-version%)
- Spring WebClient para llamadas HTTP reactivas a servicios externos
- OAuth2 ADFS (client-credentials / propagate) para autenticación hacia servicios downstream
- OpenAPI 3.0.3 para la definición del contrato REST
- Pruebas unitarias (JUnit 5, Mockito, MockWebServer)

---
```

**`## ¿Cómo funciona?` section pattern:**
```markdown
## ¿Cómo funciona?

A alto nivel, el microservicio sigue la Arquitectura Hexagonal (Ports & Adapters) descrita en el documento de Arquitectura y opera así:

1) Entrada (Driving — REST)
- Las peticiones HTTP del frontend llegan a los controladores REST (driving/api-rest).
- Los controladores implementan las interfaces generadas a partir del contrato OpenAPI.
- Validan y traducen los parámetros de la petición al modelo de dominio de la aplicación.

2) Orquestación (Aplicación)
- La capa de aplicación ejecuta los casos de uso correspondientes (application/usecases), coordinando las llamadas a los puertos de salida.
- En casos de lógica compleja se realizan llamadas paralelas mediante CompletableFuture para minimizar la latencia.

3) Salidas (Driven — WebClient)
- Cada adaptador de salida (driven/*-datasource) invoca el servicio downstream correspondiente usando Spring WebClient.
- La autenticación OAuth2 (ADFS) se gestiona de forma transparente mediante el starter fwkcna-starter-auth-webclient.
- Las respuestas se mapean al modelo de dominio y se devuelven al caso de uso.

---
```

**`## Módulos del repositorio y su rol` section pattern:**
```markdown
## Módulos del repositorio y su rol
- driving/api-rest: Adaptadores de entrada REST. Controladores que implementan el contrato OpenAPI del BFF.
- application: Casos de uso, servicios de dominio y definición de puertos (interfaces).
- driven/<service>-datasource: Adaptador de salida hacia la API de <servicio>. (repetir por cada servicio downstream)
- boot: Módulo de arranque Spring Boot (configuración, empaquetado, charts/docker para despliegue en entornos).

---
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
5. `## Observabilidad`
6. `## Configuración`
7. `## Estrategia de Tests`
8. `## Decisiones de Diseño relevantes`
9. `## Dependencias y Versionado`
10. `## Recursos relacionados`

> **Nota:** En un BFF no hay sección `## Persistencia (PostgreSQL)` porque el BFF es stateless y no tiene base de datos propia.

**Driving module entry pattern** — El único módulo driving en un BFF es el controlador REST:
```markdown
- `driving/api-rest/`: adaptadores de entrada REST que implementan el contrato OpenAPI del BFF.
```

**Driven module entry pattern** — Un módulo driven por servicio downstream:
```markdown
- `driven/<service>-datasource/`: adaptador de salida hacia la API de <Servicio> (WebClient).
```

Example entries:
```markdown
- `driven/product-datasource/`: adaptador de salida hacia la API de Producto v2.
- `driven/treasury-datasource/`: adaptador de salida hacia la API de Tesorería v1.
- `driven/masterdata-datasource/`: adaptador de salida hacia el microservicio Masterdata v1.
```

The `boot` module always closes the list:
```markdown
- `boot/`: empaquetado, configuración de arranque y charts específicos.
```

After the module list, always add the layer-to-module mapping paragraph:
```markdown
Relación capas ↔ módulos:
- Entrada (Ports In) → `driving/api-rest` implementa los puertos para la recepción de peticiones REST del frontend.
- Dominio/Aplicación → `application` contiene casos de uso, servicios y puertos.
- Salida (Ports Out) → `driven/*-datasource` implementan los puertos para la invocación de servicios externos.

Para un inventario detallado de endpoints y adaptadores, ver wiki/topics/business-logic/business_logic.md.
```

**Ports & Adapters tables pattern:**
```markdown
## Puertos y Adaptadores

**Puertos de entrada (Driving Ports):**

| Puerto | Implementación |
|--------|----------------|
| `<Domain>Port` | `<Domain>UseCase` |

**Puertos de salida (Driven Ports):**

| Puerto | Implementación |
|--------|----------------|
| `<Service>DatasourcePort` | `<Service>DatasourceAdapter` |
```

**Flujo típico de una petición pattern:**
```markdown
## Flujo típico de una petición

```
Frontend
│  HTTP Request
▼
driving/api-rest (Controller Adapter)
│  invoca Puerto de entrada
▼
application/usecases (Use Case)
│  invoca Puertos de salida
▼
driven/*-datasource (WebClient Adapters)
│  HTTP + OAuth2 ADFS
▼
Servicios downstream
```

En casos de orquestación compleja, el caso de uso realiza múltiples llamadas en paralelo usando `CompletableFuture`.
```

**Cross-reference style:** Use the full path `wiki/topics/<file>.md` (not just `<file>.md`) when referencing other wiki documents from within architecture.md. For example:
- `wiki/topics/infrastructure/infrastructure.md`
- `wiki/topics/design-tradeoffs/design-tradeoffs.md`
- `wiki/topics/dependencies/dependencies.md`
- `wiki/topics/integration_tests/integration_tests.md`

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

Example:
```markdown
### product-client-webclient-cna-21
- **GroupId:** com.mercadona
- **ArtifactId:** product-client-webclient-cna-21
- **Version:** %product-client-version%
- **Descripción:** Cliente WebClient para la **API de Producto v2**. Proporciona los modelos de respuesta y la configuración del cliente HTTP para consultar marcas, tarifas e imágenes de producto.

---
```

Key points:
- Use `GroupId`, `ArtifactId`, `Version`, `Scope` (if non-default), `Descripción` as field names.
- Document only the project-specific or notable dependencies — do NOT list every transitive dependency.
- Use `%variable%` for versions wherever a matching `v.list` variable exists.
- **No hay tabla de Esquemas Avro** — los BFF no consumen Kafka.

**Downstream services summary table — add this section at the end of `dependencies.md`:**

```markdown
## Servicios Downstream Integrados

| Servicio | Artefacto cliente | Versión |
|----------|-------------------|---------|
| <Service Name> | `<artifactId>` | %<service>-client-version% |
| ... | ... | ... |
```

Example:
```markdown
## Servicios Downstream Integrados

| Servicio | Artefacto cliente | Versión |
|----------|-------------------|---------|
| Product API v2 | `product-client-webclient-cna-21` | %product-client-version% |
| Treasury API v1 | `currency-client-webclient-cna-17` | %treasury-client-version% |
| Masterdata API v1 | `masterdata-api-client-webclient-cna-17` | %masterdata-client-version% |
```

Key points for the downstream table:
- One row per `driven/*-datasource` module.
- `Artefacto cliente` column = el `artifactId` del cliente WebClient del servicio.
- `Versión` column = `%<service>-client-version%` variable referencing a v.list entry.
- Add the corresponding `<var name="<service>-client-version" value="<version>"/>` entries to `v.list` for each row.

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
- The file may start as a placeholder (just title + intro + `---`) and be filled in incrementally.
- When adding trade-off entries, use `##` sections with the decision name as the heading. Do NOT use numbered prefixes.

**Typical BFF trade-off sections to include:**
- `## Sin base de datos propia` — el BFF es stateless; implicaciones de latencia.
- `## Un adaptador Maven por servicio downstream` — aislamiento de versiones por módulo.
- `## Contrato OpenAPI como fuente de verdad` — generación de interfaces desde spec.
- `## Paralelismo con CompletableFuture` — llamadas paralelas a servicios, tamaño del pool.
- `## WebClient sobre Feign` — razón de la elección del cliente HTTP.
- `## Sin retry ni circuit breaker propios` — delegación en el framework CNA.

---

### Business Logic (`business_logic.md`)

**Title:** `# Lógica de negocio`

**Intro paragraph template:**
```markdown
Este documento describe la lógica de negocio empleada en cada uno de los endpoints del microservicio **%app_name%**.

A diferencia de un microservicio con consumidores de eventos, este BFF expone endpoints REST y orquesta llamadas a servicios downstream para construir las respuestas. La lógica de cada caso de uso se describe a continuación.

---
```

> **Nota:** En un BFF no hay sección de "Lógica común de consumidores en modo batch" ni referencias a tombstones ni batch collections.

**Endpoint/Use case section pattern** — Every use case section MUST follow this repeatable structure:

```markdown
## <UseCase o Endpoint Name>

El endpoint `<METHOD> <path>` delega en `<UseCase>.<method>()`.

<Descripción de qué datos se obtienen, de qué servicios downstream, y cómo se combinan.>

<Si hay orquestación paralela, describirla: "El caso de uso realiza múltiples llamadas en paralelo mediante `CompletableFuture`...">

<Si hay lógica de filtrado o condiciones especiales, describirlas aquí.>

---
```

**Example — simple endpoint:**
```markdown
## Treasury — Divisas

El endpoint `GET /currencies` delega en `TreasuryUseCase.getCurrencies()`.

Invoca directamente al adaptador `CurrenciesDatasourceAdapter`, que consulta la API de Tesorería v1. Soporta filtrado por `locale` y `currencyCodeAlpha`.

---
```

**Example — endpoint with parallel orchestration:**
```markdown
## Movements — Logística de proveedores

El endpoint `GET /movements/{movementId}/sheet/{sheetId}/supplier-logistics` delega en `MovementsUseCase.getSupplierLogistics()`.

Orquesta múltiples llamadas en paralelo mediante `CompletableFuture`:
- Bloque del movimiento (API Movements).
- Cadenas de suministro (API Movements).
- Proveedores y contactos (API Masterdata).
- Formatos de servicio de tienda (API Masterdata).

El resultado combina los datos de logística con las cuotas por almacén.

---
```

**Example — endpoint with conditional logic:**
```markdown
## Site — Sites disponibles

El endpoint `GET /movements/{movementId}/selling-geography` delega en `SiteUseCase.getAvailable()`.

El caso de uso determina qué fuente de datos utilizar en función del tipo de movimiento y el `stageId`:
- Si el movimiento es de tipo **supply chain**, consulta los sites desde la cadena de suministro.
- Si el movimiento es de tipo **supply group**, consulta los almacenes inferidos desde Masterdata.

El resultado se filtra según los criterios del filtro recibido en la petición.

---
```

**Key Spanish phrases to reuse:**
- "El endpoint `<METHOD> <path>` delega en `<UseCase>.<method>()`."
- "Invoca directamente al adaptador `<Adapter>`..."
- "Orquesta múltiples llamadas en paralelo mediante `CompletableFuture`:"
- "El caso de uso determina qué fuente de datos utilizar en función de..."
- "El resultado combina los datos de... con..."
- "Soporta filtrado por `<param>`."
- "Soporta paginación (`pageIndex`, `pageSize`)."

**Separator rule:** Place `---` after each use case section to visually separate sections.

---

### Infrastructure (`infrastructure.md`)

**Title:** `# Infraestructura`

**Intro paragraph template:**
```markdown
Este documento resume la configuración de infraestructura necesaria para ejecutar el microservicio **%app_name%** tanto en local como en entornos de despliegue.
```

**Required sections in order:**
1. `## Resumen rápido` — brief bullet list of key infrastructure components and their env vars
2. `## APIs Externas` — variables de entorno para URLs de servicios downstream
3. `## Autenticación (ADFS OAuth2)` — credenciales ADFS
4. `## TrustStore / Certificados` — si aplica
5. `## Despliegue` — Docker/Helm details

> **Nota:** En un BFF no hay sección `## Mensajería (Kafka)` ni `## Base de datos (PostgreSQL)`. El BFF es stateless y no tiene persistencia propia.

**Level of detail:** The infrastructure page provides a **simplified reference** with example values rather than a full authoritative listing. Use `properties` format for environment variable examples, not tables.

**`## Resumen rápido` pattern:**
```markdown
## Resumen rápido
- APIs externas: <N> servicios downstream accesibles vía HTTP/HTTPS (variables: API_<SERVICE>_URL por servicio).
- Autenticación: OAuth2 ADFS client-credentials (variables: ADFS_AUTHORITY, ADFS_APP_ID, ADFS_APP_PASS). Un provider por servicio.
- Certificados/TrustStore (si aplica): TRUSTORE_LOCATION.
- Despliegue: Docker/Helm. Charts en devops/charts/<chart-name>.

---
```

**External APIs env vars format — use `properties` inline code blocks:**
```properties
API_PRODUCT_URL=https://product-v2.api.pre.srv.mercadona.com/...
API_MASTERDATA_URL=https://cdv-back.pre.srv.mercadona.com/web-masterdata/v1
API_MOVEMENTS_URL=https://cdv-back.pre.srv.mercadona.com/web-movements/v1
```

**ADFS auth variables format:**
```properties
ADFS_AUTHORITY=https://fed.itgmercadona.com/adfs/oauth2/token
ADFS_APP_ID=***
ADFS_APP_PASS=***
```

**Deployment section pattern:**
```markdown
## Despliegue

**Docker:**
- **Imagen base:** `<base-image>`
- **Dockerfile:** `devops/Dockerfile`
- **Puerto contenedor:** 8080

**Helm:**
- **Chart:** `devops/charts/<chart-name>/` (v%latest_version%)
- **Dependencia:** `springboot-service ~<version>`
- **Puerto servicio:** 8081
- **Ingress path:** `/<base-path>`
- **Values por entorno:** `devops/charts/env/values-{dev|itg|pre|pro}.yaml`
- **Gestión de secretos:** Conjur (K8s secrets montados desde `/var/run/secrets/`)
```

---

### Integration Tests (`integration_tests.md`)

**Title:** `# Test de integración`

Key points:
- Title is singular: `# Test de integración` (NOT `# Tests de Integración`).
- En un BFF **no hay IT tests con TestContainers** (no hay base de datos propia). La página documenta la estrategia de tests unitarios.
- **No hay páginas hijo** en `wiki.tree` para este topic — es un elemento simple (sin anidamiento).

**Intro paragraph template:**
```markdown
Este documento describe la estrategia y configuración para ejecutar pruebas en el microservicio **%app_name%**.

## Estrategia de Pruebas

Las pruebas verifican la correcta interacción entre los distintos componentes del microservicio, desde los casos de uso hasta los adaptadores de entrada y salida.

Al tratarse de un BFF sin base de datos propia, no existen pruebas de integración con contenedores (TestContainers). Las pruebas están organizadas en dos niveles:

**Tests unitarios de adaptadores driven (MockWebServer):**
Los adaptadores driven se testean de forma aislada usando `MockWebServer` (OkHttp3), que simula las respuestas HTTP de los servicios downstream. Este enfoque valida el mapeo de respuestas, la construcción correcta de las peticiones y el manejo de errores sin necesidad de levantar servicios reales.

**Tests unitarios de casos de uso (Mockito):**
Los casos de uso se testean mockeando los puertos de salida con `Mockito`, verificando la lógica de orquestación y el manejo de escenarios de error.
```

**Test inventory pattern — add the list of existing tests:**
```markdown
Tests existentes por módulo:
- `<driven-module>`: `<AdapterTest>`, `<MapperTest>`
- `application`: `<UseCaseTest>`
```

---

## wiki.tree Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE instance-profile
        SYSTEM "https://resources.jetbrains.com/writerside/1.0/product-profile.dtd">

<instance-profile id="wiki"
                 name="<project-short-name>-wiki"
                 start-page="overview.md">

    <toc-element topic="overview.md"/>
    <toc-element topic="architecture.md"/>
    <toc-element topic="dependencies.md"/>
    <toc-element topic="design-tradeoffs.md"/>
    <toc-element topic="integration_tests.md"/>
    <toc-element topic="business_logic.md">

    </toc-element>
    <toc-element topic="infrastructure.md"/>
</instance-profile>
```

Key differences from a Kafka-consumer wiki.tree:
- `integration_tests.md` es un elemento simple sin páginas hijo (no hay clases IT con TestContainers).
- No hay `<toc-element topic="<TestClassNameIT>.md"/>` anidados.

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
    <var name="spring-boot-version" value="<fwkcna-parent-version>"/>
    <!-- Add one var per downstream service client: -->
    <var name="<service>-client-version" value="<version>"/>
</vars>
```

Key differences from a Kafka-consumer v.list:
- No `instancio-version`, `commons-collections4.version`, `hypersistence-utils-version`, `testcontainers-version`, `hibernate.version` — no aplican a un BFF.
- No variables `avro-*` — el BFF no consume Kafka.
- Se añade una variable `<service>-client-version` por cada artefacto cliente WebClient de servicio downstream.

---

## Rules for Claude

- When asked to document a feature or component, follow the structure and patterns above.
- Always register new pages in `wiki.tree`.
- Always use `%variable_name%` for versions and project name instead of hardcoding.
- Keep documentation concise and technical — no filler text.
- Place diagrams in `wiki/images/` and reference them, never inline large diagrams.
- Match the existing naming conventions: lowercase with hyphens for folders, underscores or hyphens for filenames matching the topic slug.
- **When adding a new endpoint/use case:** add a section in `business_logic.md` (following the endpoint section pattern) AND update the ports table in `architecture.md` if a new port is introduced.
- **Read the source code first:** Before documenting a use case, read its controller adapter, use case class, driven adapters and mappers to entender la orquestación, las llamadas paralelas y el mapeo de datos.
- **Match the prose style:** Use the established Spanish phrases listed above. Keep the same level of detail as existing sections — describe qué endpoint delega en qué use case, qué servicios invoca, si hay paralelismo y qué lógica condicional aplica.
- **Do NOT number section headings** in any wiki file. Sections use plain `##` or `###` headings without numeric prefixes.
- **No batch flow diagram** — los BFF no tienen `flow.mmd`. Si se añade un diagrama de flujo, crear un fichero `.mmd` ad-hoc en `wiki/images/` con nombre descriptivo.
- **infrastructure.md** is a simplified reference page — do not paste exhaustive YAML configs into it. Use the simplified format described in the Infrastructure section above. No incluir sección de Kafka ni PostgreSQL.
- **business_logic.md** documents use cases and endpoints, NOT batch consumer logic. No references to tombstones, batch collections, or Kafka.
- **DOCTYPE declarations are mandatory** in all four XML files (`writerside.cfg`, `wiki.tree`, `c.list`, `v.list`). Never omit them.
- **`web-path` attributes are mandatory** on both `<topics>` and `<images>` elements in `writerside.cfg`.
- **`c.list`** always uses `id="wrs"`, `name="Writerside documentation"`, `order="1"` — do not customise these to the project name.
- **Cross-references** inside `architecture.md` and other pages should use the full `wiki/topics/<folder>/<file>.md` path style, not bare filenames.
- **`integration_tests.md` has NO child pages** in `wiki.tree` for BFF projects. Do not add nested `<toc-element>` entries under it.
