# Probe: hash-constraints-repeated-requirements

**Pattern:** `hash-constraints-repeated-requirements`
**PM:** uv 0.12.20
**Category:** checksum_signing
**Generated:** 2026-09-28T23:32:48Z

## Purpose

Tests Mend's ability to preserve cryptographic hashes from
`uv.lock` when the same package appears in both the main
group and a named dependency group — the repeated-requirement
scenario fixed in uv 0.12.20 PR #21996.

Before the fix, `--require-hashes` and `--verify-hashes`
only applied hash constraints to the first occurrence of a
repeated requirement. This probe exercises whether Mend
reads and preserves `hash` fields from `uv.lock` uniformly
for all packages, including those referenced from multiple
groups.

The probe uses `iniconfig==2.0.0` as the repeated package:
it appears in both `[project.dependencies]` (main group) and
`[dependency-groups] test`. The `uv.lock` records a single
`[[package]]` entry for `iniconfig` with both sdist and wheel
hashes. Mend must represent this as one entry in `packages`
with the `hashes` array populated.

## Dependency graph

```
hash-constraints-probe
├── iniconfig==2.0.0         (main) <-- also in test group
├── packaging==24.1          (main)
└── [test]
    ├── iniconfig==2.0.0     (repeated -- same lockfile entry)
    └── pytest==8.3.4
        ├── iniconfig        (transitive)
        ├── packaging        (transitive)
        └── pluggy==1.5.0
```

## Mend failure modes targeted

- Hash fields stripped from lockfile parse output — `hashes`
  arrays in expected tree are empty.
- Duplicate package entries for `iniconfig`: one from the
  `main` group (hashed) and one from `test` group (not
  hashed), instead of a single deduplicated entry.
- Hash algorithm label (`sha256:`) dropped, leaving only
  the raw hex string in the `hashes` array.
- Mend re-downloads artifacts at scan time instead of
  trusting the lockfile hashes.

## Hash values from uv.lock

`iniconfig==2.0.0`:
- sdist: `sha256:2d91e135bf72d31a410b17c16da610a82e7b1d0aca8fe3e7bc576b5e4eb2d2ef`
- wheel: `sha256:b6a85871a79d2e3b22d2d1b94ac2824226a63c6b741c88f7ae975f18b6778374`

`packaging==24.1`:
- sdist: `sha256:026ed72c8ed3fcce5bf8950572258698927fd1dbda10a5e981cdf0ac37f4f002`
- wheel: `sha256:5b8f2217dbdbd2f7f384c41c628544e6d52f2d0f53c6d0c3ea61aa5d1d7ff124`

`pytest==8.3.4`:
- sdist: `sha256:965370d062bce11e73868e0335abac31b4d3de0e82f4007408d242b4f8610761`
- wheel: `sha256:50e16d954148559c9a74109af1eaf0c945ba2d8f30f0a3d3335edde19788b6f6`

`pluggy==1.5.0`:
- sdist: `sha256:2cffa88e94fdc978c4c574f15f9e59b7f4201d439195c3715ca9e2486f1d0cf1`
- wheel: `sha256:44e1ad92c8ca002de6377e165f3e0f1be63266ab4d554740532335b9d75ea669`

## Python version detection

Mend uses `.python-version` (single-line `3.11`) as the
Python version signal. This takes higher precedence than
`pyproject.toml`'s `requires-python = ">=3.11"` in Mend's
PIP version detection chain. Both are present and consistent.

## Mend config

Bucket B — no `.whitesource` required. Python version is
dynamically detected from `.python-version` and
`requires-python`. The uv tool itself is not pinnable via
`install-tool` versioning.

## Source references

- https://github.com/astral-sh/uv/pull/21996
