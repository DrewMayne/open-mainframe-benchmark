# Open Mainframe Benchmark: specification, v0.4.0 (draft, unreleased)

Status: draft for comment. Items marked **(proposed)** are not yet decided. A fact marked **(verify)** has not yet been checked against a source read for this specification, and every such mark is resolved, or the fact removed, before the version is released. In this repository, "SPEC n" means section n of this file, not the Standard Performance Evaluation Corporation.

Terms used throughout: a **scored run** is a run that scores a registered hypothesis, in the scoring configuration named for it (docs/HYPOTHESES-ADDENDUM.md); a **counted run** is a scored run, or a run that gives an equal-service or scale-out result; a **counted series** is a series of counted runs; a **pilot run** is a run made before a registration to estimate variance, which scores nothing (section 10). A **headline result** is defined in section 10.

Version numbers follow semantic versioning:
results always name the spec version and track they ran against.

Changes in v0.1.1: two implementation tracks (Java first, then COBOL) under one spec (1.0, 1.5).
Changes in v0.2.0: an optional analytics module derived from industry-standard benchmark material (1.6), and rules for using third-party benchmark materials (6).
Changes in v0.3.0: further optional modules derived from benchmark material, plus pricing and energy methods (1.7).
Changes in v0.3.1: labelling of derived work aligned with the benchmark owner's published policies; written permission requested before publishing derived results (6).
Changes in v0.3.2 (2026-11-25), corrections and a withdrawal: CardDemo on z/OS needs configuration changes to run batch and online together (1.1, 4.3); CPC power is shown on the HMC for older Z machines (1.7); IBM's capacity ratings are per processor model (3); Db2 (5); version labels made consistent. The optional modules derived from benchmark material, and the rules specific to them, are withdrawn until the benchmark owners' permission is in place (1.6, 1.7, 6). HYPOTHESES.md is unchanged.
Changes in v0.4.0 (draft, unreleased):
- Comparison basis: an exploratory equal-service comparison with no registered prediction, reported after the registered verdicts; per-core results kept as a disclosed normalisation (2, 3, 8); six preconditions before any headline result, including a pilot and a power analysis before registration (10).
- Pre-registration: an external timestamp from the archive's creation time, a machine registry, a run registry and a tuning log; scoring only in each hypothesis's named configuration (4.7, 4.13, 4.14).
- Platforms and builds: a current x86 server part named before any kit run on it, held at its sustained all-core frequency; the same neighbour load on both sides; vendor-optimised libraries in accelerated mode; a HotSpot variant of the Java track; a build policy; and what the COBOL track can and cannot show (1.0, 2, 8.2).
- Workload: transport equivalence and a declared CICS topology (1.1); z/OS-against-Linux B2 sorts labelled whole-stack (1.3); accelerator evidence and equal output quality (1.4); scale factors with a cache-residency rule, and a defined request stream and loaded data (1.5); a request profile with bookkeeping steps and TLS, and data variants with a live database in two labelled forms (1.8); the Db2 variant in scope, IMS and MQ later (5).
- Run rules: at least 6 repeats over at least three boots or IPLs (4.1); confinement in place of pinning, with Z partition rules (4.4); a manifest for every node and driver host (4.5); the load generator, refusals, equal guarantees, the order of reporting, the run registry, the tuning log, storage class and acceptance tests, the statistics plan, accounting validation and tuning parity (4.9-4.18).
- Methods: a load model with open-loop arrivals, a persistent connection pool, one timeout per series, throughput counted in committed requests, decision and past-saturation points, a per-repeat knee and default parameters (7); a single-writer scale-out tier with a declared availability design and one scaling-limit test for both sides (9); CPU counted on one basis on both sides, the PR/SM view for every Z partition, with workload characterisation (3; docs/CPU-ACCOUNTING.md); no MIPS equivalence (3).
- HYPOTHESES.md is unchanged. Entry 1 of docs/HYPOTHESES-ADDENDUM.md (tag omb-v0.4.0-prereg; registration status in docs/REGISTRY.md) records the change of comparison basis and names the scoring configurations.

## 1. Workload

### 1.0 Two tracks, one specification
The benchmark has two implementation tracks. They share the same transactions, data, batch suite, accelerated
extension, driver, metrics, run rules and FDR. Results always name their track.
- **Java track (built first).** The transaction and batch logic is written in Java and runs on the same JVM family
  (IBM Semeru / OpenJ9) on x86 Linux, Linux on Z and z/OS. The source and the JVM family are the same everywhere,
  so it answers the hardware question most cleanly; the JIT's code-generation back ends and the garbage collector's
  barrier implementations still differ by architecture, and the z/OS JVM is a separate certified build, so the GC
  policy in force is recorded on every side. On z/OS it also shows the effect of zIIP engines, which run
  zIIP-eligible work, such as Java code running in the JVM; native code it calls (for example zDNN through JNI) and
  many z/OS services may run on CPs. Which parts are eligible is shown by measurement: the zIIP time, the
  zIIP-eligible time on CPs and their split by address space (SMF type 30) are reported.
  Its accelerated steps use the standard Java APIs: `java.util.zip`, JCE with OpenJCEPlus (CPACF on Z), and ONNX
  Runtime or the zDNN-backed path for scoring. Whether `java.util.zip` reaches DFLTCC on Z depends on the zlib the
  runtime uses and how it was built, so it is shown by counters, not assumed (1.4).
  On x86 Linux, and on Linux on Z, the Java track is also run on an OpenJDK HotSpot build of the same Java feature
  release (on Linux on Z, for example Eclipse Temurin, which lists Linux on s390x as a supported platform [S53], at a
  feature release Temurin publishes for s390x), as a labelled variant, so that an x86 result is not tied to one JVM.
  The registered hypothesis H1j is
  scored on OpenJ9, as registered.
