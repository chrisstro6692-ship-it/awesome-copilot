---
name: geofeed-tuner
description: >
  Use this skill whenever the user mentions IP geolocation feeds, RFC 8805, geofeeds, or wants help creating, tuning, validating, or publishing a
  self-published IP geolocation feed in CSV format. Intended user audience is a network
  operator, ISP, mobile carrier, cloud provider, hosting company, IXP, or satellite provider
  asking about IP geolocation accuracy, or geofeed authoring best practices.
  Helps create, refine, and improve CSV-format IP geolocation feeds with opinionated
  recommendations beyond RFC 8805 compliance. Do NOT use for private or internal IP address
  management — applies only to publicly routable IP addresses.
license: Apache-2.0
metadata:
  author: Sid Mathur <support@getfastah.com>
  version: "0.0.9"
compatibility: Requires Python 3
---

# Geofeed Tuner – Create Better IP Geolocation Feeds

This skill helps you create and improve IP geolocation feeds in CSV format by:
- Ensuring your CSV is well-formed and consistent
- Checking alignment with [RFC 8805](references/rfc8805.txt) (the industry standard)
- Applying **opinionated best practices** learned from real-world deployments
- Suggesting improvements for accuracy, completeness, and privacy

## When to Use This Skill

- Use this skill when a user asks for help **creating, improving, or publishing** an IP geolocation feed file in CSV format.
- Use it to **tune and troubleshoot CSV geolocation feeds** — catching errors, suggesting improvements, and ensuring real-world usability beyond RFC compliance.
- **Intended audience:**
  - Network operators, administrators, and engineers responsible for publicly routable IP address space
  - Organizations such as ISPs, mobile carriers, cloud providers, hosting and colocation companies, Internet Exchange operators, and satellite internet providers
- **Do not use** this skill for private or internal IP address management; it applies **only to publicly routable IP addresses**.

## Prerequisites

- **Python 3** is required.

## Directory Structure and File Management

This skill uses a clear separation between **distribution files** (read-only) and **working files** (generated at runtime).

### Read-Only Directories (Do Not Modify)

The following directories contain static distribution assets. **Do not create, modify, or delete files in these directories:**

| Directory      | Purpose                                                    |
|----------------|------------------------------------------------------------|
| `assets/`      | Static data files (ISO codes, examples)                    |
| `references/`  | RFC specifications and code snippets for reference         |
| `scripts/`     | Executable code and HTML template files for reports        |

### Working Directories (Generated Content)

All generated, temporary, and output files go in these directories:

| Directory       | Purpose                                              |
|-----------------|------------------------------------------------------|
| `run/`          | Working directory for all agent-generated content    |
| `run/data/`     | Downloaded CSV files from remote URLs                |
| `run/report/`   | Generated HTML tuning reports                        |

### File Management Rules

1. **Never write to `assets/`, `references/`, or `scripts/`** — these are part of the skill distribution and must remain unchanged.
2. **All downloaded input files** (from remote URLs) must be saved to `./run/data/`.
3. **All generated HTML reports** must be saved to `./run/report/`.
4. **All generated Python scripts** must be saved to `./run/`.
5. The `run/` directory may be cleared between sessions; do not store permanent data there.
6. **Working directory for execution:** All generated scripts in `./run/` must be executed with the **skill root directory** (the directory containing `SKILL.md`) as the current working directory, so that relative paths like `assets/iso3166-1.json` and `./run/data/report-data.json` resolve correctly. Do not `cd` into `./run/` before running scripts.


## Processing Pipeline: Sequential Phase Execution

All phases must be executed **in order**, from Phase 1 through Phase 6. Each phase depends on the successful completion of the previous phase. For example, **structure checks** must complete before **quality analysis** can run.

The phases are summarized below. The agent must follow the detailed steps outlined further in each phase section.

| Phase | Name                       | Description                                                                       |
|-------|----------------------------|-----------------------------------------------------------------------------------|
| 1     | Understand the Standard    | Review the key requirements of RFC 8805 for self-published IP geolocation feeds   |
| 2     | Gather Input               | Collect IP subnet data from local files or remote URLs                            |
| 3     | Checks & Suggestions       | Validate CSV structure, analyze IP prefixes, and check data quality               |
| 4     | Tuning Data Lookup         | Use Fastah's MCP tool to retrieve tuning data for improving geolocation accuracy  |
| 5     | Generate Tuning Report     | Create an HTML report summarizing the analysis and suggestions                    |
| 6     | Final Review               | Verify consistency and completeness of the report data                            |

