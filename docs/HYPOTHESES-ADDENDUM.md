# Hypotheses addendum (proposed)

Entries that add to [HYPOTHESES.md](../HYPOTHESES.md) without editing it. The registered text there stays as it was
committed, byte for byte, and every result is scored against it. Each entry here fixes how registered hypotheses are
measured and scored. An entry edits no registered sentence, but several of its choices narrow, and one widens, what
the registered text, or the spec of the day, allowed. Each such choice is listed under "Choices this entry makes",
with the reading that was not chosen, so that readers can judge it.

**How an entry is registered.** Each entry is registered at a signed git tag and deposited in an external archive
that issues a DOI, as described in [REGISTRY.md](REGISTRY.md). The registration timestamp is the archive's own
creation time of that version, not an editable date field and not the git history. An entry is never edited after
its tag; a change, or a value fixed later, is a new entry with its own tag. Values marked **(proposed)** are this
draft's values; at the tag they become the registered values. SPEC references in an entry mean the text of SPEC.md
at the commit of that entry's tag. No fact an entry relies on carries a "(verify)" or "(source needed)" mark at its
tag.

HYPOTHESES.md says that git history is the timestamp of its entries. That stays true of H1-H6 as registered: their
commits are the only record of when they were written, and the archive deposit at tag omb-v0.4.0-prereg adds an
external latest date for them. From spec v0.4.0, an entry counts as registered only with the signed tag and the
archive deposit described in REGISTRY.md (SPEC 4.7).

## Entry 1, registered at tag omb-v0.4.0-prereg: the comparison basis, the measurands and the scoring configurations

### What changed
H1-H5 were registered against spec v0.1, and H1j and H6 against v0.1.1. At that time SPEC section 2 made equal core
counts the basis of comparison ("runs use equal counts of cores or IFLs"), SPEC section 3 defined throughput as
"transactions per second, per core", with closed loop for capacity (4.2), and README principle 4 listed "equal core
counts" under fair comparison. The x86 side of phase 1 was "x86 PC on bare-metal Linux".

From spec v0.4.0 (proposed), the benchmark adds an **equal-service** comparison: for a stated throughput within a
p99 latency bound fixed before the runs, what each side needs, with cores or engines by type, memory, accelerators,
power where measurable and cost where priced all disclosed (SPEC 2 and 8). It is **exploratory**: no prediction is
registered for it, and it is not a headline result (SPEC 8 and 10). Equal core counts on a single box stay, as the
basis of the registered per-core hypotheses, and the scale-out tier is reported as its own labelled tier.

### Why
- **What prompted it.** A community member asked how an x86 cluster behaves as nodes are added,
  and how requests that carry bookkeeping and reach a live database behave under load. Equal core counts on one box
  cannot express either. The one measured precedent we found for comparing the resources each side needs at a fixed
  offered load is a study commissioned by IBM (docs/prior-art.md, AN01); we found no independent one. This benchmark
  adopts the method, not that study's assumptions or its estate model.
- **What the comparison answers.** Equal service asks what each side needs to deliver a stated throughput within a
  stated response time, which is the question a capacity plan answers. It also lets a cluster be compared with a
  single system. Comparisons at a fixed offered load, reporting the resources each side needs, are one of the
  normalisations prior work has used (docs/prior-art.md, section 5).
