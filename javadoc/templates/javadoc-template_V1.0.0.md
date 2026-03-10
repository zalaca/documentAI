---
name: javadoc-writer
description: >
  Add or update Javadoc comments in Java Spring Boot domain classes and service
  methods for WEB/SNK/BTC/BFF microservices with hexagonal architecture.
  Use when documenting domain classes, application services, strategies, or
  any class with non-trivial logic.
  Covers: class-level Javadoc, method Javadoc with @param/@return, field
  comments for non-obvious fields, and inline comments for complex logic.
  Does NOT add comments to trivial fields (id, name, etc.) or self-explanatory
  simple getters/setters. This is the V1.0.0 version.
---

# Code Documentation Guide — BTC/WEB/SNK Microservices

## General Rules

- **Write in English** — all Javadoc and inline comments must be in English.
- **Comment non-obvious things only.** If the code reads naturally, skip the comment.
- **Do not comment trivial fields** such as `id`, `name`, `description`, `code`, `value` when the type and name already make the purpose clear.
- **Do comment fields** that encode domain concepts using opaque identifiers, flags with non-intuitive semantics, or composite keys.
- **Do not add Javadoc to simple Lombok-only POJOs** if all fields are self-explanatory. Add only where there is meaningful context to convey.
- **Always document public service methods** (non-trivial ones) with full Javadoc.
- **Use inline comments** (`//`) for complex algorithm steps, non-obvious conditional branches, or workarounds.

---

## 1. Class-Level Javadoc

Place above the class declaration, below the package/import block.

### Pattern

```java
/**
 * [One-line summary of what this class represents or does.]
 *
 * [Optional: 2–3 sentences of additional context — when is it used,
 * what domain concept does it model, or what problem does it solve.]
 *
 * [Optional: note any important constraints, invariants, or relationships
 * with other domain objects.]
 */
@Data
@Builder
public class MyDomainClass { ... }
```

### When to add class Javadoc

| Class type | Add Javadoc? | Notes |
|---|---|---|
| Domain entity / value object | Yes, if non-trivial | Describe the business concept |
| Domain enum | Yes | Explain what the enum discriminates |
| Abstract strategy / service | Yes | Explain the pattern and lifecycle |
| Concrete strategy implementation | Yes | Describe which movement type it handles |
| Batch class (`*Batch`) | Yes | Explain partition logic and AutoCloseable lifecycle |
| Kafka consumer adapter | Yes | Describe the topic and routing logic |
| Port interface (driving/driven) | Yes | One-line contract description |
| Simple DTO / filter | Optional | Only if the name is ambiguous |
| Exception class | Optional | Only if the semantics are non-obvious |

### Examples

```java
/**
 * Represents a sales price entry for a specific boundary and tariff type.
 *
 * A boundary identifies the geographic or commercial scope (e.g. a country
 * code or exception zone), while the type discriminates between general and
 * exception tariffs. Together, {@code boundaryId} and {@code typeId} form the
 * composite natural key used when sending data to the external pricing API.
 */
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class SalesPrice {
    ...
}
```

```java
/**
 * Defines the allowed stage transitions for a given movement type.
 *
 * Each record establishes a directed edge in the movement workflow graph:
 * from a source stage ({@code fromStatus}) to a target stage ({@code toStatus}).
 * The {@code isRoot} flag marks the entry-point transition — i.e., the initial
 * edge that can be triggered without any prior stage.
 */
@Data
public class MovementTypeStageTransition {
    ...
}
```

```java
/**
 * Base class for all consolidation strategies applied to a movement type.
 *
 * Each concrete subclass handles the consolidation logic for a specific
 * movement type (e.g. PVP variation, cost price variation). Subclasses must
 * implement {@link #getType()}, {@link #compoundData(List)}, and
 * {@link #consolidate(MovementConsolidate)}.
 *
 * <p>Consolidation is the process of aggregating the data from multiple blocks
 * into a single domain object that is then pushed to the external golden record
 * system.</p>
 */
@AllArgsConstructor
public abstract class AbstractConsolidationMovementService<T extends MovementConsolidate> {
    ...
}
```

---

## 2. Field-Level Comments

Add a single-line `/** ... */` or `//` comment above fields that encode non-obvious domain concepts.

### Fields that DO need a comment

- Opaque identifier fields (`boundaryId`, `typeId`, `signageId`) — explain what system or domain they refer to.
- Boolean flags with non-intuitive semantics (`isUrgent`, `isRoot`).
- Composite natural keys.
- Fields whose nullability has business meaning (e.g. null = "not yet defined").
- Fields that shadow or extend a concept from an external system.
- Kafka metadata fields (`toDelete`, `timestamp`, `offset`) — see Section 8.

### Fields that do NOT need a comment

