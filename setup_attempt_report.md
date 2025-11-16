# Setup Attempt Report

This log captures the required setup steps that were attempted on Sun Nov 16 06:55:29 UTC 2025 and the blockers that prevented
completion.

## 1. Python environment (`./setup.sh`)
- Running `./setup.sh` fails immediately because `conda` is not installed in the container image, so the script cannot create the
  `b2txt25` environment. 【812fce†L1-L4】
- The script then falls through to the `pip install` commands, but every attempt to reach the PyTorch wheel index is blocked by
  the environment's HTTP proxy, which returns `403 Forbidden`. 【4d4c1f†L1-L6】
- Installing Miniconda manually via `wget https://repo.anaconda.com/...` also fails with the same `403 Forbidden` proxy error,
  so there is currently no way to add `conda` to the host in order to retry the setup script. 【38d650†L1-L6】

## 2. Language-model environment (`./setup_lm.sh`)
- The language-model setup script has the same `conda` prerequisite and therefore fails for the same reason as soon as it tries
  to source `conda.sh`. 【65d6a5†L1-L4】
- Subsequent `pip install` steps are likewise blocked by the proxy when they try to download `torch==1.13.1`, so the
  `b2txt25_lm` environment cannot be provisioned. 【04f075†L1-L6】

## 3. Redis installation
- Installing Redis per the README requires `apt-get`, but even `apt-get update` cannot reach the Ubuntu mirrors because the
  proxy again responds with `403 Forbidden`. 【6ef95a†L1-L19】
- `redis-server` is therefore not present on the machine. 【994593†L1-L2】

## 4. Data download and verification
- With the repository moved to `/workspace/nejm-brain-to-text` (so the script's safety check on the working directory succeeds),
  running `python download_data.py` still fails because `datadryad.org` is unreachable through the proxy (`Tunnel connection
  failed: 403 Forbidden`). 【6f31b0†L1-L34】
- Directly hitting the site with `curl -I https://datadryad.org` shows the same proxy block, confirming that the dataset cannot
  be downloaded from within this environment. 【faea10†L1-L9】
- As a result, the local `data/` directory still contains only the repository-provided placeholders and does **not** match the
  expected layout from the README (missing `hdf5_data_final/` and `t15_pretrained_rnn_baseline/`). 【e79821†L1-L5】
