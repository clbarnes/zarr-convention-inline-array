# Template Convention Metadata

- **UUID**: 551e1edc-9717-4795-b149-5b855b00648c
- **Name**: Inline Array
- **Schema URL**: "<https://raw.githubusercontent.com/clbarnes/zarr-convention-inline-array/refs/tags/v1/schema.json>"
- **Spec URL**: "<https://github.com/clbarnes/zarr-convention-inline-array/blob/v1/README.md>"
- **Scope**: Array, Group
- **Extension Maturity Classification**: Proposal
- **Owner**: @clbarnes

## Description

This convention defines a JSON object which can be interpreted the same as a Zarr array.
The goal is to allow small arrays to be inlined in a way which is consistent across data types and implementations;
avoiding the cost of multiple (possibly network-bound) IO operations for minuscule amounts of data.

## Motivation

In OME-Zarr, we have certain fields which are always small arrays,
for example the coefficients for affine coordinate transformations.

We explicitly allow such structs to have either a `path` (to a zarr array) or an inline form (array of arrays of numbers),
but it would be preferable to have a consistent way of defining this across ecosystems.

## Convention Registration

The convention must be registered in `zarr_conventions`:

```json
{
  "zarr_conventions": [
    {
      "schema_url": "https://raw.githubusercontent.com/clbarnes/zarr-conventions-inline-array/refs/tags/v1/schema.json",
      "spec_url": "https://github.com/clbarnes/zarr-conventions-inline-array/blob/v1/README.md",
      "uuid": "551e1edc-9717-4795-b149-5b855b00648c",
      "name": "inline-array",
      "description": "Inline Zarr arrays in metadata"
    }
  ]
}
```

## Applicable To

This convention can be used with these parts of the Zarr hierarchy:

- [x] Group
- [x] Array

## Properties

This convention defines a structure which can be used by other metadata specifications.

| Field Name      | Requirement | Type                                                   | Description                                                                              |
| --------------- | ----------- | ------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| values          | MUST        | array of anything                                      | Values are encoded as in Zarr v3 `fill_value`s, in lexicographic ("C"/ row-major) order. |
| data_type       | MAY         | Zarr data type (string or object); default `"float64"` | As in the Zarr v3 spec.                                                                  |
| shape           | MAY         | array of unsigned integer; default `[values.length]`   | As in the Zarr v3 spec. The product of the elements MUST equal the length of `values`.   |
| dimension_names | MAY         | array of string                                        | As in the Zarr v3 spec.                                                                  |
| attributes      | MAY         | object; default `{}`                                   | As in the Zarr v3 spec.                                                                  |

While `dimension_names` and `attributes` are supported for parity with the Zarr array data model,
it is RECOMMENDED that if you rely on them, you SHOULD use a separate Zarr array.

**Example**:

```jsonc
{
  "attributes": {
    "zarr_conventions": [{ "name": "inline-array", "spec_url": "..." }],
    "minimal_array": {
      "values": [1.5, "NaN", "Infinity", -30],
      // default "data_type":"float64", "shape":[4]
    },
    "integer_array": {
      "data_type": "uint8",
      "values": [1, 2, 3, 255],
      // 1.5 or 256 would be an error
    },
    "shaped": {
      "values": [1, 2, 3, 4, 5, 6],
      "shape": [2, 3],
      // | 1 2 3 |
      // | 4 5 6 |
    },
    "struct_dtype": {
      "data_type": {
        "name": "struct",
        "configuration": {
          "fields": [
            { "name": "a", "data_type": "float32" },
            { "name": "b", "data_type": "int8" },
          ],
        },
      },
      "values": [
        { "a": 1.5, "b": -1 },
        { "a": "Infinity", "b": 127 },
      ],
    },
  },
}
```

## FAQs

> Why not allow colexicographic (F/ column-major) ordering?

The default Zarr array-to-bytes codec, `bytes`, assumes lexicographic ordering.
These arrays should be so small that there is no performance benefit to changing the order;
it simply becomes a more complicated API surface to support.

> Why default to `float64`?

The vast majority of Zarr arrays are numeric.
`float64` is the numeric type JSON was designed to encode.
Having defaults for most fields was a design goal, to keep the metadata slim.

## Known Implementations

This section helps potential implementers assess the convention's maturity and adoption, and provides a way for the community to collaborate on future revisions.

### Libraries and Tools

- **[Library/Tool Name](https://link-to-repo)** - Brief description of the implementation
  - Language: Python/JavaScript/Rust/etc.
  - Status: Experimental/Stable/Production
  - Maintainer: @github-handle
  - Since: Version X.Y or Date

- **[Another Implementation](https://link)** - Description
  - Language: Language
  - Status: Status
  - Maintainer: @handle
  - Since: Version/Date

## Acknowledgements

This template is based on the [STAC extensions template](https://github.com/stac-extensions/template/blob/main/README.md).
