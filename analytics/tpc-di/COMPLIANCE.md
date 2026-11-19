# Compliance checklist: batch ETL module (Derived from TPC-DI) against TPC-DI 1.1.0

> **This workload is derived from the TPC-DI Benchmark and is not comparable to published TPC-DI Benchmark results, as this implementation does not comply with all requirements of the TPC-DI Benchmark.** "Met" below means that the harness implements the
> rule. It is not an audit finding. TPC-DI 1.1.0 is obsolete at the TPC, so no result from this module can be a TPC
> result, audited or not.

The status reflects the state on **2026-11-19**. Nothing has run yet: no data has been generated and no SQL has
been executed, so every "impl" row is **not yet demonstrated** until a run's evidence path is added here.
Requirements are stated in a few words, and the clause is the authority.

Status key: **impl** = implemented, not yet run; **met** = holds by construction; **NOT MET** = cannot be met here,
with the reason; **open** = needs a decision or a download (STATUS.md); **n/a** = does not apply.
Columns: the PC (x86 Linux, private PostgreSQL cluster), a Linux on Z guest (shared), z/OS (idea only, README).

## Clause 0-1: general
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 0.2 | No benchmark specials | Stock PostgreSQL, general bulk-load settings only (pg/); the ETL uses no knowledge of the data beyond the spec | met | met | n/a |
| 1.6 | Warehouse class ACID or OPEN | OPEN, unless the ACID demonstration of clause 3.4.10.2 is done (not planned) | met (OPEN) | met (OPEN) | n/a |

## Clause 2: source data and Staging Area
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 2.2 | Parse every source format | etl.py and sql/: pipe-delimited, CSV, XML (iterparse), FINWIRE fixed-width (substring) | impl | impl | n/a |
| 2.3 | Batch1-3 folder structure | DIGen's own output folder, used as the Staging Area | met | met | n/a |
| 2.4.1, 8.4.1.1 | Staging Area durable across a single-medium failure and power loss | One NVMe, no mirror | NOT MET (needs RAID 1 or a second device) | NOT MET (storage not ours) | n/a |
| 2.4.3-2.4.5 | DIGen files only, unchanged, no extra files | gen_data.py writes its manifest beside the Staging Area, never inside it; checksums prove the files are unchanged | met | met | n/a |

## Clause 3: Data Warehouse
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 3.2 | All tables and columns, exact numeric types | sql/ddl/warehouse.sql: NUMERIC for NUM/SNUM, no floating point, no added or removed columns | impl | impl | n/a |
| 3.3.1 | Commercially available product | PostgreSQL: generally available with commercial support from several vendors. Our reading; an auditor might not share it (same as analytics/tpch) | open | open | Db2: met |
| 3.4.3 | Surrogate keys are not physical addresses | Sequences/identity columns | met | met | n/a |
| 3.4.6, 8.4.1.2 | Warehouse durability: single-medium failure, power loss | fsync, synchronous_commit and full_page_writes on; but WAL and data on one device | NOT MET for media failure; power loss only on a PC nobody else is using | NOT MET | n/a |
| 3.4.7 | Committed data stays visible | No table is truncated or swapped during a phase; checked by the visibility counts (checks.py) | impl | impl | n/a |
| 3.4.9.1 | No warehouse tables at the start of the run | run.py drops and creates the database (clause 7.2.3) | met | met | n/a |
| 3.4.9.2 | Space for seven more incremental updates | Operator check: free disk >= 7 x Batch2's growth; recorded in notes.json | open | open | n/a |
| 3.4.10 | ACID class demonstration | Not attempted; results are OPEN class | n/a | n/a | n/a |
| 3.4.11 | SQL as given, minor modifications only, disclosed (OID 3-1, 3-2) | Our stand-in queries are ours, not modified TPC queries. TPC appendix files, once downloaded, run as supplied; any PostgreSQL changes are added to run.py's `appendix_file` and listed here | open (appendix not downloaded) | open | n/a |

