# Schema Analyzer

> An Open Source joint by [Dan Levy](https://danlevy.net/) ✨

## Analyze column type & size summary from any input JSON array!

Schema **Analyzer** is the core library behind Dan's [Schema **Generator**](https://github.com/justsml/schema-generator).

### Features

The primary goal is to support any input JSON/CSV and infer as much as possible. More data will generally yield better results.

- [x] Heuristic type analysis for arrays of objects.
- [x] Browser-based (local, no server necessary)
- [x] Automatic type detection:
    - [x] ID - Identifier column, by name and unique Integer check (detects BigInteger)
    - [x] ObjectId (MongoDB's 96 bit/12 Byte ID. 32bit timestamp + 24bit MachineID + 16bit ProcessID + 24bit Counter)
    - [x] UUID/GUID (Common 128 bit/16 Byte ID. Stored as a hex string, dash delimited in parts: 8, 4, 4, 4, 12)
    - [x] Boolean (detects obvious strings `true`, `false`, `Y`, `N`)
    - [x] Date (Smart detection via comprehensive regex pattern)
    - [x] Timestamp (integer, number of milliseconds since unix epoch)
    - [x] Currency (62 currency symbols supported)
    - [x] Float (w/ scale & precision measurements)
    - [x] Number (Integers)
    - [x] Null (sparse column data helps w/ certain inferences)
    - [x] Email (falls back to string)
    - [x] String (big text and varchar awareness)
    - [x] Array (includes min/max/avg length)
    - [x] Object
- [x] Detects column size minimum, maximum and average
- [x] Includes data points at the 30th, 60th and 90th percentiles (for detecting outliers and enum types!)
- [x] Handles some error/outliers
- [x] Quantify # of unique values per column
- [ ] Identify enum Fields w/ Values
- [ ] Identify `Not Null` fields
- [ ] Nested data structure & multi-table relational output.
- [ ] _Un-de-normalize_ JSON into flat typed objects.

### Getting Started

```js
npm install schema-analyzer
```

```js
import { schemaBuilder } from 'schema-builder'

schemaBuilder(schemaName: String, data: Array<Object>): TypeSummary
```

### Data Analysis Results: `TypeSummary`

> What does this library's analysis look like?

It consists of 3 key top-level properties:

- `totalRows` - # of rows analyzed.
- `uniques: ` - a 'map' of field names & the # of unique values found.
- `fields: FieldRangeSummary` - a map of field names with all detected types ([includes meta-data](#fieldrangesummary) for each type detected, with possible overlaps. e.g. an `Email` is also a `String`, `"42"` is a String and Number)

#### Review the raw results below

Details about nested types can be found below.

```json
{
  "totalRows": 5,
  "uniques": {
    "id": 5,
    "name": 5,
    "role": 3,
    "email": 5,
    "createdAt": 5,
    "accountConfirmed": 2
  },
  "fields": {
    "id": {
      "Number": {
        "value": { "min": 1, "avg": 3, "max": 5, "percentiles": [ 2, 4, 5 ] },
        "count": 5,
        "rank": 8
      },
      "String": {
        "length": { "min": 1, "avg": 1, "max": 1, "percentiles": [ 1, 1, 1 ] },
        "count": 5,
        "rank": 12
      }
    },
    "name": {
      "String": {
        "length": { "min": 3, "avg": 7.2, "max": 15, "percentiles": [ 3, 10, 15 ] },
        "count": 5,
        "rank": 12
      }
    },
    "role": {
      "String": {
        "length": { "min": 4, "avg": 5.4, "max": 9, "percentiles": [ 4, 5, 9 ] },
        "count": 5,
        "rank": 12
      }
    },
    "email": {
      "Email": {
        "count": 5,
        "rank": 11
      },
      "String": {
        "length": { "min": 15, "avg": 19.4, "max": 26, "percentiles": [ 5, 3, 6 ] },
        "count": 5,
        "rank": 12
      }
    },
    "createdAt": {
      "String": {
        "length": { "min": 6, "avg": 9.2, "max": 10, "percentiles": [ 0, 0, 0 ] },
        "count": 5,
        "rank": 12
      }
    },
    "accountConfirmed": {
      "Boolean": {
        "count": 5,
        "rank": 3
      },
      "String": {
        "length": { "min": 4, "avg": 4.4, "max": 5, "percentiles": [ 4, 5, 5 ] },
        "count": 5,
        "rank": 12
      }
    }
  }
}
```

#### Sample input dataset for the example results above:

| id | name            | role      | email                        | createdAt  | accountConfirmed |
|----|-----------------|-----------|------------------------------|------------|------------------|
| 1  | Eve             | poweruser | `eve@example.com`            | 01/20/2020 | false            |
| 2  | Alice           | user      | `ali@example.com`            | 02/02/2020 | true             |
| 3  | Bob             | user      | `robert@example.com`         | 12/31/2019 | true             |
| 4  | Elliot Alderson | admin     | `falkensmaze@protonmail.com` | 01/01/2001 | false            |
| 5  | Sam Sepiol      | admin     | `falkensmaze@hotmail.com`    | 9/9/99     | true             |

### Number & String Range Object Details

Numeric and String types include a summary of the observed field sizes:

#### `FieldRangeSummary<Object>`

##### Properties

- `min` the minimum number or string length
- `max` the maximum number or string length
- `avg` the average number or string length
- `percentiles[30th, 60th, 90th]` values from the `Nth` percentile number or string length

```js
{
  "min": 1, "avg": 1, "max": 1,
  "percentiles": [ 1, 1, 1 ]
}
```

## Notes

We recommend you provide at least 100+ rows. Accuracy increases greatly with 1,000 rows.

The following features require a certain minimum # of records:

- Enumeration detection.
  - 100+ Rows Required.
  - Number of unique values must not exceed 20 or 5% of the total number of records. (100 records will identify as Enum w/ 5 values. Up to 20 are possible given 400 or 1,000+.)
- `Not Null` detection.
  - where rowCount === field count

