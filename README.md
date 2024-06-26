# PAC-ID Resolver

## `PAC-ID Resolver` in a Nutshell

A `PAC-ID Resolver`, short for **P**ublicly **A**ddressable **C**ontent **ID**entifier **Resolver**, is an architectural pattern for loosely coupling applications, creating modern user experience without classical integration. The basis for the coupling is a `PAC-ID`.

Working principle: The application asks the `PAC-ID Resolver` to convert a given `PAC-ID` into an ordered list of coupling information, based on configurable coupling information tables provided to the `PAC-ID Resolver`. The application in turn selects the best coupling information from the list, based on an application intent. The coupling information, depending on the indicated service type, contains a URL for user handovers or an endpoint for standardized inter-application communication.

## Introduction

In laboratory workflows, the utilization of multiple software applications is commonplace. A seamless exchange of contextual data between these applications is essential to provide users with a state-of-the-art experience.

The conventional approach involves interfacing applications with each other. Unfortunately, due to the lack of standards, such interfaces typically require bespoke software development or customization projects, demanding substantial effort to create and maintain.

In contrast, the PAC-ID Resolver architecture offers an alternative and lightweight solution for connecting applications in a loosely coupled manner. Implemented as a neutral contract between systems, it eliminates inter-dependencies and therefore facilitates adding additional or replacing existing systems. Its workflow context awareness allows seamless interactions that result in an exceptional user experience.

## Interaction of an Application with a `PAC-ID Resolver`

1. The application submits a `PAC-ID` to the `PAC-ID Resolver`.
2. The `PAC-ID Resolver` looks up `coupling information tables`.
3. The `PAC-ID Resolver` selects coupling information entries in the coupling information table that match the `PAC-ID` and substitutes variables.
4. The `PAC-ID Resolver` returns an ordered list of coupling information entries, together with the source of the coupling information table. The order is defined by the precedence of the sources of the coupling information tables and the sequence of the entries in these tables (first entries have higher precedence).
5. The application selects the most appropriate coupling information based on the application intent.

## Coupling Information Table

### Format of a Coupling Information Table

The coupling information table SHALL be a text string, containing zero or more coupling information entries (rows), separated by “`newline`" and formatted with “`tab`" delimited columns as follows. When serializing into a file or into a binary stream or array, the contents SHALL be encoded using UTF-8 encoding.

Lines starting with the “`#`" character SHALL be ignored and treated as comments until the next “`newline`".

It is RECOMMENDED that the first line contains a version preamble in the format `# coupling information table version: 1.0`, indicating the version of the coupling information table. Currently the version is fixed to `1.0`.

The first non-comment row MUST contain a header, composed of the column names from the table below, delimited by "`tab`" and ended by “`newline`":