- **COBOL track.** The CardDemo-based application in COBOL (1.1). It answers the migration and rehosting question,
  because COBOL/CICS business applications are what those claims are about. On z/OS it uses IBM Enterprise COBOL
  under CICS. On Linux it uses GnuCOBOL. **Caveat, stated with every COBOL-track result:** GnuCOBOL compiles via C and
  is not the compiler production mainframes use, so Linux-on-Z COBOL figures don't represent Enterprise COBOL on z/OS.
  - **What the track measures is the toolchain as well as the hardware.** GnuCOBOL implements decimal arithmetic
    through the GNU MP library (or MPIR) and compiles through a C compiler; Enterprise COBOL compiles to z/Architecture
    instructions, and at ARCH(12) and later uses the vector packed-decimal facility. Business COBOL relies heavily on
    decimal arithmetic, so the two toolchains can differ by more than the hardware does. GCC has no packed-decimal
    type, and GnuCOBOL implements decimal arithmetic through GMP, so the generated code is not expected to use
    z/Architecture decimal instructions. A disassembly search of the build for the packed-decimal (AP, SP, MP, DP,
    ZAP, CP) and vector packed-decimal (VAP, VSP, VMP, VDP) opcodes is published with the build record. So every
    COBOL-track result on Linux on Z states: "This result does not exercise IBM Z's decimal hardware."
  - **It can show:** on Linux, the same build policy (below) on Linux on Z and on x86, which is a comparison of the
    hardware plus each platform's C code generation and its GMP build, whose hand-written assembly differs by
    architecture (H1); and on z/OS, Enterprise COBOL under CICS as production mainframes run it, compared with itself
    (for example, with and without batch) or in a labelled whole-stack comparison.
  - **It cannot show:** how Enterprise COBOL on z/OS compares with GnuCOBOL on x86 as a hardware comparison, since
    compiler, runtime and transaction manager all differ; or how a commercial COBOL compiler for x86 would perform.
    Those questions need the Java track, or a whole-stack label.
  - **Build policy (proposed), for both tracks and every library on the path.** Both sides build from the same source
    with the same GnuCOBOL release (and, for the Java track, the same Java feature release and JVM build family), the
    same C compiler family and major version, the same optimisation level and the same GnuCOBOL options. Code is
    generated for the measured machine on both sides (`-march` set to the Z machine's architecture level, and to the
    x86 part's microarchitecture; the C compiler's major version must support `-march` for both measured machines);
    each distribution's default build is run as a labelled variant. GnuCOBOL's
    `binary-byteorder` option, `big-endian` by default, is set to `native` on both sides, so that x86 does not
    byte-swap every binary field while s390x does not; the generated record files stay byte-identical (1.5), and the
    load step writes binary fields in each platform's native order. Before a build is measured, every program is
    checked for a REDEFINES, or a record layout, that reads a binary field as bytes; each one found is listed in
    PORTING.md, and the golden check must pass with native byte order on x86. The indexed-file handler (for example
    Berkeley DB, VBISAM or DISAM) and its version, and the GMP version, are the same on both sides. The same rule
    applies to the database, and to zlib or zlib-ng and OpenSSL in plain mode; accelerated builds follow 1.4. Every
    flag and library build is recorded (FDR 3).
  - Sources: GnuCOBOL 3.2, DEPENDENCIES file (https://sources.debian.org/src/gnucobol3/3.2-4/DEPENDENCIES); IBM
    Enterprise COBOL for z/OS 6.4, "ARCH" (https://www.ibm.com/docs/en/cobol-zos/6.4.0?topic=options-arch);
    GnuCOBOL 3.x, `config/default.conf` ("binary-byteorder: big-endian"; read on the 3.x branch, verify for the
    release used): https://sourceforge.net/p/gnucobol/code/HEAD/tree/branches/gnucobol-3.x/config/default.conf
- **Track equivalence:** both tracks must produce identical business results (account balances, posted transactions,
  statements) from the same seed. That is checked by a shared golden-output test.

### 1.1 Base application (proposed)
- **Core (COBOL track; the Java track reimplements the same transactions): [AWS CardDemo](https://github.com/aws-samples/aws-mainframe-modernization-carddemo)** (Apache 2.0). It is a COBOL credit-card management application: CICS online transactions over VSAM (KSDS with alternate indexes), plus a batch suite.
- **Why CardDemo:** it already exists, it is open, and it is used to test mainframe migration tooling, including performance. Results on it can be compared directly with migration claims.
- **On z/OS:** it installs as shipped (CICS, VSAM, JCL, DFSORT). Running its batch and online work together needs configuration changes, or changes to the batch programs if their file access is routed through CICS (see 4.3 and docs/PORTING-zOS.md).
- **On Linux (s390x and x86):** CICS and VSAM are replaced by a thin portable layer:
  - indexed files through GnuCOBOL file I/O, or, in the live-database variant, a client-server database (1.8.4);
  - a pool of long-lived COBOL workers driven by the load generator, in place of CICS;
  - a driver that bypasses the 3270 screens and calls the business logic with the same inputs.
- The port must not change business logic. Differences are listed in a PORTING.md with the reason for each.
- **Transport equivalence (proposed).** The driver speaks one wire protocol to every platform: the same request and response bytes (proposed: a fixed-layout binary message over TCP), with any character conversion done inside the system under test and counted; the same connection model (7.1); and the same TLS setting (1.8.1). On z/OS the receiving program is named (for example the CICS sockets listener and a child server program, for a binary protocol over TCP; or, where the wire protocol is HTTP, CICS web support or CICS Liberty). A transport whose code is zIIP-eligible (for example one that runs in Java) reports its zIIP time separately and is labelled "zIIP-eligible transport"; the transport class (native or Java) is the same on both sides of a comparison, or the comparison is labelled "unequal transport" (4.11).
- **CICS topology (proposed).** The FDR declares the number of CICS regions, MRO and function shipping, any file-owning region, program concurrency, Open Transaction Environment use, MXT and the TCB limits. A program is defined THREADSAFE only after its code has been reviewed and made threadsafe, because the definition states that the application logic is threadsafe and CICS does not make it so (IBM CICS TS 6.x, "Making applications threadsafe"); a program defined QUASIRENT always runs on the QR TCB. The review is published with the port. The z/OS baseline is the kit's configuration checklist for z/OS, which names the SMF record types and the RACF logging options in force, since z/OS has no distribution default for them. Data tables (CICS-maintained or user-maintained) and any other in-memory copy of the KSDS files are not used in counted runs unless the Linux side uses an equivalent in-memory copy of the same files; either way the choice is declared and labelled.
- **Linux port configuration (proposed).** The worker model (processes or threads, and whether the GnuCOBOL runtime is used from several threads), the indexed-file handler's cache, lock and sync settings, and the record locking between batch and online processes are declared and published with the port, and are the same on both Linux platforms.

### 1.2 Online transactions
- The CardDemo online set (account view and update, card list and update, transaction list, view and add, bill payment), weighted by a **transaction mix table** (to be defined in a later spec version, milestone M1b, with the weights published).
- Think time is zero in closed-loop mode. In open-loop mode, an arrival rate is offered (see 4.2 and section 7).

### 1.3 Batch suite
- **B1 posting:** transaction posting (CardDemo POSTTRAN), interest calculation (INTCALC) and statement creation (CREASTMT).
- **B2 extract → sort → merge** of the transaction history, at a stated record count and size. Sorts use the platform's normal utility: DFSORT on z/OS, GNU sort on Linux. Both are reported. A B2 comparison between z/OS and Linux is labelled "whole-stack comparison", because the sort program differs. On Linux, GNU sort runs with its buffer size (`-S`), `--parallel` and temporary directory fixed in the run plan, the same on every Linux platform; on z/OS, the DFSORT options in force are recorded, and whether the Z Sort accelerator was used is shown by DFSORT message ICE267I ("ZSORT ACCELERATOR PATH ... RSN=nnn", added by APAR PH03207).
- **B3 archive:** compress closed statements and transactions.

### 1.4 Accelerated extension (proposed, switchable)
These are optional steps added to the online path, each a single library call so that it can be switched between
"plain" and "accelerated" on every platform:
- **Sign** each bill payment, using ECDSA P-256 through OpenSSL. It uses CPACF/KDSA on Z (z15+), and software or available instructions on x86.
- **Compress** each statement or document attachment with zlib. It uses DFLTCC on Z (z15+), and software, zlib-ng or QAT on x86.
- **Score** each transaction with a small fraud model, at batch size 1. It uses zDNN or the IBM Deep Learning Compiler on Z (z16 and later: the on-chip IBM Integrated Accelerator for AI, which IBM also calls the AIU, here always "the on-chip AIU" to distinguish it from the IBM Spyre Accelerator adapter, used through the NNPA instruction), and ONNX Runtime CPU (AVX2, AVX-512 or AMX) on x86.

Results are always reported with and without the extension.
- **Plain and accelerated, defined per platform (proposed).** **Plain** is the same portable build on both sides, with every ISA-specific path masked (zlib built without DFLTCC and without x86 SIMD paths; OpenSSL with `OPENSSL_s390xcap` or `OPENSSL_ia32cap` set to disable the hardware crypto and vector paths; the mask values recorded). The exact mask strings and configure flags are fixed in the kit's configuration checklist. **Accelerated** uses each platform's hardware. On each side every available accelerated path is run and reported: on Z, for example DFLTCC through zlib and through zlib-ng, CPACF/KDSA, the AI accelerator through zDNN, and protected-key or Crypto Express paths where the key class allows (they pair only with the same class); on x86, QAT where the part has enabled devices (for compression and for signing), IAA for compression, zlib-ng with its SIMD paths, OpenSSL's default x86 paths, and AMX for scoring. The path used for any comparison is chosen on each side by a rule fixed in the run plan (proposed: the highest mean closed-loop capacity in separate selection runs, which are calibration runs and are never reused for scoring). Where a platform has no hardware for a step, the FDR says so.
- **Evidence that it ran.** Counters, not configuration: on Linux on Z, the CPU-measurement facility's crypto counters for CPACF and a DFLTCC call count from the library or a trace; on z/OS, SMF type 30 or type 113 data; for the Z Sort accelerator, DFSORT's messages or SMF data; on x86, QAT telemetry or device counters. In plain mode the same counters show no use (verify the counter names against the releases in use).
- **The same output quality.** Compression in plain mode uses zlib level 6 on both sides. In accelerated mode, DFLTCC is enabled for the levels used in the Z build, and its level setting is recorded; the compressed size of the fixed corpus is reported per side and per path. Where the two sides' selected accelerated paths give compressed sizes that differ by more than 5% (proposed), the side with the smaller output uses instead its fastest path and level whose output is at most 5% smaller than the other side's, so that the comparison is between equal work; both pairings are reported, and the size-matched one is used for any verdict. Signing names the key protection class on each side (a clear key in application memory, a CPACF protected key, or a secure key in an adapter) and pairs only the same class; clear keys are the default. Scoring names the numeric format on each side (for example FP32, BF16, DLFloat16 or INT8) and reports the model's accuracy on a fixed held-out set per side; a pairing across numeric formats is labelled.

### 1.5 Data
- **Storage per track (proposed):** the COBOL track uses GnuCOBOL indexed files on Linux and VSAM on z/OS. The Java track uses a portable keyed store, to be chosen in a later spec version. Both load from the same generated record files. In the live-database variant (1.8.4), both tracks keep the same data in a client-server database instead.
- A deterministic generator with a scale factor S (accounts = 10,000 × S, transactions per account stated).
- **Scale factors for results (proposed).** Every counted series runs at two scale factors, fixed in the run plan: a cache-resident S (proposed: S = 1), and a **memory-resident** S, the smallest S whose stored data, as loaded, is at least 8 times the largest cache level that the counted cores share on either side (on IBM Z, the drawer's virtual L4, which IBM gives as 2.8 GB per drawer on z17; on x86, the L3 that the counted cores share: the socket's L3, or a sub-NUMA cluster's, on Intel Xeon, and one core complex's L3, including any stacked cache, on AMD EPYC (verify against AMD's documentation for the part)). The scoring configurations use the memory-resident S (docs/HYPOTHESES-ADDENDUM.md). A result names its S and the ratio of stored data to that cache on each side; a cross-platform ratio reported at one S only is labelled "single scale factor". Memory per configuration follows 8.2.
- The generated record files are byte-identical on every platform; the load step may write binary fields in the platform's native byte order (1.0, build policy).
- **Request stream and loaded data (proposed; fixed before the registration tag).** The generator fixes, from the seed, the same on both sides: customers equal to accounts (proposed); the cards per account (proposed: 1, 2 or 3, with equal probability) and the transaction records per card at load (proposed: 20); the record size of every file as loaded (CardDemo's fixed record lengths; verify each against its copybook); and, for each online operation, the key-access distribution (proposed: account and card keys drawn from a Zipf distribution with exponent 0.99 over a seeded random permutation of the keys; uniform as a labelled variant) and the request and response payload sizes (proposed: the fields of the operation's screen as CardDemo defines them, in the fixed-layout message of 1.1). These are part of the transaction mix table (1.2). Every FDR reports the measured fraction of requests that reached the 1% most-used keys.
- The same seed gives byte-identical data files on every platform. Golden checks compare decompressed bytes, because hardware and software deflate output differ.

### 1.6 Optional modules (in preparation)
Optional modules derived from industry-standard benchmark material are in preparation. They will be published
only once the benchmark owners' permission is in place; until then, none is described here.

### 1.7 Pricing and energy methods (proposed)
Methods, not workloads:
- **Pricing:** the optional pricing appendix of an FDR uses published list prices only, with every assumption
  stated (docs/FDR-TEMPLATE.md section 8). The open cost model (M7) builds on it.
- **Energy:** power is measured where it can be (RAPL on x86 we control; docs/methods/energy.md).
  On IBM z17 ME1 / LinuxONE Emperor 5 (machine type 9175), per-partition power is reported to the operating system
  (Linux `zpwr`, z/OS SMF 70.1 via APAR OA66018, the z/VM monitor on z/VM 7.3 with APAR VM66823 and on z/VM 7.4),
  but IBM describes it as apportioned rather than measured. For machine type 9176 (z17 ME2 and MER, LinuxONE
  Rockhopper 5) this is not verified. It is recorded in an optional FDR field labelled "vendor estimate; not
  measured" and kept out of per-watt comparisons. Older Z generations: no partition figure is visible to the
  operating system, and the FDR says so; CPC power is shown on the HMC to the machine's owner
  (docs/methods/energy.md).

### 1.8 Request profile and data variants (proposed)

#### 1.8.1 Profiles
Every result names its request profile:
- **P0, base:** the business transaction only (1.2), in one unit of work, without TLS.
- **P1, bookkeeping:** the work a public-facing service does on every request, around the same business
  transaction. Its steps are defined in 1.8.3. P1 runs with **TLS** on the client connections, terminated inside the
  system under test, so that its processors are counted (8.4): TLS 1.3, with the cipher suite
  TLS_AES_128_GCM_SHA256 and an ECDSA P-256 server certificate (proposed), the same on both sides. The TLS library
  and the crypto path on each side (for example System SSL or OpenSSL using CPACF on Z, OpenSSL using AES-NI on x86)
  are recorded.

P0 and P1 are reported side by side on the same hardware, so the cost of bookkeeping per request can be read
directly.

#### 1.8.2 Nothing is refused by design
- No component is configured to reject, shed or throttle a request during a measured run at a decision point
  (7.2): not the application, the load balancer, the database, the cache, the transaction manager or the operating
  system.
- Every limit that could refuse work is set so that it does not, and its value is recorded: connection limits and
  listen backlogs, queue and thread limits, transaction class limits, rate limits and maximum database connections
  (docs/PLAYBOOK.md, section 4.1, lists them per platform).
- Queuing is allowed, and shows up as latency.
- At every point, a refused request is counted as a timeout, recorded at T, and reported by component and cause. At
  a decision point, any refusal makes that step unsustained in that repeat (7.4); above the knee and at
  past-saturation points it is a recorded past-saturation outcome. A refusal never makes a run invalid.
- **A limit raised after a refusal** is entered in the tuning log (4.14) before the next run. If no decision-point
  run of the series has yet been made (the refusal came in the coarse pass, in a calibration step before the fine
  pass, or in the driver checks), the series continues. Otherwise the raised limit starts a new, labelled series
  (4.13), and the results under the earlier limit are still published in full.
- Business outcomes are not refusals. They are counted and reported by type:
  - the business rejections that the CardDemo programs themselves return, and permission denials produced by the
    business data (1.8.3, step 3);
  - **duplicate-key rejections.** CardDemo assigns a new transaction ID as the highest existing key plus one, with
    no serialisation before the write (docs/ARCHITECTURE.md, section 7). Under concurrent load two requests can
    compute the same ID, and the second gets a duplicate-key response. The business logic is never changed to
    prevent this (1.1); such rejections are business errors, counted and reported on both sides.
- A unit of work rolled back by the system for a deadlock, a lock timeout or a serialisation failure is not a
  refusal either. It is an error (7.1), counted by cause and operation, and it counts against the sustained-rate
  test (7.4). The application does not retry it unless the run plan says so, and then the same retry rule applies
  on both sides. The isolation level, the lock-timeout and deadlock-detection settings are recorded; because the
  isolation level decides whether a key collision appears as a duplicate-key response or as a serialisation
  failure, it is the same on both sides of a comparison (4.11).

#### 1.8.3 Bookkeeping steps (P1)
For every request, in this order:
1. **Authenticate:** validate the session token sent with the request (proposed: a 32-byte random token carrying
   an HMAC-SHA256 tag, checked with the same key length on both sides), and look up the session record. Sessions
   are created before the run and stay valid for its whole length; there is no sign-on during the run.
2. **Count:** update the requesting client's counters for fixed one-second and one-minute windows, and compare them
   with the client's limits. The window a request falls in is taken from its **intended send time** (7.1), which
   the driver sends with the request, never from the server's clock, so the counts depend only on the seeded
   request stream. When a limit is exceeded, record a "would throttle" event and carry on. The request is never
   rejected (1.8.2).
3. **Authorise:** check that the principal may perform the operation on the account, using permission data: a
   customer principal may act on accounts linked to it by the card cross-reference, and an operator principal may
   act on any account.
4. **Business transaction:** as in P0.
5. **Record:** write an **access record** for every request (principal, operation, key, time, outcome), and an
   **audit record** for every update (principal, operation, key, time, and the changed fields before and after).

Where each kind of bookkeeping data commits:
- Where the data variant has a unit of work (D-live, and D-file "atomic unit of work"), the business transaction
  (step 4) and its audit record commit in **one unit of work**. Under the D-file label "no atomic unit of work"
  there is no unit of work to share: the audit record is written immediately after the business update, on both
  sides, and the label is shown with every P1 result.
- The counters (step 2) are **not** part of the business unit of work. They live in a counter store, and each
  counter update is atomic on its own. The counter store's **durability class** is declared and is the same on both
  sides: **volatile** (counters may be lost when the store restarts) or **durable** (each update is on stable
  storage before the request continues).
- Access records may be buffered. The buffering and the flush interval are recorded, and the same rule applies on
  both sides.
- Sessions, counters, access records and audit records may be kept in any store that meets these rules. The store
  chosen on each platform is disclosed.

The data generator (1.5) also generates, from the same seed:
- the principals: one per customer, plus a stated number of operator principals;
- one session per principal;
- per-client limits, as counts per one-second and per one-minute window, generated for each offered rate in the run
  plan from the seeded stream at that rate, so that a stated fraction of clients (proposed: 1%) exceeds a limit at
  least once in the measured window. The limits for each rate are recorded;
- the request stream's principal for each request, including a stated, seeded fraction of requests that the
  permission data denies.

#### 1.8.4 Data variants
Every result names its data variant, and for the live-database variant, its form.
- **D-file, file-based:** the stores of 1.5 (GnuCOBOL indexed files on Linux, VSAM on z/OS, and the Java track's
  keyed store). A D-file result names one of two labels, the same on both sides:
  - **"no atomic unit of work":** z/OS files RECOVERY(NONE), as CardDemo ships them; Linux files as the store
    provides them;
  - **"atomic unit of work":** z/OS recoverable files; on Linux, a transaction layer in the data adapter that
    commits or backs out all of a request's file updates, which is described and disclosed.

  A D-file result also declares its **durability class**, the same on both sides of a comparison (4.11): **durable**
  (each update is on stable storage before the response is sent) or **not durable** (updates may be lost in an
  operating-system crash). The class is established by the acceptance test of 4.15, not assumed from the store's
  documentation; on Linux, an adapter that wants the durable class forces its writes (for example with `fsync` or
  `O_DSYNC`), and says how. "Not durable" keeps the data on the same storage class as the durable class; a store
  held only in memory (for example on tmpfs, or with no writes to storage) is labelled "memory only" and is never
  paired with either class. The scoring configurations use the durable class (docs/HYPOTHESES-ADDENDUM.md).
- **D-live, live database:** the data adapter keeps the application's data in a client-server database engine
  that runs outside the application process.
  - **Same durability on both sides.** The commit record is forced to stable storage before the commit returns:
    for PostgreSQL, `fsync=on`, `synchronous_commit` not `off`, and `full_page_writes=on`; for Db2, the log written
    to stable storage before the commit returns, with the settings that ensure it named in the FDR. The storage
    class, the write-cache protection of the log device and the number of log copies follow 4.15. A weaker setting
    is a labelled variant, never paired in a comparison with a durable one.
  - **Isolation for updates.** A record read for update stays locked until commit or rollback, so that a second
    updater waits or gets a conflict status and never sees a half-applied change. The isolation level and locking
    clauses used, and the replication mode, are recorded.
  - The logical schema is the same on every platform.
  - **Versions:** for each series, each database product is the newest release generally available on every
    platform of the comparison when the series' run plan is committed. In form A, the same version and fix level
    are used on both sides for the whole series. The version and fix level are recorded.
  - **Optimiser and storage state:** after every data load, statistics are collected at the same point on both
    sides (for example Db2 RUNSTATS, PostgreSQL ANALYZE), package bind options are recorded, automatic maintenance
    (for example PostgreSQL autovacuum) is set as recorded and the same in every run of a series, and no buffer
    pool is pre-loaded on either side: the warm-up does that.
  - **Publication:** no result from a licensed program, IBM's or another vendor's, is published until its licence
    terms have been checked for conditions on benchmarking or publishing results (section 10; docs/PLAYBOOK.md,
    section 1).
  - D-live has two labelled forms, and the FDR names the form:
    - **Form A, same product (platform comparison):** the same database product and version on both sides, so
      that only the platform differs: PostgreSQL on Linux on Z against PostgreSQL on x86 Linux; and Db2 for Linux,
      UNIX, and Windows on Linux on Z against the same product on x86 Linux. (Db2 for Linux, UNIX, and Windows is
      used on Linux on IBM Z only in a release whose IBM system requirements list Linux on IBM Z, and the FDR cites
      that requirements page.)
    - **Form B, whole stack (whole-stack comparison):** Db2 for z/OS on z/OS against Db2 for Linux, UNIX, and
      Windows on x86 Linux, or against PostgreSQL on x86 Linux. Db2 for z/OS and Db2 for Linux, UNIX, and Windows
      are separate products, with their own releases, settings, optimisers, locking and logging, so the Db2-to-Db2
      pairing across z/OS and Linux is a whole-stack comparison too. A form B result is labelled "whole-stack
      comparison": its differences include the database engine as well as the platform, and are never attributed
      to the hardware alone. This is a deliberate exception to holding the program constant (docs/prior-art.md,
      section 5, "Traps to avoid"); the label keeps such a result out of any hardware-only comparison.
  - PostgreSQL is used because it runs on both platforms and its licence places no condition on publishing
    benchmark results. It is not claimed to be the usual x86 counterpart of Db2 for z/OS.
- **Batch alongside online.** How batch and online share the data is declared and labelled on each side: on z/OS,
  VSAM record-level sharing (with DFSMS Transactional VSAM Services where the files are recoverable) or batch file
  access through the external CICS interface, labelled "batch through EXCI"; on Linux, the data adapter's record
  locking between batch and online processes, described with the port; in D-live, the database.

The business logic is the same in every variant and form; only the data adapter changes (docs/PORTING-zOS.md,
"three thin adapters"). The golden business-output check (1.0) passes on every variant and form.

CardDemo's own base application keeps its data in VSAM. It also ships optional features: a Transaction Type
Management feature that keeps transaction type reference data in Db2, a credit card authorization feature that uses
IMS, Db2 and MQ together, an account extraction feature that uses MQ and VSAM, and additional JCL utilities (FTP,
text-to-PDF conversion, Db2 and IMS DB load and unload, and the internal reader) (CardDemo README, "Optional
Features"). D-live moves the base application's data into the database through the data adapter. The Transaction
Type Management feature is not in the first online set. The authorization and account extraction features and the
additional JCL utilities stay out of scope with IMS and MQ (section 5).

#### 1.8.5 Golden check and invariant check
**The golden check** (1.0) runs the golden script serially, so its output is deterministic. Under P1 it also checks:
- the number of access records equals the number of requests in the script;
- the number of audit records equals the number of committed updates;
- the access and audit record contents match the golden output, with clock-derived fields masked;
- the numbers of "would throttle" events and permission denials equal the values the seed predicts.

**The invariant check** runs after every measured run, in either profile. A measured run is concurrent, so its
final state depends on commit order (for example, a bill payment pays the whole current balance), and an exact
final state cannot be predicted. The check is of invariants instead:
- **keys:** no file or table holds a duplicate key, and each committed transaction add or bill payment created
  exactly one transaction record, except a bill payment whose transaction write got a duplicate-key response, which
  created none (see "balances conserved"). Those are counted and listed;
- **no lost or duplicated update:** for each record, the committed updates to it form one chain from its
  generated value to its final value. In that chain, each update's **read value** (the value the business logic
  read in that unit of work before changing it) equals the previous committed update's after-value, and every
  committed update appears exactly once.
  - The values come from an independent, application-level **audit stream** that the system under test writes
    alongside its work: for every update, the data adapter writes the read value and the after-value, in the same
    unit of work as the update where the data variant has one, and immediately after it otherwise. In P1 this is
    the audit record (1.8.3); in P0 it is an update log in the same format. The stream is written in every measured
    run, the same on both sides, so its cost is part of the measured work; one point per series is also run with
    it off, and the difference is reported (docs/CPU-ACCOUNTING.md, section 5).
  - A data manager's own change log (for example PostgreSQL logical decoding, the Db2 log, or a CICS log) is a
    cross-check only. Its before-images are the stored value just before each write, so they always chain, and
    cannot show a lost update.
  - Responses logged by the driver are not enough, because a request that timed out at the driver may still have
    committed;
- **balances conserved:** for each account, the final balance equals the generated balance, minus the amounts of
  the transaction records created by committed bill payments, plus the changes made by committed account updates
  (read value to after-value, from the audit stream). One exception follows from CardDemo as shipped: in bill
  payment, a duplicate-key response on the transaction write does not stop the account update that follows
  (docs/ARCHITECTURE.md, section 7). A balance change on an account with such a response in the window is
  attributed to that response, counted and listed. Any other unexplained change fails the check;
- **counts:** the committed operations by type equal the system's own counts; the driver's completed, error,
  refusal and timeout counts add up to the requests sent; and any difference between the system's count and the
  driver's completed count (late responses, 7.1) is reported.

A run that fails the invariant check is invalid, and is kept and reported (FDR section 6).

## 2. Platforms
- **Phase 1:**
  - x86 PC on bare-metal Linux, and the same PC in a VM, for development and for within-platform results;
  - a current x86 server for any cross-platform ratio, on bare metal: a general-purpose server part (not one sold as
    frequency-optimised, core-dense or with stacked cache), the newest such part the contributor has access to whose
    general availability is within 24 months (proposed) of the Z machine's, named with the reason for the choice in
    the machine registry before any kit run on it (docs/REGISTRY.md); where both Intel and AMD offer such a part and
    access allows, one of each is named. A part of another class may also be named there, and gives its own
    verdicts, labelled with its class. If no qualifying part is available, the newest available part may still be
    run, but the pair is labelled "cross-generation" and scores no cross-platform hypothesis;
  - a LinuxONE / Linux on Z guest, **only where terms permit benchmarking and publishing**.
- **A fair x86 baseline.** A ratio between parts more than 24 months apart, or between a desktop part and a server
  part, is labelled "cross-generation" or "desktop part" and scores no cross-platform hypothesis; a ratio from a part
  of another class is labelled with its class. Every machine run in a scoring configuration gives its own verdicts,
  and all of them are reported. No cross-platform figure comes from a VM with less memory than the run plan
  requires. Libraries follow the build policy of 1.0. In plain mode, and in every run that scores H1, H1j or H3-H5,
  each library is the same portable build on both sides (1.4). In accelerated mode, each library is the platform's
  optimised build where one is published (for example DFLTCC-enabled zlib or zlib-ng on Z; zlib-ng with its SIMD
  paths, and QAT-backed compression and signing, on x86; OpenSSL built for each platform's crypto instructions). The
  build of every library is recorded (FDR 3).
- **Phase 2:**
  - an x86 server with QAT and AMX (cloud bare metal);
  - z/OS with CICS, VSAM, DFSORT and WLM, and Db2 for z/OS for the live-database variant (1.8.4);
  - dedicated or characterised LinuxONE capacity.
- **Comparison tiers.** Every result names its comparison tier: equal core counts on a single box, equal service, or the scale-out tier.
- **Equal core counts on a single box,** for the registered per-core hypotheses: such runs use equal counts of cores or IFLs (2 and 4 to start), and measure closed-loop capacity (4.2). SMT state is recorded, along with whether a "vCPU" is a core or a thread. The configurations that score each hypothesis are named in docs/HYPOTHESES-ADDENDUM.md.
- **Equal service (proposed; exploratory).** For a stated throughput within a p99 latency bound fixed before the runs, what each side needs to sustain it: cores or engines by type, memory, accelerators, power where measurable and cost where priced, all disclosed (section 8). No prediction about it is registered, it is not a headline result, and it is reported after the registered verdicts (4.12, 10).
- **Per core, as a disclosed normalisation.** Per-core figures from any tier are labelled as a normalisation. Much software is licensed per core, but a core is not one licence unit: IBM's Processor Value Unit table rates a z17 ME1 core at 120 PVUs, and a core of a two-socket Intel Xeon server or of an AMD EPYC server at 70.
- **The scale-out tier** (section 9) is its own labelled tier, reported apart from single-box results.
- **Why an equal-service comparison, beside equal cores.** It allows a cluster to be compared with a single system, and it reports a resource need at a stated service level, which is the question a capacity plan answers. Neither tier counts the work of processors outside the counted cores (on IBM Z, SAPs, IFPs, the z17 DPU and the on-core and on-chip accelerators; on x86, adapter offloads and QAT, IAA and DSA); those are listed beside every result instead (8.4, 8.5). A shared Z engine is not a fixed amount of hardware, so counted runs use dedicated or entitled engines (8.2). Much z/OS software is charged on MSUs, not per core, so the FDR states each product's licence metric. The reasoning, with its sources, is in docs/ARCHITECTURE.md, section 1.1.

## 3. Metrics
- **Capacity:** closed-loop throughput at zero think time (4.2), in committed transactions per second per core or engine (7.1), with the p99 at that point reported beside it. The registered hypotheses use it: per core in H1, H1j and H2, per engine in H6, as online capacity in H3a and H3b, and as the base of H4's load points (docs/HYPOTHESES-ADDENDUM.md).
- **Sustained throughput:** transactions per second at the transaction mix of 1.2, sustained within a stated p99 bound (7.4). The equal-service result is what each side needs for a stated sustained throughput (section 8).
- **Cost per transaction:** the busy processor time of the whole system under test during the measured window, by engine type, divided by the committed requests in that window (7.1), with CPU per completed request beside it (docs/CPU-ACCOUNTING.md).
  - **x86 Linux:** the busy time (user, nice, system, irq and softirq) of every CPU in the system under test's cpuset, from `/proc/stat`, plus the confinement residue of the other online CPUs (8.2), with the kernel's CPU-time accounting options recorded, because their defaults differ between s390 and x86. With SMT on, the time is in hardware-thread seconds and is labelled so.
  - **IBM Z, one basis for every partition:** the PR/SM view, whatever the operating system: the partition's effective dispatch time by engine type (on z/OS, the sum of SMF70EDT over the partition's logical processors of each type, in the PR/SM partition data of SMF type 70 subtype 1; verify the field names against the release in use), over intervals aligned with the window. For Linux in an LPAR, the same partition data, read through `hyptop` with global performance data authority or from the RMF partition data of a z/OS partition on the same CPC. The partition's LPAR management time (total minus effective dispatch time) is reported beside it, listed and not counted. The operating system's own view (MVS busy; `/proc/stat` busy) is reported beside it, with the capture ratio on z/OS. Before the first counted run on a machine, a zero-load check shows that the field tracks load (docs/CPU-ACCOUNTING.md, section 3).
  - **Guests:** the busy time of the processors the guest runs on, from the hypervisor's view.
  - **Coupling facility partitions:** the coupling facility processor busy time (SMF type 74 subtype 4).
  - **Several nodes or partitions:** the time is summed over every one in the system under test. Driver hosts are excluded and reported separately.
  - **Breakdown only:** per-process and per-cgroup figures on Linux; SMF 30 (per job) and SMF 110 (per transaction) on z/OS.
  - The basis is validated on each platform against a known load before it is used (4.17).
- **Workload characterisation (required for every counted series):** on IBM Z, the CPU-measurement facility counters (SMF type 113 on z/OS; the `cpum_cf` counter sets on Linux in an LPAR authorised for them), with the L1 miss rate, the relative nest intensity and the LSPR workload category they imply; on x86, instructions retired, cycles and last-level-cache misses per transaction (`perf`); on both, a breakdown of CPU per transaction into transport, data adapter, business logic, data manager and operating system. Where counters cannot be read (for example in a z/VM guest), the FDR says so and the result is labelled "not characterised" (docs/CPU-ACCOUNTING.md, section 7).
- **Latency:** p50, p90, p99 and p99.9 (HdrHistogram).
- **Batch:** elapsed time and CPU time per job, alone and alongside online work.
- **Interference:** online capacity and p99 with batch present, divided by the same without batch.
- **Steal time** and other-load indicators on shared platforms.
- **Normalisations,** each labelled as a normalisation and shown beside the result: per core or engine at the stated clock (with the licence metric of each product, because a core is not one licence unit); per watt where measurable (RAPL on x86; on Z machines before z17 no partition-level power figure is visible to the operating system, and on z17 ME1 the figure reported is apportioned by IBM, not measured, and for z17 ME2 and MER this is not verified, 1.7); and per dollar, optional, using published list prices with assumptions stated and the licence metric of each product named (where no published list price exists for a hardware configuration, on either side, no per-dollar figure is given for it). Per watt and per dollar appear beside the result only when both sides are measured or priced on the same basis (8.6). Where a per-core result scores a registered hypothesis, its verdict is reported first (4.12).
- **MIPS:** never derived from Linux runs, and no MIPS-to-x86-core equivalence is computed or published. IBM's published capacity ratings are per processor model (the LSPR multi-image table, which IBM uses as the basis of its processor capacity ratings, commonly called MIPS, and of its MSU ratings); for a specific LPAR configuration, IBM's LSPR FAQ points to its zPCR tool (LPAR Configuration Capacity Planning). The prior-art review advises against MIPS equivalence (docs/prior-art.md, section 5). A comparison with an estate model, if one is ever made, is a separate, labelled analysis in the open cost model (ROADMAP M7), with every assumption disclosed, and never a result of this benchmark.

## 4. Run rules
1. **Repeats:** at least 6 runs per point (proposed), in randomised order, with warm-up excluded. The runs are spread evenly over at least 3 boots of each x86 node and 3 IPLs or re-activations of each Z partition (proposed; two runs per boot at the default of 6 repeats), with the data reloaded each time; where the owner does not allow an IPL, over 3 restarts of every component of the system under test, labelled "restarts, not IPLs". The repeat count is fixed in the run plan, and is the same at every point of a series and on both sides of a comparison. Each open-loop repeat uses its own arrival seed (7.1). Report the mean and a 95% confidence interval over all runs, the between-boot and within-boot variance components, the median and the spread (4.16).
2. **Load modes:** closed loop for capacity. Open loop at 50, 70, 90, 100, 110 and 120% of measured capacity for the degradation curve. Open-loop load, timeouts, the sustained-rate test and the equal-service procedure follow sections 7 and 8.
3. **Batch alongside online:** the measured window must span whole batch cycles, or be synchronised to the cycle start. A window that samples only one phase of a batch job biases the result; we found this in our own pilot.
   On z/OS, CardDemo as shipped does not run batch and online together: its batch sequence first closes the files in CICS (CLOSEFIL), its main VSAM clusters are SHAREOPTIONS(2 3), all 18 programs are CONCURRENCY(QUASIRENT), and all 8 files are STRINGS(1) and RECOVERY(NONE). Running both together there needs a way for batch to share the files with CICS: either VSAM record-level sharing (RLS), which needs a coupling facility (and Transactional VSAM, a separately priced feature, if the files are made recoverable), or batch file access routed through CICS (the external CICS interface, EXCI); threadsafe program definitions or several CICS regions; and string and buffer tuning. These changes are part of the z/OS configuration and are disclosed with it (docs/PORTING-zOS.md).
4. **Confinement and partition rules,** for every counted run (a run that scores a registered hypothesis, or that gives an equal-service or scale-out result): on x86, the system under test runs in a cpuset that holds all its processes, in one NUMA node, with device interrupts and packet steering directed to the same cores, and the confinement is verified for every run (8.2); on Linux on Z, Linux runs directly in an LPAR with exactly the counted IFLs online, and its processes in a cpuset; on z/OS, the partition runs only the system under test (8.2); Z engines are dedicated, or shared under the capping and entitlement rule of 8.2, with no System Recovery Boost period active and Intelligent Resource Director management off. The neighbour state is the same on both sides (8.2). The frequency governor, the x86 frequency cap (the part's sustained all-core frequency for counted series, 8.2) and SMT are fixed and recorded.
5. **A manifest per run:** for every node or partition of the system under test and every driver host, the CPU model and machine type, core and thread counts, memory, OS and kernel, compiler and library versions and configuration switches; and the git commit of the kit. It also records the platform state that decides results: on x86, the firmware and CPU microcode versions, the power profile, the memory population per channel and its speed, the NUMA mode (for example sub-NUMA clustering or NPS); on every Linux node, x86 and Linux on Z, the state of the CPU vulnerability mitigations (`/sys/devices/system/cpu/vulnerabilities/`); on Z, the CPC's microcode driver and bundle level; on z/OS, the system parameter members in force (for example IEASYSxx, IEAOPTxx, LOADxx, IFAPRDxx and CLOCKxx), the maintenance level, the CICS system initialisation overrides and an extract of its resource definitions, an export of the WLM service definition, and the Db2 subsystem parameters, in machine-readable form.
6. **Raw output:** every raw file is kept and published with the results.
7. **Pre-registration:** a hypothesis file entry exists, committed, before the first run that tests it.
   - **External timestamp.** The entry is registered with a signed git tag on the default branch and a deposit of the tagged release in a public archive that issues a DOI (for example Zenodo, or OSF Registries). The registration timestamp is the archive's own creation time of that version, before the first kit run it affects; an archive's editable date fields are never used (docs/REGISTRY.md). A git commit date alone is not a timestamp. Every FDR that scores a hypothesis cites the tag and the DOI.
   - **Machines.** Every machine that scores a registered hypothesis is named in the machine registry before any kit run on it (docs/REGISTRY.md).
   - **Scoring configuration.** A registered hypothesis is scored only by runs in the scoring configuration named for it (docs/HYPOTHESES-ADDENDUM.md). Any other run is reported as an unregistered result, and never scores or re-scores it.
   - **Other load.** A run whose other load on the system cannot be established is labelled "other load unknown" and scores no registered hypothesis.
8. **Full Disclosure Report:** every published result comes with an FDR following [docs/FDR-TEMPLATE.md](docs/FDR-TEMPLATE.md). A result without one is not a result of this benchmark.
9. **Load generator:** open-loop load comes from driver hosts on separate hardware, shown not to be the bottleneck
   (7.3). Latency is measured from each request's intended send time (7.1).
10. **Refusals are failures:** at every point, a request that is refused, shed or throttled is recorded at T and
    counted as a timeout. At a decision point, any refusal makes the step unsustained in that repeat (7.4). Refusals
    are reported by component and cause, and a refusal never makes a run invalid. Every limit that could refuse
    work is set as in 1.8.2 and is part of the frozen configuration; a limit raised after a refusal follows 1.8.2.
11. **The same guarantees on both sides.** A comparison pairs only runs that match in all of the following. A
    pairing that differs in any of them is reported as two separate results, not as a comparison.
    - Request and data: request profile and TLS setting, data variant and label, database form, isolation level,
      durability setting or class (1.8.4), counter-store durability class (1.8.3), scale factor S and seed, request
      stream (1.5), transaction mix.
    - Platform: storage class (4.15), availability design (9.2), neighbour state (8.2), SMT state, frequency policy
      (on x86; IBM Z runs at its machine's fixed frequency), hypervisor layer (PR/SM logical partitioning counts as
      no hypervisor layer for this rule, and its LPAR management time is listed, 8.4; z/VM, KVM and any x86
      hypervisor do count as one).
    - Software: accelerated-extension mode, transport class (1.1), build policy (1.0), monitoring level.
    - Method: arrival process, connection pool (7.1), latency bound L, timeout T, warm-up rule, window length,
      repeat count.
12. **Registered results first:** an FDR that scores a registered hypothesis reports its verdicts (VERIFIED or
    DIFFERS) in the first section of its summary, before any equal-service, scale-out or other figure, whatever the
    outcome and whichever platform it favours. An FDR that scores none says so in the same place. No result is
    called a headline result until the preconditions of section 10 hold.
13. **Run registry and complete disclosure:** from the first kit run, every kit run started is entered in the public
    run registry (docs/REGISTRY.md) before it starts, including calibration, exploratory and tuning runs, and runs
    that fail or are abandoned. Entries are never deleted. Every series' run plan is committed and tagged before the
    series' first run. Every series ends in a published FDR, or in a published withdrawal notice that gives the
    reason and includes all raw data gathered. A run that is not in the registry is not a result of this benchmark.
    Validity is decided by the kit's committed validity checker, applied to every run's raw data, not by the
    operator; its version is recorded. An invalid run is re-run at most twice per point (proposed), and every
    invalid run is counted in the FDR, by side and by reason. A point still invalid after its second re-run is
    recorded as "invalid after re-runs", with its reasons; for the knee (7.4) and for a fixed target (8.3, 9.3), it
    counts as not sustained in that repeat, and the number of such points is reported with the result. A value of
    the run plan, or a limit of the frozen configuration, chosen after looking at decision-point data starts a new,
    labelled series (1.8.2); the results under the committed plan are still published in full. The registry records
    what is entered in it and cannot show that no other run was made, so FDR 9 states, for each machine in the
    result, whether any kit run on it is missing from the registry.
14. **Tuning log:** every change to the configuration of the system under test or the driver during a series is
    entered in the series' tuning log, with its time (UTC), what changed and why, before the first run it affects.
    The log is published with the FDR.
15. **Storage class and acceptance tests:** the log and data devices are declared on both sides, with their type
    (for example local NVMe, SAN over FCP, or DASD over FICON or zHyperLink), write-cache protection, replication and
    the number of log copies (for example Db2 dual active logs). A baseline of synchronous 4 KiB write latency (p50
    and p99) and throughput on the log device, measured with a declared tool, is published. The **storage class** is
    defined by durability semantics and measured commit latency, not by device type or interface:
    1. the write cache of the log and data devices is protected against power loss, or writes reach stable media
       before they are acknowledged;
    2. the same number of log copies;
    3. the same replication (none, synchronous or asynchronous);
    4. the commit latency class: a synchronous 4 KiB write p99 on the log device within a factor of 2 (proposed) of
       the other side's.

    Local NVMe, SAN over FCP and DASD over FICON or zHyperLink can therefore be in one class, and a z/OS-to-x86
    pairing can be a comparison. A pairing that differs in items 1-3 is labelled "storage-unequal" and reported as
    two results. A pairing that differs only in item 4 is paired, labelled "unequal commit latency", and both
    baselines are shown beside it. Before the first counted run of a series, on each side
    and in each data variant used, the kit's acceptance tests are run and their output published (FDR 6):
    atomicity (a unit of work aborted after its first update leaves no trace, where the variant has units of work),
    isolation (two concurrent updaters of one record: one waits or gets a conflict status), and durability (the
    system under test is stopped abruptly during load at the highest planned rate, by power-off, or by an immediate
    partition deactivation where power cannot be removed; after restart, every response the driver received as
    committed is present). A configuration that fails a test is not counted in that variant or class. Where the
    owner does not allow an abrupt stop, the durability test is not run, and the configuration is labelled
    "durability not tested" and is never paired with a durable one.
16. **Statistics plan:** every verdict follows the decision rule of docs/HYPOTHESES-ADDENDUM.md: interval methods
    fixed in advance (Fieller, Welch, and the percentile bootstrap for ratios of ratios, differences of ratios and
    other derived quantities), repeats over at least three boots or IPLs (4.1), and the Holm method across the
    family of verdicts. Wherever a rule needs to show that there is no material difference (the driver cross-check,
    the warm-up test, the scale-out "flat" test, the reproduction criterion of section 10), it uses an equivalence
    test (two one-sided tests at 5%, that is a 90% interval; for the scale-out "flat" test, a one-sided one) with a
    margin fixed in advance (7.5), never the absence of a significant difference.
17. **Accounting validation:** before the first counted run on each machine, and again after any change of kernel,
    z/OS level or CPC microcode, the kit's calibration load is run on every engine type of the system under test,
    at 25%, 50% and 80% (proposed) of one engine. The calibration load is a single-threaded CPU-bound loop whose CPU
    time per operation is first measured alone on one otherwise idle engine. The CPU per operation given by the
    reported basis (section 3; docs/CPU-ACCOUNTING.md) must match that time within 3% (proposed) at each rate. The
    capture ratio (z/OS) or the confinement residue (Linux) is reported with it. A machine that fails gives no
    counted result until the cause is found and recorded in the tuning log.
18. **Tuning parity:** each side of a comparison has a named configurer, with their affiliation (GOVERNANCE.md,
    "Independence"). Both sides start from the kit's published configuration checklist for their platform.
    "Untuned" means that checklist with every other setting at the product or distribution default; it is always
    run, and reported unless a licence term forbids it (the FDR then names the term and the configuration
    withheld). Tuning beyond the checklist has the same budget on both sides, fixed in the run plan (proposed: 40
    person-hours and 30 tuning runs per side per series), and every change is entered in the tuning log. Before the
    counted runs, each side's tuned configuration is offered for review to a performance specialist of that platform
    who is not its configurer, and the review, or the fact that none was obtained, is published in FDR 9. A side
    whose budget was exceeded is labelled "unequal tuning", and its result scores nothing.

## 5. Out of scope
- Audited or official benchmark status (TPC/SPEC). Results are research results. This kit is not a TPC benchmark, and results must not be presented as TPC results or described with TPC benchmark names. A separately sponsored, audited run of an actual TPC benchmark is listed as a stretch goal in ROADMAP.md.
- IMS and MQ variants of CardDemo (later, possibly). The Db2 variant is in scope as the live-database variant (1.8.4); CardDemo's optional components that need IMS or MQ stay out of scope with them.

## 6. Benchmark materials from third parties
To stay within other benchmark owners' terms:
1. **Download, never vendor.** Third-party benchmark kits and specification text are never committed to this
   repository.
2. **Permission first.** No module derived from industry-standard benchmark material, and no result from one, is
   published until the benchmark owner's written permission is in place.
3. **No borrowed names or metrics.** Our own workloads (the CardDemo-based online track and the batch suite) are
   never described with another benchmark's name, and other benchmarks' metric names are never used for our numbers.
4. **Academic framing only:** no marketing claims based on results derived from benchmark material.

## 7. Load model (proposed)
Section 4.2 defines closed-loop capacity and the open-loop degradation curve. This section defines how open-loop
load is generated and checked, how failed requests are counted, and when a configuration **sustains** an offered
rate. Sections 8 and 9 use those definitions. Closed-loop capacity (4.2) is unchanged, and it remains the measure
the registered per-core hypotheses use (docs/HYPOTHESES-ADDENDUM.md). The default value of every parameter is in
7.5.

### 7.1 Open-loop arrivals
- The driver offers a **fixed target rate** for the whole window. Arrivals are Poisson (exponential inter-arrival
  times) by default, or at constant, evenly spaced intervals as a stated variant.
- **Seeds.** Arrivals come from a seeded pseudo-random stream. Each open-loop repeat of a point uses its own arrival
  seed, derived from the run plan's base seed and the repeat number, and the same seeds are used on both sides of a
  comparison. The schedule is generated once from the seed and then split among the driver hosts by a stated rule
  (proposed: by request sequence number, round robin), so the arrivals are the same whatever the number of hosts.
- Each request has an **intended send time** from that schedule. Latency is measured from the intended send time,
  not from the moment the request was actually sent. A system that stalls therefore cannot hide its own delay from
  the measurement (coordinated omission).
- **Connections.** Before the warm-up, the driver opens a fixed pool of P persistent connections and keeps them open
  for the whole run. P is the same on both sides, fixed in the run plan and recorded (FDR F3).
  - **Size, by Little's law.** The requests in flight equal the arrival rate times the time each one takes. P is the
    smallest power of two at or above 2 × the highest offered rate of the series × L_cal (8.1), capped at P_max
    (proposed: 1,024 connections in all, split evenly over the driver hosts).
  - **Why the cap.** P_max fits every connection limit on the path with room to spare: on z/OS, the CICS sockets
    domain's MAXSOCKETS limit and, where the transport holds a CICS task for each connection, the region's maximum
    tasks (MXT) (verify both against the CICS release in use); on Linux, the listener's and the database's limits;
    and on each driver host, the ephemeral ports to one destination (the Linux default of `ip_local_port_range` is
    32768 to 60999 [S54]). A run plan that raises P_max shows that every one of these limits still fits it.
  - **No connection is opened or closed inside the measured window.** Under P1, every TLS handshake (P full
    handshakes, with no session resumption) is therefore made before the warm-up, and the measured window has no
    handshakes: the handshake count per measured request is zero. The handshakes made are reported.
  - A connection carries one outstanding request at a time, or several over a protocol that allows responses out of
    order (for example HTTP/2 streams). Requests are never pipelined in order on one connection, because one slow
    response would delay those behind it.
  - When every connection is busy, the request waits in the driver. That wait is part of its latency, because
    latency is timed from the intended send time, and it is attributed to the system under test (7.3). The number
    of requests waiting in the driver is recorded each second.
- **Timeouts.** The driver never retries. One timeout T applies to every run of a series, calibration included. It
  is fixed in the run plan before the first run of the series and is the same on both sides (proposed: T = 10 ×
  L_cal, 8.1; no latency bound L of the series may exceed T/10). A request with no response within T is a **failed
  request** (a timeout). It is recorded in the latency histogram at the value T (censored at T), so it raises every
  percentile it reaches, and it counts against the sustained-rate test (7.4). A response that arrives after T is
  logged as late, and the request stays a timeout.
- **Which window a request belongs to.** A request belongs to the measured window of its intended send time. Its
  outcome is taken when it completes or reaches T, even if that is after the window ends. The driver keeps offering
  load at the same rate until T after the window ends, so that the window's last requests meet the same load as the
  rest.
- **Outcomes.** Every request ends in one of four classes, counted separately:
  - **completed:** a response within T, including business outcomes (the business rejections that the CardDemo
    programs themselves return, permission denials of 1.8.3, and duplicate-key rejections, 1.8.2);
  - **error:** a response within T that reports a system failure: a unit of work rolled back by the system
    (deadlock, lock timeout, serialisation failure), a resource failure, or an abnormal end;
  - **refusal:** a request that a component refuses, sheds or throttles (for example a connection refused at a full
    listen backlog, or a transaction class or rate limit). A refusal is not an error. At every point it is recorded
    in the latency histogram at T, counted as a timeout, and reported by component and cause. A refusal never makes
    a run invalid. At a decision point it makes that step unsustained in that repeat (7.4); above the knee, and at
    past-saturation points, it is a recorded past-saturation outcome (7.2, 1.8.2, 4.10);
  - **timeout:** no response within T.
- **What throughput counts.** Capacity (4.2), the achieved rate and every per-core or per-engine figure count
  **committed requests**: completed requests whose business transaction committed its update, or returned its read,
  as intended. Business outcomes (the CardDemo programs' own rejections, permission denials and duplicate-key
  rejections) are completed requests, but not committed ones. They are excluded from every throughput figure and
  reported separately, per side, as a rate per 1,000 requests, by type. Offered rates and service targets are rates
  of requests offered, and the achieved committed rate is reported beside each. Where the two sides'
  duplicate-key rejection rates differ by more than 1 per 1,000 requests (proposed), the comparison is labelled
  "unequal collision rate", and its throughput ratio is also shown with rejections counted, beside the
  committed-only ratio.
- When the system falls behind, requests queue. Queuing time is part of the measured latency.

### 7.2 Rate sweep
- Before the first run of a series, the **latency bound** L (the p99 latency, per operation or overall, above which
  an offered rate counts as not sustained) and the timeout T are fixed and recorded. In an equal-service series the
  calibration uses the calibration bound L_cal, and the bounds used by the counted runs are derived as in 8.1.
- The driver offers a ladder of fixed rates, from low to high, each with its own warm-up and measured window:
  - a **coarse pass** in even steps, which is exploratory: one run per step, used only to find the bracket between
    the last sustained and the first unsustained step;
  - a **fine pass** in smaller steps across that bracket, with the full repeat count of the run plan at every step,
    in randomised order (4.1). Only fine-pass results decide the knee (7.4).
- The sweep continues past the knee to 120% of the closed-loop capacity C (4.2) of the same configuration, measured
  before the sweep, so behaviour past saturation is recorded.
- **Decision points and past-saturation points.** The open-loop points of 4.2 at 100%, 110% and 120% of C, and the
  coarse-pass steps above the first unsustained coarse step, are **past-saturation points**. Every other point is a
  **decision point**: every fine-pass step, whatever its rate relative to C (including a step at or above C after an
  upward extension), the 50%, 70% and 90% points of 4.2, the fixed targets of sections 8 and 9, and closed-loop
  runs.
- **Past saturation.** Past-saturation points are not steady state: the queue grows for the whole window, so their
  latency depends on the window length and on T. They are labelled "past saturation", run with the same window
  length and T on both sides, and reported with the queue growth (requests sent minus requests answered at the end
  of the window), the timeout count, the refusals by component and cause (each counted as a timeout at T), and the
  achieved rate. The run stays valid. Their latency figures are compared only between runs with the same window
  length and T.

### 7.3 The load generator
- **Separate hardware.** The driver never runs on the system under test, nor on a machine, hypervisor host or CPC
  that it shares.
- **Sized to the load.** The driver hosts together have enough capacity for the highest offered rate with headroom.
  Several driver hosts may be used; their latency histograms are merged, and their clocks are synchronised to one
  source.
- **Shown not to be the bottleneck:**
  1. a self-test against a null responder, on separate hardware, at twice the highest planned rate (proposed), in
     two modes: answering at once, and answering after a fixed delay d = P / (2 × the highest planned rate), at most
     0.9 × T, where P is the connection pool of 7.1, so that every connection is busy (Little's law); d is recorded.
     In both modes, the requests completed within T, the schedule lag (actual minus intended send time) at p99.9,
     and the busiest driver core or thread stay within the bounds of 7.5. The self-test is run, or run again, after
     the series' smallest L is fixed, and again whenever the driver hosts change;
  2. during every run, driver CPU per core or thread, driver network utilisation and schedule lag are recorded.
     Schedule lag counts against the driver only while a connection was writable or a new one could be opened. Lag
     that builds up while the send was blocked by TCP flow control, or while the system under test was not accepting
     connections, is attributed to the system under test and reported; it is already inside the measured latency,
     because latency is timed from the intended send time. A run that breaks a driver limit is invalid;
  3. once per series, after the series' targets and values of L are fixed, the highest decision-point rate the
     series offers is repeated, with the full repeat count, with the load spread over twice as many driver hosts.
     For a sweep, that rate is the fine-pass step one step below the knee of the largest configuration swept; in
     sections 8 and 9, it is the highest target R. The driver is shown not to be limiting when the 90% confidence
     intervals (Welch) for the difference in mean p99, and for the difference in mean achieved rate, lie within ±5%
     of L and ±0.5% of R (proposed margins; two one-sided tests at 5%):
     - if an interval contains 0 but crosses a margin, the cross-check is inconclusive; its own repeat count (not
       the series') is doubled once, and if it is still inconclusive the series is labelled "driver adequacy
       inconclusive";
     - only an interval that excludes 0 and lies outside a margin labels the series "driver not shown adequate". The
       driver is then enlarged and the series repeated; the first series stays in the run registry (4.13).
- **Same driver for both platforms** where possible, at the same network distance. Otherwise the idle round-trip
  time from driver to system under test is measured and reported for each side (see also 8.1).

### 7.4 Sustained rate and the knee
A configuration **sustains** an offered rate R **in a repeat** when, over that repeat's measured window (7.1), with
each request's outcome taken as in 7.1:
- every request whose intended send time falls in the window has an outcome (completed, error, refusal or timeout),
  and the counts add up to the requests scheduled;
- timeouts, refusals and errors together are at most 0.1% (proposed) of those requests (the **failure
  threshold**); and
- p99 latency, with every timeout and refusal recorded at T, is at or below L.

A refusal at a decision point counts as a timeout. Any refusal makes that rate unsustained in that repeat, and the
run stays valid (4.10). A point still invalid after its re-runs (4.13) counts as not sustained in that repeat.

The number of repeats per point is fixed in the run plan before the series. It is the same at every point of a
series and on both sides of a comparison (4.1).

**The knee, per repeat.** The fine pass runs every step once in each repeat r, with repeat r's arrival seed (7.1).
The knee of repeat r, K_r, is the highest fine-pass step R such that R and every lower fine-pass step are sustained
in repeat r. If the lowest fine-pass step is not sustained in some repeat, the fine pass is extended downwards. If,
in some repeat, K_r is the highest fine-pass step, it is extended upwards. Extension continues until every repeat's
knee lies strictly inside the fine pass, and stops at 50% of C downwards and at 120% of C upwards (proposed). A
repeat whose knee still lies outside is reported as "below 50% of C", or as at the last step, and the knee interval
uses those bounds.

**Non-monotone results.** A step above K_r that is sustained in repeat r is a **non-monotone pass**. It never raises
K_r. Every non-monotone pass is reported, with its step and repeat.

**What is reported.** The **knee** of the configuration is the lower median of the per-repeat knees, K_(⌈n/2⌉), so
that it is always a fine-pass step. Its confidence interval is built from the K_r sorted from lowest to highest, so
it assumes no distribution:
- with n repeats, the interval is [K_(k), K_(n+1-k)], where K_(k) is the k-th lowest value and k is the largest
  value whose exact binomial coverage is at least 95%;
- with 6 repeats (the default, 4.1), this is [min, max], with a nominal coverage of 96.9%. That coverage assumes
  independent repeats; repeats are nested in boots or IPLs (4.1), so the between-boot flag of the statistics plan
  (4.16) is reported with the knee interval.

Every K_r is listed. The achieved rate and p99 at the step of the knee are reported with their means and 95%
confidence intervals.

**Sustaining a fixed target.** Where a single rate R is offered with the run plan's repeats (sections 8 and 9), the
configuration **sustains R** when R is sustained in more than half of the repeats. The number of repeats that
passed is reported with the result.

### 7.5 Default parameters (proposed)
A counted series uses these values. A run plan that changes one is a labelled variant, and scores nothing. A value
given as a minimum ("at least") may be raised without making a variant, if the run plan fixes it before the series
and it is the same on both sides.

| Parameter | Default (proposed) |
| --- | --- |
| Calibration bound L_cal (8.1) | p99 of 1 s, over all operations |
| Timeout T, every run of a series | 10 × L_cal (10 s) |
| Latency bounds L (8.1) | 2×, 5× and 10× the higher of the two sides' mean p99 at 50% of the lower calibration knee, each at most L_cal; a side whose mean p99 at 5% of that knee exceeds 0.8 × L reports "latency floor above L" for that L |
| Measured window W | 600 s; with batch alongside, the smallest whole number of batch cycles that lasts at least 600 s, the same on both sides (4.3) |
| CPU-accounting interval | 1 minute on both sides (on z/OS, the RMF Monitor I interval, synchronised to SMF) |
| Warm-up | at least 600 s and at most 1,800 s: it ends at the first interval boundary at which the slope test of docs/PLAYBOOK.md section 8 passes (ten 1-minute intervals; mean latency and CPU per transaction; 90% interval of each drift within ±10% of its mean); a run whose warm-up has not passed by 1,800 s is invalid |
| Repeats per point (4.1) | at least 6 (6 by default), spread evenly over at least 3 boots or IPLs |
| Failure threshold (7.4) | timeouts, refusals and errors at most 0.1% of the requests in the window |
| Fixed target sustained | in more than half of the repeats |
| Coarse and fine steps | 10% and 2% of C; fine-pass extension bounded at 50% and 120% of C |
| Closed-loop search (4.2) | stop when two doublings each raise committed throughput by less than 2%; C is the highest throughput among the steps whose p99 is at most L_cal |
| Connection pool P (7.1) | the smallest power of two at or above 2 × the highest offered rate × L_cal, at most P_max = 1,024 connections in all |
| Driver sizing (7.3) | twice the highest planned rate |
| Null-responder self-test | twice the highest planned rate; delay P / (2 × the highest planned rate), at most 0.9 × T |
| Driver limits | busiest core or thread 70%; network 50% of link speed; schedule lag p99.9 at most 1% of L and at most 1 ms; completions in the self-test within 0.1% of those scheduled |
| Driver cross-check margins | ±5% of L (p99); ±0.5% of R (achieved rate); 90% intervals (two one-sided tests at 5%) |
| Confinement bound (8.2) | residue at most 1% of the counted cores' busy time |
| Calibration size (8.1) | 4 cores or engines on each side |
| Service targets (8.1) | 25%, 50% and 80% of the lower calibration knee |
| Network-distance rule (8.1) | 10% of the smallest L |
| Shared Z pool, other load (8.2) | varies by at most 10 percentage points of the pool across the repeats of a series |
| Neighbour load, loaded state (8.2) | the neighbour processors at 80% mean utilisation (± 10 percentage points) over the window |
| x86 frequency (8.2) | capped at the part's measured sustained all-core frequency; a point whose counted-core mean is more than 2% below the cap is invalid |
| Unequal collision rate (7.1) | duplicate-key rejection rates differing by more than 1 per 1,000 requests |
| Storage latency class (4.15) | synchronous 4 KiB write p99 on the log device within a factor of 2 |
| Hardware generation window (2) | 24 months |
| Scale factors (1.5) | S = 1 (cache-resident), and the memory-resident S whose stored data is at least 8 × the largest shared cache |
| Request stream (1.5) | account and card keys drawn from a Zipf distribution with exponent 0.99 over a seeded permutation of the keys |
| Compression (1.4) | zlib level 6 in plain mode; in accelerated mode, compressed sizes matched within 5% |
| Scale-out "flat" margin (9.3) | 0.10 × K_1 (per added node, engine or core) |
| Scale-out rate ladder (9.3) | K_1, 2 × K_1, 4 × K_1 and 8 × K_1, then further doublings while the declared maximum N allows |
| Scale-out starting N (9.3) | 1, or the N at which the previous rate was sustained |
| Open-loop rate beside batch (4.3) | 70% of C |
| Batch streams beside online (4.3) | one per counted core or engine |
| Accounting validation (4.17) | a calibration load at 25%, 50% and 80% of one engine; CPU per operation within 3% |
| Zero-load check of the Z basis (3) | effective dispatch time below 5% of online time with no load offered |
| Tuning budget (4.18) | 40 person-hours and 30 tuning runs per side per series |
| Reproduction margin (10) | 90% interval of the difference within ±10% of the original estimate |
| Pilot and power (10) | at least 6 runs over at least 3 boots or IPLs per side, at one point of each measurand; 80% power |
| Bootstrap | 10,000 resamples, seed in the run plan |
| Family-wise error rate (4.16) | 0.05, by the Holm method |
| Re-runs of an invalid run | at most 2 per point |

### 7.6 Reporting
- For every offered rate: offered and achieved rates; p50, p90, p99 and p99.9 (section 3) per operation and
  overall, with timeouts and refusals recorded at T; the maximum; counts of completed and committed requests,
  business outcomes by type, errors by cause, refusals by component and cause, and timeouts; late responses; the
  driver's schedule lag; and the requests waiting in the driver for a free connection.
- The principal latency figures are p50, p99 and p99.9.
- The raw histograms are published (4.6).

## 8. Equal-service comparison (proposed)
For a stated throughput within a stated p99 bound, this comparison reports what each side needs to sustain it. It
is **exploratory**: no prediction about it is registered (docs/HYPOTHESES-ADDENDUM.md), it scores no registered
hypothesis, and no equal-service result is a headline result. That changes only when a prediction about it has been
registered in a later addendum entry, at its own tag, before the first kit run that tests it, and the preconditions
of section 10 hold. Until then every equal-service result is reported as exploratory. Because it carries no
prediction, an equal-service result is descriptive: it is not summarised as one platform winning, and no claim of
advantage is drawn from it (8.7). Every FDR reports the verdicts of the registered hypotheses it scores first,
before any equal-service figure (4.12).

### 8.1 Service targets and latency bounds
- A **service target** is a stated throughput R (transactions per second at the transaction mix of 1.2), with its
  request profile and data variant (1.8), and a p99 latency bound L (7.2). The same R and L apply to both sides.
- **Fixed before the first run of the series,** calibration included, in the run plan: the calibration size, the
  calibration bound L_cal, the timeout T, the rule that derives the targets and the rule that derives the values of
  L.
- **Calibration size.** One size is declared for both sides before either is measured: the same number of x86 cores
  and of Z engines (on z/OS, CPs plus zIIPs, with the split declared), proposed 4. Each side's declared set of
  configurations (8.2) includes the calibration size and extends to at least twice it; a side whose declared set
  does not gives no equal-service result.
- **Calibration.** Each side runs the sweep of 7.2 at the calibration size, with L = L_cal (proposed: p99 of 1 s,
  overall) and the series' T (proposed: 10 × L_cal). The calibration knees are found with L_cal. The calibrations
  are reported with the result and score nothing.
- **Targets come from both calibrations.** The targets are declared fractions of the **lower** of the two
  calibration knees (proposed: 25%, 50% and 80% of it). They are never derived from one side's calibration alone.
  100% is not used: at the median knee about half the repeats of the lower side sustain, so a target there would
  report mostly noise.
- **Latency bounds come from both sides too.** After both calibrations, each side is offered 50% of the lower
  calibration knee, open loop, at the calibration size, with the run plan's repeats, and its mean p99 is recorded.
  Each side also records its mean p99 at 5% of the lower calibration knee, as its **latency floor**.
  - The series declares at least three values of L by a rule applied to both sides (proposed: 2×, 5× and 10× the
    **higher** of the two sides' mean p99 at 50% of the lower calibration knee). The higher is used so that no side
    fails a bound by its base latency alone, which adding engines or cores cannot shorten.
  - No value may exceed L_cal (that is, T/10). A value the rule would set higher is set to L_cal, and the cap is
    reported.
  - For each L, a side whose latency floor exceeds 0.8 × L (proposed) reports "latency floor above L" for that bound
    instead of a sustaining configuration.
- The derived targets and values of L are added to the run plan, committed and registered (4.13) before the first
  counted run of the series.
- **Network distance.** If the idle round-trip time from driver to system under test differs between the sides by
  more than 10% (proposed) of the smallest L, the result is labelled "unequal network distance", and the sustaining
  configuration is also reported with L applied to latency minus each side's idle round-trip time.
- A series may have several targets. Each target is a separate result.

### 8.2 Declared configurations
Before the series, each side declares an ordered set of configurations it will try, from smallest to largest. The
rules below apply to every counted run.
- **Z engines:**
  - Linux on Z: 1, 2, 3 ... IFLs. z/OS: a declared sequence of (CP, zIIP) pairs with at least one CP, for example
    (1,0), (1,1), (2,1), (2,2) ..., with the order fixed before the series.
  - **Dedicated or entitled.** A counted run uses dedicated engines, or shared engines that meet all four of these:
    1. absolute capping of the partition at the declared engine count, a whole number;
    2. exactly that many logical processors of the type online;
    3. a weight for that processor type, fixed for the series, that alone entitles the partition to at least the
       capped capacity: the partition's weight for the type ÷ the total weight for that type of the active
       partitions sharing the pool × the physical processors of the type in the pool ≥ the cap;
    4. the other partitions' consumption of the pool recorded for every run, and stable within the limit of 7.5 (a
       repeat outside it is invalid, not unsustained).

    Results on such engines are labelled "shared, capped"; results on dedicated engines are labelled "dedicated".
  - A configuration whose entitlement is below its cap is labelled "shared, capped above entitlement"; a shared,
    uncapped one is labelled "shared uncapped" and reports the partition's measured physical processor consumption
    beside its logical engine count. Both are disclosed only: not counted, and never a headline result.
  - Fractional caps on Z, and `cpu.max` caps on x86, are a labelled variant on both sides or on neither.
  - **Partition hygiene, for every counted run:** no System Recovery Boost period is active (boosts disabled, or no
    boost active in the window, shown by the system log and the RMF data for the window; the boost setting and the
    IPL time are recorded); no temporary
    capacity record is activated or deactivated during the series; Intelligent Resource Director weight management
    and vary CPU management are off; the HiperDispatch state and the polarisation of each logical processor are
    recorded at the start and end of each run; the CPC's microcode driver and bundle level is recorded; and the
    drawer and chip of each logical processor are recorded where the platform shows them.
  - The CP capacity setting and the multithreading (MT) mode of IFLs and zIIPs are fixed for the series. A series
    on sub-capacity CPs is labelled "sub-capacity CPs".
  - **z/OS: the partition runs only the system under test** (the z/OS counterpart of the x86 confinement): nothing
    but the system under test and the system's own address spaces, as listed in the run plan, checked against the
    SMF type 30 and type 72 data for every window (docs/CPU-ACCOUNTING.md, section 3). A run with other work is
    invalid.
  - **Linux on Z:** counted runs run Linux directly in an LPAR, with exactly the counted IFLs online (the others
    deconfigured, or offline with `chcpu`), so that interrupt handling and every other kernel activity of the
    partition can run only on counted IFLs; the processes of the system under test run in a cpuset that holds them
    all. The CPU dispatching mode (polarisation: horizontal or vertical, as `chcpu` sets it [S56]) is recorded at
    the start and end of each run. A z/VM or KVM guest is a labelled variant, with the hypervisor's time counted
    (docs/CPU-ACCOUNTING.md, section 2).
- **x86:**
  - One server on bare metal, with 1, 2, 3 ... physical cores in use. The cores in use form a **cpuset** (cgroup v2
    `cpuset.cpus`):
    - every process of the system under test runs in it: the application, the database, any cache and their
      background workers;
    - its memory nodes (`cpuset.mems`) are the NUMA node of the counted cores, and the counted cores stay on one
      socket, in one NUMA node and in one last-level-cache domain until the configuration needs more; the placement
      is recorded;
    - device interrupts are handled on the same cores, in one of these ways:
      - where an interrupt's affinity can be set, through `/proc/irq/*/smp_affinity_list` [S39];
      - for kernel-managed interrupts (for example NVMe, and many multi-queue network drivers), whose affinity
        cannot be changed from user space, through their queue mapping (with one NVMe queue per CPU, I/O submitted
        from the counted cores completes on them);
      - by limiting the NIC's RSS indirection table, and the RPS and XPS masks, to queues whose interrupts are on the
        cpuset's CPUs;
      - where none of these is enough, with the boot parameter `isolcpus=managed_irq,<the other cores>` (verify
        against the kernel in use).

      The per-CPU interrupt counts from `/proc/interrupts` over the window are published. The cores outside the
      cpuset are never taken offline, because the loaded neighbour state needs them;
    - kernel threads that can be moved are moved there; CPU affinity for the application alone is not enough.
  - The confinement is **verified for every run**. The **residue** is the busy time of the CPUs outside the cpuset
    during the window (user, nice, system, irq and softirq, from `/proc/stat`), less the CPU time of the background
    load's cgroup (loaded state), and less a baseline measured with the system under test stopped and the
    background load, if any, running. The residue is at most 1% (proposed) of the busy time of the counted cores,
    or the run is invalid. It is added to the reported CPU figure (docs/CPU-ACCOUNTING.md), so that work outside the
    cpuset is still counted, as all work in a Z partition is.
  - Both hardware threads of a counted core are in the cpuset, and the SMT state is fixed for the series.
  - **Frequency.** The frequency governor is fixed for the series. Counted series hold the counted cores at the
    part's **sustained all-core frequency**, not at base frequency: turbo is enabled and the counted cores are capped
    at the mean core frequency measured with every core of the socket running the kit's background load, measured
    once per machine before the series and recorded (for example with `cpupower frequency-set --max`, or the
    platform's equivalent). Pinning that frequency, and reporting the setting, are part of the run. A point whose
    counted-core mean frequency is more than 2% (proposed) below the cap is invalid. Turbo uncapped (single-core and
    light-load boost peaks) and turbo off at base frequency are labelled variants. IBM Z processors run at their
    machine's fixed rated frequency, which is recorded.
  - **The frequency asymmetry that remains,** stated with every per-core verdict: an x86 part can run a few cores
    above the cap when the rest of the socket is idle, and below it under heavier vector or power load, while an
    IBM Z core runs at one fixed frequency whatever its neighbours do. Each per-core verdict reports the cap, the
    measured mean frequency of the counted cores, the part's published base and maximum boost frequencies, and the
    Z machine's rated frequency; the turbo-uncapped variant is run and reported beside it, and scores nothing.
  - **SKU.** The x86 part is named, with its class (frequency-optimised, general-purpose, core-dense, or with
    stacked cache), its last-level cache per core and the reason it was chosen, in the machine registry before any
    kit run on it (docs/REGISTRY.md). Memory is populated on every channel at the part's rated speed. Scored runs
    use a general-purpose part, or a part of another class named in the machine registry, whose verdicts are then
    labelled with its class (docs/HYPOTHESES-ADDENDUM.md); every named part is in the one Holm family.
- **Neighbour load, the same rule on both sides.** Every counted series declares one neighbour state and runs it on
  both sides. The neighbour processors are those that share the counted cores' largest shared cache or memory
  path: on x86, the other cores of the socket; on IBM Z, the other physical processors of the drawers that hold the
  counted logical processors.
  - **Loaded (the scored state):** the neighbour processors are busy at a defined level over the window: 80% mean
    utilisation (proposed), within ± 10 percentage points. On x86 the kit's background program provides it, in its
    own cgroup, excluded from the count and from the confinement residue. On IBM Z the kit's background program
    provides it in another partition on the same drawers where the owner allows one; otherwise the other
    partitions' own work provides it, labelled "loaded by other partitions". On Z the level is shown for every run
    from the RMF partition data and the processor topology data (verify the records: SMF type 70 subtype 1, and
    SMF type 99 subtype 14 for the topology). The background type is reported with every verdict.
  - **Quiet (a labelled variant, on both sides):** on x86, the other cores of the socket idle; on IBM Z, no logical
    processor of any other partition dispatched on the drawers that hold the counted logical processors, shown in
    the same way. Where one side cannot be made quiet, the quiet variant is not paired.
  - A run whose declared state cannot be shown is invalid for that state. The drawer and chip of every counted
    logical processor are recorded for every run, and a series whose placement changes between runs is labelled
    "placement changed".
- **Both sides:** a small configuration still shares the last-level cache and memory bandwidth of its socket, core
  complex, chip or drawer. Each configuration states its memory, the last-level cache available to its counted cores, and the
  frequency policy. Memory follows the **same rule on both sides**, stated before the series: the same amount at
  each count, or the same multiple of the stored data.
- Settings that change with size (worker pools, buffer pools, connection limits) follow a rule stated before the
  series. Every other setting is the same across the set.
- A single-box series changes the size of one machine or partition. A series that adds machines is the scale-out
  tier (9).

### 8.3 Procedure
For each target, from the smallest declared configuration upwards:
1. Offer R open loop (7.1), with warm-up, window and the run plan's repeats (section 4).
2. If the configuration sustains R (7.4), it is the **sustaining configuration** for that target. Stop.
3. Otherwise move to the next declared configuration.

The result for the next smaller declared configuration, which did not sustain R, is reported with the sustaining
configuration, so the reader can see that one size less was not enough. If no declared configuration sustains R,
the result is "not sustained within the declared configurations", and the result for the largest declared
configuration is reported.

If the smallest declared configuration sustains R, the result is "sustained at the smallest declared
configuration". That is an upper bound on the need, not the need. It is reported with the CPU time per transaction
(8.4) and without a next smaller configuration. When both sides sustain R at their smallest declared configuration,
no difference in counted cores or engines is reported, only the CPU time per transaction.

### 8.4 What is counted, and what is disclosed
For each target and each side, the sustaining configuration is reported with:
- **cores or engines by type,** counted over every tier of the system under test (including any database, cache,
  load balancer and coupling facility partitions; a load-balancer appliance is listed instead, unless it terminates
  TLS or runs a pipeline step):
  - on Z: CPs, zIIPs, IFLs and ICFs (or the shared CPs a coupling facility runs on), each dedicated or shared, with
    weight, entitlement and capping for shared engines, and the MT mode of IFLs and zIIPs;
  - on x86: physical cores, by CPU model, with the SMT state;
  - cores or engines of tiers that exist only for availability (standbys, replicas that serve no reads) are counted
    separately from those that serve the load, and each side's count is given with and without them (9.2);
- **everything that runs a pipeline step on its own processors is counted.** A component that executes a step of
  the request pipeline (docs/ARCHITECTURE.md, section 3), or terminates TLS sessions, on processors of its own and
  independently of the counted cores (an appliance, a gateway, a TLS-terminating proxy, or a SmartNIC that runs such
  a step) is part of the system under test, and its processors are counted. A device that only performs an
  operation requested by a counted core (a cryptographic or compression operation by CPACF, a Crypto Express
  adapter, QAT or IAA, or a network adapter's inline TLS record encryption) is listed, not counted, on both sides;
- **CPU time per transaction by engine type,** on the basis of docs/CPU-ACCOUNTING.md, so that a tie at whole-core
  or whole-engine steps is not read as equal need;
- **processors and adapters not counted as cores,** listed on both sides:
  - on Z: System Assist Processors (SAPs), Integrated Firmware Processors (IFPs), the on-chip Data Processing Unit
    (DPU, z17), channels, network adapters and their offloads, and cryptographic adapters; for every z/OS run, the
    SAP utilisation from the RMF I/O queuing activity data (verify the record, SMF type 78 subtype 3), and for Linux
    on Z the same figure from the CPC owner where it is available;
  - on x86: network adapters and their offloads (and any DPU or SmartNIC that runs no pipeline step), storage
    controllers, and on-die accelerators such as QAT, IAA and DSA where present;
  - on both: the storage subsystem and its processors;
- **accelerators** present and in use (1.4), with the counters that show their use: on Z, CPACF, the on-chip
  compression accelerator (zEDC, used through DFLTCC), CMPSC (the compression call instruction), the Z Sort
  accelerator (SORTL), the IBM Integrated Accelerator for AI, with the number of AI accelerators the partition's
  cores can reach, IBM Spyre Accelerator adapters and Crypto Express adapters; on x86, AES-NI/SHA, AVX-512, AMX, QAT,
  IAA and DSA. Hardware that only performs operations requested by the counted cores (CPACF, the on-chip
  accelerators, QAT, IAA, Crypto Express) stays listed, not counted;
- **memory,** over every tier;
- **hardware generation:** machine type and model, or CPU model, with its general-availability date, on both
  sides. If the two dates are more than 24 months apart (proposed), the result is labelled "cross-generation";
- **power,** where measurable (1.7; docs/methods/energy.md), or "not measurable" with the reason;
- **cost,** where priced (FDR section 8), with the licence metric of each product;
- utilisation of each tier at the sustaining configuration.

Driver hosts are never counted, and are reported separately (7.3).

### 8.5 What equal service does and does not capture
Equal service counts the cores and engines a workload needs to meet a stated service, and lists every processor,
adapter and accelerator that does work outside them. It does not count that outside work: neither an equal-core run
nor an equal-service run does, on either platform. What equal service adds is that each side runs at the size it
needs for the same service, instead of at a size fixed in advance; that a cluster can be compared with a single
system; and that the result is a resource need at a stated service level.

One asymmetry of this kind is stated with every result: on x86, the interrupt and softirq time of network and
storage I/O lands on the counted cores, while on IBM Z channel-program processing runs on SAPs, which are listed and
not counted. For I/O-heavy work this favours Z in CPU time per transaction, which is why the SAP utilisation is
reported beside every z/OS result (8.4).

### 8.6 Normalisations reported with it
- At the sustaining configuration, the target per counted core or engine (R divided by the count) is shown beside
  the result, labelled as a normalisation of the target, not a capacity. Per-core capacity (the knee, 7.4, divided
  by the count) is reported only from a sweep of that configuration, and labelled as such.
- A per-watt or per-dollar figure is shown beside the result only when both sides are measured or priced on the
  same basis. Otherwise neither appears there, and the available side's figure is given in the FDR body, labelled
  "one side only; not comparable". Where no published list price exists for a hardware configuration, on either
  side, no per-dollar figure is given for it.
- Equal-service results are measured resource needs at one service target. They are not consolidation ratios, and
  they are never scaled to an estate (docs/prior-art.md, section 5, "Traps to avoid"). The only measured precedent
  we found for a fixed offered load with the resources each side needs is a study commissioned by IBM
  (docs/prior-art.md, AN01); we found no independent one.

### 8.7 How an equal-service result may be summarised
An equal-service result is the table of 8.4 for each target. It is never reduced to one ratio of engine counts
across engine types. Where one figure is given, it is the ratio of CPU time per transaction (8.4) on the two sides,
labelled with the engine types it compares, with its 95% confidence interval and the result of the next smaller
configuration. Its inverse is not a replacement ratio either. The table shows each side's configurations in their
own units; on z/OS the declared (CP, zIIP) steps can be finer than whole x86 cores, which is stated, and the figure
compared is the CPU time per transaction, not the number of steps. Every FDR summary that reports equal service
carries this sentence: "An equal-service result is a resource need at one service target on the configurations
tested. It is exploratory. It is not a consolidation ratio, and it does not state how many x86 cores replace an IBM
Z engine."

## 9. Scale-out tier (proposed)

### 9.1 Purpose
This tier is its own labelled tier (section 2). It measures what each added application node buys at a fixed
offered rate, and where adding nodes stops raising capacity (the **scaling limit**, if one is found). It runs the
live-database variant (1.8.4), because file-based stores do not span nodes, and names its form. Its results are
exploratory, like those of section 8.

The x86 cluster of this tier is **single-writer**: one database primary takes every write. Results are labelled
"scale-out tier, single-writer". A scaling limit found here is the limit of a single-writer design with the
declared fixed tiers, not of x86 clusters in general. A multi-writer x86 variant (a shared-data or distributed
database, as declared) is optional and may be added in a later spec version. No claim is made that the
single-writer cluster, or any later variant, matches Parallel Sysplex data sharing.

### 9.2 Topology and availability design
- The x86 cluster has a load balancer, N identical stateless application nodes, a counter and session store, one
  database primary and optional batch nodes (docs/ARCHITECTURE.md, section 5.2). It has database replicas only in
  variant S2, where they serve reads and are counted, or under availability design A1.
- Within a series only N changes. Every other tier is fixed, and declared before the series starts.
- **Availability design,** declared for each side and the same on both sides of a comparison (4.11):
  - **A0 (default): no standby on either side.** On x86, one database primary with no synchronous standby; on Z,
    the declared configuration with no standby partition or CPC.
  - **A1: a synchronous standby on both sides, at the same layer.** Either a database-level synchronous standby on
    both sides (on x86, for example PostgreSQL synchronous replication; on Z, a database standby), or storage-level
    synchronous mirroring on both sides (on Z, for example Metro Mirror; on x86, synchronous array replication). The
    layer is declared; a pairing across layers is labelled "unequal availability design".

  A comparison between different availability designs is labelled "unequal availability design" and reported as
  two separate results, not as a comparison. Cores or engines of tiers that exist only for availability are
  counted separately (8.4).
- Two variants:
  - **S1 (default):** only application nodes grow;
  - **S2:** application nodes and read replicas grow together, with read-only operations routed to replicas.
    Replication lag is measured, and the invariant check (1.8.5) is run after each point.
- The load generator runs on its own hardware (7.3) and is sized for the highest rate in the series.

### 9.3 Procedure
- The series fixes L and T in the run plan before K_1 is measured (proposed: L = L_cal and the T of 7.5), and every
  K_N, and every (R, N) point, uses them.
- The single-node knee K_1 is found by the sweep of 7.2 with N = 1.
- A **rate ladder** of fixed target rates is set from K_1 before the series starts, and recorded.
- At every N used in the series, C and the knee K_N are found by the procedures of 4.2 and 7.2 (coarse and fine
  passes).
- For each rate R, N grows one node at a time. Each (R, N) point follows the run rules of section 4.
- **Flat, rising or inconclusive.** For each added node, the 95% percentile bootstrap interval (7.5) of
  D_N = K_N − K_(N−1) is computed, with the per-repeat knees at N and at N − 1 resampled independently. The node
  takes the first label that applies, so that each node is in exactly one class:
  - **flat** if the interval lies wholly below 0.10 × K_1 (proposed), that is, the node is shown to add less than a
    tenth of the one-node knee (a one-sided equivalence test, 4.16);
  - **rising** if the interval lies wholly above 0;
  - **inconclusive** otherwise.

  "Flat" is always reported with its margin, never as "adds nothing".
- For each R the series ends in one of three outcomes, and records it:
  - **sustained:** N_min(R), the smallest N that sustains R (7.4). The series then adds two more nodes, to show
    whether more nodes keep helping;
  - **the scaling limit:** two consecutive added nodes are flat. The series records the N where this began, and
    the tier whose utilisation or waiting rose (load balancer, application, counter and session store, database or
    network);
  - **the declared maximum N** is reached first. Inconclusive steps are reported as such.

### 9.4 The Z comparator
The same ladder of rates is offered to the declared Z configuration: a single LPAR at a declared set of engine
counts (8.2), and in a later phase a Parallel Sysplex with an increasing number of members. For each R the result
records whether it is sustained, and the smallest declared configuration that sustains it (8.3).

The same scaling-limit test applies on both sides. The single-LPAR series finds C and the knee at every engine count
in its declared set (4.2, 7.2), classes each added engine flat, rising or inconclusive by the rule of 9.3, with K_1
the one-engine knee, and reports two consecutive flat engines as the scaling limit of that configuration, with the
resource whose utilisation or waiting rose. Where an x86 single-server series is run beside the tier, each added
core is classed in the same way, with K_1 the one-core knee. A sysplex with
Db2 data sharing is multi-writer; it is labelled so, and a comparison of it with the single-writer x86 cluster is
labelled "multi-writer sysplex against single-writer cluster".

### 9.5 What the tier reports
- For each R: the sustaining configuration on each side, counted and disclosed as in 8.4, with and without the
  tiers that exist only for availability.
- Per-node capacity (K_N divided by N) against N; and, for each R, the achieved rate against N.
- The scaling limit, where found, and the saturated tier.
- Any fit of a scalability model to the pairs (N, K_N) is reported as analysis, never as a measurement.
- Results of this tier are labelled "scale-out tier" and reported apart from single-box results. They score no
  registered hypothesis, and none is a headline result.

## 10. Preconditions before any headline result (proposed)
A **headline result** is a result that the project presents as a finding of the benchmark: in an FDR summary, the
README, a release note or an announcement. None exists today, and none can exist until every precondition below
holds. Until then, every FDR summary begins with "Preliminary: the preconditions of SPEC section 10 are not all
met", followed by the list of those not met.

| Precondition | What is missing today | What must exist |
| --- | --- | --- |
| 1. A published kit | No kit code is published: the kit's data generator, transaction mix table, load driver, Linux replacement for CICS and VSAM, Java reimplementation, z/OS wrapper programs and golden script are not released. | A tagged kit release with all of these, a configuration checklist for each platform, the validity checker of 4.13, and reference code for the interval methods and the Holm procedure of 4.16, which passes its golden check on every platform of the result. |
| 2. A sanctioned platform | The project has no IBM Z or LinuxONE system whose terms allow benchmarking and publishing results (docs/platforms.md). | For every machine in the result, the written permission of its owner to benchmark it and publish the results, recorded in FDR 9. |
| 3. Licence terms checked | The licence terms of the programs a run would use have not been checked for conditions on benchmarking or publishing results. | For every licensed program in the run, on both sides (IBM's and any other vendor's: for example z/OS, CICS TS, Db2, Enterprise COBOL, a Java runtime, a commercial Linux distribution, a hypervisor or a vendor accelerator library), its terms checked and the outcome recorded in FDR 9. A configuration whose publication a term forbids, or makes conditional on something the run does not meet, is not published. |
| 4. An independent reproduction | No one has re-run anything from this specification. | A reproduction by a team independent of the maintainer (neither the maintainer nor anyone working under the maintainer's direction), from the published kit and run plan, on its own x86 hardware and on the same or an equivalent Z machine (the same machine type, engine type and count, engine mode and capacity setting), published with its own FDR. A figure to be called a headline is reproduced when the 90% interval of the difference between the reproduction's estimate and the original's (Welch for means, Fieller for ratios, the percentile bootstrap for derived quantities, over both sets of runs) lies within ±10% (proposed) of the original estimate. A registered verdict to be called a headline must be the same verdict in the reproduction. A figure or verdict that does not reproduce is not a headline, and the difference is reported. |
| 5. External pre-registration | The hypotheses are timestamped only by git history. | The signed tag `omb-v0.4.0-prereg` and its archive deposit with a DOI, recorded in docs/REGISTRY.md, with an archive creation time before the first counted kit run (4.7); the machine registry entries of every machine in the result, made before any kit run on it; the run registry and tuning logs of every series behind the result (4.13, 4.14). For an equal-service or scale-out result, a prediction about it registered in the same way before its first kit run (section 8). |
| 6. A pilot and a power analysis | No pilot has been run and no power analysis made. | Before the tag `omb-v0.4.0-prereg`: a pilot of at least 6 runs over at least 3 boots or IPLs (proposed) on each side, at one point of each measurand, entered in the run registry as kind "pilot", scoring nothing and published; and, from it, a power analysis recorded in the addendum entry: for every claim, its Holm-adjusted level, the repeat count, and the smallest distance from the predicted region's boundary that the claim resolves with 80% probability (proposed) at that count (docs/REGISTRY.md, section 2). Every DIFFERS verdict is quoted with that power statement. |

Results produced before the preconditions hold are published under the run rules, labelled "preliminary", and they
stay preliminary. Preconditions 2 and 3 are never waived: no result, preliminary or not, is published from a machine
without its owner's written permission to benchmark it and publish the results, or from a licensed program whose
terms have not been checked. Only the independent reproduction (precondition 4) necessarily comes after the result it
reproduces.

Trademarks used in this specification are listed in [NOTICE](NOTICE).