- `id`, `name`, `code`, `description`, `value` — self-explanatory.
- Standard audit fields (`createdAt`, `updatedAt`).
- Fields whose name matches exactly the column or API attribute they map to.
- Fields that merely restate the field name: `/** Tariff Identifier */` over `tariffId` is noise.

### Pattern

```java
/** [What this field represents and where the value comes from or how it is used.] */
private String myField;
```

### Examples

```java
/**
 * Geographic or commercial boundary identifier.
 * For general tariffs this is a country ISO code (e.g. "ES", "PT").
 * For exception tariffs this is a zone code (e.g. "T3", "T5").
 * Together with {@code typeId} it forms the composite PK sent to the pricing API.
 */
@NotBlank
@Size(max = 10)
private String boundaryId;

/**
 * Tariff type discriminator sent to the pricing API.
 * Matches the {@code id} field of {@link TariffType} (e.g. "1" for GENERAL, "2" for EXCEPTION).
 * Note: this is NOT the human-readable name — it is the numeric API code.
 */
@NotBlank
@Size(max = 5)
private String typeId;

/**
 * When {@code true}, the price change must be processed with priority
 * and communicated to stores ahead of the normal schedule.
 */
private Boolean isUrgent;

/**
 * Marks this transition as the entry point of the workflow.
 * A root transition has no required predecessor stage and can be triggered
 * directly after movement creation.
 */
private Boolean isRoot;
```

---

## 3. Method-Level Javadoc

### Pattern

```java
/**
 * [One-line summary of what the method does.]
 *
 * [Optional: describe the algorithm or business logic at a high level,
 * especially if it is non-trivial. Mention side effects if any.]
 *
 * @param paramName [what the param represents and any relevant constraints]
 * @param otherParam [...]
 * @return [what the return value represents; omit for void]
 * @throws SomeException [when and why this exception is thrown]
 */
```

### When to add method Javadoc

| Method type | Add Javadoc? |
|---|---|
| Public service/use-case method | Always |
| Abstract method defining a contract | Always |
| Public utility / factory method | Yes if non-trivial |
| Private helper with complex logic | Inline comment preferred |
| Simple getter / setter (non-Lombok) | No |
| Override of a well-known framework method | Optional — only if behaviour differs |

### Examples

```java
/**
 * Retrieves a paginated list of employees matching the provided filter criteria.
 *
 * @param pageConfig pagination configuration (page number, size, sort)
 * @param filter filter criteria to apply when searching for employees
 * @return paginated list of employees matching the filter
 */
public Page<Employee> findEmployees(PageConfig pageConfig, EmployeeFilter filter) { ... }
```

```java
/**
 * Validates that all declared fields in the consolidate object are non-null.
 *
 * Uses reflection to iterate over the fields of the concrete consolidate class.
 * If any field is null, throws {@link UnauthorizedException} indicating that
 * the movement is not ready to be consolidated (missing required concepts).
 *
 * @param consolidate the movement consolidate object to validate
 * @throws UnauthorizedException if one or more required fields are null
 */
protected void validateConceptsNullability(MovementConsolidate consolidate) { ... }
```

```java
/**
 * Builds the composite natural key used to match this price entry against
 * existing records in the pricing API.
 *
 * The key is formed by concatenating {@code boundaryId} and {@code typeId}
 * with an underscore separator (e.g. {@code "ES_1"} or {@code "T3_2"}).
 *
 * @return composite key string in the format {@code "<boundaryId>_<typeId>"}
 */
public String toSalesPriceBoundaryTypePk() { ... }
```

```java
/**
 * Returns the movement type this strategy is responsible for consolidating.
 *
 * Used by the framework to route consolidation requests to the correct
 * strategy implementation at runtime.
 *
 * @return the {@link MovementTypeEnum} value associated with this strategy
 */
public abstract MovementTypeEnum getType();
```

---

## 4. Inline Comments for Complex Logic

Use `//` inline comments inside method bodies when:

- A loop or branch implements a non-obvious algorithm.
- A condition encodes a business rule that is not readable from the code alone.
- A workaround or known limitation exists.
- The order of operations matters and is not self-evident.

### Pattern

```java
// [Why this is done, not what — the code already shows what]
someComplexOperation();

// [Business rule]: [explanation]
if (someFlag && otherCondition) { ... }
```

### Examples

```java
// Use reflection to check all declared fields of the concrete class at runtime,
// since the set of required fields varies per movement type.
for (Field field : consolidate.getClass().getDeclaredFields()) {
        field.setAccessible(true);
    try {
Object value = field.get(consolidate);
// A single null field is enough to consider the consolidate incomplete.
        if (value == null) {
isValid = false;
        break;
        }
        } catch (IllegalAccessException e) {
        // Field access should never fail here; accessibility is set above.
        }
        }
```