- **What it does not change.** Neither an equal-core run nor an equal-service run counts all the processing
  involved. On IBM Z, System Assist Processors run I/O operations and Integrated Firmware Processors handle firmware
  and PCIe functions; on z17 a Data Processing Unit on each processor chip takes over I/O-adapter functions. Each
  core has its own CPACF cryptographic unit. IBM describes the Z Sort accelerator as on-core on z16, and lists it
  among each core's integrated accelerators on z17; the compression accelerator (zEDC) and the AI accelerator are on
  each processor chip (IBM Redbooks, *IBM z16 Technical Introduction*, SG24-8950-01; *IBM z17 Technical
  Introduction*, SG24-8580-01). On z16 the AI accelerator serves the cores of its own chip; on z17 each AI
  accelerator accepts work from any core in the same drawer (IBM, "New Telum II Processor and IBM Spyre
  Accelerator", https://www.ibm.com/blog/announcement/telum-ii). x86 servers also move work off the cores, for
  example segmentation offload in network adapters (Linux kernel documentation, "Segmentation Offloads"), and on
  some parts the QAT, IAA and DSA accelerators (docs/platforms.md). Equal service does not remove this. It lists the
  uncounted processors, adapters and accelerators beside every result, on both sides.
- **Shared engines.** A shared engine's share of the physical processors depends on the partition's weight relative
  to the other active partitions and on any capping (IBM Support, "Introduction to Logical Partitions"), so counted
  runs use dedicated engines, or shared engines whose weight alone entitles them to the capped capacity (SPEC 8.2).
- **Per-core figures stay, as a normalisation.** Much software is licensed per core, but a core is not one licence
  unit: IBM's Processor Value Unit table rates a z17 ME1 core at 120 PVUs and a core of a two-socket Intel Xeon or of
  an AMD EPYC server at 70 (IBM, "Processor Value Units (PVUs)"). Much z/OS software is charged on MSUs instead (IBM
  z/OS 3.1, "What is Tailored Fit Pricing for IBM Z?"), and adding zIIPs or IFLs does not change a machine's MSU
  rating (SG24-8580-01). The FDR names each product's licence metric.

### Effect on the registered hypotheses
None is edited or withdrawn, and none has been scored. The choices below narrow the configurations that may score
them.

**Measurands, as registered.**
- "Throughput" in H1, H1j, H2 and H6 means **closed-loop capacity at zero think time** C (SPEC v0.1, sections 3 and
  4.2; the procedure is in docs/PLAYBOOK.md, section 6.1), per core in H1, H1j and H2, and per engine in H6. It is
  counted in **committed requests** (proposed SPEC 7.1): business rejections, duplicate-key rejections included, are
  reported separately and excluded. C is the highest committed throughput among the doubling steps whose p99 is at
  most L_cal (proposed SPEC 7.5), and its p99 is reported beside it. The sustained rate and the knee of proposed
  SPEC 7.4 are not used to score any of them.
- "Online capacity" in H3a and H3b is the same closed-loop capacity, with and without batch, at the same connection
  count.
- In H4, "100% offered load" means 100% of that closed-loop capacity C. "Within the next 10-20% of load" is read as
  "by 120% of C at the latest": the ratio scored is p99 at 120% of C over p99 at 100% of C, offered open loop
  (SPEC 4.2), and the ratio at 110% of C is reported beside it.
- H5 is scored with online work in closed loop at zero think time, at a fixed concurrency set by the core count
  (the H5 row below), not at the connection count that gave C.

**H6, read so that it can come out either way.** The registered text is: "enabling zIIPs will leave Java-track
throughput per engine within ±10%, while moving ≥ 80% of the CPU time off general CPs". It is read as follows, and
this reading was chosen before any kit run:
- "Per engine" counts general CPs and zIIPs together: closed-loop capacity divided by (CPs + zIIPs).
- The two configurations compared have the **same total engine count**: 2 CPs against 1 CP + 1 zIIP, and 4 CPs
  against 2 CPs + 2 zIIPs, each pair giving its own verdict. The CPs are full-capacity CPs, so that a zIIP and a CP
  run at the same speed.
- Claim 1 ("within ±10%") is measured closed loop, at capacity, in both configurations.
- Claim 2 ("≥ 80% of the CPU time off general CPs") is the zIIP share of the Java workload's CPU time (SMF type 30:
  zIIP time over CP plus zIIP time), measured in the configuration with zIIPs, **open loop at 25% of that
  configuration's C** (proposed), where the zIIPs are not saturated. The mean zIIP utilisation over the window must
  be at most 50% (proposed); if it is higher, the rate is halved once, and that is reported. The zIIP-eligible time
  that ran on CPs, and the zIIP-eligible share ((zIIP time + zIIP-eligible time on CPs) over CP plus zIIP time), are
  reported beside it and score nothing.
