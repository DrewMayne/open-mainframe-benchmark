# Status: Derived from TPC-H analytics track (milestone A1)

Updated: 2026-11-19, before GO. Status: **built and unit-tested; nothing has run on a database yet. Waiting for
"GO: realreq finished".**
All files are uncommitted in the working tree (branch `main` of the local clone), for review.

## Done (verified)
- Kit extracted to `E:\tpc\TPC-H_v3.0.1\` (renamed from the zip's "TPC-H V3.0.1", 83 MB), outside every repo.
  EULA, dbgen README, check_answers and reference READMEs, and spec clauses 1-5 and 8 were read.
- `analytics/tpch/` written. No TPC code, template, spec text or reference data is in it. Scripts read the kit
  through `TPCH_KIT`.
- Unit tests: `python -m unittest discover -s analytics/tpch/tests` gives **28 tests, OK** (Windows Python
  3.11.3). They cover:
  - the dialect changes, and the clause 2.1.3.5 comparison rules with their overrides;
  - the 5.3.6/5.3.7.5 rounding, the 5.4 figures and the 5.4.1.4 adjustment;
  - the ACID arithmetic, RF parsing and the seed format;
  - the FDR generator, full and public editions: label at the top, all sections present, no TPC metric names on
    our numbers, the release.h mismatch line, and the "SF 10 cannot be checked" statement;
  - the publisher: allow-list, sanitised validation file, and refusal of query-like text.
- Dry check (in memory, nothing written): over the kit's 22 templates, the dialect step adds LIMIT to exactly Q2,
  Q3, Q10, Q18 and Q21 (clause 2.1.2.9's list), applies MOD-1 to Q1 only, and splits Q15 into three statements.
  permute.h parses as 41 ordered sets.
- `bash -n` passes on every script, and every Python module compiles.

## Decisions recorded (coordinator, 2026-11-19; the instructor is offline)
1. **Repeats:** the track follows TPC-H's structure. A result is one benchmark: one load, then Run 1 and Run 2.
   Repeatability means at least 3 full benchmarks per configuration on fresh loads, when time allows. This is the
   track's exemption from SPEC 4.1, noted in the README ("Repeats"). The coordinator adds the matching line to
   SPEC. Tonight's window: 1 full benchmark at SF 1.
2. **Role:** a `tpch` role on oobe with CREATEDB and membership in `pg_checkpoint`, no superuser. It stays until
   the SF 10 window and is dropped at the final teardown. It is recorded in every manifest
   (`postgresql.role`), and the preflight refuses to start without both privileges.
3. **Network:** pip from PyPI inside oobe is fine; `psycopg==3.2.3` stays pinned.
4. **Results in the repo:** publish timings, validation verdicts, manifests, the FDR and the seeds, but no qgen
   output or query text. `py/publish.py` writes the public copy to `results/<stamp>-pc-wsl2-sf1-analytics/`:
   - it copies an allow-list only;
   - validation mismatch values become counts, because they quote the reference answers;
   - it adds `seeds.json` and the public FDR;
   - it refuses the copy if anything looks like query text.

   It also leaves out query output rows (my reading: those rows are derived from TPC data and answers). The full
   raw run, query text included, goes to `E:\tpc\results\` only.
5. **Clause 1.5.1:** COMPLIANCE.md now says "met for availability; pricing not addressed".
6. **FDR disclosures:** every FDR now states the release.h 3.0.0 versus 3.0.1 mismatch, and states "SF <n> cannot
   be checked against TPC reference data" whenever the kit ships no reference data for that SF (SF 10 and 30).
7. **TPC Policies v6.20, 8.1.4 and 8.1.5** (checked by the coordinator, policies text lines 3085-3109):
   - every public mention of the name for our work now reads "Derived from TPC-H ...";
   - every FDR (both editions) and the README carry the 8.1.5 disclaimer verbatim (`common.DISCLAIMER`);
   - written TPC permission is being requested. Until it is granted, `py/publish.py` refuses to write the public
     results copy unless `TPC_PERMISSION=granted`. In the SF 1 window this means the public-copy step prints
     REFUSED while the raw run is copied to `E:\tpc\results\` as before. Unit-tested.

## Waiting for GO (the SF 1 window, about 1.5-2 h, on the existing `oobe` rig)
Setup before the run:
1. Create the role, as an administrator in oobe:
   `CREATE ROLE tpch LOGIN CREATEDB; GRANT pg_checkpoint TO tpch;`. Its password goes in `~/.pgpass` inside the
   distro, never in a repo file.
2. Write `config.env`: `TPCH_DISTRO=oobe`, `PGHOST`/`PGPORT`/`PGUSER=tpch`, and the database names.

Command, from Windows Git Bash: `bash analytics/tpch/scripts/wsl.sh bash scripts/sf1_window.sh`. It runs:
1. preflight: gcc, make, Python 3.8+, psql reach, CREATEDB and pg_checkpoint;
2. a venv with psycopg 3.2.3, then the unit tests;
3. build_kit, then verify_kit at SF 1 (byte comparison with ref_data/1);
4. gen_data SF 1 (9 RF pairs);
5. a snapshot of the instance settings;
6. create `tpch_qual` and `tpch_sf1`, with per-database settings only;
7. load the qualification database, then the validation run (it stops if validation fails);
8. the timed load of tpch_sf1, then stream queries from its end-of-load seed;
9. Run 1 and Run 2;
10. the full FDR;
11. a second snapshot and the compare (expected result: "instance settings unchanged");
12. the public copy into the repo's `results/`;
13. the full raw copy to `E:\tpc\results\`.

Nothing is torn down. The databases are kept unless `DROP_DBS=1`.

Rough time at SF 1 on 2 vCPU (an estimate, not measured):
- build and verify: under 5 min;
- dbgen: about 1 min;
- each load: 3-6 min;
- validation: 5-15 min (Q17, Q20 and Q21 dominate on PostgreSQL);
- each run's power and throughput tests: 15-30 min.

If Q17 or Q20 runs far longer than that, I'll report it with the plan (GEN-05) rather than add a timeout.

## Not in this window
- SF 10: a later window. E: needs 30 GB or more free: about 11 GB of data plus about 25 GB for the database.
- ACID tests: written (`RUN_ACID=1` adds them after validation), but not in tonight's scope.
- Durability by crash or power loss: not on the shared oobe instance, because it would kill the other study's
  PostgreSQL. It needs a dedicated VM and the owner's agreement.

## Findings so far (disclosed in the FDR and COMPLIANCE.md)
1. The kit's `dbgen/release.h` says **3.0.0 build 0**, while the kit and the spec PDF are 3.0.1. Clauses 2.1.4.3
   and 4.2.1.3 want the versions to match. We use the kit as distributed and disclose the mismatch.
   (`makefile.suite` also carries stale VERSION=2 and RELEASE=13 variables, which are unused.)
2. The kit ships reference data for SF 1, 100, 300, 1000 and up, **but not for SF 10 or 30**, so an SF 10
   population cannot be checked against TPC reference data. The SF 1 check proves the build.
3. The kit's `check_answers/cmpq.pl` tests ratios ("rat") as an absolute difference of 1, where clause 2.1.3.5 (b)
   says within 1%. It also skips the clause's comments: exact SUM(l_quantity) in Q1 and Q18, exact Q12 sums, and
   both rules for Q8, Q14 and Q17. `py/answers.py` follows the specification.
4. The consistency condition (clause 3.3.2.1) gets its first check on real dbgen data at the first ACID run. If
   dbgen's O_TOTALPRICE does not satisfy it exactly, that is a finding, not something to loosen.

## Open questions
- **Policies 8.1.4, second paragraph:** "The use of any Primary Metric or Optional Metric of a TPC Benchmark
  Standard in a work that is derived from TPC material is not allowed."
  - The harness computes the composite with the clause 5.4.3 formula and prints it under a neutral name in both
    FDR editions. It does the same for the power and throughput figures, which the spec calls secondary numerical
    quantities, not primary metrics.
  - A neutral name may not be enough if the number is the primary metric itself.
  - Options: (a) keep it, and ask about it in the permission request; (b) leave the composite out of the public
    FDR until permission is granted, and keep it in the full edition as an internal sanity number; (c) drop it.
  - Nothing is changed until you decide.
- One reading for the coordinator to confirm: the public copy also leaves out query **output rows** (decision 4
  listed what to publish but did not name output rows).

## Uncommitted
Branch `main` of `C:\Users\Andrew\source\open-mainframe-benchmark`. The harness was committed by someone else as
`da59611` ("Analytics track A1: TPC-H-derived harness ..."; that subject line predates the "Derived from TPC-H"
wording).

The Policies 8.1.4/8.1.5 update is uncommitted, for review:
- modified: COMPLIANCE.md, QUERY-MODIFICATIONS.md, README.md, STATUS.md, pg/postgresql.tpch.conf, py/common.py,
  py/fdr.py, py/publish.py, sql/schema.sql, tests/test_pure.py;
- new: `.gitattributes`, which keeps LF endings in a Windows checkout. Every working file had CRLF endings, which
  would break the bash scripts in WSL. They are now LF; git's diff ignores the change because the committed blobs
  are already LF.

The public results folder (`results/`) appears only after a run, and only with `TPC_PERMISSION=granted`.
