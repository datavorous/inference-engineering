# stage 0

## aim

**short term:** get a working test bed on Ada - environment, GPU verified, a model running inference and training end to end, benchmarked, containerized.

**long term:** serve LLMs as an API on cluster infrastructure, with distributed training across nodes, quantization workflows, and Kubernetes-managed self-serve infra for the lab.


## infrastructure

**Ada** (~90 nodes, free):
- ~40 nodes with GTX 1080 (older CUDA only), ~50 nodes with RTX 2080/3080
- 11GB VRAM per GPU, up to 4 GPUs per job = 44GB ceiling
- caps out at ~7B FP32 or ~10B FP16; no FlashAttention, no FP8
- training an 8B FP16 model takes ~4 days vs ~1 day on Turing
- 4-day wall-time limit; nlp/irel accounts get 12 GPUs with no time limit
- storage: home 30GB + share1 100GB (both persistent), tmp 1TB (7-day delete)
- cross-node InfiniBand is underperforming - multi-gnode training not viable
- current use: dev and experimentation; target: distributed training + serving

**Turing** (pay-per-use):
- LS40/RTX6000 cards, 48GB VRAM each, up to 4 per job = 192GB ceiling
- same 4-day wall-time, same storage layout
- modern enough for FP16 training but still no mxfp4/mxfp8 or FlashAttention (needs H/B series)
- current use: when Ada can't fit the model; target: burst workloads once containerized

**proposed fixes** (from infra recommendations):
- NAS purchase for shared model/data cache replicated to Ada and Turing
- Docker containers with pinned libraries for reproducibility and portability
- dev-mode vs burst-mode distinction: dev on Ada, burst movable to Turing or cloud

## constraints

| constraint | impact |
|---|---|
| Ada 1080s: older CUDA only | no FlashAttention, no FP8, no modern kernel opts |
| 44GB VRAM ceiling (4x 11GB) | max ~7B FP32 or ~10B FP16 without quantization |
| share1: 100GB persistent | env + cache + checkpoints must all fit here |
| 4-day wall-time on all jobs | checkpoint every run without exception |
| InfiniBand underperforming | cross-node training not viable yet; 4-GPU single-node is the ceiling |
| <10% GPU utilization cluster-wide | no benchmarking baseline exists yet |

## todo

- [ ] get VPN + SSH access, confirm Ada/Turing/nlp/irel account status
- [ ] run assessment commands on first node (nvidia-smi, nvcc, df -h)
- [ ] set cache env vars, create conda env in share1
- [ ] verify GPU visible to PyTorch, check VRAM
- [ ] run opt-125m inference, confirm model cached in share1
- [ ] decide: what model, what precision, what task
- [ ] set up HF streaming dataset, confirm no I/O stall
- [ ] LoRA finetune on toy data, single GPU, checkpoint works
- [ ] torchrun 4-GPU DDP, measure throughput vs 1 GPU
- [ ] write down benchmark numbers (GPU util, tokens/sec, latency)
- [ ] Dockerfile - pin torch + transformers + CUDA version
- [ ] vLLM on single node, hit it with curl
- [ ] investigate InfiniBand bandwidth, NCCL IB support
- [ ] quantization workflow (INT8/INT4 for larger models on Ada)
- [ ] Kubernetes (long term - don't touch until serving is stable)