- Why this reading can come out either way is set out under "Notes on mechanisms and limits".

**Choices this entry makes.** Each was open in the registered text or in the spec of the day.

| Item | Registered text, or spec at registration | Chosen here | Not chosen |
| --- | --- | --- | --- |
| x86 comparator for H1, H1j, H2, H4 | v0.1 phase 1: "x86 PC on bare-metal Linux" | a current general-purpose x86 server part on bare metal (SPEC 2), named in the machine registry before any kit run on it; a part of another class named there gives its own labelled verdicts | the PC; a part chosen after calibration |
| Machines and multiplicity | not stated | every machine named in the machine registry before the first counted run of this entry; one Holm family over all named machine pairs | machines chosen later; one family per machine pair |
| Core count for "one LinuxONE IFL and one x86 core" | one of each | per-core figures from 2-core and 4-core runs | 1-core runs |
| Z machine and layer for H1 | "LinuxONE IFL" | IFLs of a LinuxONE or an IBM Z system, Linux directly in an LPAR (this widens "LinuxONE") | LinuxONE only; z/VM or KVM guests |
| SMT | "SMT state is recorded" | off on x86; IFLs and zIIPs in single-thread mode | on, or each side's default |
| x86 frequency | not stated | turbo on, the counted cores capped at the part's measured sustained all-core frequency (SPEC 8.2); turbo uncapped and turbo off as labelled variants | turbo off at base frequency; turbo uncapped |
| Neighbour load | not stated | loaded on both sides, at the defined background level (SPEC 8.2); quiet as a labelled variant on both sides | x86 quiet only; Z neighbours uncontrolled |
| Throughput | "transactions per second" | committed requests; rejections reported separately; "unequal collision rate" labelled (SPEC 7.1) | completed requests, rejections included |
| Build | "the same compiler (GnuCOBOL)" | the build policy of SPEC 1.0: code generated for the measured machine on both sides, native binary byte order, the same file handler and GMP | each distribution's default target; big-endian binary fields on x86 |
| Scale factor | not stated | the memory-resident scale factor of SPEC 1.5 | the cache-resident one |
| Request stream | not stated | the request stream and loaded data of SPEC 1.5 (Zipf 0.99 key access, proposed) | uniform key access |
| Data variant | file stores (spec v0.1) | D-file, "no atomic unit of work", durability class **durable** on both sides; a side that fails the durability acceptance test (SPEC 4.15) gives no verdict | the other label; "not durable"; a class chosen per series; D-live |
| "Default priority" (H3a, H5) | "default priority" | L0: online and batch in one task group | the distribution's default placement (run as L0d and reported, unregistered) |
| Concurrency for H3a, H3b | not stated | online at the connection count that gave C alone; batch in one stream per counted core (proposed) | one batch job at a time, whose single thread bounds the online fraction below by about 1 − 1/c on c cores |
| Concurrency for H5 | not stated | online at one connection per counted core (proposed), with batch as for H3a | the connection count that gave C, which would set the batch slowdown by the count found |
| H3b weights | "cgroups `cpu.weight` favouring online work" | online 1000, batch 100 | other weights |
| H3b "batch elapsed time will grow" | relative to what is not stated | L2 over L0 | L2 over batch alone |
| H4 "within the next 10-20% of load" | 110% or 120% | by 120% of C (110% reported) | 110% of C; either point |
| H4 window and timeout | not stated | W = 600 s and T = 10 s, fixed here | set in a run plan |
| H2 "plain" and the accelerated paths | not stated | plain: ISA-specific paths masked on both sides; accelerated: on each side, the path chosen by a fixed rule from separate selection runs, with compression size-matched | an x86 accelerator named in a run plan; the fastest path picked from the scored runs |
| H6 "per engine" | zIIPs added, or zIIPs in place of CPs | the same total engine count | zIIPs added to a fixed CP count |
| H6 "≥ 80% of the CPU time off general CPs" | measuring condition not stated | below zIIP saturation, at 25% of C | at closed-loop capacity (where, in these equal-count pairs, the zIIP share cannot exceed 0.5) |
| H1 claim 2, H3a claim 2 | "more than run-to-run noise"; "roughly constant" | interval rules below | point estimates |
| Decision rule | none | the whole interval inside the region, with the Holm correction, after a pilot and a power analysis | the point estimate |
| Boots or IPLs per point | not stated | at least 3 | 2 |
| One verdict or one per platform | H3a "on both architectures"; H4 "on both platforms" | one verdict that needs both for H3a and H4; one verdict per platform for H3b and H5 | one verdict per platform for H3a and H4; one verdict needing both for H3b and H5 |

