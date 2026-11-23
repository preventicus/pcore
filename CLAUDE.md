# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

pcore (PreventicusCore) is a **specification-only** repository: it defines a protobuf3 file format for compressed time-based sensor data (PPG, ACC, ECG). There is no implementation here; implementations live in other repositories and are expected to follow this spec. Data files should use the `.pcore` extension.

The parts of the spec must stay consistent with each other:

1. `protobuf_definitions/pcore.proto` — the wire format (package `com.preventicus.pcore`). Source of truth for field names.
2. `README.md` — scope/constraints, class diagram, pseudocode for the compression/decompression algorithms, validation rules (with the exception type per rule), a worked example in protobuf text format, and the description of the test file format.
3. `unittest/**/*.json` — language-neutral test cases that every implementation must reproduce (the only source of the tests; there is no Markdown version or generator):
   - `compression/`, `decompression/` — timestamp and int/double value cases
   - `validation/dataValidation.json` — cases expecting `InvalidDataException` (structural problems), `WrongValueException` (value out of range) or no error (`null`)

When changing an algorithm or validation rule, update the README pseudocode/rules, the affected JSON test cases, and (if relevant) the README example.

## Commands

```bash
# Check that the proto compiles
protoc --proto_path=protobuf_definitions --descriptor_set_out=/dev/null pcore.proto

# Check that all test files are valid JSON
for f in unittest/*/*.json; do python3 -m json.tool "$f" > /dev/null || echo "invalid: $f"; done

# Check that the README example is valid protobuf text format
python3 -c "s=open('README.md').read(); print(s[s.index('    compressed_timestamps_container {'):s.index('## Test Specifications')])" \
  | protoc --proto_path=protobuf_definitions --encode=com.preventicus.pcore.Data pcore.proto > /dev/null
```

## Editing test cases

The JSON format (operations, `expected` vs. `expected_error`, sensor inputs) is documented in the README section "Test Specifications". Keep the existing layout when editing by hand:

- 2-space indentation, key order `name`, `operation`, `purpose`, `input`, `expected`/`expected_error`; lists of numbers stay on one line.
- Test names are unique per file; `purpose` explains intent, including where values are derived from limits (e.g. "max and min int32 values") since the JSON contains only the resulting numbers.
- Integers beyond ±(2^53−1) (e.g. `int64` limits) are written as decimal strings.
- Validation tests target exactly one rule and keep all other inputs valid.

## Format concepts that span multiple files

- **Shared timestamps**: all sensors in a `Data` message share one `CompressedTimestampsContainer`; every sensor must have exactly as many values as there are timestamps. Different sampling rates require separate `Data` messages.
- **Timestamp compression**: timestamps (strictly increasing) are split into sections with a constant interval. Per section the container stores the interval (`inner_sections_durations_ms`), the distance from the previous section's start (`outer_sections_durations_ms`, first element always `0`), and the point count (`sections_sizes`). Inner duration is `0` exactly when a section has a single timestamp; the compressor only produces that for the last section.
- **Value compression**: `IntValuesContainer` (`sint64`) stores deltas, with the first value as a delta from zero; `DoubleValuesContainer` stores values unchanged.
- **Sensor identification**: a sensor is identified by its type message with all fields (PPG: light + `channel`, ACC: `type` + `channel`, ECG: `channel`), never by array position. Uniqueness and a set type are enforced only when compressing; decompression must accept legacy files (written before the PPG/ACC `channel` fields) with duplicate identifications and preserve sensor order.
- **Schema evolution**: `Sensor` reserves field numbers 3–5 for future value containers and 9–15 for future sensor types. Don't renumber existing fields.

## Conventions

- Commit subjects are prefixed with the ticket id from the branch name (e.g. `FOR-517 …`); changes land on `main` via PRs.