## Clause 4: transformations
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 4.2.1, 4.3.1 | Implemented with a commercially available DI System | **Our own SQL and Python.** Deliberate: one readable implementation for x86 and s390x | NOT MET (standing deviation) | NOT MET | n/a |
| 4.3.3 | Data dependencies honoured | sql/hist and sql/incr run in dependency order (numbered files) | impl | impl | n/a |
| 4.3.5.2 | Commit subsets only when complete and well-formed | etl.run_phase commits per step | impl | impl | n/a |
| 4.3.5.4 | Read only the current batch's folder | run_phase receives one BatchN folder | met | met | n/a |
| 4.3.5.6 | Warehouse open to other sessions from update 1 on | The visibility session (run.py) | impl | impl | n/a |
| 4.3.7 | No auxiliary structures from earlier runs | Database dropped and created per run | met | met | n/a |
| 4.4.1 | Type 2 history, daily grain, cascades | sql/hist, sql/incr; see the transformation table below | impl | impl | n/a |
| 4.4.2 | BatchID 0-3 per phase | run.py passes 0-3; checks.audit tests the range | impl | impl | n/a |
| 4.5, 4.6 | Each historical and incremental transformation | See "Transformations" below | impl | impl | n/a |

### Transformations (clauses 4.5 and 4.6)
Filled from the ETL implementation's review; every `DEVIATION` comment in sql/ is listed here.

| Clause | Target | File | Status |
| --- | --- | --- | --- |
| 4.5.2 | DimBroker | sql/hist/01_dimbroker.sql | impl |
| 4.5.3 | DimCompany (daily grain, unchanged records dropped, SPrating alert, isLowGrade) | hist/02 | impl |
| 4.5.14 | Financial (CIK or name lookup) | hist/03 | impl; **DEVIATION (4.5.14.2):** a name matching several companies takes the highest surrogate key, so each FIN record gives one row |
| 4.5.6 | DimSecurity (daily grain, DimCompany cascade 4.4.1.6.1) | hist/04 | impl; same name-match **DEVIATION (4.5.6.2)** |
| 4.5.4 | DimCustomer (carry-forward, phones, tax rates, prospect demographics, tier/DOB alerts) | hist/05 | impl |
| 4.5.1 | DimAccount (carry-forward, CLOSEACCT, INACT, DimCustomer cascade) | hist/06 | impl |
| 4.5.15 | Prospect (MarketingNameplate, IsCustomer, inserted-rows message) | hist/07 | impl |
| 4.5.8 | DimTrade (create/close from TradeHistory, fee/commission alerts) | hist/08 | impl |
| 4.5.9 | FactCashBalances (running total per account) | hist/09 | impl |
| 4.5.10 | FactHoldings | hist/10 | impl |
| 4.5.12 | FactWatches | hist/11 | impl |
| 4.5.11 | FactMarketHistory (52-week high/low, PE ratio, yield, no-earnings alert) | hist/12 | impl |
| 4.5.5, 4.5.7, 4.5.13, 4.5.16-4.5.18 | DimDate, DimTime, Industry, StatusType, TaxRate, TradeType | etl.py (direct COPY) | impl |
| 4.6.4 | DimCustomer | sql/incr/01_dimcustomer.sql | impl |
| 4.6.1 | DimAccount (customer cascade, INACTIVE) | incr/02 | impl |
| 4.6.14 | Prospect (change detection, messages, IsCustomer refresh) | incr/03 | impl |
| 4.6.7 | DimTrade (update existing, insert new) | incr/04 | impl |
| 4.6.8 | FactCashBalances | incr/05 | impl; **DEVIATION (4.6.8.1):** one row per account per batch, dated the Batch Date |
| 4.6.9 | FactHoldings | incr/06 | impl |
| 4.6.10 | FactMarketHistory (52-week window over stored history) | incr/07 | impl |
| 4.6.11 | FactWatches | incr/08 | impl |
| 4.6.2, 4.6.3, 4.6.5, 4.6.6, 4.6.12, 4.6.13, 4.6.15-4.6.17 | tables with no incremental work | none | met (no-ops by the clause) |

Interpretation choices where the clauses are silent or loose (ours, open to challenge; listed so a reviewer can check
them against the spec):
- Point-in-time surrogate-key lookups use EffectiveDate <= d < EndDate, also for the customer lookup of 4.5.1.3,
  which the clause states with both ends inclusive.
- Customer status is compared upper-cased, because the historical load writes 'ACTIVE' while the updates copy
  StatusType names.