**Scoring configurations, fixed before any kit run.** Only runs in the configuration named here score a
hypothesis. Results from any other track, profile, data variant, scale factor, core count, tier, mode or machine
are reported as unregistered results, and are never used to score, or to re-score, a registered hypothesis
(proposed SPEC 4.7).

Common to all, unless a row says otherwise:
- single box; profile P0; data variant D-file, label "no atomic unit of work", durability class durable on both
  sides (SPEC 1.8.4; the "Data variant" row above); accelerated extension off; the transaction mix of SPEC 1.2; the
  request stream and the memory-resident scale factor of SPEC 1.5;
- throughput counted in committed requests (SPEC 7.1);
- x86: a current general-purpose server part on bare metal (SPEC 2), named in the machine registry before any kit
  run on it, calibration included; SMT off; turbo on, with the counted cores capped at the part's sustained
  all-core frequency (SPEC 8.2); the counted cores in one NUMA node, in a verified cpuset;
- the loaded neighbour state on both sides (SPEC 8.2), with the background type reported with every verdict;
- Linux on Z: Linux directly in an LPAR, with exactly the counted IFLs online, in single-thread mode;
- Z engines: dedicated, or shared under the capping and entitlement rule of SPEC 8.2; no System Recovery Boost
  period active; no temporary capacity change during a series; Intelligent Resource Director weight and vary CPU
  management off (SPEC 8.2);
- the build policy of SPEC 1.0, the same on both sides;
- the repeats of SPEC 4.1: at least 6 runs per point, spread evenly over at least three boots or IPLs, the same on
  both sides, at the repeat count fixed by the power analysis below;
- a measured window W of 600 s and a timeout T of 10 s (proposed) for every scored run, closed loop and open loop.
  For H3a, H3b and H5, the window is instead the smallest whole number of batch cycles that lasts at least 600 s,
  the same on both sides (SPEC 4.3), and the cycle length is reported;
- per-transaction monitoring off in scored runs, on both sides (docs/CPU-ACCOUNTING.md, section 5);
- **machines named before the first counted run.** Every machine that scores this entry is named in the machine
  registry (REGISTRY.md) before the first counted run of this entry, and before any kit run on that machine: for
  IBM Z, the machine type and model, the capacity setting, the number of CPC drawers and the engine mode; for x86,
  the part, its class and the server model. Each named Z machine is paired with each named x86 part. A machine named
  later scores nothing under this entry. A named machine that is not run is reported as "named, not run", with the
  reason, and its verdicts enter the Holm procedure at p = 1. Where both Intel and AMD offer a current
  general-purpose server part and access allows, one of each is named. Every named machine's results are reported;
  none is left out.

| Hypothesis | Scoring configuration | Measurand and verdict rule |
| --- | --- | --- |
| H1 (both claims) | COBOL track; Linux on IFLs, against x86 Linux; 2 cores or IFLs on each side, and separately 4 | Ratio of closed-loop capacity per IFL to per x86 core. Claim 1 (within a factor of 2): VERIFIED if the interval of the ratio lies within [0.5, 2]. Claim 2 (differs by workload class): each class's ratio is the ratio of closed-loop capacities per core for a mix made only of that class's transaction types, in their mix-table proportions (SPEC 1.2). For each of the q pairs of non-empty classes, the interval of the ratio of their ratios is computed at level 1 − α/q, where α is the claim's Holm-adjusted level (a Bonferroni split within the verdict). VERIFIED if, for at least one pair, that interval excludes 1. Claim 2 is one verdict per core count and machine pair; its p is q times the smallest pair p, capped at 1. Each core count gives its own verdicts. |
| H1j | As H1, Java track, the same IBM Semeru / OpenJ9 release on both sides | VERIFIED if the interval of (Java-track ratio minus COBOL-track ratio), for the same transaction set and core count, lies within [-0.15, 0.15]. |
| H2 | As H1, with the signing step on alone, and separately the compression step on alone, in accelerated mode on both sides, against the same step in plain mode on both sides. **Plain** on both sides: the same portable builds of zlib and OpenSSL with every ISA-specific path masked (zlib built without DFLTCC and without x86 SIMD paths; OpenSSL with `OPENSSL_s390xcap` or `OPENSSL_ia32cap` set to disable the hardware crypto and vector paths; the mask values recorded), compression at zlib level 6. **Accelerated** on each side: every available path is run and reported, and the path scored is chosen on each side by the rule of SPEC 1.4 (the highest mean closed-loop capacity in separate selection runs, which are never reused for scoring). On Z: for compression, DFLTCC through zlib and through zlib-ng, with DFLTCC enabled for the levels used and its level setting recorded; for signing, OpenSSL with CPACF (KDSA for ECDSA). On x86: for compression, QAT and IAA where the part has enabled devices, and zlib-ng with its x86 SIMD paths; for signing, QAT where the part has enabled devices, and OpenSSL with its default x86 paths. Compression output is size-matched as in SPEC 1.4, and both pairings are reported; signing with clear keys in application memory on both sides. That each path ran, or did not, is shown by counters (SPEC 1.4). | Per step: VERIFIED if the interval of (accelerated ratio / plain ratio) lies above 1. The size of the shift is reported per step. AI inference: no verdict, as registered ("open"). |
| H3a | COBOL track; Linux on Z (in an LPAR) and x86 Linux, each measured separately; c = 2 cores or IFLs, and separately c = 4; the batch suite of SPEC 1.3 on the same cores; priority variant L0 (docs/PLAYBOOK.md, 11.3); online work at the connection count that gave C alone on that platform and core count, one worker per connection; batch in c streams (one per counted core, proposed), the suite's data split among them by account key range, each stream running the jobs one at a time in the order of SPEC 1.3, in one process, with GNU sort at `--parallel=1`; the runnable-thread counts of online and batch reported with every verdict | Online capacity with batch divided by online capacity alone. Claim 1: VERIFIED if, on each architecture, its interval lies within [0.45, 0.65] (one verdict, which needs both). Claim 2 ("roughly constant across transaction mixes"): VERIFIED if, on each architecture and for every mix in the table, the interval of (that mix's fraction minus the mean fraction over all mixes), at a level Bonferroni-adjusted over the mixes within the claim's Holm level, lies within [−0.05, 0.05] (proposed) (one verdict, which needs both architectures). The range across mixes is reported beside it and is not used for the verdict. |
| H3b | As H3a, with priority variant L2: online and batch in separate cgroups, `cpu.weight` 1000 for online and 100 for batch (proposed) | One verdict per platform, independent of the other: VERIFIED if the lower confidence bound of online capacity with batch over alone is at least 0.85, and the interval of batch elapsed time under L2 over batch elapsed time under L0 lies above 1. |
| H3c | None | No prediction, as registered. |
| H4 | As H1; open loop at 100%, 110% and 120% of C, with W = 600 s and T = 10 s (proposed), fixed here, the same on both sides | p99 with every timeout, and every refusal at these points, recorded at T (SPEC 7.1). VERIFIED if, on each platform, the interval of p99 at 120% of C over p99 at 100% of C lies above 10 (one verdict, which needs both platforms). If the lower confidence bound of p99 at 100% of C is at or above T/10 on a platform, the ratio cannot exceed 10 there: the verdict is DIFFERS, and the report says "bounded by T". The ratio at 110% of C is reported beside it. The curve shapes are reported, not ranked. |
| H5 | As H3a (L0, c streams of batch), with online work in closed loop at zero think time at c connections (one per counted core, proposed) | One verdict per Linux platform: VERIFIED if the interval of batch-suite elapsed time with online work over elapsed time alone (the same c streams, from the first start to the last end) lies within [1.2, 2.0]. |
| H6 | z/OS, Java track (IBM Semeru Runtime Certified Edition for z/OS, at the Java feature release named in the later entry that names the keyed store), P0, the Java track's keyed store (named in that entry), extension off; full-capacity CPs; IEAOPTxx `IIPHONORPRIORITY=YES` (the default), with the IEAOPTxx member in force and the HiperDispatch state recorded and fixed for the series; zIIPs in single-thread mode; (2 CP, 0 zIIP) against (1 CP, 1 zIIP), and separately (4, 0) against (2, 2) | One verdict per pair. VERIFIED if the interval of per-engine capacity with zIIPs over per-engine capacity without lies within [0.90, 1.10] (closed loop), and the lower confidence bound of the zIIP share of CPU time, measured open loop at 25% of C in the configuration with zIIPs as set out above, is at least 0.80. |