```java
// The authFilter must be registered first so the framework's client-credentials
// token is always present before the propagation filter potentially overwrites it.
return WebClient.builder()
    .filter(authFilter())
        .filter(propagateAuthorizationFilter())
        .build();
```

```java
// Only overwrite the Authorization header if the downstream request did not
// already carry one (e.g. user-delegated token takes precedence).
if (!request.headers().containsKey(AUTHORIZATION)) {
        ...
        }
```

---

## 5. Enum Javadoc

Document the enum class and any non-obvious constants or fields.

```java
/**
 * Defines the tariff categories recognised by the external pricing API.
 *
 * Each constant holds two identifiers:
 * <ul>
 *   <li>{@code id} — the numeric code sent to the API (e.g. {@code "1"})</li>
 *   <li>{@code value} — the string used internally as the {@code typeId} in
 *       {@link SalesPrice} and {@link SalesPriceVariation}</li>
 * </ul>
 */
@Getter
@RequiredArgsConstructor
public enum TariffType {

  /** Standard country-wide tariff. */
  GENERAL("1", "COUNTRY"),

  /** Zone-specific exception tariff (e.g. border or special regions). */
  EXCEPTION("2", "EXCEPTION");

  /** Numeric code expected by the pricing API. */
  private final String id;

  /**
   * String discriminator stored in {@link SalesPrice#typeId}.
   * Maps to the {@code typeId} field — not the API numeric code.
   */
  private final String value;
}
```

---

## 6. What NOT to Comment

- Do not add Javadoc to Lombok-generated methods (`@Data`, `@Builder`, etc.).
- Do not comment `@Override` methods that simply delegate without additional logic.
- Do not state the obvious: `// sets the value` above `this.value = value` is noise.
- Do not duplicate the method signature in the Javadoc: `@param id the id` adds no value.
- Do not add `@author`, `@since`, or `@version` tags — version control handles that.
- Do not comment import blocks.
- Do not add field comments that merely restate the field name (e.g. `/** Tariff Identifier */` over `tariffId`).

---

## 7. SNK: Kafka Metadata Fields

These three fields appear in all SNK domain entities. Their names look simple but their semantics are non-obvious — **always document them**.

```java
/** When {@code true}, this record must be deleted instead of upserted. */
private Boolean toDelete;

/** Kafka message timestamp; used alongside {@code offset} for deduplication ordering within a batch. */
private Long timestamp;

/** Kafka partition offset; combined with {@code timestamp} to resolve the latest record per key. */
private Long offset;
```

---

## 8. SNK: Batch Classes

Batch classes aggregate Kafka events into two disjoint sets (upsert vs. delete) and implement `AutoCloseable` for explicit memory release.

```java
/**
 * Aggregates a list of [Domain] Kafka events into records to upsert and IDs to delete.
 *
 * <p>Records are sorted by offset and timestamp (descending) before partitioning,
 * so the latest event per key takes precedence within the batch.</p>
 *
 * <p>Implements {@link AutoCloseable} to allow explicit release of both sets
 * after the batch has been processed.</p>
 */
public class XxxBatch implements AutoCloseable { ... }
```

**Field comments inside batch classes are not needed** — `xxxToProcess` and `idsToDelete` are self-explanatory.

---

## 9. SNK: Kafka Consumer Adapters

```java
/**
 * Kafka consumer adapter for processing [Domain] events.
 *
 * Partitions the incoming batch into valid records and deserialization errors,
 * delegates valid records to {@link XxxPort}, and tracks failures via Micrometer counters.
 */
@Slf4j
@Component
public class XxxConsumerAdapter extends MercadonaKafkaBatchConsumerListener<K, V> { ... }
```

Standard method patterns — one-line summaries only:

```java
/** Processes a batch of consumer records, routing deserialization errors to the error handler. */
@Override
public void consume(List<ConsumerRecord<K, V>> events) { ... }

/**
 * Processes a list of valid records as a single batch.
 * Falls back to per-record processing if the batch fails.
 *
 * @param eventsToProcess the list of valid consumer records to process
 */
public void processRecords(List<ConsumerRecord<K, V>> eventsToProcess) { ... }

/**
 * Processes a single consumer record, delegating errors to the error handler.
 *
 * @param eventToProcess the consumer record to process
 */
public void processRecord(ConsumerRecord<K, V> eventToProcess) { ... }
```

Private methods (`processError`, `logError`) do not need Javadoc.

---

## 10. SNK: Port Interfaces

```java
/**
 * Driving port defining the contract for [Domain] operations.
 */
public interface XxxPort {

  /**
   * Processes a batch of [domain] records (batch path).
   *
   * @param batch the batch to process
   */
  void processAll(XxxBatch batch);

  /**
   * Processes a single [domain] record with retry (individual fallback path).
   *
   * @param batch the single-record batch to process
   */
  void processOne(XxxBatch batch);
}
```

