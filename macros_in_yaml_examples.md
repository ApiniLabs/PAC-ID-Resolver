# Macros in YAML

## Purpose

YAML macros provide a way to **reduce repetition** in resolver configurations.  
They allow commonly used entry definitions—such as recurring service types or shared fields—to be defined once and reused across multiple rules.

Macros increase maintainability and readability of larger resolver configurations, where several entries share identical properties except for the `template_url` or rule condition.

---

## Definition

Macros are defined under a top-level key named `macros`.  
Each macro is declared as a named mapping and can be referenced elsewhere in the YAML using the YAML merge operator (`<<: *alias`).

The `macros` section **MUST** appear before the `config` section.

---

## Example

```yaml
macros:
  shop: &shop
    service_type: userhandover-generic
    service_name: Shop
    application_intents:
      - reorder

config:
# Instruments
- if: $.categories[?(@.key == '-MD')]
  entries:
  - <<: *shop
    template_url: https://shop.mettorius.com/instruments

# Consumables
- if: $.categories[?(@.key == '-MC')]
  entries:
  - <<: *shop
    template_url: https://shop.mettorius.com/consumables
```