**Statistics and decision rule, for every verdict.**
- **The unit** is one run (one repeat of a point). Each point has the repeat count of SPEC 4.1, as fixed by the
  power analysis below.
- **Intervals.** For a ratio of two means: Fieller's method. For a difference of two means: Welch's two-sample
  interval. For a ratio of ratios (H1 claim 2, H2), a difference of ratios (H1j), a deviation from the mean across
  mixes (H3a claim 2) and any other derived quantity: a percentile bootstrap over the runs, with each side's and each
  configuration's runs resampled independently, 10,000 resamples (proposed), and the bootstrap seed fixed in the
  run plan.
- **Boots and IPLs.** The runs of a point are spread evenly over at least three boots or IPLs (proposed; two runs
  per boot at the default of 6 repeats). The between-boot and within-boot variance components are reported. If the
  between-boot mean square exceeds the within-boot mean square, the interval is also computed from the per-boot
  means (Welch or Fieller on the boot means, with b − 1 degrees of freedom per side for b boots, or, for derived
  quantities, the percentile bootstrap over boots), and the wider of the two intervals is used.
- **The decision.** A claim is VERIFIED only when the whole interval, at the claim's Holm-adjusted level, lies in
  the predicted region. Otherwise it is DIFFERS. Every DIFFERS is quoted, in the same sentence wherever it appears,
  with its qualifier, "interval wholly outside the predicted region" or "interval straddles the region boundary",
  and with the power statement of the power analysis below (the smallest distance from the boundary that the claim
  resolves with 80% probability at the repeat count used).
- **Multiplicity (Holm).** The family is every verdict this entry defines, over every machine pair named in the
  machine registry before the first counted run of this entry, counted per machine pair, core count, platform, step
  and configuration pair; its size m is fixed when that first counted run starts, whatever the number of FDRs that
  report the verdicts. For each verdict, p is the smallest α for which the two-sided (1 − α) interval lies wholly in
  the predicted region (p = 1 if none does), except for H1 claim 2, whose p is set out in its row. With the p-values
  sorted in ascending order, the k-th verdict is VERIFIED only if p_(j) ≤ 0.05 / (m − j + 1) for every j ≤ k. A
  verdict not yet computed when an FDR is published enters the procedure at p = 1, and a published verdict is never
  revised by later ones. A hypothesis is summarised only as the number of named machine pairs VERIFIED out of all
  named pairs, never by one pair's verdict alone.