**Do not skip phases.** Each phase provides critical checks or data transformations required by subsequent stages.


### Execution Plan Rules

Before executing each phase, the agent MUST generate a visible TODO checklist.

The plan MUST:
- Appear at the very start of the phase
- List every step in order
- Use a checkbox format
- Be updated live as steps complete


### Phase 1: Understand the Standard

The key requirements from RFC 8805 that this skill enforces are summarized below. **Use this summary as your working reference.** Only consult the full [RFC 8805 text](references/rfc8805.txt) for edge cases, ambiguous situations, or when the user asks a standards question not covered here.

#### RFC 8805 Key Facts

**Purpose:** A self-published IP geolocation feed lets network operators publish authoritative location data for their IP address space in a simple CSV format, allowing geolocation providers to incorporate operator-supplied corrections.

**CSV Column Order (Sections 2.1.1.1–2.1.1.5):**

| Column | Field         | Required | Notes                                                      |
|--------|---------------|----------|------------------------------------------------------------|
| 1      | `ip_prefix`   | Yes      | CIDR notation; IPv4 or IPv6; must be a network address     |
| 2      | `alpha2code`  | No       | ISO 3166-1 alpha-2 country code; empty or "ZZ" = do-not-geolocate |
| 3      | `region`      | No       | ISO 3166-2 subdivision code (e.g., `US-CA`)               |
| 4      | `city`        | No       | Free-text city name; no authoritative validation set       |
| 5      | `postal_code` | No       | **Deprecated** — must be left empty or absent             |

**Structural rules:**
- Files may contain comment lines beginning with `#` (including the header, if present).
- A header row is optional; if present, it is treated as a comment if it starts with `#`.
- Files must be encoded in UTF-8.
- Subnet host bits must not be set (i.e., `192.168.1.1/24` is invalid; use `192.168.1.0/24`).
- Applies only to **globally routable** unicast addresses — not private, loopback, link-local, or multicast space.

**Do-not-geolocate:** An entry with an empty `alpha2code` or case-insensitive `ZZ` (irrespective of values of region/city) is an explicit signal that the operator does not want geolocation applied to that prefix.

**Postal codes deprecated (Section 2.1.1.5):** The fifth column must not contain postal or ZIP codes. They are too fine-grained for IP-range mapping and raise privacy concerns.


### Phase 2: Gather Input

- If the user has not already provided a list of IP subnets or ranges (sometimes referred to as `inetnum` or `inet6num`), prompt them to supply it. Accepted input formats:
  - Text pasted into the chat
  - A local CSV file
  - A remote URL pointing to a CSV file

- If the input is a **remote URL**:
  - Attempt to download the CSV file to `./run/data/` before processing.
  - On HTTP error (4xx, 5xx, timeout, or redirect loop), **stop immediately** and report to the user:
    `Feed URL is not reachable: HTTP {status_code}. Please verify the URL is publicly accessible.`
  - Do not proceed to Phase 3 with an incomplete or empty download.

- If the input is a **local file**, process it directly without downloading.

- **Encoding detection and normalization:**
  1. Attempt to read the file as UTF-8 first.
  2. If a `UnicodeDecodeError` is raised, try `utf-8-sig` (UTF-8 with BOM), then `latin-1`.
  3. Once successfully decoded, re-encode and write the working copy as UTF-8.
  4. If no encoding succeeds, stop and report: `Unable to decode input file. Please save it as UTF-8 and try again.`


### Phase 3: Checks & Suggestions

#### Execution Rules
- Generate a **script** for this phase.
- Do NOT combine this phase with others.
- Do NOT precompute future-phase data.
- Store the output as a JSON file at: `./run/data/report-data.json`

#### Schema Definition and Validation Rules Reference

Read before running Phase 3 checks. The report-data schema and the full validation rules table. See [references/phase3-schema-and-validation-rules.md](references/phase3-schema-and-validation-rules.md) for:

- Schema Definition
- Validation Rules Reference

#### Populating Messages

When a validation check matches, add a message to the entry's `Messages` array using the values from the reference table:
```python
entry["Messages"].append({
    "ID": "1201",      # From the table
    "Type": "ERROR",   # From the table
    "Text": "Invalid country code: not a valid ISO 3166-1 alpha-2 value",  # From the table
    "Checked": True    # From the table (True = tunable)
})
```

