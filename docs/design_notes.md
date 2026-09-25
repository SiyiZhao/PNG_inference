# Design Notes

## features in plan

```
PNG type
├── local
├── equilateral
├── orthogonal
└── ...

tracer setup
├── single tracer
└── multi-tracer

backend
├── desilike
├── native
└── other inference frameworks
```

## feature matrix

| PNG type \ Backend | Native backend | `desilike` |
| --- | --- | --- |
| Local | dev:single tracer | dev:single tracer |
| Equilateral | Future | Unable |
| Orthogonal | Future | Unable |
| Cosmological Collider | Future | Unable |

## design boundary

```
project repositories
        |
        | stable public API
        v
   PNG_inference
   /     |       \
models  core   provenance
          |
          v
      backend adapter
          |
     +----+------+
     |           |
 desilike      native
```

When using `PNG_inference`, avoid to directly import modules from the outside backend like `desilike` or `cobaya`. 

## Public API 

define a task (`PNGInference`?) -> run -> load results (`InferenceResults`?) -> plot results ?

### PNGInference

Define a task:

```
model:
  png_type: local
  parameterization: ...

data / observables:
  ...

parameters:
  ...

priors:
  ...

backend:
  name: desilike

inference:
  sampler: ...
```

API could be like:

```python
from png_inference import PNGInference, LocalPNG

inference = PNGInference(
    model=LocalPNG(...),
    tracers=...,
    data=...,
    backend="desilike",
)

result = inference.run()
```

## Outputs

Outputs for each inference run:

```
outputs/<run-id>/
├── manifest.yaml
├── resolved_config.yaml
├── chains/ # contains the original outputs from the backend.
├── products/ # contains the derived products from the backend outputs, e.g. bestfit, covariance, etc.
└── figures/
```

- `manifest.yaml`: contains status of `png_inference`, `backend` (commit SHA, clean or not), as well as enviroment and run information, like: 

```manifest.yaml
png_inference:
  version: 0.1.0
  git_commit: ...
  dirty: false

backend:
  name: desilike
  version: ...
  git_commit: ...
  dirty: false

environment:
  python: ...
  lockfile_hash: ...

run:
  config_hash: ...
  random_seed: ...
```