- **What is reported.** The point estimate, the 95% interval, the adjusted level, N, the variance components and
  the power statement are always reported with the verdict.

**Pilot and power, before the tag.** Every other choice in this entry is first committed and registered at a
preliminary tag, `omb-v0.4.0-choices` (REGISTRY.md, section 2). Then, on the named machines, a pre-registration
pilot of at least 6 runs over at least 3 boots or IPLs (proposed) is run at one point of each measurand on each side.
It is entered in the run registry as kind "pilot", scores nothing, and is published with this entry. From it, this
entry records at its tag, for every claim: the Holm-adjusted level planned for it, the repeat count, and the
smallest distance from the predicted region's boundary that the claim resolves with 80% probability (proposed) at
that count. Where that distance is wider than half the predicted region (for a region bounded on one side, wider
than 10% of the boundary value, proposed), the repeat count is raised until it is not, or the claim is marked "not
resolvable at this repeat count" before any scored run; a marked claim is still scored, and its verdict carries the
mark. After the pilot, only repeat counts and boot counts may change. The pilot has not been run.

**Fixed now, applied when the mix table exists.** The transaction mix table (SPEC 1.2, milestone M1b) does not exist
yet. The rules below are fixed here. The classes, mixes, request-stream values and keyed store they produce are
recorded in a later entry on this page, at its own tag, before the first kit run on IBM Z or LinuxONE of any series
that could score a registered hypothesis, the pre-registration pilot and calibration included.
- **H1 workload classes.** Each transaction type is classed by a property measured on x86 alone, before any
  cross-platform kit run: its CPU time per transaction divided by its response time, closed loop with one
  connection. CPU-bound if the ratio is at least 0.8, I/O-bound if at most 0.5, mixed otherwise (proposed). If fewer
  than two classes are non-empty, claim 2 receives no verdict, the report says why, and it stays OPEN.
- **H3a mixes.** Every mix in the table is scored; none is left out.
- **The request stream** of SPEC 1.5 at the values fixed there.
- **H6's keyed store** on z/OS, and whether its I/O runs in Java or through native code, which changes the
  zIIP-eligible share; and the Java feature release used for H6.

### Readings of three registered sentences
Three registered sentences state an explanation or an interpretation rather than a measurable prediction. They are
not edited. Each is read as follows:
- H3a: "The fraction will be **roughly constant across transaction mixes**, because the scheduler shares CPU per
  runnable thread, not per transaction cost." The clause after "because" is the expected explanation, not part of
  the scored claim. A fraction that varies with the mix counts against that explanation.
- H1j: "If the gap is larger, the compiler (GnuCOBOL vs JIT), not the hardware, is driving H1." This is how a
  DIFFERS verdict for H1j will be interpreted. It is not scored separately.
- H6: "That is a cost effect, not a speed effect." This interprets the two H6 predictions taken together (which
  engine type is charged, not how fast the work runs). It is not scored separately.

### Notes on mechanisms and limits
- **H3a and the Linux scheduler.** Under group scheduling, a nice value acts only among threads in the same task
  group, and a new autogroup is created for each new session (Linux man-pages, sched(7)). L0 puts online and batch
  in one group, which is the condition the registered mechanism describes. It is not the distribution default: a
  systemd host places services in separate cgroups. That placement is also run, as "L0d, distribution default", and
  reported as an unregistered result.
- **H3a, H5 and concurrency.** Under the registered mechanism, the online fraction is about n / (n + b) for n
  runnable online workers and b runnable batch threads. With b = 1 on c cores, the fraction could not fall below
  about 1 − 1/c (0.75 at 4 cores), outside H3a's range on every platform. Batch therefore runs one stream per
  counted core, so that the range is reachable on every platform; n is the connection count that gave C, and is
  reported with every verdict, because the fraction depends on it. For H5, the online concurrency is fixed by the
  core count, not by the connection count the capacity search found, so that the batch slowdown is not set by that
  count.
