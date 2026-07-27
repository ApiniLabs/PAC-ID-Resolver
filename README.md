# PAC-ID Resolver

## `PAC-ID Resolver` in a Nutshell

A `PAC-ID Resolver`, short for **P**ublicly **A**ddressable **C**ontent **ID**entifier **Resolver**, is an architectural pattern for loosely coupling applications, creating modern user experience without classical integration. The basis for the coupling is a `PAC-ID`.

Working principle: The application asks the `PAC-ID Resolver` to convert a given `PAC-ID` into an ordered list of coupling information, based on resolver configurations provided to the `PAC-ID Resolver`. The application in turn selects the best coupling information from the list, based on an application intent. The coupling information, depending on the indicated service type, contains a URL for user handovers or an endpoint for standardized inter-application communication.

## Introduction

In laboratory workflows, the utilization of multiple software applications is commonplace. A seamless exchange of contextual data between these applications is essential to provide users with a state-of-the-art experience.

The conventional approach involves interfacing applications with each other. Unfortunately, due to the lack of standards, such interfaces typically require bespoke software development or customization projects, demanding substantial effort to create and maintain.

In contrast, the PAC-ID Resolver architecture offers an alternative and lightweight solution for connecting applications in a loosely coupled manner. Implemented as a neutral contract between systems, it eliminates inter-dependencies and therefore facilitates adding additional or replacing existing systems. Its workflow context awareness allows seamless interactions that result in an exceptional user experience.

## Interaction of an Application with a `PAC-ID Resolver`

1. The application submits a `PAC-ID` to the `PAC-ID Resolver`.
2. The `PAC-ID Resolver` looks up `resolver configuration`.
3. The `PAC-ID Resolver` selects service entries in the resolver configuration that match the `PAC-ID` and substitutes variables.
4. The `PAC-ID Resolver` returns an ordered list of coupling information entries, together with the source of the resolver configuration. The order is defined by the precedence of the sources of the resolver configurations and the sequence of the entries in these configurations (first entries have higher precedence).
5. The application selects the most appropriate coupling information based on the application intent.

## Resolver Configuration

### Format of a Resolver Configuration

A resolver configuration is a UTF-8 encoded YAML document that defines how `PAC-ID`s map to services.  
It is composed of three top-level keys:

- **`schema_version`**: identifies the version of the configuration format. MUST be `2.0`.
- **`origin`**: specifies the provenance or ownership of the configuration, e.g., `PERSONAL`, `ORGANIZATIONAL`, or `GLOBAL`.
- **`config`**: an array of rule blocks. Each block contains an `if` condition that determines when it applies and an `entries` list defining one or more services.

A resolver configuration MAY additionally have a top-level **`macros`** key, used to reduce repetition via YAML anchors and merge keys — see [Macros in YAML](macros_in_yaml_examples.md). Since `macros` is resolved by the YAML parser itself, it MUST appear before `config` in the document.

Each service entry describes a single integration point or handover option.  

#### Example

```yaml
schema_version: '2.0'
origin: PERSONAL

config:
# This is a comment
- if: $.issuer == METTORIUS.COM AND $.categories[?(@.key == '-MD')]
  entries:
  - service_type: userhandover-generic
    service_name: Product Page
    application_intents:
      - ProdInfo
    template_url: https://www.mettorius.com/inventory/{$.categories[0].segments[?(@.key == '240')].value}/{$.categories[0].segments[?(@.key == '21')].value}

- if: $.issuer == METTORIUS.COM
  entries:
  - service_type: attributes-generic
    service_name: Attributes
    application_intents:
      - ChemProps 
    template_url: https://attributes.mettorius.com/{$.issuer}/{$.categories[0].key}/{$.categories[0].segments[?(@.key == '21')].value}
```