- Every SEC record becomes a DimSecurity version; only DimCompany drops unchanged records.
- A NULL or zero EPS sum counts as "no earnings" (NULL PE ratio plus an alert).
- Prospect matching: missing fields match missing fields; ties go to the lowest AgencyID (historical) or to this
  batch's file, then the latest SK_RecordDateID (incremental).
- The DImessages result extract leaves out both visibility message types (clause 8.2.2's filter names a type that
  the clause 3.2.8 list does not use).
- NOT NULL restrictions were read from a table-mode extraction of spec pages 37-46, because the layout-mode text
  misaligns several tables.
- load_audit also loads the root-level Batch1-3_audit.csv files that DIGen writes beside the batch folders (to be
  confirmed on the first generation).

## Clause 6: DIGen
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 6.1, 6.2.1 | Source data from DIGen | gen_data.sh runs the kit's DIGen | impl | impl | n/a |
| 6.2.2, 6.2.4 | DIGen and its PDGF unmodified | The kit is copied, not changed; only a `pdgf -> PDGF` link is added in the copy (FEASIBILITY.md R2); checksums in build-info.txt | met | met | n/a |
| 6.2.3 | DIGen version matches the spec (1.1.0) | build_kit.sh refuses another version | met | met | n/a |
| 6.2.6 | JVM at least Java SE 7 | Java 8 pinned; build_kit.sh refuses < 7 | met | met | n/a |
| 6.3 | digen_report.txt kept unmodified, used for the row counts | Read only; metrics.parse_digen_report fails loudly on an unexpected format | impl | impl | n/a |

## Clause 7: execution rules and metrics
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 7.1.2 | Timings to 0.1 s, rounded up | common.round_up_tenth, unit-tested | met | met | n/a |
| 7.1.3 | No outside interaction during the run | One command; operator only monitors; disclosed in the FDR | impl | impl | n/a |
| 7.2.1.1 | One SF for all batches; SF keeps each update <= 3600 s | One DIGen run per SF; run.py flags an update over 3600 s | impl | impl | n/a |
| 7.2.2 | Phases in order, no pause, no intervention | run.py | impl | impl | n/a |
| 7.2.3 | Clean SUT; no Staging Area access in initialization | Drop/create database; etl.init reads no source data | impl | impl | n/a |
| 7.2.3-7.2.5 | Batch validation, then a PCR, at the end of every phase | run.py `batch_validation` and `pcr` | impl | impl | n/a |
| 7.2.6 | Audit phase: load audit files (header skipped), final visibility 1, audit query, all OK | etl.load_audit; run.py; **our stand-in audit until the TPC appendix is downloaded** | open | open | n/a |
| 7.3 | Visibility queries every <= 5 min, own session | run.py Visibility thread (default every 240 s); the FDR reports the largest gap | impl | impl | n/a |
| 7.4 | Batch validation query | Our stand-in (checks.batch_validation) or the TPC's file | open | open | n/a |
| 7.5 | Elapsed times from PCR timestamps; rows from digen_report; 1800 s floor | metrics.figures, unit-tested | met | met | n/a |
| 7.6 | Combined metric and price/performance | Computed with the same arithmetic under a neutral name; never called by the TPC metric name. No price/performance figure | met | met | n/a |

## Clause 8: qualification
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 8.1 | Qualification on the same SUT at SF 5, same DI application | run.py --qual (enforces SF 5), same settings file | impl | impl | n/a |
| 8.2.2 | Result files, pipe-delimited, defined formats | py/extract.py with checks.RESULT_TABLES | impl | impl | n/a |
| 8.2.1.1, 8.3 | Compare with the TPC's expected results using the TPC's tool | **Tool and expected data not downloaded** | open | open | n/a |
| 8.4 | Durability tests | Not possible on this rig (one device, shared machine) | NOT MET | NOT MET | n/a |

## Clauses 9-11: pricing, FDR, audit
| Clause | Requirement | How we meet it | PC | Linux on Z | z/OS |
| --- | --- | --- | --- | --- | --- |
| 9 | Priced configuration under TPC-Pricing | Optional appendix by docs/methods/pricing.md; list prices only | open | open | n/a |
| 10 | Full Disclosure Report | py/fdr.py: FDR-TEMPLATE sections plus the DIT/OID items; not a TPC FDR | impl | impl | n/a |
| 11 | Independent audit | None; impossible for an obsolete benchmark | NOT MET | NOT MET | n/a |
