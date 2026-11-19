# OLTP module derived from TPC-E (ROADMAP T3): plan

Status: plan only, 2026-11-19. No code and no runs. Sources:
- the public TPC-E specification, revision 1.14.0 (downloaded from tpc.org to `E:\tpc\TPC-E-spec\`, outside this
  repository);
- TPC Policies v6.20;
- the project pages of the open-source kits named below.

Clause numbers point into the TPC-E specification. Rules are paraphrased, never copied (SPEC.md section 6).

> This workload is derived from the TPC-E Benchmark and is not comparable to published TPC-E Benchmark results, as
> this implementation does not comply with all requirements of the TPC-E Benchmark. (Disclaimer wording from TPC
> Policies v6.20, 8.1.5.)

It sits beside the CardDemo online track (SPEC 1.7). **CardDemo is never derived from it**, and nothing here changes
the online track.

## Verdict
- **x86 PostgreSQL, derived, using DBT-5 plus the TPC's EGen: feasible.**
  - A minimum-scale run (5,000 customers) fits the bigger x86 host.
  - It fits the 2 vCPU / 2 GB rig only as an I/O-bound, disclosed-deviation run.
  - About 3 weeks to a first derived result with harness and FDR **(estimate)**.
- **A compliant-shape implementation of our own on EGen** (driver, connector, frames): **feasible but not
  recommended first**. About 8-12 weeks **(estimate)**; DBT-5 already does this work under an open licence.
- **Db2 (later): feasible**, as new frame and connector code beside DBT-5's PostgreSQL code. About 4-8 weeks after the
  PostgreSQL leg **(estimate)**.

## 1. What EGen provides (clause 5, Appendix A)
EGen is TPC-provided C++, built by the sponsor. A compliant implementation must use it, unmodified except through the
TPC's portability process (clauses 5.3, 5.6).

| Part | Provides | Sponsor's side |
| --- | --- | --- |
| **EGenInputFiles** | the tab-separated source data (names, companies, securities, zip codes, ...) | none; modification not permitted (5.5) |
| **EGenLoader** | generates the initial database (all tables, exact cardinalities) from a customer count, scale factor and Initial Trade Days | a loader: flat files plus our bulk load (`COPY`), or a custom loader class. Extension code is disclosed (5.7.4) |
| **EGenDriverCE** (class CCE) | the customer-emulator transaction mix (a card-deck generator) and all transaction inputs | `CCESUTInterface`: send each request to the SUT, time it, log it |
| **EGenDriverMEE** (CMEE) | the market exchange emulator: the ticker, triggered limit orders, Trade-Result and Market-Feed inputs | `CMEESUTInterface`, plus the listener for the SUT's send-to-market messages |
| **EGenDriverDM** (CDM) | the Data-Maintenance and Trade-Cleanup inputs, one every 60 s | `CDMSUTInterface`; exactly one DM instance per run (5.8.7.2) |
| **EGenTxnHarness** | the frame-flow control of every transaction (which frame runs next, with which inputs) | one `C<Txn>DBInterface` per transaction (`CTradeOrderDBInterface` and the others): the frame bodies against the database |
| **EGenValidate** | checks Trade-Results per Load Unit over the Measurement Interval (5.10) | feed it our logs |
| **EGenLogger** | logs the driver and loader configuration and flags any non-default value (5.8.1) | keep its output; any "NO" makes the result non-compliant |

**EGen does not provide any of these:**
- a network protocol between driver and SUT, or the processes that host CE, MEE and DM;
- a database connector, schema DDL or frame SQL;
- timing and logging, a report generator, or ACID tests.

## 2. What we must build (or take from DBT-5)

| Piece | Built ourselves | With DBT-5 |
| --- | --- | --- |
| Schema DDL, keys and indexes for PostgreSQL (clause 2) | write from clause 2 | provided (PostgreSQL) |
| Loader (EGenLoader flat files, then `COPY`, then indexes and constraints) | write | provided |
| Frames: 10 transactions, plus Data-Maintenance and Trade-Cleanup, as PL/pgSQL functions (or client-side SQL) following clause 3's frame definitions | write (the bulk of the work) | provided: PL/pgSQL by default, C functions optional |
| SUT connector: the "Brokerage House" process implementing EGenTxnHarness and the DB interfaces over libpq | write (C++) | provided (`BrokerageHouse`) |
| Driver harness: CE threads (CCE), MEE (CMEE) with the send-to-market listener, DM (CDM), the wire protocol, connection pooling, the ramp-up and ramp-down controller | write (C++) | provided (`Driver`, `MarketExchange`) |
| Timing: start and end stamps at the driver, before the first byte sent and after the last byte received, resolution at least 0.01 s (6.5.1.1) | write | provided; our harness re-derives every statistic from the raw log |
| Metrics: per-type 90th percentile, average, mix, Trade-Result rate per second, rate per minute for the sustainability check | write | partly provided; **our `checks.py` is the scorer** |
| ACID tests (clause 7): atomicity, consistency conditions, isolation tests, durability | write | partly provided (to check); the rest is ours |
| Manifest, repeats, FDR, compliance map, labelling | ours either way | ours |
| Db2 (phase 2): DDL, loader, frames in SQL PL, connector over Db2 CLI | ours either way | ours |

**Recommended route (Option A):**
- Use DBT-5 at a pinned release (v0.11.0 at the time of writing). It is fetched by the scripts, not vendored, and it
  keeps its own Artistic-2.0 licence.
- Build it against the user's own EGen 1.14.0 at `TPCE_KIT`.
- Wrap it in our harness: `scripts/` plus `py/`, the same conventions as `analytics/tpch/`.
- Read the TPCx-V kit's PL/pgSQL frames (`E:\tpc\TPCx-V\...\VDb\pgsql\dml`) only as a second reference, never copied:
  it is a derived from TPC-E schema under the TPC EULA.

## 3. Minimum scale

**Rules (verified in the specification):**
- At least 5,000 customers, in 1,000-customer Load Units (2.6.1.2).
- Scale Factor 500 (2.6.1.4); Initial Trade Days 300 (2.6.1.5).
- Configured customers must equal the customers accessed (2.6.1.6).
- TRADE starts at 17,280 rows per customer (2.6.1.7), so 86.4 million at 5,000 customers.
- Nominal rate is 1 Trade-Result/s per 500 customers. The measured rate must lie between 80% and 102% of it (6.6.8.2,
  6.7.1.2).

| Scale | Customers | Nominal Trade-Result/s | Database size on PostgreSQL (estimate) | Use |
| --- | --- | --- | --- | --- |
| dev | 1,000 (EGenLoader's smallest; below the 5,000 minimum) | 2 | ~15-20 GB | build and smoke tests only; a disclosed deviation |
| minimum | 5,000 | 10 | ~50-100 GB | the first derived result |
| bigger x86 | sized to measured capacity: LU = rate / 2 | e.g. 200 at 100 LU | ~1.8 TB at 100 LU | the scaled comparison |

The database sizes are estimates, scaled from the TPCx-V kit's measured PostgreSQL sizes (about 7.4 GB per LU at 125
trade days) by the trade-day ratio (300/125). DBT-5's docs say only "tens of gigabytes" for 5,000 customers. **They
must be measured on the first load.**

**The 2 vCPU / 2 GB rig:**
- **Fit:**
  - 5,000 customers (about 50-100 GB) against 2 GB of RAM is a buffer hit ratio well under 10% **(estimate)**: the run
    is disk-bound.
  - 10 Trade-Result/s means about 100 transactions/s in total (Trade-Result is 10% of the mix, clause 6.3.1).
    Whether 2 vCPUs reach that is **unknown until measured**.
  - If they don't, the 80% floor fails. That is a reported result, not a reason to lower the scale.
- **The driver must not share the SUT's 2 vCPUs:** run CE, MEE and DM on other pinned cores or another host.
- **The pilot's hard limit is the WSL2 VHDX.** Keep the database on native ext4 inside it (as the TPC-H track's README
  rule says), with at least 150 GB free.

**Bigger x86** (the 16-thread desktop on bare-metal Linux, or cloud bare metal):
- First measure capacity at 5,000 customers.
- Then regenerate at LU = measured rate / 2, so that the run lands inside 80-102%.
- Repeat until it lands. Each regeneration is a full EGenLoader run plus load.

## 4. Run rules to implement (compliance-readiness)

Key:
- **auto:** our checker scores it from raw logs.
- **op:** operator-stated in the FDR notes.
- **dev:** cannot be met on that platform; always listed as a deviation.

| Rule (paraphrased) | Clause | Rig | Bigger x86 |
| --- | --- | --- | --- |
| EGen version matches the spec version (both 1.14.0) | 5.3.1 | auto (`GetEGenVersion`) | auto |
| EGen sources and input files unmodified; any portability fix disclosed | 5.3, 5.5, 5.6 | auto (checksums against the kit) | auto |
| EGenLogger shows all defaults ("NO" anywhere is non-compliant) | 5.8.1 | auto | auto |
| CE/MEE/DM built with the constructors without an RNG seed; unique IDs; exactly one DM | 5.8.2-3, 5.8.7.2 | auto (logged) | auto |
| Database populated only by EGenLoader; at least 5,000 customers in 1,000 steps; SF 500, ITD 300 | 2.6.1 | auto | auto |
| No pending or submitted trades at start; Trade-Cleanup runs before anything else | 6.6.2.2, 6.3.4 | auto | auto |
| Mix inside the required ranges, reported to 3 decimals | 6.3.1-2 | auto | auto |
| Data-Maintenance every 60 s (58-62 s apart), each within 55 s | 6.3.3 | auto | auto |
| 90th-percentile response-time limit per transaction type (1-3 s); resolution at least 0.01 s | 6.5.1 | auto | auto |
| Measurement Interval of at least 2 h, inside Steady State; every CE thread active before and after it | 6.6.5 | auto | auto |
| Sustainable for a Business Day (8 h) from the start of Steady State (sliding-window variation limits) | 6.6.3 | **dev** by default (8 h runs need a quiet machine all night) | op: one 8 h run per campaign |
| Durable copy of the database no more than 15 min behind any commit | 6.6.5.3 | auto (checkpoint settings in the manifest) | auto |
| Measured rate 80-102% of nominal; EGenValidate passes (Trade-Results per Load Unit) | 6.6.8, 6.7.1.2, 5.10 | auto | auto |
| Atomicity, consistency and isolation tests | 7.2-7.4 | auto (ours) | auto |
| Durability: Loss of Processing, Loss of a Vulnerable Storage Component, Loss of all external power; Business Recovery at 95% of reported for 20 min | 7.5 | Loss of Processing only (kill PostgreSQL, restart) | all three, on hardware we own (the power test as in the TPC-H track) |
| Data Accessibility: survive a permanent failure of a durable medium while running | 7.6 | **dev** (single disk) | met with md RAID-1 over two NVMe disks |
| 60-Day space computed from measured growth | 6.6.6, 8.2 | auto (computed, not priced) | auto |
| Pricing (TPC-Pricing); audit; Executive Summary in TPC form | 8, 9, 10 | **dev** | **dev** |
| Primary metric name and price-performance | Policies 8.1.4 | never used | never used |

Our own figures are named in plain words:
- "Trade-Result transactions per second (derived from TPC-E; not comparable)", per core;
- CPU per transaction, and p50/p90/p99/p99.9 per type (SPEC section 3).

## 5. Existing open-source TPC-E implementations

| Project | What it is | Licence | Status (checked 2026-11-19) | Use to us |
| --- | --- | --- | --- | --- |
| **DBT-5** (osdldbt/dbt5) | "fair-use" TPC-E implementation on EGen; PostgreSQL only; PL/pgSQL (default), C or client-side frames; `BrokerageHouse`, `MarketExchange` and `Driver` processes; AppImage builds | Artistic-2.0 (GitHub) | **active**: v0.11.0, released 2026-10-31; 932 commits | **base for Option A** |
| osdldbt/egen | a mirror of the TPC-E Tools (EGen) used as DBT-5's submodule; carries the TPC EULA and the required legend | TPC EULA | 2 commits | **do not use**: SPEC section 6 says download, never vendor. Build DBT-5 against our own `TPCE_KIT` instead (DBT-5 says the TPC-E Tools "must be downloaded separately") |
| BenchBase (cmu-db) | Java OLTP framework: TPC-C, TPC-H, TATP, YCSB and others | Apache-2.0 | active; 2,300 commits | **no TPC-E**: not in its benchmark list, even as pending. Possible later for a derived from TPC-C cross-check only |
| Percona-Lab/tpce-mysql | TPC-E-like kit for MySQL (and SAP HANA), using EGen; says its results are not comparable to TPC-E | TPC EULA files included | 3 commits; maintenance unknown | read only, for MySQL ideas |
| H-Store TPC-E | academic, marked incomplete by its own docs | (not checked) | inactive (inferred) | none |
| The TPCx-V kit's VDb (`E:\tpc\TPCx-V`) | PL/pgSQL frames for a derived from TPC-E schema (125 trade days, two databases per group) | TPC EULA | TPC 2.1.10 | second reference only, never copied |

**Results from DBT-5 are derived from TPC-E under the same rules:**
- they use the TPC's EGen and follow the TPC-E specification, so EULA clause 4(b) and TPC Policies 8.1.4, 8.1.5 and
  8.3 apply;
- the "Derived from" prefix, the disclaimer, no TPC metric names, every deviation listed, and academic framing only;
- DBT-5's own documents call it a fair-use implementation and point to the TPC fair-use policy.

DBT-5's code is not TPC material: it may be fetched and patched in our harness. EGen and anything built from it may
not enter this repository.

## 6. Effort and risk

| Work | Effort (estimate) | Risk |
| --- | --- | --- |
| Confirm the kit (EGen 1.14.0), extract it to `E:\tpc\TPC-E\`, read its EULA | 0.5 day | low |
| Build DBT-5 v0.11.0 against our EGen on Linux x86; dev load at 1,000 customers; first short run | 2-3 days | medium: whether DBT-5's EGen patches apply cleanly to 1.14.0 is unconfirmed |
| Harness: config/env, manifest, repeat campaign, raw-log re-scoring (`checks.py`), FDR, compliance map | 5-7 days | low (patterns from the TPC-H track) |
| ACID tests DBT-5 does not cover (to be listed after reading its source) | 3-5 days | medium |
| Minimum-scale (5,000-customer) load and first derived 2 h result on the rig and on bigger x86 | 2-3 days of machine time | medium: disk size, run length |
| **x86 PostgreSQL module, total** | **about 3 weeks** | medium |
| 8 h sustainability, power-loss durability and RAID-1 data accessibility on our own hardware | 1 week, mostly machine time | medium |
| Db2 (phase 2): DDL, loader, frames in SQL PL, connector | 4-8 weeks | high (no open reference implementation) |
| s390x leg: EGen and DBT-5 built on s390x; loader output compared byte for byte with x86 | 1-2 weeks once a platform permits | medium: char signedness, byte order (the TPCx-V kit sends raw structs on the wire, FEASIBILITY.md section 4) |

**Main risks:**
1. **Isolation on PostgreSQL:** the specification's isolation levels against PostgreSQL's snapshot semantics, and the
   serialisation-failure retries this brings. Retries must happen in the SUT, not the driver.
2. **Run length:** 2 h measurement intervals and 8 h days need a quiet machine.
3. **Disk at scale:** about 18 GB per LU **(estimate)**.
4. **Licensing:** the "Derived from" naming and the TPC's written permission (Policies 8.1.4) are open for the whole
   project.

## Choices I made
1. **DBT-5 as the base (Option A).** Recommended over writing our own driver and frames. Proposal only; the maintainer
   decides.
2. **Database sizes:** estimated by scaling TPCx-V's measured PostgreSQL sizes. They are not measurements.
3. **Reading "the 2 vCPU / 2 GB rig":** taken as the pilot rig (WSL2 capped at 2 vCPU / 2 GB), with the driver run
   outside those 2 vCPUs.
4. **Wording:** the Policies v6.20 "Derived from" prefix and 8.1.5 disclaimer are used alongside the SPEC section 6
   wording. Whether SPEC section 6 adopts them is open (FEASIBILITY.md in `virtualization/tpcx-v/`, choice 3).
