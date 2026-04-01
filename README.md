# ocean-current-omp

## What it does

Simulates steady-state geostrophic circulation in a 2D ocean basin using iterative Gauss-Seidel relaxation on an n×n finite-difference grid.  
After the pressure field converges, Lagrangian particles are advected through the velocity field using 2nd-order Runge-Kutta integration — mimicking virtual vessel drift paths under real ocean currents.

**Real-world connection (perfect for NVIDIA N.Ex.T interviews):**  
Ocean currents are the dominant factor in vessel fuel consumption and ETA prediction on transoceanic routes. A fast, accurate current solver enables real-time route optimisation — exactly the kind of HPC problem shipping companies and maritime AI systems solve.

## Key contributions (what I built as a fresher)

1. **Fixed race condition** — The original OpenMP version summed `diff` across threads without a reduction clause, producing non-deterministic results. Fixed with `#pragma omp parallel for reduction(max:diff)`.
2. **Proper red-black ordering** — Split the grid into red/black phases so every thread updates independent cells with zero dependency conflicts.
3. **Cache-locality tuning** — Used `schedule(static)` with empirically chosen chunk size to keep each thread’s working set inside L2 cache.
4. **Lagrangian particle advection** — Added embarrassingly parallel RK2 tracer (10k+ virtual vessels) — directly relevant to maritime routing.
5. **Full benchmark harness** — `std::chrono` timing across grid sizes (256/512/1024) and thread counts 1–8, plus automated `benchmark.sh` script.

## Folder Structure

```
ocean-current-omp/
├── Makefile
├── README.md
├── BENCHMARKS.md
├── src/
│   ├── main.cpp              # timing harness + CLI
│   ├── grid.h / grid.cpp     # 2D flat-array grid + init
│   ├── solver_serial.h / solver_serial.cpp
│   ├── solver_omp.h / solver_omp.cpp     ← MY CORE CONTRIBUTION
│   ├── particle.h / particle.cpp         ← Lagrangian advection
├── scripts/
│   └── benchmark.sh
├── results/
│   └── speedup_n1024.csv
└── docs/
└── architecture.md

```

## Build (Ubuntu 22.04 / WSL2)

```bash
sudo apt update && sudo apt install -y g++ make
make
Run
Bash# Format: ./ocean_omp <grid_size> <threads>
./ocean_omp 1024 8
Example output:
textn=1024 threads=8 serial=3412ms omp=508ms speedup=6.71x iterations=142
```

## Results (measured on my Intel i7-12700H, 8 cores)
<img width="638" height="328" alt="image" src="https://github.com/user-attachments/assets/10d9bcc3-0c81-4494-b76e-09404d8d42b3" />


<img width="753" height="370" alt="image" src="https://github.com/user-attachments/assets/c6e63a19-ff73-49f1-93d1-42832a1a92e7" />
