# Benchmark – Qiskit Aer CPU vs GPU

## Technologies
- **Qiskit 2.5.2** + **Qiskit Aer 0.17.2** (statevector simulator), Python 3.12
- **GPU:** Aer compiled with **CUDA 12.8** (Thrust backend) + **cuStateVec 1.15** (NVIDIA cuQuantum)
- **Container:** Singularity image `qiskit-gpu.sif` (base `nvidia/cuda:12.8.1-runtime-ubuntu24.04`), built with Docker
- **Cluster:** MetaCentrum, PBS Pro (`qsub`), GPUs with compute capability 7.5+ (RTX 2080 … H100, Blackwell)
- **Workload:** `bench.py` – Quantum Volume circuits, 24–28 qubits, depth 20, 100 shots, double precision

## GPU comparison (total time 24–28 qubits)

| GPU | Memory | Host (CPU threads) | CPU | GPU (Aer CUDA) | GPU (cuStateVec) | Speedup vs own CPU | vs RTX 3060 (cuStateVec) |
|---|---|---|---|---|---|---|---|
| GeForce RTX 3060 | 12 GB | local PC (12) | 217.46 s | 36.53 s | 23.62 s | 9.2× | 1.0× |
| RTX A4000 | 16 GB | MetaCentrum `fer2` (8) | 272.67 s | 22.38 s | 14.63 s | 18.6× | 1.6× |
| RTX PRO 6000 Blackwell Server | 96 GB | MetaCentrum `grogu3` (16) | 69.83 s | 4.61 s | 3.29 s | 21.2× | 7.2× |
| H100 NVL | 94 GB | MetaCentrum `bee5` (8) | 170.79 s | 1.48 s | 0.93 s | 184.5× | 25.4× |

cuStateVec is faster than Aer's own CUDA kernels on every GPU (1.4–1.6× on the totals).

## Detailed results

### GeForce RTX 3060 12 GB – local PC, 12 CPU threads
| Qubits | State | CPU | GPU (Aer CUDA) | GPU (cuStateVec) |
|---|---|---|---|---|
| 24 | 0.25 GB | 7.74 s | 1.33 s | 0.72 s |
| 25 | 0.50 GB | 13.68 s | 2.33 s | 1.34 s |
| 26 | 1.00 GB | 26.39 s | 4.67 s | 2.95 s |
| 27 | 2.00 GB | 52.76 s | 8.95 s | 5.96 s |
| 28 | 4.00 GB | 116.88 s | 19.26 s | 12.65 s |
| **total** | | **217.46 s** | **36.53 s (6.0×)** | **23.62 s (9.2×)** |

### RTX A4000 16 GB – MetaCentrum `fer2.natur.cuni.cz`, 8 CPU threads (job 24112172)
| Qubits | State | CPU | GPU (Aer CUDA) | GPU (cuStateVec) |
|---|---|---|---|---|
| 24 | 0.25 GB | 10.44 s | 0.66 s | 0.48 s |
| 25 | 0.50 GB | 18.89 s | 1.40 s | 0.86 s |
| 26 | 1.00 GB | 34.35 s | 2.86 s | 1.88 s |
| 27 | 2.00 GB | 64.96 s | 5.59 s | 3.64 s |
| 28 | 4.00 GB | 144.02 s | 11.88 s | 7.76 s |
| **total** | | **272.67 s** | **22.38 s (12.2×)** | **14.63 s (18.6×)** |

### RTX PRO 6000 Blackwell Server Edition 96 GB – MetaCentrum `grogu3.cerit-sc.cz`, 16 CPU threads (job 24112181)
| Qubits | State | CPU | GPU (Aer CUDA) | GPU (cuStateVec) |
|---|---|---|---|---|
| 24 | 0.25 GB | 1.86 s | 0.15 s | 0.36 s |
| 25 | 0.50 GB | 3.99 s | 0.30 s | 0.20 s |
| 26 | 1.00 GB | 8.21 s | 0.62 s | 0.41 s |
| 27 | 2.00 GB | 16.94 s | 1.14 s | 0.75 s |
| 28 | 4.00 GB | 38.81 s | 2.41 s | 1.57 s |
| **total** | | **69.83 s** | **4.61 s (15.1×)** | **3.29 s (21.2×)** |

### H100 NVL 94 GB – MetaCentrum `bee5.cerit-sc.cz`, 8 CPU threads (job 24120119)
| Qubits | State | CPU | GPU (Aer CUDA) | GPU (cuStateVec) |
|---|---|---|---|---|
| 24 | 0.25 GB | 3.98 s | 0.05 s | 0.09 s |
| 25 | 0.50 GB | 8.48 s | 0.09 s | 0.06 s |
| 26 | 1.00 GB | 18.42 s | 0.18 s | 0.11 s |
| 27 | 2.00 GB | 38.46 s | 0.36 s | 0.21 s |
| 28 | 4.00 GB | 101.45 s | 0.80 s | 0.46 s |
| **total** | | **170.79 s** | **1.48 s (115.0×)** | **0.93 s (184.5×)** |

Notes:
- CPU times depend on the host's cores (`ncpus` of the job), so "speedup vs own CPU" is not comparable across machines; compare the GPU columns.
- The Blackwell GPU (compute capability 12.0) is newer than the compiled targets (7.5–9.0); it most likely runs via CUDA's JIT compilation of the 9.0 code.
  Adding `12.0` to `ARCH` in `Dockerfile.gpu` may make it faster (CUDA 12.8 supports it).
- The H100 (compute capability 9.0, a native compile target) is the fastest by far: ~3.5× faster than the Blackwell RTX PRO 6000 and ~25× faster than the RTX 3060 (cuStateVec totals). At 24 qubits it is fast enough that cuStateVec's fixed overhead makes it slower than Aer CUDA.
- 28 qubits = 4 GB state, so all GPUs could run the whole range; the 94–96 GB cards (H100, RTX PRO 6000) could go up to 32 qubits (`BENCH_QUBITS=24-32`).
