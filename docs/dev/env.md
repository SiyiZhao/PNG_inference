# Environment Setup

This project uses `uv` to manage the development environment. 

We run the following command to initialize the project:

1. Initialize:
```bash
uv init --lib --name png-inference
```

2. Setup the path to project environment (**this step depends on NERSC!**):
```bash
export UV_PROJECT_ENVIRONMENT=/pscratch/sd/s/siyizhao/envs/png_inference
```
I also write it to local `.env.sh` file. 
Next time open a new terminal, run `source .env.sh` to set the environment variable.

Also set up the MPI environment:
```bash
module load cray-mpich-abi
export MPICH_GPU_SUPPORT_ENABLED=0
```

3. Add `desilike` to dependencies, and use the `refactor-jax` branch for the latest PNG API:
```bash
uv add --optional desilike git+https://github.com/cosmodesi/desilike.git --branch refactor-jax
```

Also add `matplotlib` for plotting:
```bash
uv add matplotlib
```

4. Sync the environment:
```bash
uv sync --extra desilike
```

5. Verify the new PNG API:
```bash
uv run --extra desilike python -c "from desilike.theories.galaxy_clustering import PNGTracerSpectrum2Poles; print(PNGTracerSpectrum2Poles)"
```
