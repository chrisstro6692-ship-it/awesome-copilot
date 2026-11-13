# Schema Definition and Validation Rules Reference

#### Schema Definition

The JSON structure below is **IMMUTABLE** during Phase 3. Phase 4 will later add a `TunedEntry` object to each object in `Entries` — this is the only permitted schema extension and happens in a separate phase.

JSON keys map directly to template placeholders like `{{.CountryCode}}`, `{{.HasError}}`, etc.

```json
{
  "InputFile": "",
  "Timestamp": 0,

  "TotalEntries": 0,
  "IpV4Entries": 0,
  "IpV6Entries": 0,
  "InvalidEntries": 0,

  "Errors": 0,
  "Warnings": 0,
  "OK": 0,
  "Suggestions": 0,

  "CityLevelAccuracy": 0,
  "RegionLevelAccuracy": 0,
  "CountryLevelAccuracy": 0,
  "DoNotGeolocate": 0,

  "Entries": [
    {
      "Line": 0,
      "IPPrefix": "",
      "CountryCode": "",
      "RegionCode": "",
      "City": "",

      "Status": "",
      "IPVersion": "",

      "Messages": [
        {
          "ID": "",
          "Type": "",
          "Text": "",
          "Checked": false
        }
      ],

      "HasError": false,
      "HasWarning": false,
      "HasSuggestion": false,
      "DoNotGeolocate": false,
      "GeocodingHint": "",
      "Tunable": false
    }
  ]
}
```

Field definitions:

**Top-level metadata:**
- `InputFile`: The original input source, either a local filename or a remote URL.
- `Timestamp`: Milliseconds since Unix epoch when the tuning was performed.
- `TotalEntries`: Total number of data rows processed (excluding comment and blank lines).
- `IpV4Entries`: Count of entries that are IPv4 subnets.
- `IpV6Entries`: Count of entries that are IPv6 subnets.
- `InvalidEntries`: Count of entries that failed IP prefix parsing and CSV parsing.
- `Errors`: Total entries whose `Status` is `ERROR`.
- `Warnings`: Total entries whose `Status` is `WARNING`.
- `OK`: Total entries whose `Status` is `OK`.
- `Suggestions`: Total entries whose `Status` is `SUGGESTION`.
- `CityLevelAccuracy`: Count of valid entries where `City` is non-empty.
- `RegionLevelAccuracy`: Count of valid entries where `RegionCode` is non-empty and `City` is empty.
- `CountryLevelAccuracy`: Count of valid entries where `CountryCode` is non-empty, `RegionCode` is empty, and `City` is empty.
- `DoNotGeolocate` (metadata): Count of valid entries where `CountryCode`, `RegionCode`, and `City` are all empty.

**Entry fields:**
- `Entries`: Array of objects, one per data row, with the following per-entry fields:
  - `Line`: 1-based line number in the original CSV (counting all lines including comments and blanks).
  - `IPPrefix`: The normalized IP prefix in CIDR slash notation.
  - `CountryCode`: The ISO 3166-1 alpha-2 country code, or empty string.
  - `RegionCode`: The ISO 3166-2 region code (e.g., `US-CA`), or empty string.
  - `City`: The city name, or empty string.
  - `Status`: Highest severity assigned: `ERROR` > `WARNING` > `SUGGESTION` > `OK`.
  - `IPVersion`: `"IPv4"` or `"IPv6"` based on the parsed IP prefix.
  - `Messages`: Array of message objects, each with:
    - `ID`: String identifier from the **Validation Rules Reference** table below (e.g., `"1101"`, `"3301"`).
    - `Type`: The severity type: `"ERROR"`, `"WARNING"`, or `"SUGGESTION"`.
    - `Text`: The human-readable validation message string.
    - `Checked`: `true` if the validation rule is auto-tunable (`Tunable: true` in the reference table), `false` otherwise. Controls whether the checkbox in the report is `checked` or `disabled`.
  - `HasError`: `true` if any message has `Type` `"ERROR"`.
  - `HasWarning`: `true` if any message has `Type` `"WARNING"`.
  - `HasSuggestion`: `true` if any message has `Type` `"SUGGESTION"`.
  - `DoNotGeolocate` (entry): `true` if `CountryCode` is empty or `"ZZ"` — the entry is an explicit do-not-geolocate signal.
  - `GeocodingHint`: Always empty string `""` in Phase 3. Reserved for future use.
  - `Tunable`: `true` if **any** message in the entry has `Checked: true`. Computed as logical OR across all messages' `Checked` values. This flag drives the "Tune" button visibility in the report.

#### Validation Rules Reference

When adding messages to an entry, use the `ID`, `Type`, `Text`, and `Checked` values from this table.

| ID     | Type         | Text                                                                                           | Checked | Condition Reference                    |
|--------|--------------|------------------------------------------------------------------------------------------------|---------|----------------------------------------|
| `1101` | `ERROR`      | IP prefix is empty                                                                             | `false` | IP Prefix Analysis: empty              |
| `1102` | `ERROR`      | Invalid IP prefix: unable to parse as IPv4 or IPv6 network                                     | `false` | IP Prefix Analysis: invalid syntax     |
| `1103` | `ERROR`      | Non-public IP range is not allowed in an RFC 8805 feed                                         | `false` | IP Prefix Analysis: non-public         |
| `3101` | `SUGGESTION` | IPv4 prefix is unusually large and may indicate a typo                                         | `false` | IP Prefix Analysis: IPv4 < /22         |
| `3102` | `SUGGESTION` | IPv6 prefix is unusually large and may indicate a typo                                         | `false` | IP Prefix Analysis: IPv6 < /64         |
| `1201` | `ERROR`      | Invalid country code: not a valid ISO 3166-1 alpha-2 value                                     | `true`  | Country Code Analysis: invalid         |
| `1301` | `ERROR`      | Invalid region format; expected COUNTRY-SUBDIVISION (e.g., US-CA)                              | `true`  | Region Code Analysis: bad format       |
| `1302` | `ERROR`      | Invalid region code: not a valid ISO 3166-2 subdivision                                        | `true`  | Region Code Analysis: unknown code     |
| `1303` | `ERROR`      | Region code does not match the specified country code                                          | `true`  | Region Code Analysis: mismatch         |
| `1401` | `ERROR`      | Invalid city name: placeholder value is not allowed                                            | `false` | City Name Analysis: placeholder        |
| `1402` | `ERROR`      | Invalid city name: abbreviated or code-based value detected                                    | `true`  | City Name Analysis: abbreviation       |
| `2401` | `WARNING`    | City name formatting is inconsistent; consider normalizing the value                           | `true`  | City Name Analysis: formatting         |
| `1501` | `ERROR`      | Postal codes are deprecated by RFC 8805 and must be removed for privacy reasons                | `true`  | Postal Code Check                      |
| `3301` | `SUGGESTION` | Region is usually unnecessary for small territories; consider removing the region value        | `true`  | Tuning: small territory region         |
| `3402` | `SUGGESTION` | City-level granularity is usually unnecessary for small territories; consider removing the city value | `true`  | Tuning: small territory city           |
| `3303` | `SUGGESTION` | Region code is recommended when a city is specified; choose a region from the dropdown         | `true`  | Tuning: missing region with city       |
| `3104` | `SUGGESTION` | Confirm whether this subnet is intentionally marked as do-not-geolocate or missing location data | `true`  | Tuning: unspecified geolocation        |
