# JSONPath Query Examples

The following examples illustrate how JSONPath expressions operate on the Resolver Context (see [Constructing the Resolver Context](README.md#constructing-the-resolver-context)).

These examples correspond to this `PAC-ID`: `HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211`
and its equivalent JSON representation:

```json
{
  "issuer": "METTORIUS.COM",
  "identifier": [
    { "key": null, "value": "-MD" },
    { "key": null, "value": "BAL500" },
    { "key": null, "value": "210263" },
    { "key": null, "value": "+ACMELABS.COM" },
    { "key": "250", "value": "1" }
  ],
  "categories": [
    {
      "key": "-MD",
      "segments": [
        { "key": "240", "value": "BAL500" },
        { "key": "21", "value": "210263" },
        { "key": "250", "value": "1", "derivation_namespace": "ACMELABS.COM" }
      ]
    }
  ],
  "extensions": [
    {
      "name": "SUM",
      "type": "TREX",
      "data": "11$T.D:20231121+FOO$T.A:BAR"
    },
    {
      "name": "CAL",
      "type": "T.D",
      "data": "20231211"
    }
  ],
  "as_url": "HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211"
}
```

<br>

| **Query** | **Description** | **Result** | **Logical Result** |
| :-- | :-- | :-- | :-- |
| `$.issuer` | Returns the PAC-ID issuer | `"METTORIUS.COM"` | True |
| `$.identifier[2]` | Returns the third identifier segment (positional — short notation carries no keys) | `{ "key": null, "value": "210263" }` | True |
| `$.categories[0].key` | Returns the key of the first (main) category | `"-MD"` | True |
| `$.categories[?(@.key == '-MD')]` | Selects the category whose `key` is `-MD` | `{ "key": "-MD", "segments": [...] }` | True |
| `$.categories[0].segments[?(@.key == '240')].value` | Returns the value of the well-known segment `240` (model code) within the `-MD` category | `"BAL500"` | True |
| `$.categories[0].segments[?(@.key == '21')]` | Returns the segment with well-known key `21` (serial number) within the `-MD` category | `{ "key": "21", "value": "210263" }` | True |
| `$.categories[0].segments[?(@.key == '250')].value` | Returns the value of segment `250`, regardless of who added it — access by `key` is uniform whether the segment came from the original issuer or a third party | `"1"` | True |
| `$.categories[0].segments[?(@.derivation_namespace == 'ACMELABS.COM')]` | Returns the segment(s) added by the third party `ACMELABS.COM` via a `+<namespace>` derivation segment | `{ "key": "250", "value": "1", "derivation_namespace": "ACMELABS.COM" }` | True |
| `$.categories[0].segments[?(@.key == '240')].derivation_namespace` | `derivation_namespace` is absent/`null` for a segment added by the `PAC-ID`'s own issuer | `null` | True |
| `$.extensions[?(@.type == 'TREX')].data` | Returns the raw T-REX payload of the `SUM` extension | `"11$T.D:20231121+FOO$T.A:BAR"` | True |
| `$.identifier[?(@.key == 'FOO')].value` | Tries to match a non-existent identifier key | `[]` | False |
| `$.as_url` | Returns the `PAC-ID` re-serialized as a URL string, including extensions | `"HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211"` | True |


