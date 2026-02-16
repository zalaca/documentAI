# Project Documentation Guide (Writerside)

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
    └── integration_tests/  # Can have nested child pages
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
                 name="snk-movements-wiki"
                 start-page="overview.md">

    <toc-element topic="overview.md"/>
    <toc-element topic="architecture.md"/>
    <toc-element topic="dependencies.md"/>
    <toc-element topic="design-tradeoffs.md"/>
    <toc-element topic="integration_tests.md">
        <toc-element topic="<TestClassNameIT>.md"/>
        <!-- Add one <toc-element> per IT test child page -->
    </toc-element>
    <toc-element topic="business_logic.md">

    </toc-element>
    <toc-element topic="infrastructure.md"/>
</instance-profile>
```

Key points:
- DOCTYPE is mandatory.
- `name` attribute uses the short form: `snk-movements-wiki` (NOT the full repo name).
- `business_logic.md` element has an intentional blank inner element (not self-closing).
- Integration test child pages use the `IT.md` suffix pattern (e.g. `GeographicalBoundaryUseCaseServiceTestIT.md`).
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
    <var name="instancio-version" value="<version>"/>
    <var name="commons-collections4.version" value="<version>"/>
    <var name="hypersistence-utils-version" value="<version>"/>
    <var name="testcontainers-version" value="<version>"/>
    <var name="spring-boot-version" value="<version>"/>
    <var name="hibernate.version" value="<version>"/>
</vars>
```

Key points:
- DOCTYPE is mandatory.
- The canonical variable set is: `product`, `latest_version`, `java-version`, `app_name`, `instancio-version`, `commons-collections4.version`, `hypersistence-utils-version`, `testcontainers-version`, `spring-boot-version`, `hibernate.version`.
- Do NOT add Avro schema version variables here unless the project explicitly documents per-schema versions. Avro versions belong in `dependencies.md` only.
- Reference variables in markdown as `%variable_name%`.

---

## Required Sections and Writing Patterns per Topic Type

### Overview (`overview.md`)

**Title:** `# Descripción`

**Required sections in order:**
1. Header block with `**bold label:**` fields
2. `## ¿Cómo funciona?` — numbered steps explaining the hexagonal flow
3. `## Módulos del repositorio y su rol` — bulleted list of modules and their roles
4. `## Errores, resiliencia y pruebas` — brief paragraph on error handling and test approach
5. `## Integraciones y datos` — brief paragraph on database and messaging integrations

**Intro header block pattern (use `---` after this block):**
```markdown
# Descripción

**Nombre del Microservicio:** %app_name%.

**Descripción:**
Microservicio responsable de la gestión del consumo y actualización de los datos.

**Propósito:**
Proveer una servicio robusto y lógica de negocio para manejar la actualización de los datos necesarios para la aplicación de movements.

**Responsabilidades Principales:**
- Gestionar la creación, actualización y eliminación de datos.
- Validar reglas de negocio relacionadas con las transformaciones.

**Componentes Principales:**
- **Casos de Uso / Servicios:** Lógica de negocio para transformaciones de productos.
- **Adaptadores / Consumidores:** Capa que contiene los adapters que gestionan el consumo de los eventos de Kafka.
- **Capa de Persistencia:** Repositorios y entidades para transformaciones, sites, geographical boundaries, etc.

**Tecnologías:**
- Java %java-version% / Spring Boot %spring-boot-version%
- JPA / Hibernate para persistencia %hibernate.version%
- Manejo de excepciones y framework de validación
- Pruebas unitarias e integración (JUnit, Mockito, etc.)

---
```

**`## ¿Cómo funciona?` section pattern:**
```markdown
## ¿Cómo funciona?

A alto nivel, el microservicio sigue la Arquitectura Hexagonal (Ports & Adapters) descrita en el documento de Arquitectura y opera así:

1) Entrada (Driving/PUERTO de entrada)
- Los eventos de Kafka llegan al adaptador (driving/*-consumer).
- Los controladores validan y traducen al modelo de aplicación/dominio.

2) Orquestación (Aplicación)
- La capa de aplicación ejecuta casos de uso (application) coordinando reglas de negocio y llamadas a puertos de salida.
- Se asegura la consistencia de datos, las validaciones de negocio y las transiciones de estado.

3) Salidas (Driven/ADAPTADORES)
- Persistencia: Repositorios PostgreSQL (driven/postgres-repository) guardan y consultan datos de producto, surtido, sites, etc.
---
```

**`## Módulos del repositorio y su rol` section pattern:**
```markdown
## Módulos del repositorio y su rol
- driving/*-consumer: Adaptadores Kafka (puertos de entrada).
- application: Casos de uso y orquestación de la lógica de negocio.
- driven/postgres-repository: Adaptadores de salida para persistencia en PostgreSQL (scripts SQL en sql/migration).
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
5. `## Persistencia (PostgreSQL)`
6. `## Observabilidad`
7. `## Configuración`
8. `## Estrategia de Tests`
9. `## Decisiones de Diseño relevantes`
10. `## Dependencias y Versionado`
11. `## Recursos relacionados`

**Module entry pattern** — Each driving adapter module follows this exact one-liner format:
```markdown
- `driving/<module-name>-consumer/`: adaptadores de entrada (Kafka consumer) para eventos de <descripción breve>.
```

Example entries:
```markdown
- `driving/site-consumer/`: adaptadores de entrada (Kafka consumer) para eventos de sites.
- `driving/tariff-consumer/`: adaptadores de entrada (Kafka consumer) para eventos de tarifas.
- `driving/conversion-consumer/`: adaptadores de entrada (Kafka consumer) para eventos de reglas de conversión entre formatos de producto.
```

The `driven` and `boot` modules always close the list:
```markdown
- `driven/postgres-repository/`: adaptadores de salida hacia PostgreSQL (repositorios, mappers, migraciones SQL).
- `boot/`: empaquetado, configuración de arranque y charts específicos.
```

