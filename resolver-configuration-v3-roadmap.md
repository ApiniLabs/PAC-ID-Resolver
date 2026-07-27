# Resolver Configuration v3 — Forward-Looking Roadmap

## Status

This is a **non-normative, forward-looking working document**. It is not part of the current, implemented specification — see [README.md](README.md) for the actual `PAC-ID Resolver` v2 specification. Nothing in this file is binding on any current implementation.

## Design Principle: v3 MUST Reduce to v2

Every v3 construct MUST have a well-defined, mechanical mapping to equivalent v2 syntax/structure. This guarantees:

- A v3 resolver configuration can always be mechanically reduced ("compiled down") to an equivalent v2 resolver configuration.
- An issuer publishing a v3 configuration SHOULD also publish the equivalent v2 configuration (e.g. generated automatically from the v3 source), so that resolvers which don't yet support v3 still get correct results.

v3 is therefore additive and semantic-preserving: it does not introduce any behavior that couldn't already be expressed in v2, only more convenient or more explicit ways to express it.

## Serving v3 Separately from v2

v3 SHALL be served at a **separate, distinct endpoint** from v2's Global Resolver Configuration, rather than replacing it:

- v2 (existing): `https://pac.{issuer}/resolver_config.yaml` — see [Sources and Precedence of a Resolver Configuration](README.md#sources-and-precedence-of-a-resolver-configuration).
- v3 (proposed): `https://pac.{issuer}/resolver_config_v3.yaml` *(exact naming convention TBD)*.

A `PAC-ID Resolver` that supports v3 SHOULD request the v3 endpoint first, falling back to the v2 endpoint if the v3 endpoint is unavailable — mirroring how the v2 endpoint already falls back to the legacy v1 endpoint. A resolver that does not support v3 simply never requests the v3 endpoint, and behaves exactly as it does today. This means a v3-capable resolver gets the richer format, an older resolver is unaffected, and no runtime feature-negotiation logic is needed inside either file.

<span style="color:red">Open question for the team: exact v3 endpoint naming convention; and whether "unavailable" for fallback purposes means an HTTP error status specifically, or also includes "fetched successfully but this resolver doesn't understand the schema_version it declares."</span>

## Candidate v3 Semantic Augmentations and Shortcuts

Each candidate below MUST state its mechanical mapping back to v2. Add new candidates as their own subsection, following this pattern.

### 1. Bracket-key shorthand

`[key]` (quoted, either `'` or `"`) as shorthand for `[?(@.key == key)]`, to select an element of an array of `{key, ...}` objects by its `key`.

- **v3 form:** `$.categories["-MD"]`
- **v2 equivalent:** `$.categories[?(@.key == '-MD')]`

Already exists today as an unofficial, Python-only authoring convenience (see [known-implementation-bugs.md](known-implementation-bugs.md), item 2) — it is not part of the current v2 spec and is not guaranteed to be expanded correctly by the reference implementation. Promoting it to an official, well-defined v3 construct would resolve that ambiguity.

### 2. `primary_category` / `issuing_system` convenience fields

Per [PAC-CAT](https://github.com/ApiniLabs/PAC-CAT), a `PAC-ID`'s categories are now restricted to at most two: the **primary category** (first, describing the item itself) and, optionally, the **issuing system** (second). v3 exposes these directly as named top-level fields in the JSON serialization, instead of requiring positional access into the `categories` array.

- **v3 form:** `$.primary_category` / `$.issuing_system`
- **v2 equivalent:** `$.categories[0]` / `$.categories[1]`

`$.issuing_system` is `null` if no second category is present.

### 3. Resolver Context: beyond just the `PAC-ID`

What a resolver configuration's rules and templates are evaluated against should no longer be described as just "the `PAC-ID` serialized to JSON" — it's better understood as a broader **Resolver Context**, of which the `PAC-ID`'s own data is the primary (and, in v2, the *only*) content. This renaming applies regardless of how the context ends up being structured (see open question below) — it's a conceptual/naming shift, not itself a schema change.

Candidate v3 extension: enrich the Resolver Context with client/request information the resolver has available at resolve time but which isn't part of the `PAC-ID` itself — e.g. `client.language`, `client.location`. This lets a rule or `template_url` account for who's asking, not just what's being asked about (e.g. a localized product page).

**This candidate does not fit the "reduces to v2" principle above.** Unlike candidates 1 and 2 (pure syntactic sugar over data v2 already has), client/request context is new input data a v2 resolver has no way to access at all — there is no v2-equivalent rule for "if client language is French." A resolver configuration block that depends on `$.client` needs a sensible lower-precedence fallback block that matches on the `PAC-ID` alone, for situations where that context isn't available. It can't be mechanically rewritten into an equivalent v2 rule the way candidates 1 and 2 can.

<span style="color:red">Open question for the team: should the Resolver Context be a nested wrapper — `{ pac_id: { issuer, identifier, categories, extensions, as_url }, client: {...} }`, cleanly separating "`PAC-ID` data" from "everything else," but requiring every existing v2 query to gain a `pac_id.` prefix — or kept flat, with today's fields staying exactly where they are at the root and `client` added as a new sibling (fully backward compatible, but `client` then sits at the same level as `issuer` despite being a conceptually different kind of thing)? Not yet decided.</span>

### 4. Derivation namespace segments (`+<namespace>`)

`PAC-ID` supports appending a **derivation namespace segment** (`+<namespace>`, e.g. `+ACMELABS.COM`) so a third party can extend someone else's `PAC-ID` with its own segments under its own namespace, without claiming authority over the original issuer's segment vocabulary — e.g. `HTTPS://PAC.OMNIZYME.COM/-MS/240:AMYLASE/10:AB9876/21:9876/+ACMELABS.COM/250:1`.

Neither the `PAC-ID`/`PAC-CAT` specs nor the reference implementation currently say anything about how these segments relate to `PAC-CAT` categories — today, the reference implementation doesn't recognize `+` as special at all, so a derivation segment and everything after it silently gets folded into whichever category preceded it (an accident of the parser, not a decision).

Decided (per Reto):

- Segments following a `+<namespace>` marker belong to the **primary category**, never to the issuing system (second) category — regardless of where in the identifier the `+<namespace>` marker appears.
- They still live in that category's ordinary `segments` array, not a separate `derivations` field — so a resolver configuration accessing a well-known segment (e.g. a container number) works identically regardless of whether the original issuer or a third party added it. Uniform, provenance-agnostic access is the default case and stays simple.
- Provenance is still recoverable for those who need it: each entry in `segments` gains an optional field, **`segment_issuer`**, holding the domain of the party that added it. E.g. `{ "key": "250", "value": "1", "segment_issuer": "ACMELABS.COM" }`. This lets `$.primary_category.segments[?(@.key == '250')].value` work the same regardless of origin, while `$.primary_category.segments[?(@.segment_issuer == 'ACMELABS.COM')]` remains available when provenance actually matters. The name deliberately echoes the top-level `issuer` field, just at segment granularity.
- **Sentinel for the original issuer:** rather than repeating the issuer's own domain string on every ordinary segment, a segment from the original issuer simply omits the field (`null`/absent) — consistent with how the rest of this spec already represents "not applicable" (`key: null` for an unkeyed segment, `issuing_system: null` when there's no second category, etc.). A presence check like `$.primary_category.segments[?(@.segment_issuer)]` then cleanly means "segments added by a third party."
- The `+<namespace>` marker segment itself is structural (like a category's own `-XX` marker) and doesn't need its own entry in `segments` — it just sets which `segment_issuer` subsequent segments are tagged with, until the next category or `+` marker.

Reto is updating the `PAC-ID`/`PAC-CAT` specs in parallel to reflect this; this section should be reconciled with that once published, to stay consistent rather than diverge. A concrete proposed addition to the `PAC-CAT` spec text follows below.

<span style="color:red">Open question for the team: whether this is an existing gap needing a fix in the current v2 spec (if `+`-derived `PAC-ID`s are already in use) or genuinely forward-looking v3 material; and what happens if a `+`-derivation appears with no primary category at all.</span>

#### Proposed Addition to the `PAC-CAT` Specification

`PAC-CAT`'s ["Concatenate a second category to identify the issuing system"](https://github.com/ApiniLabs/PAC-CAT#concatenate-a-second-category-to-identify-the-issuing-system) section is the natural place for this — it's the only existing place that discusses what happens beyond the primary category's own segments. Proposed new subsection, immediately following it, in the same style:

> #### Segments added via a derivation namespace (`+`)
>
> A `PAC-ID` MAY be extended by a third party using a **derivation namespace segment** (`+<namespace>`), as defined in the [`PAC-ID` specification](https://github.com/ApiniLabs/PAC-ID). Where `PAC-CAT` is used, any `category segment`s that follow a `+<namespace>` marker MUST be treated as further `category segment`s of the **primary category** — they MUST NOT start a new category, and MUST NOT be attributed to the issuing system.
>
> Example:
>
> ```
> HTTPS://PAC.OMNIZYME.COM/-MS/240:AMYLASE/10:AB9876/21:9876/+ACMELABS.COM/250:1
>                         | primary category                | added by ACMELABS.COM
> ```
>
> Here, `250:1` (Aliquot) is a `category segment` of the `-MS` primary category, even though it was added by `ACMELABS.COM` rather than the original issuer, `OMNIZYME.COM`.
>
> \> [!NOTE]
> \> Which party added a given `category segment` MAY be relevant to systems consuming a `PAC-ID` (e.g. a [`PAC-ID Resolver`](https://github.com/ApiniLabs/pac-id-resolver)), even though it does not change which category the segment belongs to. A `category segment` that appears before any `+<namespace>` marker was added by the `PAC-ID`'s own `issuer`; a `category segment` that appears after a `+<namespace>` marker was added by that namespace.

This is written to be lifted directly into `PAC-CAT`'s `README.md` once confirmed — no separate drafting needed elsewhere.
