# OLTP module derived from TPC-E (ROADMAP T3): status

Status: **waiting for the kit.** Updated 2026-11-19.

## Verified
- The plan is written: [PLAN.md](PLAN.md).
- The public TPC-E specification, revision 1.14.0, was downloaded from tpc.org to `E:\tpc\TPC-E-spec\` (outside this
  repository) and was the source for the plan.
- DBT-5 v0.11.0 (osdldbt/dbt5, Artistic-2.0) was released 2026-10-31. It is the proposed base (PLAN.md section 5).

## Waiting on
1. **The TPC-E Tools (EGen) kit** (`TPC-E_Tools_v1.14.0.zip`), being fetched by the maintainer. Each user downloads it
   from tpc.org and accepts the TPC EULA. It is never committed here (SPEC.md section 6).
   - **Possibly already on disk:** a TPC download named `035B2DD7-...-TPC-E-Tool.zip` (2.8 MB, 831 entries) sits with
     the other TPC kits.
   - Its `ChangeLog.txt` begins "1.14.0b1 to 1.14.0 ... Bless 1.14.0b1 as 1.14.0", which reads as EGen 1.14.0.
   - It was listed but **not extracted**. The maintainer should confirm that it is the intended kit before it is
     extracted to `E:\tpc\TPC-E\`.
2. A decision on PLAN.md's Option A (DBT-5 as the base) against writing our own driver and frames.
3. A decision on the TPC Policies v6.20 naming ("Derived from" prefix, the 8.1.5 disclaimer, and written permission to
   use the name) for SPEC section 6.

## Next, once the kit is confirmed
1. Extract it to `E:\tpc\TPC-E\`. Read its EULA. Record `GetEGenVersion()` and the checksums of the kit tree.
2. Build DBT-5 v0.11.0 against that EGen on x86 Linux. This is a compile, so it needs a quiet machine and must not run
   during a timed benchmark.
3. Dev load at 1,000 customers and a short smoke run. Then measure the database size and correct PLAN.md section 3's
   estimates.
4. Write `scripts/`, `py/` and `config.example.env` on the `analytics/tpch/` pattern. Write a pre-registered hypothesis
   (SPEC 4.7) before the first measured run.

## Not started
- Code, loads and runs. No results exist.
