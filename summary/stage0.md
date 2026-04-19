# stage 0

## aim

**short term:** get a working test bed on Turing - environment, GPU verified, a model running inference and training end to end, benchmarked, containerized.

**long term:** serve LLMs as an API on cluster infrastructure, with distributed training across nodes, quantization workflows, and Kubernetes-managed self-serve infra for the lab.


## fit to pilot

Stage0 outputs feed directly into Phase A of the LTRC Infrastructure Upgrade Pilot. The verified Turing environment, Docker image with pinned dependencies, benchmark baseline (tokens/sec, GPU util, TTFT), and vLLM endpoint with Prometheus metrics are all explicit Phase A inputs — not prerequisites to some later phase.

NAS and Kubernetes are parallel tracks run by the broader team. NAS is a Phase A prerequisite currently blocked on hardware purchase; once available, all cache paths must migrate to the NAS mount. K8s is also Phase A, not sequenced after serving is stable. The intern's job is to produce K8s-compatible artefacts: no hardcoded paths, all configurable values via environment variables, health check and metrics endpoints on every serving process.

See [knowledge/pilot_alignment.md](../knowledge/pilot_alignment.md) for the full phase breakdown and alignment table.


## infrastructure

**Turing** (pay-per-use):
- heterogeneous cluster: node01-04 have RTX 6000 (48GB VRAM), node05-14 have L40S (48GB VRAM, Ada Lovelace, FP8 supported), node10 has 8× A100 40GB
- up to 4 GPUs per job = 192GB ceiling (RTX 6000 / L40S nodes); node10 allows up to 8 GPUs
- 4-hour wall-time limit; nlp/irel accounts get 12 GPUs with no time limit
- storage: home 50GB (persistent), /scratch 14TB per node (15-day auto-delete)
- CUDA modules available: 9.2, 11.7, 11.8, 12.4, 12.9 — load via `module load u22/cuda/12.4`
- cross-node network is 10GbE — multi-gnode training not yet viable
- containers: Turing runs Singularity (singularity-ce/4.2.2); build Docker images, push to GHCR, pull via Singularity
- current target: all dev, training, and serving runs

**proposed fixes** (from infra recommendations):
- NAS purchase for shared model/data cache replicated to Turing
- Docker containers with pinned libraries for reproducibility and portability
- dev-mode vs burst-mode distinction: dev on Turing, burst movable to cloud

## storage

| location | size | persists? | use for |
|---|---|---|---|
| `~/` (home) | 50GB | yes | dotfiles, bashrc, small configs |
| `/scratch/$USER/` | 14TB/node | no - 15 day delete | env, model cache, checkpoints |

everything goes in /scratch. home fills up fast. /scratch auto-deletes after 15 days — back up anything important before it disappears. A parallel filesystem /pfs is planned but does not exist yet.


## environment setup

<img src="../media/stage0/IMG_20260419_105659.jpg">
<img src="../media/stage0/IMG_20260419_105642.jpg">

0. check if a shared conda env or Singularity image already exists on /scratch — do not rebuild from scratch if a teammate has already set one up; confirm PyTorch + CUDA versions match before inheriting
1. assess the node — `nvidia-smi`, `module avail`, `df -h /scratch`, `ls /scratch/`
2. point all cache dirs (HF, pip, conda) at `/scratch/$USER/` before installing anything
3. load the right CUDA module first: `module load u22/cuda/12.4` (or 12.9); create conda env under `/scratch/$USER/envs/` — match torch build to loaded CUDA version; env + cache should stay well under 14TB but confirm /scratch is not full
4. verify GPU visible to PyTorch and VRAM reads 48GB
5. proof of life — run `opt-125m` inference; confirm model lands in /scratch not home
6. verify all 4 GPUs enumerate correctly
7. multi-GPU smoke test with `torchrun --nproc_per_node=4`
8. once bare-metal env is stable, build a Docker image (pin torch + transformers + CUDA version), push to GHCR, and pull on Turing via Singularity: `singularity pull docker://ghcr.io/<org>/<image>:<tag>`


## constraints

| constraint | impact |
|---|---|
| 192GB VRAM ceiling (4x 48GB) on RTX 6000 / L40S nodes | 70B FP16 fits; FP8 supported on L40S (node05-14) and A100 (node10); mxfp4 not available |
| /scratch: 14TB per node, 15-day auto-delete | env + cache + checkpoints go here; not persistent — back up what matters |
| 4-hour wall-time on all jobs | checkpoint every run without exception |
| 10GbE cross-node network | cross-node training not viable yet; 4-GPU single-node is the ceiling |
| <10% GPU utilization cluster-wide | no benchmarking baseline exists yet |
| K8s-compatible Docker images required | no hardcoded paths; use env vars; expose `/health` and `/metrics` on all serving processes |

## todo

- [ ] get VPN + SSH access, confirm Turing/nlp/irel account status
- [ ] confirm NAS purchase status with supervisor — if available, update HF_HOME and all cache paths to NAS mount; if not, use /scratch/$USER as interim
- [ ] run assessment commands on first node (nvidia-smi, nvcc, df -h)
- [ ] set cache env vars, create conda env in /scratch/$USER
- [ ] verify GPU visible to PyTorch — CUDA version matched, confirmed with teammates on torch version
- [ ] run opt-125m inference, confirm model cached in /scratch not home
- [ ] all 4 GPUs verified, torchrun multi-GPU smoke test passes
- [ ] decide: what model, what precision, what task
- [ ] set up HF streaming dataset, confirm no I/O stall
- [ ] LoRA finetune on toy data, single GPU, checkpoint saves mid-run and reloads (mandatory given 4-hour wall-time)
- [ ] torchrun 4-GPU DDP, measure throughput vs 1 GPU
- [ ] write down benchmark numbers (GPU util, tokens/sec, latency)
- [ ] Dockerfile — pin torch + transformers + CUDA version; push to GHCR; pull on Turing with `singularity pull docker://ghcr.io/...`
- [ ] vLLM on single node, hit it with curl
- [ ] wire vLLM Prometheus metrics endpoint — required for pilot telemetry
- [ ] investigate 10GbE cross-node bandwidth, NCCL tuning for Ethernet
- [ ] quantization workflow (INT8/INT4 for larger models)
- [ ] Kubernetes (parallel track — not your implementation task, but design all Docker images to run inside K8s pods from day one)