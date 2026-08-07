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

- **`schema_version`**: identifies the version of the configuration format. MUST be `2.0`, written as a quoted string (e.g. `'2.0'`), to avoid being parsed as a number by the YAML document.
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
- if: $.pac.issuer == METTORIUS.COM AND $.pac.categories[?(@.key == '-MD')]
  entries:
  - service_type: userhandover-generic
    service_name: Datasheet
    application_intents:
      - document-datasheet
    key: https://www.wikidata.org/wiki/Q20819677
    template_url: https://www.mettorius.com/datasheets/{$.pac.categories[0].segments[?(@.key == '240')].value}.pdf

- if: $.pac.issuer == METTORIUS.COM
  entries:
  - service_type: attributes-generic
    service_name: Attributes
    application_intents:
      - ChemProps 
    template_url: https://attributes.mettorius.com/{$.pac.issuer}/{$.pac.categories[0].key}/{$.pac.categories[0].segments[?(@.key == '21')].value}
```

**Field Name** | **Description** |
| :--- | :--- |
**if** | REQUIRED. A `rule` defining which conditions a `PAC-ID` SHOULD fulfil in order to match the entry.  (see [matching](#matching-a-pac-id-to-entries-in-a-resolver-configuration)) 
**entries** | A list of services, each containing the following fields:
**service_name** | Describes the entry in human readable, US-English language. The service name SHALL contain only letters `a-zA-Z`, numbers `0-9`, spaces ` ` or hyphens `-`. It MUST contain at least one but not more than 255 characters. |
**application_intents**  | A list of strings identifying the use case(s) the entry fulfills (see [Application Intent](#application-intent)). Strings ending with `-generic` SHALL be reserved for future use. Each string SHALL contain only letters `a-zA-Z`, numbers `0-9` or hyphens `-`, but no spaces. Each string MUST contain at least one but not more than 64 characters. |
**service_type** | The service type MUST be one of the following strings:<ul><li>`userhandover-generic`: The template URL is a navigable URL leading to content made for human consumption (e.g. a HTML page, a PDF file).</li><li>`attributes-generic`: The template URL MUST lead to an `Attributes Service` endpoint.</li><li>`action-generic`: The template URL, when requested, triggers an action (e.g. a backend operation) rather than returning content for display or an attributes payload.</li></ul> |
**template_url** | A template URL which MAY contain one or more instances of a `variable`. The Template Url is inspired by [RFC 6570 URI Template](https://datatracker.ietf.org/doc/html/rfc6570): A `variable` corresponds to an RFC6570 "*expression*"; the `PAC-ID Resolver` to a RFC 6570 "*template processor*". When replacing the `variable`s with the appropriate values from a `PAC-ID`, the result MUST become a valid URL. |
**key** | OPTIONAL. An absolute IRI identifying what the entry semantically *is*, drawn from a shared vocabulary (see [Key](#key)). |



### Application Intent

`application_intents` lets the creator of a resolver configuration tailor entries to specific use cases. An application resolving a `PAC-ID` looks for the entries whose `application_intents` include the intent(s) it cares about, and prioritizes those matches over entries that don't declare a matching intent. This means the same `PAC-ID` can resolve to different links depending on which application — or which specific action within that application — is doing the resolving: for example, an inventory application might look for the intent `reorder`, while a lab notebook might look for `chem-info`, even though both resolve the same `PAC-ID` against the same resolver configuration.

<span style="color:red">Open question for the team: MAY `application_intents` be empty or absent, i.e. an entry declares no intent at all? If so, how should such an entry be treated when an application filters by a specific intent — always match, as a fallback/default entry, or never match?</span>

### Key

`key` identifies *what the entry actually is*. Its source SHALL follow the same convention as `PAC-ID Attributes` already established (see [`well_known_keys.md`](https://github.com/ApiniLabs/PAC-Attributes/blob/main/well_known_keys.md)): reuse a term from an existing, already-authoritative external vocabulary appropriate to the concept (e.g. [schema.org](https://schema.org), [Wikidata](https://www.wikidata.org), a [GS1 Application Identifier](https://ref.gs1.org/ai/), [QUDT](https://qudt.org/vocab/quantitykind/)) rather than minting a new LabFREED-specific namespace. 

Example — `https://www.wikidata.org/wiki/Q222067` is `PAC-ID Attributes`' own recommended key for "Safety Data Sheet" (see `well_known_keys.md`'s "Documents" section), reused here unchanged as the `key`:

```yaml
- if: True
  entries:
  - service_type: userhandover-generic
    service_name: MSDS
    application_intents:
      - document-msds
    key: https://www.wikidata.org/wiki/Q222067
    template_url: https://www.mettorius.com/msds/{$.pac.categories[0].segments[?(@.key == '240')].value}
```

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

The `if` condition MUST always be present and MUST contain an explicit expression — it MUST NOT be omitted, empty, or `null`. To make a rule block unconditionally applicable (or unconditionally inapplicable), use the boolean literal predicate below explicitly.

A **predicate** is one of the following:

- A **JSONPath expression** that checks for the existence of elements in the Resolver Context.  
  Example:

  ```text
  $.pac.categories[?(@.key == '-MD')]
  ```

- A **JSONPath expression with a comparison** to a literal value (quoted or unquoted), or to another JSONPath expression — the JSONPath expression MAY appear on either side of the comparison. Comparisons use one of `==`, `!=`, `<`, `<=`, `>`, `>=` and are performed on the string representation of both operands, case-insensitively.  
  Example (both equivalent, regardless of case):

  ```text
  $.pac.issuer == METTORIUS.COM
  metTORius.com == $.pac.issuer
  ```

- The literal `True` or `False`, case-insensitive, making the rule block unconditionally applicable or unconditionally inapplicable. It MAY be written as a quoted YAML string (e.g. `if: "True"`) or as YAML's native boolean (e.g. `if: True`); either way it MUST be interpreted as the corresponding boolean, not merely a non-empty, always-truthy string.  
  Example:

  ```text
  if: False
  ```

> **Design Choice:** JSONPath is used because it provides a standardized and expressive way to query nested JSON structures—such as those derived from `PAC-ID` with `PAC-CAT` and extensions. This enables flexible conditions without custom parsing logic.
>
> **Design Choice:** Comparisons are case-insensitive because `PAC-ID` components (e.g. the issuer) are RECOMMENDED, but not required, to be written in upper case — matching should not depend on an author or issuer having followed that recommendation. This applies uniformly to every comparison in a rule, including one written *inside* a JSONPath filter (e.g. `$.pac.categories[?(@.key == '-md')]` MUST match a category whose key is `-MD`) — a rule's matching behavior MUST NOT depend on where in the expression a comparison happens to be written.

Example logical expression, which matches any `PAC-ID` issued by `METTORIUS.COM` that contains the `-MD` category.
```text
$.pac.issuer == METTORIUS.COM AND $.pac.categories[?(@.key == '-MD')]
```

---

### Querying the Resolver Context with JSONPath

Before a rule or template can be evaluated, the `PAC-ID Resolver` MUST construct a **Resolver Context**: a JSON object, in a standardized structure, representing everything a resolver configuration's rules and templates can query against.

#### Constructing the Resolver Context

The Resolver Context MUST be built as follows:

| Field | Type | Description |
| :--- | :--- | :--- |
| `pac` | `object` | Everything derived from the `PAC-ID` being resolved, nested under this single node (see Design Choice below). Contains the fields below. |
| `pac.issuer` | `string` | The domain name of the `PAC-ID` issuer. |
| `pac.identifier` | `array` of objects | Ordered list of identifier segments, exactly as parsed from the `PAC-ID`. Key omission/implication is a `PAC-CAT` category-level concept (see `pac.categories` below) and does NOT apply at this base level. Each object contains:<br>• `key`: string or `null` if no key was given for that segment.<br>• `value`: string representing the segment value. |
| `pac.categories` | `array` of objects | Categories derived from the identifier per `PAC-CAT`, in identifier order; `[]` if the identifier has no category segments. Each category object contains:<br>• `key`: the category key (e.g., `-MD`).<br>• `segments`: array of `{key, value}` belonging to that category. <br>`PAC-CAT` defines, per category, how a segment's key MAY be omitted from the `PAC-ID` and implied by segment order instead; here, any such implied key MUST be stated explicitly — every entry in `segments` carries its actual well-known key (e.g., `21` for a serial number), whether or not that key was written out in the `PAC-ID` itself. `key` is `null` only for a segment that has no well-known key at all, neither explicit nor implied.|
| `pac.extensions` | `array` of objects *(optional)* | List of extensions attached to the `PAC-ID`. Each object contains:<br>• `name`: extension name, or `null` if the extension has none.<br>• `type`: extension type, or `null` if the extension has none.<br>• `data`: raw extension payload as a string. |
| `pac.serialized_as_url_including_extensions` | `string` | The `PAC-ID`, re-serialized as a URL string, including any extensions. |
| `pac.serialized_as_url_without_extensions` | `string` | The `PAC-ID`, re-serialized as a URL string, omitting any extensions. |

