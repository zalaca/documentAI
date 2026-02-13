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

## Required Sections and Writing Patterns per Topic Type

### Overview (`overview.md`)
- Brief description of the microservice/project using `%app_name%`
- Key responsibilities
- Technology stack summary (reference versions from `v.list` with `%var%` syntax)

### Architecture (`architecture.md`)

**Structure:**
1. Estilo Arquitectónico (Hexagonal Architecture explanation)
2. Estructura de Módulos y Capas (module listing)
3. Puertos y Adaptadores
4. Flujo típico de una petición
5. Persistencia
6. Observabilidad
7. Configuración
8. Estrategia de Tests
9. Decisiones de Diseño relevantes
10. Dependencias y Versionado
11. Recursos relacionados

**Module entry pattern** — Each driving adapter module follows this exact one-liner format:
```markdown
- `driving/<module-name>-consumer/`: adaptadores de entrada (Kafka consumer) para eventos de <descripción breve>.
```

Example:
```markdown
- `driving/tariff-consumer/`: adaptadores de entrada (Kafka consumer) para eventos de tarifas.
- `driving/conversion-consumer/`: adaptadores de entrada (Kafka consumer) para eventos de reglas de conversión entre formatos de producto.
```

### Dependencies (`dependencies.md`)
- One `##` section per external dependency
- Purpose, version (use `%var%`), and why it was chosen

### Design Tradeoffs (`design-tradeoffs.md`)
- Document architectural concessions and their rationale
- Trade-off analysis for key decisions

### Business Logic (`business_logic.md`)

**File structure:**
1. Title: `# Lógica de negocio`
2. Intro paragraph referencing `%app_name%`
3. Common batch processing section with Mermaid flow diagram
4. One `##` section per consumer/topic

**Consumer section pattern** — Every consumer section MUST follow this repeatable structure:

```markdown
## <ConsumerName>

En esta réplica guardamos <qué datos se replican y qué campos principales se almacenan>.

<Si hay filtrado o condiciones especiales, describirlos aquí con bloque de código si aplica>

En el objeto `<BatchClassName>` contamos con <N> colecciones:
- `<collectionName>`: donde se almacenan <los datos a insertar o actualizar>.
- `<collectionName>`: donde se almacenan <los ids/datos a eliminar>.

<Lógica adicional si aplica: mapeo complejo, desnormalización, relación con otras entidades, etc.>

<Manejo de tombstones: "Cuando llega un tombstone, se extrae el `<campo>` de la clave del mensaje y se marca para eliminación.">
```

**Example — simple consumer:**
```markdown
## Site gruping

En esta réplica guardamos los datos agrupación de sites.
```

**Example — consumer with filtering:**
```markdown
## NonSalesArticleFormatArticle

En esta réplica, guardamos los datos básicos de los artículos con formato no comercial, pero sólo los que cumplen ciertas condiciones:

\```java
articleTypeId == 2 && (01,02,03,04,05).contains(logisticPackagingTypeId)
\```

Además, guardamos sólo la descripción comercial del artículo en todos los idiomas disponibles.
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
- "Cuando llega un tombstone, se extrae el `<campo>` de la clave del mensaje..."
- "sólo los que cumplen ciertas condiciones:"
- "No replicamos los datos de... ya que no son necesarios en el sistema destino."
- "no se ha implementado la lógica de borrado, puesto que no está previsto que lleguen tombstones en este tópico."

### Infrastructure (`infrastructure.md`)
- Messaging config (Kafka topics, consumer groups)
- Database config (connection, credentials pattern)
- Environment variables
- Deployment details (Docker, Helm, etc.)

### Integration Tests (`integration_tests/`)
- General testing strategy in parent page
- One child page per test class: `<TestClassName>.md`
- Numbered test cases with description and expected assertions

## wiki.tree Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<instance-profile id="wiki"
                 name="<project-name>-wiki"
                 start-page="overview.md">
    <toc-element topic="overview.md"/>
    <toc-element topic="architecture.md"/>
    <toc-element topic="dependencies.md"/>
    <toc-element topic="design-tradeoffs.md"/>
    <toc-element topic="integration_tests.md">
        <!-- Child test pages here -->
    </toc-element>
    <toc-element topic="business_logic.md"/>
    <toc-element topic="infrastructure.md"/>
</instance-profile>
```

## v.list Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<vars>
    <var name="product" value="Writerside"/>
    <var name="latest_version" value="0.1.0"/>
    <var name="java-version" value="21"/>
    <var name="app_name" value="<project-name>"/>
    <!-- Add dependency versions here -->
</vars>
```

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
