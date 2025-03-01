# Setup Evo2 on h4h UHN cluster

Due to the privacy policy of the UHN cluster, successfully loading Evo2 is a bit complicated. Follow the instructions below to do this seamlessly (and hopefully not waste several hours of labour, the way that I just did!)

## On the login cluster

1. Create a new virtual environment
```bash
conda create -n evo2 python=3.11
```

2. Install all build dependencies
```bash
cd vortex
pip install biopython huggingface_hub torch>=2.6.0 rich ruff pre-commit 
pip install .
pip install transformer_engine[pytorch] --no-build-isolation
```

This last operation will likely fail due to its dependency on a CUDA executable. This can be circumvented, as per Zhibin, by installing CUDA locally in your home folder (good luck. this warrants a guide of its own). Alternatively, you might be able to install the cached downloads in the compute node.

3. Download and cache the Evo2 model (in this case, the 1B model)
```bash

python -c "import huggingface_hub; huggingface_hub.hf_hub_download(repo_id='arcinstitute/evo2_1b_base', filename='evo2_1b_base.pt')"

```

## On the compute cluster

1. Start an interactive slurm session. You'll need at least one GPU.
2. (optional) rerun any pip installs that needed a nvcc executable to install.
2. Compile the CUDA kernels.
```bash
cd vortex/ops/attn
MAX_JOBS=32 pip install -v -e . --no-build-isolation
```
4. Cleanup
```bash
cd ../../..
rm -rf .venv build/ dist/ *.egg-info/
find . -type d -name __pycache__ -exec rm -rf {} +
find . -type f -name "*.pyc" -delete
``````

5. Install Evo2
```bash
pip install .
```

## Conclusion

This should get you up and running with Evo2 on the h4h UHN cluster. Fair warning that I tried many, many approaches to successfully load Evo2 before running this particular set of commands, so there may be important intermediate steps that I missed in this guide. Should any issues arise following this guide, please shoot me a message on slack or submit a PR to edit this README.

Now, go forth and conquer biology!
