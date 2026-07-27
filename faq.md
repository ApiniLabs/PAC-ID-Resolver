# [`PAC-ID Resolver`](README.md) FAQ

**Q: Why do I need a `PAC-ID Resolver` — why not just put a URL in the `PAC-ID`?**

**A:** A `PAC-ID` is meant to identify an object for its entire lifetime, across every system that ever needs to work with it. A URL points at one specific system's view of that object today. Baking a URL into the identifier would mean re-issuing every `PAC-ID` whenever a system is replaced, moved, or added. The resolver decouples the two: the `PAC-ID` stays stable, and the mapping to services lives in a resolver configuration that can be updated independently.

**Q: What's the difference between the previous (v1) format and this one?**

**A:** The previous format ([Coupling Information Table v1, deprecated](resolver-configuration-v1-deprecated.md)) is a flat, tab-delimited table with one fixed matching syntax per row. This resolver configuration format replaces it with a YAML document whose rules are JSONPath-based logical expressions, which can express richer conditions (categories, nested identifier segments, extensions) than v1's fixed placeholders could. A resolver MAY still support v1 alongside this format for backward compatibility.

**Q: Why JSONPath instead of a custom matching syntax?**

**A:** JSONPath is a standardized, expressive way to query nested structures. Reusing an existing query language avoids inventing and maintaining a bespoke one.

**Q: What happens if more than one coupling information entry matches a `PAC-ID`?**

**A:** The resolver returns all matching entries, grouped by the resolver configuration they came from, in source-precedence order (see [Sources and Precedence of a Resolver Configuration](README.md#sources-and-precedence-of-a-resolver-configuration)). The application then picks the entry (or entries) it actually wants using [`application_intents`](README.md#application-intent) and the indicated `service_type`.

**Q: Do I have to host my own resolver configuration?**

**A:** As a lab *consuming* `PAC-ID`s (resolving them in your own applications), no — a `PAC-ID Resolver` retrieves the issuer's Global Resolver Configuration automatically by default, so you can resolve any `PAC-ID` without hosting anything yourself.

As an *issuer* of `PAC-ID`s, however, you are strongly recommended to host your own resolver configuration at `https://pac.{your-issuer-domain}/resolver_config.yaml`, so that other applications can resolve the `PAC-ID`s you create. Configuring additional (personal or organizational) resolver configurations lets a consumer add or override entries with higher precedence on top of what an issuer publishes.

**Q: Can resolver rules use `PAC-CAT` categories, or only raw identifier segments?**

**A:** Both. Rules are evaluated against the full Resolver Context, which includes `PAC-CAT`-derived `categories` (and their `segments`) alongside the raw `identifier` array — see [Querying the Resolver Context with JSONPath](README.md#querying-the-resolver-context-with-jsonpath).

**Q: When should I use `attributes-generic` versus a `userhandover-generic`/`action-generic` entry, and how complex should `template_url` templating get?**

**A:** As a general pattern:

- Use `attributes-generic` for data that changes dynamically over the object's lifetime (e.g. current status, latest measurement, stock level) — the Attributes Service is queried live rather than baked into a resolver configuration entry. An attributes response MAY itself include URLs for handover to another system, letting an application chain from "get current attributes" to "open the relevant page" without a separate resolve step.
- Use `userhandover-generic`/`action-generic` entries for the stable, structural mapping from a `PAC-ID` to a service — i.e. where this kind of object lives, not what its current state is.

Templating in `template_url` SHOULD NOT be overused. The sweet spot: one, at most two, nodes substituted verbatim — e.g. `{$.identifier[1].value}` for a single segment or `{$.as_url}` for the whole `PAC-ID`. This only works if the target's own URLs use that exact string, e.g. `https://mettorius.com/instruments/BAL500`.

Existing IT systems rarely have a standardized vocabulary for what a `PAC-ID` cleanly separates into segments (e.g. a "model" field mixing the model code with a free-text display name) — that's a limitation of those systems, not of `PAC-ID`. When this means the target's own URL doesn't match the segment value verbatim (e.g. the shop page is `https://shop.mettorius.com/Analytical-Balance-BAL-500`, not `.../BAL500`), it is RECOMMENDED to add a lightweight redirect endpoint (`https://pac.mettorius.com/redirect/shop/BAL500`) that performs the mapping, rather than encoding it in the template.