After populating all messages for an entry, derive the entry-level flags:
```python
entry["HasError"] = any(m["Type"] == "ERROR" for m in entry["Messages"])
entry["HasWarning"] = any(m["Type"] == "WARNING" for m in entry["Messages"])
entry["HasSuggestion"] = any(m["Type"] == "SUGGESTION" for m in entry["Messages"])
entry["Tunable"] = any(m["Checked"] for m in entry["Messages"])
```

#### Accuracy Level Counting Rules

Accuracy levels are **mutually exclusive**. Assign each valid (non-ERROR, non-invalid) entry to exactly one bucket based on the most granular non-empty geo field:

| Condition                                                    | Bucket                      |
|--------------------------------------------------------------|-----------------------------|
| `City` is non-empty                                          | `CityLevelAccuracy`         |
| `RegionCode` non-empty AND `City` is empty                   | `RegionLevelAccuracy`       |
| `CountryCode` non-empty, `RegionCode` and `City` empty       | `CountryLevelAccuracy`      |
| `DoNotGeolocate` (entry) is `true`                           | `DoNotGeolocate` (metadata) |

**Do not count** entries with `HasError: true` or entries in `InvalidEntries` in any accuracy bucket.

The agent MUST NOT:
- Rename fields
- Add or remove fields
- Change data types
- Reorder keys
- Alter nesting
- Wrap the object
- Split into multiple files

If a value is unknown, **leave it empty** — never invent data.

#### Structure & Format Check

This phase verifies that your feed is well-formed and parseable. **Critical structural errors** must be resolved before the tuner can analyze geolocation quality.

##### CSV Structure

This subsection defines rules for **CSV-formatted input files** used for IP geolocation feeds.
The goal is to ensure the file can be parsed reliably and normalized into a **consistent internal representation**.

- **CSV Structure Checks**
  - If `pandas` is available, use it for CSV parsing.
  - Otherwise, fall back to Python's built-in `csv` module.

  - Ensure the CSV contains **exactly 4 or 5 logical columns**.
  - Comment lines are allowed.
  - A header row **may or may not** be present.
  - If no header row exists, assume the implicit column order:
    ```
    ip_prefix, alpha2code, region, city, postal code (deprecated)
    ```
  - Refer to the example input file:
    [`assets/example/01-user-input-rfc8805-feed.csv`](assets/example/01-user-input-rfc8805-feed.csv)

- **CSV Cleansing and Normalization**
  - Clean and normalize the CSV using Python logic equivalent to the following operations:
    - Select only the **first five columns**, dropping any columns beyond the fifth.
    - Write the output file with a **UTF-8 BOM**.

  - **Comments**
    - Remove comment rows where the **first column begins with `#`**.
    - This also removes a header row if it begins with `#`.
    - Create a map of comments using the **1-based line number** as the key and the full original line as the value. Also store blank lines.
    - Store this map in a JSON file at: `./run/data/comments.json`
    - Example: `{ "4": "# It's OK for small city states to leave state ISO2 code unspecified" }`

- **Notes**
  - Both implementation paths (`pandas` and built-in `csv`) must write output using
    the `utf-8-sig` encoding to ensure a **UTF-8 BOM** is present.

#### IP Prefix Analysis
  - Check that the `IPPrefix` field is present and non-empty for each entry.
  - Check for duplicate `IPPrefix` values across entries.
  - If duplicates are found, stop the skill and report to the user with the message: `Duplicate IP prefix detected: {ip_prefix_value} appears on lines {line_numbers}`
  - If no duplicates are found, continue with the analysis.

  - **Checks**
    - Each subnet must parse cleanly as either an **IPv4 or IPv6 network** using the code snippets in the `references/` folder.
    - Subnets must be normalized and displayed in **CIDR slash notation**.
      - Single-host IPv4 subnets must be represented as **`/32`**.
      - Single-host IPv6 subnets must be represented as **`/128`**.

  - **ERROR**
    - Report the following conditions as **ERROR**:

    - **Invalid subnet syntax**
      - Message ID: `1102`

    - **Non-public address space**
      - Applies to subnets that are **private, loopback, link-local, multicast, or otherwise non-public**
        - In Python, detect non-public ranges using `is_private` and related address properties as shown in `./references`.
      - Message ID: `1103`

  - **SUGGESTION**
    - Report the following conditions as **SUGGESTION**:

    - **Overly large IPv6 subnets**
      - Prefixes shorter than `/64`
      - Message ID: `3102`

    - **Overly large IPv4 subnets**
      - Prefixes shorter than `/22`
      - Message ID: `3101`

