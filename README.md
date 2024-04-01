# PAC-ID Resolver

## What Is the `PAC-ID Resolver`?
> [!TIP]
> A `PAC-ID Resolver`, short for **P**ublicly **A**ddressable **C**ontent **ID**entifier **Resolver**, is a tool designed to convert a given `PAC-ID` into a URL. This URL can then be utilized to access additional information associated with the corresponding `PAC-ID`.

## Introduction
In laboratory workflows, it is often necessary to utilize multiple software applications and devices. Dealing with chemicals, samples, and instruments requires a seamless exchange of contextual data between these applications and devices, significantly benefiting scientists and lab technicians.

The conventional approach involves interfacing applications and devices with each other. Unfortunately, due to the lack of standardized support, such interfaces usually require bespoke software development or customization projects, demanding substantial effort to create and maintain.

The PAC-ID resolver architecture introduced here offers an alternative and lightweight solution for implementing handover between applications and devices in a loosely coupled manner, providing an exceptional user experience.

By utilizing PAC-IDs to identify chemicals, samples, and instruments, the PAC-ID resolver architecture supports the efficient conversion from a PAC-ID into a browsable link that directs users to the application most relevant to the current workflow context.

![User perspective on PAC-ID resolver](images/pac-id-resolver-user-perspective.png)

*User Perspective: Example of resolving a PAC-ID of a  “material XY” within a LIMS system and forwarding the user to the corresponding structural info in the ELN*

The PAC-ID resolver architecture has been designed with the following design goals:
- **Decentralized operation**: The PAC-ID resolver architecture should be able to operate independently without relying on a central registry.
- **Local customization**: It should allow local overrides of PAC-ID to link conversion mechanisms, enabling customization for local context specific needs.
- **Flexibility and independence**: The PAC-ID resolver architecture should function without strict governance, providing flexibility and independence in its operation.
- **Isolated network compatibility**: It should be capable of operating effectively in isolated networks, even with limited or no connectivity to external resources.

## Architectural Overview and Scope
The process of resolving a PAC-ID into a browsable link follows these steps, as illustrated below:
1. The source application sends a PAC-ID to the PAC-ID resolver
2. The resolver looks up mapping tables
3. The resolver selects entries in the mapping tables that match the PAC-ID and substitutes existing placeholders (see below)
4. The resolver returns an ordered list, sorted by relevance of browsable links, display names and intents to the source application
5. The source application selects the most appropriate link based on the returned of browsable links, display names and intents, considering additional context-dependent or user-provided information

![PAC-ID resolver architecture](images/pac-id-resolver-architectural-overview.png)

*Architectural overview of resolving a PAC-ID into a browsable link.*

## Specification
A PAC-ID resolver is a service or library used by a source application to resolve a PAC-ID into one or more browsable link(s). 

A PAC-ID resolver expects a PAC-ID as input. In order to resolve the PAC-ID, the PAC-ID resolver uses one or more mapping table(s). It selects all entries that match the current PAC-ID and substitutes information in these resolved entries. After completion, it returns a list of zero or more browsable links in conjunction with display names and intents to the source application.

The PAC-ID resolver performs the following steps:
1. Retrieve mapping table(s)
2. Match the PAC-ID to the entries in the mapping table(s) and select matching entries
3. Substitute the placeholders in the matching entries (see below)
4. Return an ordered list of browsable links, display names and intents, derived from all matching entries. The order SHALL correspond to the source of the matching entry (user, corporate or global mapping table, in this order of precedence)

### Retrieving Mapping Table(s)
The following methods SHALL be used to retrieve mapping table(s).

#### User Mapping Table
The user mapping table SHALL be retrieved by reading a file or via a HTTP GET request to an URL. The file or URL SHALL be configurable. It is RECOMMENDED as a best practice to use a local file called “`pac.mapping`", located in the user's home directory.

#### Corporate Mapping Table
The corporate mapping SHALL be retrieved by reading a file or via a HTTP GET request to an URL. The file or URL SHALL be configurable. It is RECOMMENDED as a best practice to use the URL “`pac.local/pac.mapping`".

#### Global Mapping Table
The global mapping table SHALL be retrieved via a HTTP GET request to the URL “`pac.{issuer}/pac.mapping`”. Where “`issuer`“ MUST be replaced by the appropriate value from the PAC-ID.

### Matching PAC-ID to Entries in the Mapping Table
To identify matching entries in a mapping table, compare the PAC-ID’s issuer and category against the **Issuer** and **Category** columns of the mapping table.

### Substituting Placeholders
Mapping table entries MAY contain placeholders in the **Link** column. Replace the placeholders with the corresponding values given by the PAC-ID by performing a text replacement.

### Mapping Table Format

It is RECOMMENDED to encode mapping tables using UTF-8 encoding.

| **Column #** | **Column Name** | **Description** |
| :--- | :--- | :--- |
| 1 | **Service Name** | Describes the entry in human readable, US-English language. SHALL contain only letters `a-zA-Z`, numbers `0-9`, spaces ` ` or hyphens `-`. MUST contain at least one but not more than 255 characters. |
| 2 | **User Intent**  | List of intents that can be fulfilled by this entry.<br>CAN be empty.<br>Intents are usually specified by the calling application and a corresponding matching allows a precise routing to the most adequate service available.<br>Multiple intents MUST be separated by `;`<br>Intents ending with `-generic` SHALL be reserved for future use.<br>Each intent MUST match the regular expression `^[A-Za-z0-9-]{0,64}$`. |
| 3 | **Service Type** | Can be one of the following:<ul><li>`userhandover-generic`: The resolved URL MUST be a navigable URL leading to content made for human consumption (e.g. a HTML page, a PDF file). The resolved URL MUST NOT lead to service endpoint, e.g. a RESTful API.</li><li>`attributes-generic`: The resolved URL MUST lead to an [Attributes Service](https://github.com/ApiniLabs/Attributes-Service) endpoint.</li></ul> |
| 4 | **Applicable If** |  |
| 5 | **Template Url** |  |

#### Rules


#### Placeholders


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