All string values **MUST** be encoded in UTF-8 and all keys are **case-sensitive**.  
Unknown or unsupported fields **MUST** be ignored by implementations.

> **Design Choice:** All `PAC-ID`-derived fields are nested under a single `pac` node (`$.pac.issuer`, `$.pac.categories`, ...) rather than living directly at the Resolver Context's top level. This leaves room for future versions of the Resolver Context to add fields unrelated to the `PAC-ID` itself (e.g. resolution-time or caller/application context) alongside `pac`, without those new fields ever risking a name collision with `issuer`, `identifier`, `categories`, `extensions`, `serialized_as_url_including_extensions`, or `serialized_as_url_without_extensions`.
>
> **Design Choice:** Access to T-REX extension content via JSONPath is not supported: an extension's `data` is always the raw, unparsed payload string, regardless of `type`. No further breakdown is provided for any extension type, since not all extension types can be reliably represented as hierarchical JSON structures, and keeping `extensions` uniform across types keeps queries against it predictable.



#### Use in JSONPath Queries

JSONPath expressions in resolver configurations operate on the Resolver Context.  
If a queried node exists, the condition evaluates to **True**; if multiple matches occur, the **first** match is used. [Examples](jsonpath_examples.md)


### Substituting Variables in a Resolver Configuration

Variables in `template_url` fields are defined using JSONPath expressions enclosed in `{}`. These expressions are evaluated against the Resolver Context as defined in this specification. If a query has one or more matches, the value of the **first** match is substituted; if it has no match, it is substituted with the empty string. If the first match is a JSON object or array rather than a scalar (string, number, or boolean), it MUST be serialized as compact JSON text before substitution — this allows a template to carry a whole node of the Resolver Context (e.g. an entire category) as an embedded, parseable JSON value. The substituted value MUST be percent-encoded so the resulting URL remains valid even if the matched value contains characters that would otherwise not be valid in a URL. Leading and trailing whitespace in the resulting URL SHALL be removed.

It is the responsibility of the creator of the resolver configuration to write queries that address a single value (e.g. by indexing into an array, or filtering down to one element). If a query matches multiple values, only the first is used, so relying on this for anything other than a single, unambiguous match SHOULD be avoided.

> **Design Choice:** This replaces the former {id}-style placeholder system. Template expansion is now fully JSONPath-based, providing a unified mechanism for referencing any part of the Resolver Context.

Example:
```text
https://attributes.mettorius.com/{$.pac.categories[0].segments[?(@.key == '21')].value}
```
For the `PAC-ID` `HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263`, this resolves to: https://attributes.mettorius.com/210263


## Related Documents

- [JSONPath Query Examples](jsonpath_examples.md) — worked examples of JSONPath queries against the Resolver Context.
- [Macros in YAML](macros_in_yaml_examples.md) — reducing repetition in resolver configurations using YAML anchors and merge keys.
- [Coupling Information Table v1 (deprecated)](resolver-configuration-v1-deprecated.md) — the previous, tab-delimited format. Retained for backward compatibility: a `PAC-ID Resolver` MAY still support it alongside the resolver configuration format described here.
- [Resolver Configuration v3 Roadmap](resolver-configuration-v3-roadmap.md) *(non-normative, forward-looking)* — candidate semantic augmentations and shortcuts for a future v3, each mapped back to this v2 specification.

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