#### Geolocation Quality Check

Analyze the **accuracy and consistency** of geolocation data:
  - Country codes
  - Region codes
  - City names
  - Deprecated fields

This phase runs after structural checks pass.

##### Country Code Analysis
  - Use the locally available data table [`ISO3166-1`](assets/iso3166-1.json) for checking.
    - JSON array of countries and territories with ISO codes
    - Each object includes:
      - `alpha_2`: two-letter country code
      - `name`: short country name
      - `flag`: flag emoji
    - This file represents the **superset of valid `CountryCode` values** for an RFC 8805 CSV.
  - Check the entry's `CountryCode` (RFC 8805 Section 2.1.1.2, column `alpha2code`) against the `alpha_2` attribute.
  - Sample code is available in the `references/` directory.

  - If a country is found in [`assets/small-territories.json`](assets/small-territories.json), mark the entry internally as a small territory. This flag is used in later checks and suggestions but is **not stored in the output JSON** (it is transient validation state).

  - **Note:** `small-territories.json` contains some historic/disputed codes (`AN`, `CS`, `XK`) that are not present in `iso3166-1.json`. An entry using one of these as its `CountryCode` will fail the country code validation (ERROR) even though it matches as a small territory. The country code ERROR takes precedence — do not suppress it based on the small-territory flag.

  - **ERROR**
    - Report the following conditions as **ERROR**:
    - **Invalid country code**
      - Condition: `CountryCode` is present but not found in the `alpha_2` set
      - Message ID: `1201`

  - **SUGGESTION**
    - Report the following conditions as **SUGGESTION**:

    - **Unspecified geolocation for subnet**
      - Condition: All geographical fields (`CountryCode`, `RegionCode`, `City`) are empty for a subnet.
      - Action: 
        - Set `DoNotGeolocate = true` for the entry.
        - Set `CountryCode` to `ZZ` for the entry.
      - Message ID: `3104`


##### Region Code Analysis
  - Use the locally available data table [`ISO3166-2`](assets/iso3166-2.json) for checking.
    - JSON array of country subdivisions with ISO-assigned codes
    - Each object includes:
      - `code`: subdivision code prefixed with country code (e.g., `US-CA`)
      - `name`: short subdivision name
    - This file represents the **superset of valid `RegionCode` values** for an RFC 8805 CSV.
  - If a `RegionCode` value is provided (RFC 8805 Section 2.1.1.3):
    - Check that the format matches `{COUNTRY}-{SUBDIVISION}` (e.g., `US-CA`, `AU-NSW`).
    - Check the value against the `code` attribute (already prefixed with the country code).

  - **Small-territory exception:** If the entry is a small territory **and** the `RegionCode` value equals the entry's `CountryCode` (e.g., `SG` as both country and region for Singapore), treat the region as acceptable — skip all region validation checks for this entry. Small territories are effectively city-states with no meaningful ISO 3166-2 administrative subdivisions.

  - **ERROR**
    - Report the following conditions as **ERROR**:
    - **Invalid region format**
      - Condition: `RegionCode` does not match `{COUNTRY}-{SUBDIVISION}` **and** the small-territory exception does not apply
      - Message ID: `1301`
    - **Unknown region code**
      - Condition: `RegionCode` value is not found in the `code` set **and** the small-territory exception does not apply
      - Message ID: `1302`
    - **Country–region mismatch**
      - Condition: Country portion of `RegionCode` does not match `CountryCode`
      - Message ID: `1303`

##### City Name Analysis

  - City names are validated using **heuristic checks only**.
  - There is currently **no authoritative dataset** available for validating city names.

  - **ERROR**
    - Report the following conditions as **ERROR**:
    - **Placeholder or non-meaningful values**
      - Condition: Placeholder or non-meaningful values including but not limited to:
        - `undefined`
        - `Please select`
        - `null`
        - `N/A`
        - `TBD`
        - `unknown`
      - Message ID: `1401`

    - **Truncated names, abbreviations, or airport codes**
      - Condition: Truncated names, abbreviations, or airport codes that do not represent valid city names:
        - `LA`
        - `Frft`
        - `sin01`
        - `LHR`
        - `SIN`
        - `MAA`
      - Message ID: `1402`

  - **WARNING**
    - Report the following conditions as **WARNING**:
    - **Inconsistent casing or formatting**
      - Condition: City names with inconsistent casing, spacing, or formatting that may reduce data quality, for example:
        - `HongKong` vs `Hong Kong`
        - Mixed casing or unexpected script usage
      - Message ID: `2401`

