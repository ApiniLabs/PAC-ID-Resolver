# PAC-ID Resolver

## What Is a `PAC-ID Resolver`?
> [!TIP]
> A `PAC-ID Resolver`, short for **P**ublicly **A**ddressable **C**ontent **ID**entifier **Resolver**, is a tool designed to convert a given `PAC-ID` into a URL. This URL can then be utilized to access additional information associated with the corresponding `PAC-ID`.

## Introduction
In laboratory workflows, it is often necessary to utilize multiple software applications and devices. Dealing with chemicals, samples, and instruments requires a seamless exchange of contextual data between these applications and devices, significantly benefiting scientists and lab technicians.

The conventional approach involves interfacing applications and devices with each other. Unfortunately, due to the lack of standardized support, such interfaces usually require bespoke software development or customization projects, demanding substantial effort to create and maintain.

The `PAC-ID Resolver` architecture introduced below offers an alternative and lightweight solution for implementing handover between applications and devices in a loosely coupled manner, providing an exceptional user experience.

By utilizing `PAC-ID`s to identify chemicals, samples, and instruments, the `PAC-ID Resolver` architecture supports the efficient conversion from a `PAC-ID` into a URL, depending on the current workflow context. The URL can then be used to direct users to the application most relevant or to retrieve additional information.

![User perspective on PAC-ID resolver](images/pac-id-resolver-user-perspective.png)

*User Perspective: Example of resolving a `PAC-ID` of a  “material XY” within a LIMS system and forwarding the user to the corresponding structural info in the ELN.*

The `PAC-ID Resolver` architecture has been designed with the following design goals:
- **Decentralized operation**: A `PAC-ID Resolver`  should be able to operate independently without relying on a central registry.
- **Local customization**: It should allow local overrides of `PAC-ID` to URL conversion mechanisms, enabling customization for local context specific needs.
- **Flexibility and independence**: A `PAC-ID Resolver`  should function without strict governance, providing flexibility and independence in its operation.
- **Isolated network compatibility**: It should be capable of operating effectively in isolated networks, even with limited or no connectivity to external resources.

## `PAC-ID Resolver` Architectural Overview and Scope

A `PAC-ID Resolver` is a program, service or library that transforms a `PAC-ID` into a URL, based on a mapping table and optionally additional information, like a **User Intent**.

The process of resolving a `PAC-ID` into a URL follows these steps, as illustrated below:

1. The end user software application submits a `PAC-ID` to a `PAC-ID Resolver`
2. The `PAC-ID Resolver` looks up mapping tables
3. The `PAC-ID Resolver` selects entries in the mapping tables that match the `PAC-ID` and substitutes variables
4. The `PAC-ID Resolver` returns a list of **Service Name**s, **User Intent**s, **Service Type**s and resolved **URL**s, 

![PAC-ID resolver architecture](images/pac-id-resolver-architectural-overview.png)

*Architectural overview of resolving a PAC-ID into a browsable link.*

## Specification
A `PAC-ID Resolver` is a program, service or library used by a end user software application to resolve a PAC-ID into a (list of) URL(s). 

A `PAC-ID Resolver` expects a `PAC-ID` as input. In order to resolve the `PAC-ID`, the `PAC-ID Resolver` uses one or more mapping table(s). It selects all entries that match the current `PAC-ID` and substitutes information in these resolved entries. After completion, it returns a list of zero or more **Service Name**s, **User Intent**s, **Service Type**s and resolved **URL**s.

The `PAC-ID Resolver` performs the following steps:
1. Retrieve mapping table(s)
2. Match the `PAC-ID` to the entries in the mapping table(s) and select matching entries
3. Substitute the variables in the matching entries
4. Return an ordered list of **Service Name**s, **User Intent**s, **Service Type**s and resolved **URL**s

### 1. Retrieving Mapping Table(s)
The following methods SHALL be used to retrieve mapping table(s).

#### 1.1 User Mapping Table
The user mapping table SHALL be retrieved by reading a file or via a HTTP GET request to an URL. The file or URL SHALL be configurable. It is RECOMMENDED as a best practice to use a local file called “`pac.mapping`", located in the user's home directory.

#### 1.2 Corporate Mapping Table
The corporate mapping SHALL be retrieved by reading a file or via a HTTP GET request to an URL. The file or URL SHALL be configurable. It is RECOMMENDED as a best practice to use the URL “`pac.local/pac.mapping`".

#### 1.3 Global Mapping Table
The global mapping table SHALL be retrieved via a HTTP GET request to the URL “`pac.{issuer}/pac.mapping`”. Where “`issuer`“ MUST be replaced by the appropriate value from the PAC-ID.

### 2. Matching PAC-ID to Entries in the Mapping Table
To identify matching entries by evaluating the **Applicable If** column of all found mapping tables.

### 3. Substituting Variables
Mapping table entries MAY contain variables in the **Template Url** column. Replace the varaibles with the corresponding values given by the PAC-ID by performing a text replacement.

### 4. Returning
Return an ordered list of **Service Name**s, **User Intent**s, **Service Type**s and resolved **URL**s, derived from all matching entries of all found mapping tables. The order SHALL correspond to the source of the matching entry (user, corporate or global mapping table, in this order of precedence) and the order within each mapping table.

