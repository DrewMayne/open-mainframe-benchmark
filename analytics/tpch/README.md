# Analytics track: TPC-H-derived (milestone A1)

> **TPC-H-derived; not comparable to published TPC-H results.** These are academic research results with no
> marketing claims (TPC EULA clause 4.c). They are not TPC Benchmark Results and have not been audited, and TPC
> metric names are never used for them (SPEC.md section 6).

This track measures the reporting and analytics side of mainframe work: large scans, joins, sorts and
aggregations over the TPC-H data set and its 22 queries, run on PostgreSQL (x86 first, then Linux on Z; Db2 for
z/OS in phase 2). It follows SPEC.md 1.6. The harness implements every TPC-H rule that can be implemented on the
platform. [COMPLIANCE.md](COMPLIANCE.md) maps each clause to how it is met, or to why it cannot be on a given
platform.

**No TPC material is in this repository**: no kit code, query templates, specification text or reference data.
Every script reads your own extracted kit through `TPCH_KIT`.

## 1. Get the kit
1. Download **TPC-H Tools** from tpc.org (TPC Current Specifications > TPC-H > download the tools). Accept the TPC
   End User License Agreement. This harness was written against kit revision 3.0.1 (its `release.h` says 3.0.0,
   see STATUS.md).
2. Extract it **outside any git repository**, e.g. `E:\tpc\TPC-H_v3.0.1\` (`/mnt/e/tpc/TPC-H_v3.0.1` from WSL).
   The kit's top folder name contains a space ("TPC-H V3.0.1"), so rename it to a name without one.
3. Read `EULA.txt`. Every result you publish must carry the label above.

## 2. Prerequisites (Linux, x86_64 or s390x)
- gcc and make (to build dbgen and qgen), bash and coreutils (`/usr/bin/time` is used if present)
- Python 3.8 or later, with `venv` (the driver's only dependency is `psycopg==3.2.3`, see requirements.txt;
  it uses the system libpq)
- PostgreSQL 15 or later (`psql` and libpq on the client side), and a role with CREATEDB and membership in
  `pg_checkpoint` (needed for the CHECKPOINT that ends the load). It does not need superuser. An administrator
  creates it once, for example:
  `CREATE ROLE tpch LOGIN CREATEDB; GRANT pg_checkpoint TO tpch;` (then set its password with `\password tpch`).
  The role's attributes are recorded in every manifest.
- Disk at SF 1: about 1.1 GB of .tbl files and 2.5-3 GB per database. At SF 10, about ten times that.
- Put the working area (`TPCH_WORK`) on a native Linux filesystem. On WSL2 that means the distro's ext4, never
  `/mnt/c` (9P).

Copy `config.example.env` to `config.env` (git-ignored) and set `TPCH_KIT`, `TPCH_WORK`, `TPCH_DISTRO`, the PG*
connection variables and the two database names. Never put a password in it: use `PGPASSWORD` or `~/.pgpass`.

## 3. Steps
Every step is one script. `scripts/sf1_window.sh` runs them all for SF 1 on an existing PostgreSQL instance. From
Windows, `scripts/wsl.sh <command>` runs any of them inside the `TPCH_DISTRO` distro.

| # | Step | Command | Spec |
| --- | --- | --- | --- |
| 1 | Build dbgen/qgen from your kit, unmodified (make variables on the command line; no patch) | `bash scripts/build_kit.sh` | 2.1.4, 4.2.1 |
| 2 | Prove the build reproduces the TPC reference data (base, update and qgen parameters) | `bash scripts/verify_kit.sh 1` | 4.2.1.1, kit dbgen/reference/README.txt |
| 3 | Generate the population and the RF update sets | `bash scripts/gen_data.sh 1` | 4.2, 2.6-2.7 |
| 4 | Create our databases with per-database settings only, and snapshot the instance settings | `py/dbsetup.py snapshot / create` | 5.2.7 |
| 5 | Load the qualification database (SF 1) | `py/load.py --db tpch_qual --sf 1 --out R/qual` | 4.1.2, 4.3 |
| 6 | Validation query text (qgen -d) and the validation run against the reference answers | `py/gen_queries.py validation`, `py/validate.py` | 2.3, 2.1.3.5 |
| 7 | ACID tests on the qualification database (after validation) | `py/acid.py`, `py/durability.py` | 3 |
| 8 | Load test of the test database (timed) | `py/load.py --db tpch_sf1 --sf 1 --out R/sf1` | 4.3, 5.1.1.2 |
| 9 | Stream query text from the end-of-load seed | `py/gen_queries.py streams --load-json R/sf1/load.json` | 2.1.3.3, 5.3.5 |
| 10 | Performance test: Run 1 and Run 2, each a power test, then a throughput test with the refresh stream | `py/run_perf.py --runs 2` | 5.3 |
| 11 | FDR (full edition, for the raw run) | `py/fdr.py --run R/sf1 --qual R/qual [--notes notes.json]` | 8 |
| 12 | Public copy for the repository | `py/publish.py --raw R --test sf1 --out results/<date>-<platform>-sf1-analytics` | SPEC 6 |

Every script has `--help`. Raw results go in `R` (`$TPCH_WORK/results/<stamp>-sf<SF>/`), and the full raw run is
copied to `E:\tpc\results\`, outside any repository. Each run keeps its raw output (every query's rows, every
timing, qgen's raw and parameter logs) and a `manifest.json` (CPU, cores, memory, OS, PostgreSQL version,
non-default and per-database settings, the role, the kit revision and release.h, the reference-data check, build
hashes, SF, seeds and the git commit).

**What is published in the repository** (`results/`): timings, validation verdicts, ACID verdicts, manifests,
instance-settings snapshots, the seeds (`seeds.json`) and a public edition of the FDR. Not published: query text,
qgen output (raw text and parameter logs) and query output rows, because all of them are derived from TPC
material. Anyone with the kit regenerates the exact queries from the seeds (`py/gen_queries.py ... --seed0`).
`publish.py` copies an allow-list only, replaces validation mismatch values (which quote the reference answers)
with counts, and refuses the copy if anything resembling query text gets in.

## 4. What is measured, and how it is named
- **Load time**: from CREATE TABLE to the end of CHECKPOINT. It includes COPY, keys, indexes and VACUUM (ANALYZE)
  (clause 4.3; `load.json`).
- **Power test**: RF1, then the 22 queries in ordered set 0, then RF2, on separate query and refresh sessions
  (clause 5.3.3).
- **Throughput test**: S query streams (the clause 5.3.4 minimum: 2 at SF 1, 3 at SF 10) plus one refresh stream
  running S RF1/RF2 pairs (clause 5.3.4, 5.3.7.7).
- **Timing**: each query from the submission of its text to the submission of the next one, and the last query to
  its last row (5.3.7.2). Rounding follows 5.3.7.5, and Ts is rounded up per 5.3.6.2.
- **Derived figures** (clause 5.4, computed as sanity numbers only), named neutrally:
  "power-test figure", "throughput-test figure" and "composite queries-per-hour x SF (TPC-H-derived, not
  comparable)". The run reported is the one with the lower composite (5.3.2).
- **Validation**: PASS or FAIL per query against the kit's `dbgen/answers`, with the clause 2.1.3.5 precision rules
  (`py/answers.py` lists where they are stricter than the kit's cmpq.pl).

### Repeats: this track's exemption from SPEC.md 4.1
SPEC.md 4.1 asks for at least 5 runs per point. The analytics track follows TPC-H's own structure instead:
- **A result is one benchmark:** one load test, then Run 1 and Run 2. The two runs are the two samples within it,
  and the result reports the lower one (clause 5.3.2).
- **Why:** TPC-H defines its measurement as that sequence (clauses 5.1.1.1-5.1.1.5). Repeating a run inside one
  benchmark would apply more refresh pairs and evolve the database (clause 2.8), so a third run would no longer
  measure the same thing. Calling 5 such runs "repeats" would also not match what the TPC-H rules define.
- **Repeatability:** at least 3 full benchmarks per configuration, each on a fresh load, when time allows. They
  are reported as separate results, with their spread.
- The first SF 1 window does one full benchmark.

Design choices (indexes, keys, settings, RF transactions, isolation) are listed in COMPLIANCE.md and STATUS.md.
The query text changes are in [QUERY-MODIFICATIONS.md](QUERY-MODIFICATIONS.md).

## 5. Rig (PC x86, WSL2)
The PC rig is the WSL2 distro described in the project's outer-bounds study: 2 vCPU, 2 GB RAM and 512 MB swap, set
in `%USERPROFILE%\.wslconfig`, with the distro's ext4 vhdx on the E: NVMe. On a shared instance (the `oobe`
distro's PostgreSQL 16), the track creates its own databases and sets only per-database parameters
(`pg/tpch-database.conf`). The instance-wide settings are recorded, never changed, and snapshotted before and after.
On a dedicated rig, `scripts/setup_pg.sh init` creates a private cluster with `pg/postgresql.tpch.conf`.

Durability tests that kill PostgreSQL or reset the VM (`py/durability.py --failure ...`) run only on an instance
and VM nobody else uses, with the owner's agreement (`--own-instance`).

## 6. Tests
`python -m unittest discover -s analytics/tpch/tests -v` covers the logic that needs no database, on synthetic
data: dialect changes, answer comparison rules, timing rounding and the derived figures, the ACID arithmetic,
the RF file parsing, the FDR generator (full and public editions, including the version-mismatch and SF 10
reference-data statements) and the publisher's allow-list and query-text guard.