- **H1 and the partition layer.** Linux directly in an LPAR is chosen so that neither side has a hypervisor layer
  the other lacks. Deployments under z/VM or KVM are run as labelled variants, and an H1 verdict describes the LPAR
  configuration only.
- **H4 and the load model.** At 100% of C the open-loop queue is at the stability limit, so p99 there depends on the
  window length. Above it, the queue grows for the whole window, and with latency timed from the intended send time
  (proposed SPEC 7.1), p99 reaches T. The ratio scored is therefore bounded above by T / p99 at 100% of C. With
  T = 10 s, H4 is close to a test of whether p99 at 100% of C stays below 1 s, and its verdict depends on T and W as
  much as on the platform. That is why both are fixed here, before any kit calibration run, and not in a run plan.
  The verdict carries little information about differences between the platforms; the curve shapes, reported for
  both, carry more.
- **Why H6, as read, can come out either way.** Let e be the zIIP-eligible fraction of the Java workload's CPU time.
  With the same total engine count and `IIPHONORPRIORITY=YES`, the CP can run zIIP-eligible work when the zIIP
  falls behind, so both engines can be kept busy only while the zIIP has enough eligible work: in (1 CP, 1 zIIP),
  per-engine capacity relative to (2 CP) is about min(1, 1 / (2(1 − e))), which is within 10% only when e is about
  0.45 or more. Below saturation, the zIIP share of CPU time is close to e itself, so claim 2 needs e of about 0.8
  or more. e depends on how much of the Java track's path (the transport, the keyed store's I/O, and the z/OS
  services it calls) is zIIP-eligible, and is not known before the runs. Both claims can therefore pass or fail.
  The model assumes that CPs help at once; when they do in practice depends on the system's settings, which is why
  the IEAOPTxx member and the HiperDispatch state are recorded. The readings not chosen fail this test in these
  pairs: at closed-loop capacity, the zIIP share of CPU time is at most zIIPs / (CPs + zIIPs), which is 0.5 in both
  pairs, so claim 2 could not pass there. It could pass at capacity only with at least four zIIPs per CP, a
  configuration the registered text does not name.
- **H6 caveats.** zIIPs run at full speed while CPs may be sub-capacity, which is why full-capacity CPs are named.
  With `IIPHONORPRIORITY=YES`, standard CPs may run zIIP-eligible work when the zIIPs cannot keep up (IBM z/OS
  documentation, "Reviewing z/OS parameter settings"), so the zIIP-eligible time that ran on CPs is reported. zIIP
  multithreading changes how busy time relates to capacity, which is why single-thread mode is named. During a
  System Recovery Boost period, zIIPs may run general CP work, which would change both claims; no scored run may
  fall in one (SPEC 8.2). Native code called through JNI, such as the zDNN scoring path, may not be zIIP-eligible;
  the accelerated extension is off in the scoring configuration for that reason.

### Equal service and the other new results
- **Equal-service results: exploratory, with no prediction.** No prediction is made about which side needs fewer
  resources at any service target, or by how much. Equal-service results score no registered hypothesis, and none
  is a headline result. Any prediction about them is a later entry on this page, registered at its own tag before
  the first kit run that tests it; until then, every equal-service result is reported as exploratory.
- **Scale-out results, and results under P1 or D-live,** are likewise unregistered, with no prediction.

### Reporting, whatever the outcome
Every result that scores a registered hypothesis reports its verdicts (VERIFIED or DIFFERS) in the first section of
the FDR summary, before any equal-service, scale-out or other figure, whatever the outcome and whichever platform
it favours (proposed SPEC 4.12). Per-core results for H1, H1j, H2 and H6 are published whether or not an
equal-service series is run on the same hardware. No registered hypothesis is left out of a report because an
equal-service result points the other way.

### Timing
No kit run has been made; the kit is not yet published, no machine has been named, and the pre-registration pilot
has not been run. The x86 PC pilot that informed H3a is disclosed in HYPOTHESES.md (Notes). All registered
hypotheses remain OPEN.