| **Column #** | **Column Name** | **Description** |
| :--- | :--- | :--- |
| 1 | **Service Name** | Describes the entry in human readable, US-English language. The service name SHALL contain only letters `a-zA-Z`, numbers `0-9`, spaces ` ` or hyphens `-`. It MUST contain at least one but not more than 255 characters. |
| 2 | **Application Intent**  | A list of strings separated by `;`. Strings ending with `-generic` SHALL be reserved for future use. Each string SHALL contain only letters `a-zA-Z`, numbers `0-9` or hyphens `-`, but no spaces. Each string MUST contain at least one but not more than 64 characters. |
| 3 | **Service Type** | The service type MUST be one of the following strings:<ul><li>`userhandover-generic`: The template URL is a navigable URL leading to content made for human consumption (e.g. a HTML page, a PDF file).</li><li>`attributes-generic`: The template URL MUST lead to an `Attributes Service` endpoint.</li></ul> |
| 4 | **Applicable If** | A list of `rule`s a `PAC-ID` SHOULD fulfil in order to match the entry. The list MAY be empty.<br>Multiple `rule`s MUST be separated by `;`. |
| 5 | **Template Url** | A template URL which MAY contain one or more instances of a `variable`. The Template Url is inspired by [RFC 6570 URI Template](https://datatracker.ietf.org/doc/html/rfc6570): A `variable` corresponds to an RFC6570 "*expression*"; the `PAC-ID Resolver` to a RFC 6570 "*template processor*". When replacing the `variable`s with the appropriate values from a `PAC-ID`, the result MUST become a valid URL. |

Example of a coupling information table:
```
# coupling information table version: 1.0
# This is a comment
Service Name        ⇨ Application Intent ⇨ Service Type         ⇨ Applicable if                       ⇨ Template Url
Product Page        ⇨ ProdInfo           ⇨ userhandover-generic ⇨ {isu}=METTORIUS.COM;{idSeg1}=DEVICE ⇨ https://www.mettorius.com/inventory/{idSeg1}/{idVal21}
Attributes          ⇨ ChemProps          ⇨ attributes-generic   ⇨ {isu}=METTORIUS.COM                 ⇨ https://attributes.mettorius.com/{id}
# ... more entries ...
```
`⇨` denotes a tab character.

### Sources and Precedence of a Coupling Information Table

A `PAC-ID Resolver` SHALL be able to retrieve coupling information tables from multiple sources:

- loading from a file system
- retrieving from a http(s) URL
- retrieving the **Global Coupling Information Table** from “`https://pac.{issuer}/coupling-information-table`”. Where “`issuer`“ MUST be replaced by the appropriate value from the `PAC-ID`.

The sources and their precedence SHALL be configurable.

### Matching a `PAC-ID` to Entries in a Coupling Information Table

The `PAC-ID Resolver` SHALL match coupling information entries by evaluating the `rule`s in the **Applicable If** column of all found coupling information tables.

In the context of this specification, a `rule` consists of a `variable` followed by an `=` sign and a `value`. Alternatively, if a `variable` merely needs to exist in the `PAC-ID`, without specifying an exact value, the `variable` can stand alone without an assigned value.

All `rule`s MUST match, if multiple `rule`s are specified (AND logic - for OR logic simply create additional rows). Matching SHALL be case-insensitive.

Example of a `rule` matching `PAC-ID`s with `issuer` "METTORIUS.COM" and having and for which the `id segment key` "21" exists:
```
{isu}=METTROIUS.COM;{idVal21}
```

### Substituting Variables in a Coupling Information Table

For **Template Url** and **Applicable If** columns, the `variable`s outlined below MAY be used.

The following `PAC-ID` (with two [T-REX](https://github.com/ApiniLabs/T-REX) extensions) is used as example for the **Example** column below:
```
HTTPS://PAC.METTORIUS.COM/DEVICE/21:210263*11$T.D:20231121+FOO$T.A:BAR*CAL$T.D:20231211
```

| **Variable** | **Description** | **Example** |
| :--- | :--- | :--- |
| {isu} | The `issuer` of the `PAC-ID` | {isu} → METTORIUS.COM |
| {pac} | The complete `PAC-ID` in URL representation (without extensions) | {pac} → HTTPS://PAC.METTORIUS.COM/DEVICE/21:210263 |
| {id} | The `identifier` of the `PAC-ID` | {id} → DEVICE/21:210263 |
| {idSegN} | The Nth `id segment` of the `PAC-ID` | {idSeg2} → 21:210263 |
| {idValFOO} | The `id segment value` of the first `id segment` with `id segment key` "FOO".[^1]  | {idVal21} → 210263 |
| {ext} | All extensions separated by `*`. | {ext} → 11$T.D:20231121+FOO$T.A:BAR*CAL$T.D:20231211 |
| {extN} | The Nth extension. | {ext1} → 11$T.D:20231121+FOO$T.A:BAR |
| {extNSegM} | The Mth segment of the Nth extension. A `+` character separates an extension into segments. | {ext1Seg1} → 11$T.D:20231121 |
| {extNValFOO} | The value of the extension segment with key "FOO"[^1]  of the Nth extension. Key and value inside a segment are separated by the `:` character. | {ext1Val11$T.D} → 20231121 |

[^1]: Any occurrence of curly brackets (`{` and `}`) inside "FOO" need to be escaped with a backslash `\` (resulting in `\{` and `\}`).

Given the following `PAC-ID`:
```
HTTPS://PAC.METTORIUS.COM/DEVICE/21:210263
```
the first row of the coupling information table example above would resolve to the following URL:
```
https://www.mettorius.com/inventory/DEVICE/210263
```
the second would resolve to this URL:
```
https://attributes.mettorius.com/DEVICE/21:210263
```

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
