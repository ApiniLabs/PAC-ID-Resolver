# JSONPath Query Examples

The following examples illustrate how JSONPath expressions operate on the canonical JSON serialization of a `PAC-ID`.

| **Query** | **Description** | **Result** | **Logical Result** |
| :-- | :-- | :-- | :-- |
| `$.issuer` | Returns the PAC-ID issuer | `"METTORIUS.COM"` | True |
| `$.identifier[2]` | Returns the third identifier segment (positional — short notation carries no keys) | `{ "key": null, "value": "210263" }` | True |
| `$.categories[0].key` | Returns the key of the first (main) category | `"-MD"` | True |
| `$.categories[?(@.key == '-MD')]` | Selects the category whose `key` is `-MD` | `{ "key": "-MD", "segments": [...] }` | True |
| `$.categories[0].segments[?(@.key == '240')].value` | Returns the value of the well-known segment `240` (model code) within the `-MD` category | `"BAL500"` | True |
| `$.categories[0].segments[?(@.key == '21')]` | Returns the segment with well-known key `21` (serial number) within the `-MD` category | `{ "key": "21", "value": "210263" }` | True |
| `$.extensions[?(@.type == 'TREX')].data` | Returns the raw T-REX payload of the `SUM` extension | `"11$T.D:20231121+FOO$T.A:BAR"` | True |
| `$.identifier[?(@.key == 'FOO')].value` | Tries to match a non-existent identifier key | `[]` | False |
| `$.as_url` | Returns the `PAC-ID` re-serialized as a URL string, including extensions | `"HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211"` | True |

---

These examples correspond to this `PAC-ID`: `HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211`
and its equivalent JSON representation:

```json
{
  "issuer": "METTORIUS.COM",
  "identifier": [
    { "key": null, "value": "-MD" },
    { "key": null, "value": "BAL500" },
    { "key": null, "value": "210263" }
  ],
  "categories": [
    {
      "key": "-MD",
      "segments": [
        { "key": "240", "value": "BAL500" },
        { "key": "21", "value": "210263" }
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
  "as_url": "HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211"
}
```
