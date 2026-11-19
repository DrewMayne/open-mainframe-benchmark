# Pre-registered hypotheses

Each hypothesis was committed before any kit run that tests it. Git history is the timestamp. When results arrive, each
claim is scored **VERIFIED** or **DIFFERS**, and the original text is never edited. Revised predictions are added as new,
dated entries.

Status: all OPEN (no kit runs yet). Spec version: v0.1.

## H1: hardware, per core, plain software
For CPU-bound COBOL business logic built by the same compiler (GnuCOBOL) with no accelerators, the throughput of one
LinuxONE IFL and one x86 core will be **within a factor of 2 of each other**. The ratio will **differ by workload class**
(CPU-bound, I/O-bound, mixed) by more than run-to-run noise.

## H2: accelerators, both sides
With accelerators on, on both sides, the Z/x86 per-core ratio will **move toward Z by more than the plain-software ratio**
for the compression and signing steps. The size of that shift will be reported per step. For AI inference at batch size 1,
the result is **open**: no prediction is made until the platform's AIU availability is confirmed.

## H3: batch and online together
- **H3a:** with batch on the same cores at default priority, online capacity on Linux will fall to **0.45-0.65 of the
  alone figure** on both architectures. The fraction will be **roughly constant across transaction mixes**, because the
  scheduler shares CPU per runnable thread, not per transaction cost.
- **H3b:** with tuned priorities (cgroups `cpu.weight` favouring online work), online capacity will recover to
  **≥ 0.85 of alone**, and batch elapsed time will grow.
- **H3c:** the same comparison under z/OS WLM is phase 2. No prediction until then.

## H4: past saturation
Above 100% offered load, p99 latency will rise by **more than 10× within the next 10-20% of load** on both platforms. The
platforms' curves will differ in shape, and those differences will be reported, not ranked.

## H5: batch elapsed time
The batch suite's elapsed time alongside online work will be **1.2-2.0× its alone time** on Linux at default priority.

## Added 2026-11-19 (spec v0.1.1, two tracks; still before any kit run)
- **H1j (Java track):** with the same JVM family (Semeru/OpenJ9) and no accelerators, the per-core throughput ratio of
  LinuxONE IFL to x86 core will be **within 0.15 of the COBOL track's ratio** (H1) for the same transaction set. If the
  gap is larger, the compiler (GnuCOBOL vs JIT), not the hardware, is driving H1.
- **H6 (z/OS, Java track, phase 2):** enabling zIIPs will leave Java-track throughput per engine **within ±10%**, while
  moving **≥ 80% of the CPU time** off general CPs. That is a cost effect, not a speed effect.

## Notes
- These are the maintainer's predictions, made to be tested, not claims.
- A single pilot run on a 2-core x86 PC (not this kit) informed H3a. It found ~0.57 of alone, flat across 27 mixes. It is
  stated here so that H3a is not presented as a blind prediction.
