# Coupling Information Table v1 (Deprecated)

> [!NOTE]
> This describes the **v1** Coupling Information Table format (tab-delimited, single `if`-condition-per-row). It has been superseded by the JSONPath/YAML-based CIT v2 format described in the [`PAC-ID Resolver` specification](README.md). It is kept here for reference: a `PAC-ID Resolver` MAY still support CIT v1 tables for backward compatibility, alongside CIT v2.

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