##### Postal Code Check
  - RFC 8805 Section 2.1.1.5 explicitly **deprecates postal or ZIP codes**.
  - Postal codes can represent very small populations and are **not considered privacy-safe** for mapping IP address ranges, which are statistical in nature.

  - **ERROR**
    - Report the following conditions as **ERROR**:
    - **Postal code present**
      - Condition: A non-empty value is present in the postal/ZIP code field.
      - Message ID: `1501`

#### Tuning & Recommendations

This phase applies **opinionated recommendations** beyond RFC 8805, learned from real-world geofeed deployments, that improve accuracy and usability.

- **SUGGESTION**
  - Report the following conditions as **SUGGESTION**:

  - **Region or city specified for small territory**
    - Condition:
      - Entry is a small territory
      - `RegionCode` is non-empty **OR**
      - `City` is non-empty.
    - Message IDs: `3301` (for region), `3402` (for city)

  - **Missing region code when city is specified**
    - Condition:
      - `City` is non-empty
      - `RegionCode` is empty
      - Entry is **not** a small territory
    - Message ID: `3303`

### Phase 4: Tuning Data Lookup

#### Objective
Lookup all the `Entries` using Fastah's `rfc8805-row-place-search` tool.

#### Execution Rules
- Generate a new **script** _only_ for payload generation (read the dataset and write one or more payload JSON files; do not call MCP from this script).
- Server only accepts 1000 entries per request, so if there are more than 1000 entries, split into multiple requests.
- The agent must read the generated payload files, construct the requests from them, and send those requests to the MCP server in batches of at most 1000 entries each.
- **On MCP failure:** If the MCP server is unreachable, returns an error, or returns no results for any batch, log a warning and continue to Phase 5. Set `TunedEntry: {}` for all affected entries. Do not block report generation. Notify the user clearly: `Tuning data lookup unavailable; the report will show validation results only.`
- Suggestions are **advisory only** — **never auto-populate** them.

#### Step 1: Build Lookup Payload with Deduplication

Load the dataset from: `./run/data/report-data.json`
- Read the `Entries` array. Each entry will be used to build the MCP lookup payload.

Reduce server requests by deduplicating identical entries:
- For each entry in `Entries`, compute a content hash (hash of `CountryCode` + `RegionCode` + `City`).
- Create a deduplication map: `{ contentHash -> { rowKey, payload, entryIndices: [] } }`. rowKey is a UUID that will be sent to the MCP server for matching responses.
- If an entry's hash already exists, append its **0-based array index** in `Entries` to that deduplication entry's `entryIndices` array.
- If hash is new, generate a **UUID (rowKey)** and create a new deduplication entry.

Build request batches:
- Extract unique deduplicated entries from the map, keeping them in deduplication order.
- Build request batches of up to 1000 items each.
- For each batch, keep an in-memory structure like `[{ rowKey, payload, entryIndices }, ...]` to match responses back by rowKey.
- When writing the MCP payload file, include the `rowKey` field with each payload object:

```json
[
    {"rowKey": "550e8400-e29b-41d4-a716-446655440000", "countryCode":"CA","regionCode":"CA-ON","cityName":"Toronto"},
    {"rowKey": "6ba7b810-9dad-11d1-80b4-00c04fd430c8", "countryCode":"IN","regionCode":"IN-KA","cityName":"Bangalore"},
    {"rowKey": "6ba7b811-9dad-11d1-80b4-00c04fd430c8", "countryCode":"IN","regionCode":"IN-KA"}
]
```

- When reading responses, match each response `rowKey` field to the corresponding deduplication entry to retrieve all associated `entryIndices`.

Rules:
- Write payload to: `./run/data/mcp-server-payload.json`
- Exit the script after writing the payload.

#### Step 2: Invoke Fastah MCP Tool

Read before running Phase 4 Step 2. Full instructions for calling the Fastah MCP tool, handling its responses, and merging tuning data into the report. See [references/phase4-fastah-lookup.md](references/phase4-fastah-lookup.md) for:

- Step 2: Invoke Fastah MCP Tool