After the module list, always add the layer-to-module mapping paragraph:
```markdown
Relación capas ↔ módulos:
- Entrada (Ports In) → `driving/*` implementan los puertos para el consumo de eventos de Kafka.
- Dominio/Aplicación → `application` contiene casos de uso, servicios y puertos.
- Salida (Ports Out) → `driven/postgres-repository` implementan los puertos para persistencia.

Para un inventario detallado de adapters y métodos, ver wiki/topics/structure.md.
```

**Cross-reference style:** Use the full path `wiki/topics/<file>.md` (not just `<file>.md`) when referencing other wiki documents from within architecture.md. For example:
- `wiki/topics/infrastructure.md`
- `wiki/topics/design-tradeoffs.md`
- `wiki/topics/dependencies.md`
- `wiki/topics/integration_tests.md`

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
### instancio-junit
- **GroupId:** org.instancio
- **ArtifactId:** instancio-junit
- **Version:** %instancio-version%
- **Scope:** test
- **Descripción:** Librería de **generación automática de datos** para pruebas unitarias con **JUnit**. Facilita la creación de objetos con datos de prueba sin necesidad de inicializarlos manualmente.

---
```

Key points:
- Use `GroupId`, `ArtifactId`, `Version`, `Scope` (if non-default), `Descripción` as field names — not `Propósito`, `Artefacto`, or `Versión`.
- Document only the project-specific or notable dependencies — do NOT list every transitive dependency.
- Use `%variable%` for versions wherever a matching `v.list` variable exists.

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

---

### Business Logic (`business_logic.md`)

**Title:** `# Lógica de negocio`

**Intro paragraph template:**
```markdown
Este documento describe la lógica de negocio empleada en cada uno de los consumos de los tópicos del microservicio **%app_name%**.
```

**Common batch section — use this exact title and paragraph:**
```markdown
## Lógica común de los consumidores en modo batch

Por eficiencia en el consumo de eventos y con el objetivo de maximizar la velocidad de replicación, todos los consumidores de los tópicos funcionan en modo batch.
Esto significa que, en lugar de procesar cada evento individualmente a medida que llega, los eventos se agrupan en lotes (batches) y se procesan juntos. Este enfoque tiene una serie de ventajas, pero también implica una complejidad adicional en la gestión de los datos.
Para ello, cada consumidor dispone de un objeto específico para gestionar los datos en batch, que contiene una serie de colecciones para almacenar los datos a insertar/actualizar, borrar, etc.
Hay que tener en cuenta que, al consumir tópicos compactados, el evento que debe prevalecer es el último recibido para cada clave. Por tanto, en la lógica de negocio de cada consumidor se ha tenido en cuenta este aspecto para evitar procesar datos obsoletos.
El proceso de consumo en batch sería el siguiente:

<code-block lang="Mermaid" src="../../images/flow.mmd"/>
```

Key points on the Mermaid reference:
- The correct filename is `flow.mmd` — NOT `batch-flow.mmd`.

**Consumer section title casing:** Use the original document's mixed-case style that matches the actual Kafka/domain naming. In practice this means:
- Simple two-word names: Title Case is acceptable (`## Site Grouping`, `## Supply Chain`)
- Compound proper names: follow the Java class name casing (`## NonSalesArticleFormatArticle`, `## GeographicalBoundary`)
- When in doubt, match the casing used in the original wiki's `business_logic.md`.
- The original uses lowercase multi-word titles (e.g. `## Employee job functions`, `## Site gruping`) — when updating or extending, prefer the original's lowercase style for existing sections to avoid drift.

**Consumer section pattern** — Every consumer section MUST follow this repeatable structure:

```markdown
## <ConsumerName>

En esta réplica guardamos <qué datos se replican y qué campos principales se almacenan>.

<Si hay filtrado o condiciones especiales, describirlos aquí con bloque de código si aplica>

En el objeto `<BatchClassName>` contamos con <N> colecciones:
- `<collectionName>`: donde se almacenan <los datos a insertar o actualizar>.
- `<collectionName>`: donde se almacenan <los ids/datos a eliminar>.

<Lógica adicional si aplica: mapeo complejo, desnormalización, relación con otras entidades, etc.>

Cuando llega un tombstone, se extrae el `<campo>` de la clave del mensaje y se marca para eliminación.
```

**Example — simple consumer:**
```markdown
## Site Grouping

En esta réplica simplemente guardamos los datos de agrupación de sites.

En el objeto `SiteGroupingBatch` contamos con 2 colecciones:
- `siteGroupingsToProcess`: donde se almacenan las agrupaciones a insertar o actualizar.
- `siteGroupingIdsToDelete`: donde se almacenan los ids de agrupaciones a eliminar.

Cuando llega un tombstone, se extrae el id de la clave del mensaje y se marca para eliminación.
```

**Example — consumer with filtering:**
```markdown
## NonSalesArticleFormatArticle

En esta réplica guardamos los datos básicos de los artículos con formato no comercial, pero sólo los que cumplen ciertas condiciones:

```java
articleTypeId == 2 && (01, 02, 03, 04, 05).contains(logisticPackagingTypeId)
```

Además, de todas las descripciones del artículo, guardamos sólo la descripción de tipo comercial (`COM`) en todos los idiomas disponibles.

En el objeto `NonSalesArticleFormatArticleBatch` contamos con 2 colecciones:
- `articlesToProcess`: donde se almacenan los artículos a insertar o actualizar.
- `articleIdsToDelete`: donde se almacenan los ids de artículos a eliminar.

Cuando llega un tombstone, se extrae el id de la clave del mensaje y se marca para eliminación.
```

**Example — consumer with batch collections and tombstone handling:**
```markdown
## GeographicalBoundary Country

En esta réplica guardamos los datos de los países asociados a los límites geográficos. Para cada país se almacenan sus códigos ISO (alpha-2, alpha-3 y numérico), prefijo telefónico, código de marcado directo, nivel de código postal, URL de la imagen de la bandera, nombres en múltiples idiomas y los niveles de jerarquía administrativa del país.

En el objeto `CountryBatch` contamos con 2 colecciones:
- `countriesToProcess`: donde se almacenan los países a insertar o actualizar.
- `geographicalBoundaryCountriesIdsToDelete`: donde se almacenan los ids de geographical boundary de los países a eliminar.

Cuando llega un tombstone, se extrae el `geographicalBoundaryId` de la clave del mensaje y se marca para eliminación.
```

**Key Spanish phrases to reuse:**
- "En esta réplica guardamos..." / "En esta réplica simplemente guardamos..."
- "contamos con N colecciones:"
- "donde se almacenan los/las..."
- "Cuando llega un tombstone, se extrae el `<campo>` de la clave del mensaje y se marca para eliminación."
- "sólo los que cumplen ciertas condiciones:"
- "No replicamos los datos de... ya que no son necesarios en el sistema destino."
- "no se ha implementado la lógica de borrado, puesto que no está previsto que lleguen tombstones en este tópico."
- "guardamos sólo la descripción comercial del artículo en todos los idiomas disponibles."
- "almacenamos un array de los ancestros de cada <entidad> con una consulta recursiva."
- "aplanándolo en la propia tabla de <entidad>."

**Separator rule:** Place `---` after each consumer section to visually separate sections.

---

### Infrastructure (`infrastructure.md`)

**Title:** `# Infraestructura`

**Intro paragraph template:**
```markdown
Este documento resume la configuración de infraestructura necesaria para ejecutar el microservicio **%app_name%** tanto en local como en entornos de despliegue.
```

**Required sections in order:**
1. `## Resumen rápido` — brief bullet list of key infrastructure components and their env vars
2. `## Mensajería (Kafka)` — Kafka configuration
3. `## Base de datos (PostgreSQL)` — DB connection variables
4. `## Despliegue` — Docker/Helm details

**Level of detail:** The infrastructure page provides a **simplified reference** with example values rather than a full authoritative listing. Use `properties` format for environment variable examples, not tables. Use simplified `yaml` snippets with illustrative (non-exhaustive) topic examples.

**`## Resumen rápido` pattern:**
```markdown
## Resumen rápido
- Base de datos: PostgreSQL (variables: LOCAL_DB_URL, LOCAL_DB_USER, LOCAL_DB_PASSWORD; migraciones: FLYWAY_DATABASE_USERNAME, FLYWAY_DATABASE_PASSWORD).
- Mensajería: Kafka (variables: BOOTSTRAP_SERVERS, SCHEMA_REGISTRY_URL, KAFKA_USER, KAFKA_PASSWORD). Tópico de email configurado en application.yaml.
- Certificados/TrustStore (si aplica): TRUSTORE_LOCATION.
- Despliegue: Docker/Helm. Charts en devops/charts/<chart-name> y boot/charts/<chart-name>.

---
```

**Kafka config snippet style (simplified example — not exhaustive topic list):**
```yaml
fwkcna:
  kafka:
    consumer:
      topics:
          input-0: <topic-name-0>
          input-1: <topic-name-1>
          group-id-0: <consumer-group-0>
          group-id-1: <consumer-group-1>
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

---

### Integration Tests (`integration_tests.md`)

**Title:** `# Test de integración`

Key points:
- Title is singular: `# Test de integración` (NOT `# Tests de Integración`).

**Intro paragraph template:**
```markdown
Este documento describe la estrategia y configuración para ejecutar pruebas de integración en el microservicio **%app_name%**.

## Estrategia de Pruebas de Integración

Las pruebas de integración verifican la correcta interacción entre los distintos componentes del microservicio, desde el caso de uso hasta las adaptaciones de entrada y salida.
```

Key points:
- The parent `integration_tests.md` page is intentionally brief. It contains the general strategy section only.
- Each individual test class gets its own child page registered in `wiki.tree`.
- Child page filenames use the `IT.md` suffix pattern matching the Java test class names (e.g. `GeographicalBoundaryUseCaseServiceTestIT.md`).

---

## wiki.tree Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE instance-profile
        SYSTEM "https://resources.jetbrains.com/writerside/1.0/product-profile.dtd">

<instance-profile id="wiki"
                 name="snk-movements-wiki"
                 start-page="overview.md">

    <toc-element topic="overview.md"/>
    <toc-element topic="architecture.md"/>
    <toc-element topic="dependencies.md"/>
    <toc-element topic="design-tradeoffs.md"/>
    <toc-element topic="integration_tests.md">
        <toc-element topic="GeographicalBoundaryUseCaseServiceTestIT.md"/>
        <toc-element topic="SiteGroupingUseCaseServiceTestIT.md"/>
        <toc-element topic="SiteUseCaseServiceTestIT.md"/>
        <!-- Add one <toc-element> per IT test child page -->
    </toc-element>
    <toc-element topic="business_logic.md">

    </toc-element>
    <toc-element topic="infrastructure.md"/>
</instance-profile>
```

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
    <var name="hypersistence-utils-version" value="<version>"/>
    <var name="testcontainers-version" value="<version>"/>
    <var name="spring-boot-version" value="<version>"/>
    <var name="hibernate.version" value="<version>"/>
</vars>
```

---

## Rules for Claude

- When asked to document a feature or component, follow the structure and patterns above.
- Always register new pages in `wiki.tree`.
- Always use `%variable_name%` for versions and project name instead of hardcoding.
- Keep documentation concise and technical — no filler text.
- Place diagrams in `wiki/images/` and reference them, never inline large diagrams.
- Match the existing naming conventions: lowercase with hyphens for folders, underscores or hyphens for filenames matching the topic slug.
- **When adding a new consumer:** add both a section in `business_logic.md` (following the consumer section pattern) AND a module entry in `architecture.md` (following the one-liner module pattern).
- **Read the source code first:** Before documenting a consumer, read its adapter, mapper, batch class, and use case service to understand filtering, special logic, and batch collections.
- **Match the prose style:** Use the established Spanish phrases listed above. Keep the same level of detail as existing sections — describe what's stored, any filtering conditions, batch collections, and tombstone handling.
- **Do NOT number section headings** in any wiki file. Sections use plain `##` or `###` headings without numeric prefixes.
- **Use `flow.mmd`** (not `batch-flow.mmd`) as the Mermaid diagram filename in `business_logic.md`.
- **infrastructure.md** is a simplified reference page — do not paste exhaustive topic tables or full YAML configs into it. Use the simplified format described in the Infrastructure section above.
- **DOCTYPE declarations are mandatory** in all four XML files (`writerside.cfg`, `wiki.tree`, `c.list`, `v.list`). Never omit them.
- **`web-path` attributes are mandatory** on both `<topics>` and `<images>` elements in `writerside.cfg`.
- **`c.list`** always uses `id="wrs"`, `name="Writerside documentation"`, `order="1"` — do not customise these to the project name.
- **Cross-references** inside `architecture.md` and other pages should use the full `wiki/topics/<file>.md` path style, not bare filenames.