Driven ports (datasource) follow the same pattern — one-line class Javadoc, `@param` on each method.

---

## 11. WEB: REST Controller Adapters

REST controller adapters in `driving/api-rest` translate HTTP requests to application port calls. They are worth documenting when they implement non-trivial routing, parameter transformation, or business-conditional logic.

### Class-level pattern

```java
/**
 * REST controller adapter for <domain>-related API endpoints.
 *
 * <p>This adapter implements the <Name>Api interface and provides REST endpoints
 * for <short description of what it exposes>.
 */
@RestController
@RequiredArgsConstructor
public class XxxControllerAdapter implements XxxApi { ... }
```

### Method-level pattern

```java
/**
 * Retrieves <resource> for the given <context>.
 *
 * @param locale     the locale used to localise the response
 * @param categoryId the category identifier used to filter results
 * @return a {@link ResponseEntity} containing a list of {@link XxxDto} objects
 */
@Override
public ResponseEntity<List<XxxDto>> getXxx(String locale, String categoryId) { ... }
```

### Rules specific to controllers

- Add method Javadoc when the method has **2 or more parameters** or performs any non-obvious mapping (e.g. enum resolution, null fallback, conditional logic).
- One-liner delegate methods with a single parameter may use a one-line summary without `@param`.
- Do **not** document `@Override` methods that are pure pass-throughs with zero transformation logic.

---

## 12. WEB: API-Driven Adapters (WebClient / HTTP clients)

Driven adapters in `driven/*-datasource` that call external HTTP services deserve class and method Javadoc to make their contract visible without reading the service layer.

### Class-level pattern

```java
/**
 * Driven adapter for the <ServiceName> HTTP API.
 *
 * <p>Provides access to <describe what data it fetches> via WebClient.
 * Implements {@link XxxDatasourcePort}.
 */
@Component
@RequiredArgsConstructor
public class XxxDatasourceAdapter implements XxxDatasourcePort { ... }
```

### Method-level pattern (same as service methods)

```java
/**
 * Retrieves <resource> matching the given <criteria>.
 *
 * @param description partial name or description used to search
 * @return list of matching {@link XxxDomain} objects; empty list if none found
 */
@Override
public List<XxxDomain> getXxx(String description) { ... }
```

### Example (BFF adapter)

```java
/**
 * Driven adapter for the corporate BFF HTTP API.
 *
 * <p>Provides product brand and tariff data that is not available
 * directly from Golden Record or Masterdata. Implements {@link BffDatasourcePort}.
 */
public class BffDatasourceAdapter implements BffDatasourcePort {

    /**
     * Retrieves product brands whose name matches the given description.
     *
     * @param description partial brand name used as search criterion
     * @return list of matching {@link ProductBrand} objects
     */
    public List<ProductBrand> getBrands(String description) { ... }

    /**
     * Retrieves tariffs for the given tariff type code.
     *
     * @param tariffType the tariff type identifier (e.g. "1" for GENERAL, "2" for EXCEPTION)
     * @return list of {@link Tariff} objects for the requested type
     */
    public List<Tariff> getTariffs(String tariffType) { ... }
}
```

---

## 13. Quick Reference Checklist

Before finishing documentation on a file, verify:

- [ ] Class has Javadoc if it models a non-trivial domain concept or strategy.
- [ ] All public service/use-case methods have `@param` and `@return` Javadoc.
- [ ] Abstract contract methods have Javadoc explaining the expected contract.
- [ ] Non-obvious fields (`boundaryId`, `typeId`, boolean flags, composite keys) have a comment.
- [ ] Complex loops, conditionals, or ordering-sensitive code have inline `//` comments.
- [ ] Enum constants with domain-specific meaning have a brief `/** ... */` comment.
- [ ] No trivial fields (`id`, `name`, `value`) were over-commented.
- [ ] No comments merely restate what the code already says.
- [ ] No `@author`, `@version`, or `@since` tags present.
- [ ] Batch class has Javadoc explaining its partition logic and `AutoCloseable` lifecycle.
- [ ] Consumer adapter class has Javadoc; `consume`/`processRecords`/`processRecord` have one-line summaries.
- [ ] Port interface has class-level Javadoc; each method has `@param`.
- [ ] Kafka metadata fields (`toDelete`, `timestamp`, `offset`) are documented in every domain entity.
- [ ] REST controller adapter has class-level Javadoc; methods with 2+ params or non-trivial logic have `@param`/`@return`.
- [ ] API-driven adapter (WebClient/HTTP) has class-level Javadoc and method Javadoc on all public methods.