### Mapping Table Format

The mapping table SHALL be a text string, containing zero or more mapping entries (rows), separated by “`newline`" and formatted with “`tab`" delimited columns as follows. When serializing into a file or into a binary stream or array, the contents SHALL be encoded using UTF-8 encoding. 

Lines starting with the “`#`" character SHALL be ignored and treated as comments until the next “`newline`". 

It is RECOMMENDED that the first line contains a version preamble in the format `# mapping table version: 1.0`, indicating the version of the mapping table. Currently the version is fixed to `1.0`.

The first non-comment row MUST contain a header, composed of the column names from the table below, delimited by "`tab`" and ended by “`newline`":

| **Column #** | **Column Name** | **Description** |
| :--- | :--- | :--- |
| 1 | **Service Name** | Describes the entry in human readable, US-English language. SHALL contain only letters `a-zA-Z`, numbers `0-9`, spaces ` ` or hyphens `-`. MUST contain at least one but not more than 255 characters. |
| 2 | **User Intent**  | List of intents that can be fulfilled by this entry.<br>CAN be empty.<br>Intents are usually specified by the calling application and a corresponding matching allows a precise routing to the most adequate service available.<br>Multiple intents MUST be separated by `;`<br>Intents ending with `-generic` SHALL be reserved for future use.<br>Each intent MUST match the regular expression `^[A-Za-z0-9-]{0,64}$`. |
| 3 | **Service Type** | MUST be one of the following:<ul><li>`userhandover-generic`: The resolved URL MUST be a navigable URL leading to content made for human consumption (e.g. a HTML page, a PDF file). The resolved URL MUST NOT lead to service endpoint, e.g. a RESTful API.</li><li>`attributes-generic`: The resolved URL MUST lead to an [Attributes Service](https://github.com/ApiniLabs/Attributes-Service) endpoint.</li></ul> |
| 4 | **Applicable If** | A list of `rule`s a `PAC-ID` SHOULD fulfil in order to be relevant for the service.<br>CAN be empty.<br>Multiple `rule`s MUST be separated by `;`. All `rule`s MUST match if multiple `rule`s are specified (AND logic - for OR logic simply create additional rows.)<br>Matching SHALL be case-insensitive. |
| 5 | **Template Url** | A URL that points to the service outlined in this entry.<br>MAY contain one or more instances of a `variable`. The **Template Url** is inspired by [RFC 6570 URI Template](https://datatracker.ietf.org/doc/html/rfc6570): A `variable` corresponds to an RFC6570 "*expression*"; the `PAC-ID Resolver` to a RFC 6570 "*template processor*".<br>When replacing the `variable`s with the appropriate values from a `PAC-ID`, the result MUST become a valid URL. |

> [!NOTE]
> Example of a mapping table:
> ```
> # mapping table version: 1.0
> # This is a comment
> Service Name        ⇨ User Intent ⇨ Service Type         ⇨ Applicable if                       ⇨ Template Url
> Product Information ⇨ ProdInfo    ⇨ userhandover-generic ⇨ {isu}=METTORIUS.COM;{idSeg1}=DEVICE ⇨ https://www.mettorius.com/inventory/{idSeg1}/{idVal21}
> Attributes          ⇨ Attributes  ⇨ attributes-generic   ⇨ {isu}=METTORIUS.COM                 ⇨ https://attributes.mettorius.com/{id}
> # ... more entries ...`
> ```
> `⇨` denotes a tab character.
> 
> Given the following `PAC-ID`:
> ```
> HTTPS://PAC.METTORIUS.COM/DEVICE/21:210263
> ```
> the first row of the mapping table would resolve to the following URL:
> ```
> https://www.mettorius.com/inventory/DEVICE/210263
> ```
> the second would resolve to this URL:
> ```
> https://attributes.mettorius.com/DEVICE/21:210263
> ```

#### Variables

For **Template Url** and **Applicable If** colums, the `variable`s outlined below MAY be used.

> [!NOTE]
> The following `PAC-ID` (with two [T-REX](https://github.com/ApiniLabs/T-REX) extensions) is used as example for the **Example** column below:
> ```
> HTTPS://PAC.METTORIUS.COM/DEVICE/21:210263*11$T.D:20231121+FOO$T.A:BAR*CAL$T.D:20231211
> ```

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

#### Rules

In the context of this specification, a `rule` consists of a `variable` followed by an `=` sign and a `value`, serving for comparison against a specific value the `variable` refers to inside the `PAC-ID`. Alternatively, if a value for the `variable` merely needs to exist without specifying an exact value, the `variable` can stand alone without an assigned value.

> [!NOTE]
> Example of a `rule` matching `PAC-ID`s with `issuer` "METTORIUS.COM" and having any non-empty `id segment value` available for the `id segment key` "21":
> ```
> {isu}=METTROIUS.COM;{idVal21}
> ```

## Terminology Used
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) "Key words for use in RFCs to Indicate Requirement Levels".

## FAQ

## License
Shield: [![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg
