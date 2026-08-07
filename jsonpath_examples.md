# JSONPath Query Examples

The following examples illustrate how JSONPath expressions operate on the Resolver Context (see [Constructing the Resolver Context](README.md#constructing-the-resolver-context)).

These examples correspond to this `PAC-ID`: `HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211`
and its equivalent JSON representation:

```json
{
  "pac": {
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
          { "key": "250", "value": "1" }
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
    "serialized_as_url_including_extensions": "HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211",
    "serialized_as_url_without_extensions": "HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1"
  }
}
```

<br>

| **Query** | **Description** | **Result** | **Logical Result** |
| :-- | :-- | :-- | :-- |
| `$.pac.issuer` | Returns the PAC-ID issuer | `"METTORIUS.COM"` | True |
| `$.pac.identifier[2]` | Returns the third identifier segment (positional — short notation carries no keys) | `{ "key": null, "value": "210263" }` | True |
| `$.pac.categories[0].key` | Returns the key of the first (main) category | `"-MD"` | True |
| `$.pac.categories[?(@.key == '-MD')]` | Selects the category whose `key` is `-MD` | `{ "key": "-MD", "segments": [...] }` | True |
| `$.pac.categories[0].segments[?(@.key == '240')].value` | Returns the value of the well-known segment `240` (model code) within the `-MD` category | `"BAL500"` | True |
| `$.pac.categories[0].segments[?(@.key == '21')]` | Returns the segment with well-known key `21` (serial number) within the `-MD` category | `{ "key": "21", "value": "210263" }` | True |
| `$.pac.categories[0].segments[?(@.key == '250')].value` | Returns the value of segment `250`, regardless of who added it — access by `key` is uniform whether the segment came from the original issuer or a third party | `"1"` | True |
| `$.pac.extensions[?(@.type == 'TREX')].data` | Returns the raw T-REX payload of the `SUM` extension | `"11$T.D:20231121+FOO$T.A:BAR"` | True |
| `$.pac.identifier[?(@.key == 'FOO')].value` | Tries to match a non-existent identifier key | `[]` | False |
| `$.pac.serialized_as_url_including_extensions` | Returns the `PAC-ID` re-serialized as a URL string, including extensions | `"HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1*SUM$TREX/11$T.D:20231121+FOO$T.A:BAR*CAL$T.D/20231211"` | True |
| `$.pac.serialized_as_url_without_extensions` | Returns the `PAC-ID` re-serialized as a URL string, omitting extensions | `"HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/+ACMELABS.COM/250:1"` | True |


