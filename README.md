# Minimal repro: staging + `ignore_run_exports`

This recipe is intended to reproduce the staging behavior seen during a conda-forge
v1 migration:

- the staging output has `python` and `numpy` in `host`
- the inheriting non-Python output adds:
  - `ignore_run_exports.by_name: [python, numpy]`
- but the final emitted package still gets `python_abi` and `numpy` in `run`

## Command

Run this from inside this directory:

```bash
RATTLER_BUILD_EXPERIMENTAL=true \
rattler-build build -r . --target-platform linux-64
```

## Expected bad behavior

In the log for `plain-output`, inspect the `Finalized run dependencies` section.

The repro is successful if `plain-output` still ends up with Python-related
runtime metadata such as:

- `numpy ...`
- `python_abi 3.13.* *_cp313`

even though `plain-output` declared:

```yaml
requirements:
  ignore_run_exports:
    by_name:
      - python
      - numpy
```

## Observed locally

Observed on `rattler-build 0.58.4` with:

```bash
cd report-error-minimal-repro
RATTLER_BUILD_EXPERIMENTAL=true \
rattler-build build -r . --target-platform linux-64
```

The emitted package was:

```text
plain-output-0.0.1-np2py313hedf0d3e_0
```

and the final build summary still contained:

```text
Run dependencies:
  numpy      >=1.23,<3
  python_abi 3.13.* *_cp313
```