**Field Name** | **Description** |
| :--- | :--- |
**if** | A `rule` defining which conditions a `PAC-ID` SHOULD fulfil in order to match the entry. (see [matching](#matching-a-pac-id-to-entries-in-a-resolver-configuration)) 
**entries** | A list of services, each containing the following fields:
**service_name** | Describes the entry in human readable, US-English language. The service name SHALL contain only letters `a-zA-Z`, numbers `0-9`, spaces ` ` or hyphens `-`. It MUST contain at least one but not more than 255 characters. |
**application_intents**  | A list of strings identifying the use case(s) the entry fulfills (see [Application Intent](#application-intent)). Strings ending with `-generic` SHALL be reserved for future use. Each string SHALL contain only letters `a-zA-Z`, numbers `0-9` or hyphens `-`, but no spaces. Each string MUST contain at least one but not more than 64 characters. |
**service_type** | The service type MUST be one of the following strings:<ul><li>`userhandover-generic`: The template URL is a navigable URL leading to content made for human consumption (e.g. a HTML page, a PDF file).</li><li>`attributes-generic`: The template URL MUST lead to an `Attributes Service` endpoint.</li><li>`action-generic`: The template URL, when requested, triggers an action (e.g. a backend operation) rather than returning content for display or an attributes payload.</li></ul> |
**template_url** | A template URL which MAY contain one or more instances of a `variable`. The Template Url is inspired by [RFC 6570 URI Template](https://datatracker.ietf.org/doc/html/rfc6570): A `variable` corresponds to an RFC6570 "*expression*"; the `PAC-ID Resolver` to a RFC 6570 "*template processor*". When replacing the `variable`s with the appropriate values from a `PAC-ID`, the result MUST become a valid URL. |



### Application Intent

`application_intents` lets the creator of a resolver configuration tailor entries to specific use cases. An application resolving a `PAC-ID` looks for the entries whose `application_intents` include the intent(s) it cares about, and prioritizes those matches over entries that don't declare a matching intent. This means the same `PAC-ID` can resolve to different links depending on which application — or which specific action within that application — is doing the resolving: for example, an inventory application might look for the intent `reorder`, while a lab notebook might look for `chem-info`, even though both resolve the same `PAC-ID` against the same resolver configuration.

<span style="color:red">Open question for the team: MAY `application_intents` be empty or absent, i.e. an entry declares no intent at all? If so, how should such an entry be treated when an application filters by a specific intent — always match, as a fallback/default entry, or never match?</span>

### Sources and Precedence of a Resolver Configuration

A `PAC-ID Resolver` SHALL be able to retrieve resolver configurations from multiple sources:

- loading from a file system
- retrieving from a http(s) URL
- retrieving the issuer's **Global Resolver Configuration**. A `PAC-ID Resolver` SHALL first attempt “`https://pac.{issuer}/resolver_config.yaml`”. Where “`issuer`“ MUST be replaced by the appropriate value from the `PAC-ID`.

An issuer of `PAC-ID`s SHOULD host its Global Resolver Configuration at this location, so that any `PAC-ID Resolver` can resolve the `PAC-ID`s it issues without additional configuration.

By default, a `PAC-ID Resolver` SHALL retrieve and include the issuer's Global Resolver Configuration automatically, without the application having to supply it explicitly. A `PAC-ID Resolver` MAY offer a way to opt out of this (e.g. for offline use, or when an application wants full control over which sources are consulted).

The sources and their precedence SHALL be configurable. Precedence across sources is defined by the order in which their resolver configurations are supplied to the resolver, followed by the sequence of entries within each configuration (first entries have higher precedence). The issuer's Global Resolver Configuration, when retrieved automatically, SHALL have the lowest precedence, i.e. it is considered after any explicitly configured resolver configurations. The application MAY inspect the source of each returned coupling information entry to make this precedence visible.

### Matching a `PAC-ID` to Entries in a Resolver Configuration


#### Rule Syntax

Rules are written as logical expressions that combine predicates using the following operators:

- Logical operators: `AND`, `OR`, `NOT`. These keywords are case-sensitive and MUST be written in upper case.
- Parentheses `(` `)` for grouping, including repeated negation (e.g. `NOT NOT (...)`).

An `if` condition that is omitted, empty, or `null` SHALL always be considered applicable.

A **predicate** is one of the following:

- A **JSONPath expression** that checks for the existence of elements in the `PAC-ID` JSON.  
  Example:

  ```text
  $.categories[?(@.key == '-MD')]
  ```

- A **JSONPath expression with a comparison** to a literal value (quoted or unquoted), or to another JSONPath expression — the JSONPath expression MAY appear on either side of the comparison. Comparisons use one of `==`, `!=`, `<`, `<=`, `>`, `>=` and are performed on the string representation of both operands; `==` and `!=` are case-insensitive.  
  Example (both equivalent):

  ```text
  $.issuer == METTORIUS.COM
  METTORIUS.COM == $.issuer
  ```

> **Design Choice:** JSONPath is used because it provides a standardized and expressive way to query nested JSON structures—such as those derived from `PAC-ID` with `PAC-CAT` and extensions. This enables flexible conditions without custom parsing logic.

Example logical expression, which matches any `PAC-ID` issued by `METTORIUS.COM` that contains the `-MD` category.
```text
$.issuer == METTORIUS.COM AND $.categories[?(@.key == '-MD')]
```

---

### Querying the PAC-ID with JSONPath

Before JSONPath queries can be evaluated, a `PAC-ID` **MUST** be represented as a JSON object in a standardized structure.  
This structure defines how the components of a `PAC-ID` (issuer, identifier, categories, and extensions) are serialized for evaluation.

#### JSON Serialization of a PAC-ID

A `PAC-ID` **MUST** be serialized into JSON as follows:

| Field | Type | Description |
| :--- | :--- | :--- |
| `issuer` | `string` | The domain name of the `PAC-ID` issuer. |
| `identifier` | `array` of objects | Ordered list of identifier segments, as parsed (independent of any `PAC-CAT` interpretation). Each object contains:<br>• `key`: string or `null` if no key is present.<br>• `value`: string representing the segment value. |
| `categories` | `array` of objects | Categories derived from the identifier per `PAC-CAT`, in identifier order; `[]` if the identifier has no category segments. Each category object contains:<br>• `key`: the category key (e.g., `-MD`).<br>• `segments`: array of `{key, value}` pairs belonging to that category, keyed by the segment's well-known key as defined by `PAC-CAT` (e.g., `21` for a serial number) or `null` if the segment has no key. |
| `extensions` | `array` of objects *(optional)* | List of extensions attached to the `PAC-ID`. Each object contains:<br>• `name`: extension name, or `null` if the extension has none.<br>• `type`: extension type, or `null` if the extension has none.<br>• `data`: raw extension payload as a string. |
| `as_url` | `string` | The `PAC-ID`, re-serialized as a URL string, including any extensions. |

All string values **MUST** be encoded in UTF-8 and all keys are **case-sensitive**.  
Unknown or unsupported fields **MUST** be ignored by implementations.

> **Design Choice:** Access to T-REX extension content via JSONPath is not supported: an extension's `data` is always the raw, unparsed payload string, regardless of `type`. No further breakdown is provided for any extension type, since not all extension types can be reliably represented as hierarchical JSON structures, and keeping `extensions` uniform across types keeps queries against it predictable.



#### Use in JSONPath Queries

JSONPath expressions in resolver configurations operate on this JSON structure.  
If a queried node exists, the condition evaluates to **True**; if multiple matches occur, the **first** match is used. [Examples](jsonpath_examples.md)


### Substituting Variables in a Resolver Configuration

Variables in `template_url` fields are defined using JSONPath expressions enclosed in `{}`. These expressions are evaluated against the canonical JSON serialization of a `PAC-ID` as defined in this specification. If a query has one or more matches, the value of the **first** match is substituted; if it has no match, it is substituted with the empty string. If the first match is a JSON object or array rather than a scalar (string, number, or boolean), it MUST be serialized as compact JSON text before substitution — this allows a template to carry a whole node of the `PAC-ID`'s JSON serialization (e.g. an entire category) as an embedded, parseable JSON value. The substituted value MUST be percent-encoded so the resulting URL remains valid even if the matched value contains characters that would otherwise not be valid in a URL. Leading and trailing whitespace in the resulting URL SHALL be removed.

It is the responsibility of the creator of the resolver configuration to write queries that address a single value (e.g. by indexing into an array, or filtering down to one element). If a query matches multiple values, only the first is used, so relying on this for anything other than a single, unambiguous match SHOULD be avoided.

> **Design Choice:** This replaces the former {id}-style placeholder system. Template expansion is now fully JSONPath-based, providing a unified mechanism for referencing any part of a PAC-ID.

Example:
```text
https://attributes.mettorius.com/{$.categories[0].segments[?(@.key == '21')].value}
```
For the `PAC-ID` `HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263`, this resolves to: https://attributes.mettorius.com/210263


## Related Documents

- [JSONPath Query Examples](jsonpath_examples.md) — worked examples of JSONPath queries against a `PAC-ID`'s JSON serialization.
- [Macros in YAML](macros_in_yaml_examples.md) — reducing repetition in resolver configurations using YAML anchors and merge keys.
- [Coupling Information Table v1 (deprecated)](resolver-configuration-v1-deprecated.md) — the previous, tab-delimited format. Retained for backward compatibility: a `PAC-ID Resolver` MAY still support it alongside the resolver configuration format described here.

## Terminology Used
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) "Key words for use in RFCs to Indicate Requirement Levels".

## FAQ

See [here](faq.md).

## License

Shield: [![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg
