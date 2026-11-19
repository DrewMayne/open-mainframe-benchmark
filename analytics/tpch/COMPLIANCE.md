# Compliance checklist: Derived from TPC-H analytics track, against TPC-H revision 3.0.1

> **This workload is derived from the TPC-H Benchmark and is not comparable to published TPC-H Benchmark results, as this implementation does not comply with all requirements of the TPC-H Benchmark.**
>
> "Met" below means that the harness implements the
> rule. It is not an audit finding, and no result from this track is a TPC result.

The status reflects the state on **2026-11-19**. Nothing has run on a database yet, so every "implemented" row is
**not yet demonstrated** until a run's evidence path is added here. Requirements are stated in a few words, and the
clause is the authority.

Status key:
- **impl** = implemented, not yet run;
- **met** = holds by construction (no run needed);
- **NOT MET** = cannot be met on that platform, with the reason;
- **open** = needs a decision (see STATUS.md, questions);
- **n/a** = does not apply.

The three status columns are the PC (WSL2 VM, shared PostgreSQL 16 instance on the `oobe` rig), a Linux on Z guest
(shared), and z/OS with Db2 (phase 2, not built).

## Clause 0-1: general and logical database design
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 0.2 | No benchmark specials | Stock PostgreSQL; general DSS settings only (pg/) | met | met | n/a yet |
| 1.3, 1.4.1 | Tables, columns, datatypes | sql/schema.sql, written from the clause; mapping in its header; NUMERIC(15,2), CHAR/VARCHAR, DATE | met | met | n/a yet |
| 1.3.1 | Identifier range above SF 300 | `/*ID*/` columns switch to BIGINT when SF > 300 (load.py) | met | met | n/a yet |
| 1.4.2.1-.3 | Optional constraints; all FKs or none | NOT NULL everywhere; all 8 PKs; all 10 FKs; no CHECKs (sql/constraints.sql) | met | met | n/a yet |
| 1.4.2 | Constraints enforced and validated before the load ends | FKs created VALID inside the timed load | impl | impl | n/a yet |
| 1.5.1 | Commercially available DBMS | PostgreSQL 16 is generally available open source. We have not priced commercial support, so the pricing side of "commercially available" is not addressed | met for availability; pricing not addressed | met for availability; pricing not addressed | Db2: met |
| 1.5.3 | Exact row counts after load | load.py checks the counts against clause 4.2.5 and against dbgen's file line counts | impl | impl | n/a yet |
| 1.5.4-1.5.6 | Partitioning and replication limits | None used | met | met | n/a yet |
| 1.5.7 | Auxiliary structures: one table, one permitted column set each | sql/indexes.sql: FK column sets plus two date columns | met | met | n/a yet |
| 1.5.8-1.5.12 | Names; no added or split columns | As the clause | met | met | n/a yet |
| 1.6 | Data access transparency | No partitioning, so nothing to hide | met | met | n/a yet |

## Clause 2: queries and refresh functions
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 2.1.2.3 | Unique entity names per stream | qgen `-p s` puts the stream number into Q15's view name | met | met | n/a yet |
| 2.1.2.9 | Return exactly N rows (Q2, 3, 10, 18, 21) | Method 3: `limit N`, applied consistently (dialect.py) | met | met | n/a yet |
| 2.1.3.3 | Seeds: seed0 = end-of-load mmddhhmmss; streams seed0+s | load.json `seed0`, used by gen_queries.py; Run 2 reuses Run 1's seeds (permitted) | impl | impl | n/a yet |
| 2.1.3.5 | Validation precision rules | answers.py, including the clause comments for Q1, Q8, Q12, Q14, Q17 and Q18; stricter than cmpq.pl for ratios | met (unit-tested) | met | n/a yet |
| 2.1.4.1 | qgen must generate the query text | The kit's qgen, built unmodified | impl | impl | n/a yet |
| 2.1.4.3, 4.2.1.3 | dbgen/qgen version must match the spec | The kit's release.h reports 3.0.0 build 0, but the kit and specification are 3.0.1 (TPC packaging). We use the kit as distributed and disclose the mismatch in every FDR (fdr.py `version_line`) and manifest | NOT MET as distributed; disclosed | same | same |
| 2.2.3 | Minor modifications only, disclosed | QUERY-MODIFICATIONS.md (MOD-1, LIMIT, DELIM) | met | met | n/a yet |
| 2.2.4 | Approved variants | None used | met | met | n/a yet |
| 2.3.1 | Validation on an unmodified SF 1 qualification DB, same text and driver, default parameters | validate.py refuses a DB with RF or ACID history; it uses driver.py, as the performance test does | impl | impl | n/a yet |
| 2.3.2 | No SUT change between validation and the performance test | Same instance, same per-DB settings; instance snapshot before and after (dbsetup.py) | impl | impl | n/a yet |
| 2.5.2 | RF transactions keep the database consistent | Each RF is one transaction holding both tables' changes | met | met | n/a yet |
| 2.5.3.1, 5.3.7.10 | Only generation and relocation before the RF; formatting inside its timing | Reading the file and stripping the `\|` happen inside the RF interval | met | met | n/a yet |
| 2.6, 2.7 | RF data from dbgen -U | gen_data.sh; refresh.py | impl | impl | n/a yet |
| 2.8 | Database evolution tracked | rfstate-<db>.json, with before and after key checks around each set | impl | impl | n/a yet |

## Clause 3: ACID
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 3.1.5, 5.2.3 c1 | ACID tests on the qualification DB; same session attributes as the streams | acid.py; db.connect roles: query sessions for the ACID query, refresh sessions for the ACID transaction | impl | impl | n/a yet |
| 3.1.6 | ACID transaction and query, HISTORY table | acid.py (`acid_txn`, `acid_query`; the arithmetic is unit-tested); sql/acid_history.sql | impl | impl | n/a yet |
| 3.2.2 | Atomicity: commit and rollback | `atomicity` | impl | impl | n/a yet |
| 3.3.3 | Consistency: a sample of 10 orders; at least 100 transactions per stream; at least S+1 streams | `consistency --streams 4` (covers S=2 at SF 1 and S=3 at SF 10) | impl | impl | n/a yet |
| 3.4.2.1-.6 | Isolation tests 1-6 | `iso1`-`iso6`. MVCC plus SELECT ... FOR UPDATE; the readers in tests 1 and 2 do not wait, which is disclosed as a non-locking scheme | impl | impl | n/a yet |
| 3.5.2.2 | Durable success file, outside the DB | durability.py: success and in-flight logs, fsync per record | impl | impl | n/a yet |
| 3.5.3 | Instantaneous interruption (crash, reboot) | `--failure sysrq` (guest reboot without sync) or `kill9` (DBMS only) | NOT MET on the shared oobe instance (it would kill the other study); impl for a dedicated VM | NOT MET on a shared guest unless the owner agrees | n/a yet |
| 3.5.3 | Loss of memory | Covered by the sysrq reset (the guest's memory is lost). The Windows host may keep vhdx blocks in its cache, which we disclose | as above | as above | n/a yet |
| 3.5.3 | SUT power failure | Manual host power cut, procedure only (durability.py `--failure none`, then cut power). Needs a PC nobody else is using. The E: drive is a consumer NVMe (Samsung 980 PRO), which we believe has no power-loss protection (to be confirmed from the datasheet), so durability rests on fsync and flush reaching the media | NOT MET until run on a dedicated PC with the owner's agreement | NOT MET (the hypervisor and power are not ours) | n/a yet |
| 3.5.3 | Permanent failure of one durable medium | Would need WAL and archives on a second device (or RAID 1) plus a restore. Not built | NOT MET (one NVMe holds data and WAL) | NOT MET | n/a yet |

## Clause 4: scaling and population
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 4.1.2 | Qualification DB like the test DB in every way but size | The same load.py, SQL and per-DB settings; separate DB `tpch_qual` | impl | impl | n/a yet |
| 4.1.3.1 | Permitted SFs only | common.check_sf and gen_data.sh refuse others | met | met | met |
| 4.1.3.3, 8.3.7.7 | Data storage ratio | fdr.py computes it from operator-supplied storage | open (operator input) | open | open |
| 4.2.1 | dbgen must populate; the population must match | The kit's dbgen, unmodified; verify_kit.sh byte-compares it with ref_data | impl | impl (the s390x portability check) | n/a yet |
| 4.2.1 | Reference data exists for the SF | The kit ships ref_data for 1, 100, 300, 1000 and up, **not 10 or 30**, so SF 10 cannot be checked against a reference (SF 1 proves the build). Every FDR states "SF <n> cannot be checked against TPC reference data" when that applies (fdr.py `reference_line`) | SF 1: impl; SF 10: NOT MET | same | same |
| 4.3 | Load time from CREATE TABLE to durable, ready state; all components disclosed | load.py phases, ending with CHECKPOINT. The `tpch` role has CREATEDB and pg_checkpoint, not superuser; it is recorded in the manifest | impl | impl | n/a yet |
| 4.3.9 | Generation resources excluded | dbgen runs before the timed load, in the same VM | met | met | n/a yet |

## Clause 5: metrics and execution rules
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 5.1.1.3-.5 | Two runs, Run 1 first after the load | run_perf.py refuses Run 1 unless RF set 1 is next | impl | impl | n/a yet |
| 5.1.1.2, 5.3.1.4 | No queries in the load; only harmless activity between load and Run 1 | Only qgen runs in between; disclosed in the manifest | impl | impl | n/a yet |
| 5.2.3 | One session per stream; query sessions initialised identically | db.connect: no session settings; same application_name per role | met | met | n/a yet |
| 5.2.7 | Settings fixed for the whole test; nothing table- or query-specific | Per-DB settings set once. Instance-wide settings belong to the instance owner: snapshotted before and after, and the result is invalid if they differ | impl | impl | n/a yet |
| 5.2.8 | Same statistics quality for every column | VACUUM (ANALYZE) at default_statistics_target 100 | met | met | n/a yet |
| 5.3.3 | Power test: RF1, 22 queries, RF2 | run_perf.py `power_test` | impl | impl | n/a yet |
| 5.3.4 | Throughput: at least the minimum S streams plus 1 refresh stream with S pairs | `throughput_test`; S is checked against Table 11 | impl | impl | n/a yet |
| 5.3.5 | Query order per Appendix A | qgen `-p s`; the order is cross-checked against the kit's permute.h | impl | impl | n/a yet |
| 5.3.6, 5.3.7 | Ts and interval definitions; 0.001 s clock; rounding | driver.py, metrics.py (perf_counter; rounding unit-tested) | met (unit-tested) | met | n/a yet |
| 5.3.7.8 | Refresh pair scheduling | Pairs run back to back from the start of the throughput test (our choice) | met | met | n/a yet |
| 5.4.1-5.4.3 | Power, throughput and composite figures, including the 5.4.1.4 adjustment | metrics.py, reported only under neutral names | met (unit-tested) | met | n/a yet |
| 5.3.2 | Report the lower run | metrics.reported_run | met | met | n/a yet |
| 5.4.4, 5.4.6 | Price/performance and the required public components | Not produced: the figures are not TPC metrics and are not published as such | NOT MET (by design; SPEC.md 6) | same | same |
| 5.4.5 | Different SFs are not comparable | Every result names its SF. We never compare across SFs | met | met | met |

## Clauses 6-9: SUT, driver, pricing, reporting, audit
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 6.2, 6.3 | SUT and driver definition | The driver (Python) runs inside the SUT VM and uses its CPUs. Disclosed in the FDR | met (disclosed) | met (disclosed) | n/a yet |
| 7 | Priced configuration (TPC-Pricing) | Not done. Milestone T1 adds a list-price appendix that is not TPC-priced | NOT MET | NOT MET (shared Z has no list price for a slice) | NOT MET |
| 8.1, 8.2 | FDR as PDF, executive summary, three supporting zips | fdr.py writes a markdown FDR that follows the project template plus a clause 8.3 index; the raw files stay in the run folder. No PDF, zips or TPC executive-summary layout | NOT MET (format) | same | same |
| 8.3 | FDR contents | fdr.py fills every item it can measure; the rest is marked "TODO (operator)" | impl | impl | n/a yet |
| 8.3.8.8 | Times truncated to 0.01 s | metrics.truncate_hundredth | met | met | n/a yet |
| 9 | Independent audit by a TPC-certified auditor | None. This is the end goal only if a sponsor funds it (SPEC.md 1.6) | NOT MET | NOT MET | NOT MET |

## TPC Policies v6.20 (use of the name, derived works)
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| Policies 8.1.4 | The prefix "Derived from" before every use of the TPC-H name for our work | README, COMPLIANCE, STATUS, the FDR and the figure names say "Derived from TPC-H ..." (common.LABEL and the names); the unit tests check that the FDR and README carry no "TPC-H-derived" wording | met | met | met |
| Policies 8.1.4 | A derived work too similar to the standard is subject to fair use (8.2); using the name needs the TPC's written permission | **Written TPC permission is being requested** before any result using the name is published. Until it is granted, publish.py refuses to write the public copy unless `TPC_PERMISSION=granted` (unit-tested) | open (permission requested) | same | same |
| Policies 8.1.4 | No Primary or Optional Metric of the standard in a derived work | Metric names are never used. The composite figure is still computed with the clause 5.4.3 formula under a neutral name, which may count as using the metric: open question in STATUS.md | open | open | open |
| Policies 8.1.5 | The derived-work disclaimer, verbatim | common.DISCLAIMER in every FDR (both editions) and in the README, COMPLIANCE and QUERY-MODIFICATIONS (unit-tested against the policy text) | met | met | met |

## Project run rules that interact with TPC-H
- SPEC.md 4.1 (at least 5 runs per point): **exempted** for this track (coordinator's decision, 2026-11-19; see the
  README, "Repeats"). A result is one benchmark: one load, then Run 1 and Run 2. Repeatability is at least 3 full
  benchmarks per configuration on fresh loads, when time allows.
- SPEC.md 4.6 (raw output kept and published) is limited by SPEC.md 6. The repository gets timings, verdicts,
  manifests, seeds and a public FDR (py/publish.py). Query text, qgen output and query output rows stay with the
  full raw run outside the repository (`E:\tpc\results\`).
- SPEC.md 4.4, pinning: the WSL2 VM's vCPUs float on the host. The manifest records SMT and the governor where the
  guest can see them.
