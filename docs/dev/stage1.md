# Stage 1 Roadmap

Aim: implement a backend adapter for `desilike`, and support local PNG inference with single tracer.

dependency: `desilike` refactor-jax branch

## Steps

1. Environment Setup
2. Reference Run
3. Core API & backend adapter
4. Results & provennance
5. Validation & documentation

## Checklist

- [ ] backend provenance: record the `desilike` commit hash, and `png_inference` version and commit hash in the inference results.
- [ ] environment reproducibility: use `uv` to record the environment and dependencies, and provide a `uv` environment file for reproducibility.
- [ ] version control" `git` + `uv` 

- [ ] local PNG 'b-p', 'bphi', 'bfnl' three modes all works
- [ ] results saved in structure and readable
- [ ] the results of adapter is same as directly run desilike 

## Stage define

```
                    Stage 1

       Local PNG galaxy P(k) inference
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       theory           data         priors
          │              │              │
          └──────────────┼──────────────┘
                         │
                  Gaussian likelihood
                         │
                       sampler
                         │
                       chain
                         │
                  standardized result
                         │
                   provenance record
```

## Whole Picture of the package

```
                    PNG_inference
                         │
              user-facing inference API
                         │
          ┌──────────────┼──────────────┐
          │              │              │
        theory          data        inference setup
          │              │              │
          │              │         priors / sampler
          │              │              │
          └──────────────┼──────────────┘
                         │
                         ▼
                  backend adapter
                         │
                  desilike backend
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      theory          observable      likelihood
        │                │                │
        └────────────────┼────────────────┘
                         │
                      sampler
                         │
                         ▼
                       chain
                         │
                         ▼
               PNG_inference result
```

### User Configuration

4 modules: `model`, `data`, `likelihood`, `sampler`, each has a `name` and a set of parameters (priors for model, input path for data, etc.)

```yaml
output_root: outputs/
model:
  name: LocalPNG
  parameters: # priors
    f_NL: {prior: uniform(-100, 100)}
    b1: {prior: uniform(0, 5)}
    p: {prior: uniform(0, 3)}
data:
    name: GalaxyPk
    input_path: ...
likelihood:
    name: GaussianLikelihood
    covariance_path: ...
sampler:
    name: dynesty
    nlive: 1000
backend: desilike
```

### Model

```
theory
│
├── PNG model
│     ├── local PNG
│     └── parameterization
│
├── tracer model
│     ├── b1
│     ├── bphi / p / product
│     ├── RSD
│     ├── FoG
│     └── shot noise
│
└── cosmology
      ├── Pm(k)
      ├── primordial P(k)
      ├── growth
      └── AP
```

### Data

use `lsstype` for the data format?

```
galaxy P(k) measurement
│
├── P0(k), P2(k), ...
├── k bins
├── covariance
├── window / projection
└── scale cuts
```

### Likelihood

Gaussian galaxy-clustering likelihood

### Sampler

input: parameters, priors, log-likelihood
output: chain / profile / posterior samples

### Backend

```
PNG_inference concept
          ↓ translate
desilike concept
```
