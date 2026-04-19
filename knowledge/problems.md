# problems to solve

ordered immediate to long term. each problem blocks or shapes what comes after it.

### at a glance

| Fact | Why it matters |
|---|---|
| `home` = 50GB persistent | Never install large envs or model caches here |
| `/scratch` = 14TB/node, 15-day delete | Use for envs and caches; not persistent — back up what matters |
| Turing: heterogeneous — node01-04 RTX 6000, node05-14 L40S (FP8), node10 8×A100 40GB | Max 4 GPUs × 48GB = 192GB on RTX/L40S nodes; 70B FP16 fits |
| 4-hour wall-time limit | Checkpoint early, checkpoint often |
| Cross-node network is 10GbE (not InfiniBand) | Single-node 4-GPU is the reliable ceiling for now |
| `nlp`/`irel` accounts have 12 GPUs + no time limit | Ask about access - changes what's feasible |

<img src="../media/stage0/IMG_20260419_105731.jpg">

## immediate

<img src="../media/stage0/IMG_20260419_105713.jpg">

**access**
- VPN and SSH access not set up for everyone on the team
- email accounts pending for new members
- unclear which accounts (Turing, nlp, irel) each person has access to

**environment**
- different PyTorch versions installed across nodes - jobs that work on one node may silently fail on another
- PyTorch and all libraries installed in home (~50GB limit) - fills up before any model is downloaded
- no shared conda env or Singularity image - every student sets up from scratch, diverging over time
- HF model cache not pointed at /scratch - models re-downloaded repeatedly by each student

## short term

**compute constraints on Turing**
- RTX 6000 nodes (01-04): 48GB VRAM × 4 = 192GB; 70B FP16 fits; no FP8
- L40S nodes (05-14): 48GB VRAM × 4 = 192GB; FP8 supported (Ada Lovelace architecture); node10 has 8× A100 40GB
- 4-hour wall-time limit on all jobs - any training run longer than that gets killed; checkpointing is mandatory

**storage**
- home: 50GB persistent - cannot hold an env and large models simultaneously
- /scratch: 14TB per node, auto-deleted after 15 days — viable for large data but not persistent
- no /pfs (parallel filesystem) yet — planned but not available; no shared persistent storage beyond home
- no shared model/data cache - each student downloads the same models independently, filling /scratch

**single-node utilization**
- cluster running at <10% GPU utilization - hardware exists but nobody is using it properly
- no benchmarking harness in place - no way to measure whether changes actually help

**observability**
- no telemetry or logging infrastructure — pilot requires usage metrics and visualization as a Phase A deliverable; vLLM Prometheus endpoint and per-job GPU utilization logs must be wired from first deployment

## medium term

**cross-node training**
- cross-node network is 10GbE — not InfiniBand; this is the structural reason multi-node gradient sync is not feasible, not a misconfiguration
- distributing a model across gnodes fails due to 10GbE bandwidth ceiling (~1 GB/s vs ~600 GB/s NVLink intra-node)
- NCCL tuning for 10GbE Ethernet (not IB) is the relevant investigation

**dependency and portability**
- no Singularity images in place — Turing runs Singularity (singularity-ce/4.2.2), not Docker directly; Docker images pushed to GHCR can be pulled via Singularity on the cluster
- no pinned, reproducible environment - experiments are not reliably reproducible across nodes or students
- no dev-mode vs burst-mode distinction in current workflow — dev (interactive Turing sessions) and burst (scheduled K8s jobs, cloud-portable) require different image designs

**data pipeline**
- no structured data pipeline - training can stall on I/O if data loading is not set up properly
- large datasets have nowhere to live persistently given storage constraints

## long term

**serving**
- no model is currently served as an API - the stated goal of the project requires this
- no load testing or latency/throughput baseline established
- no decision made on internal vs external API exposure or authentication

**infrastructure**
- no NAS in place - proposed solution to shared model/data cache problem; not yet purchased or set up
  > **NOTE:** In the pilot proposal, NAS is a Phase A deliverable and a hard prerequisite for the shared model cache. This is currently blocked pending hardware purchase. Track its status actively with the supervisor — once available, all cache paths (`HF_HOME`, pip, conda) must move to the NAS mount.
- Kubernetes layer discussed but not in place - infra is not self-serve for other students/projects
  > **NOTE:** In the pilot proposal, Kubernetes is funded and scoped to Phase A — running in parallel with storage and container work, not sequenced after serving. The intern does not implement K8s, but all Singularity/Docker work must be K8s-compatible from the start: no hardcoded paths, env variable injection for all configurable values, health check endpoints on all serving processes.
- no autoscaling or GPU-aware job scheduling beyond what the existing scheduler provides
- B/H-series GPU capabilities (mxfp4, mxfp8, larger VRAM) completely unavailable on current hardware - limits what optimizations are even possible

**model optimization**
- no quantization workflow established - needed for running the very largest models efficiently
- no evaluation harness - no way to verify that a quantized or fine-tuned model still performs acceptably
