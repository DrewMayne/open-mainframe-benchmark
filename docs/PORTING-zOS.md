# Porting to z/OS (draft, 2026-11-19)

How each part of the benchmark gets to z/OS, what it needs from the system, and roughly how long it takes. Nothing here
has been run on z/OS yet; the effort figures are estimates.

## Principle: three thin adapters, one body of business logic
Each track keeps its business logic identical on every platform and swaps three adapters:
- **Data access:** indexed files or PostgreSQL on Linux; VSAM or Db2 for z/OS on z/OS.
- **Transport:** the TCP worker pool on Linux; CICS (web services / REST via CICS Liberty, or a socket listener) for
  the COBOL track on z/OS, and the JVM's TCP server for the Java track.
- **Accelerators:** zlib, OpenSSL and ONNX Runtime on Linux; zlib with zEDC/DFLTCC, ICSF/CPACF and zDNN on z/OS.

The load generator always runs off the system under test and reaches it over TCP.

## Java track (about 1-2 weeks)
- Runs on IBM Semeru Java under z/OS UNIX. Java work is zIIP-eligible, so results are reported with and without zIIPs
  (hypothesis H6).
- **Changes:**
  - EBCDIC: set the file encoding explicitly, and name the charset in all file and network I/O;
  - crypto through z/OS JCE providers backed by CPACF/ICSF;
  - compression through java.util.zip (zEDC/DFLTCC);
  - AI scoring through zDNN via JNI;
  - the keyed store through JZOS VSAM access, or Db2 via JDBC;
  - batch jobs started from JCL with the JZOS batch launcher.

## COBOL track, CardDemo (about 3-6 weeks)
- CardDemo is a z/OS application already: CICS, VSAM, JCL and BMS, installed with its own JCL and CSD definitions.
- **Driver interface:** CardDemo is driven by 3270 screens, so the benchmark needs small wrapper programs that take the
  same inputs as the Linux worker pool. They are exposed through CICS web services / REST or a CICS socket listener, and
  call the business logic with a COMMAREA or channel. This is the main piece of work.
- **The accelerated extension in COBOL:**
  - signing via ICSF callable services;
  - compression via a small Language Environment-callable C routine over zlib/zEDC;
  - AI scoring via a C wrapper around zDNN.
- **Batch:** CardDemo's JCL, with DFSORT (and `OPTION ZSORT` where eligible) in place of GNU sort.
- **Data conversion:** generated data must be converted to CardDemo's record layouts: EBCDIC text, plus packed-decimal
  and zoned-decimal fields. The converter needs its own validation (round trip, and the golden business results).

## Analytics and TPC-derived modules
All follow SPEC section 6.
- **Derived from TPC-H, on Db2 for z/OS (about 1-2 weeks):**
  - build dbgen with z/OS Open XL C (watching for EBCDIC), or generate off-box and transfer;
  - load with the Db2 LOAD utility;
  - run the queries via JDBC, with the dialect changes the specification permits;
  - an accelerator (IDAA) run is an optional, separately labelled variant.
- **Derived from TPC-E, on Db2 for z/OS (about 3-4 weeks):** EGen built with Open XL C++ or run off-box; transactions as
  Db2 stored procedures, driven from off-box.
- **Derived from TPC-DI (about 2-3 weeks):** generator run on z/OS Java or off-box; ETL as COBOL, DFSORT and Db2 batch.
  This is the most z/OS-native module.
- **Derived from TPCx-V:** not applicable to z/OS. On IBM Z, consolidation means Linux guests under z/VM or KVM, which is a
  separate port.

## Measurement on z/OS (about 1-2 weeks)
- **Data sources:** SMF type 30 (job CPU, split into general CP and zIIP), SMF type 110 (CICS per-transaction
  performance) and RMF / SMF types 70-79 (LPAR, WLM, DASD) take the place of Linux /proc sampling.
- A parser turns them into the same manifest and CSV columns as on Linux.
- The manifest must record:
  - the processor capacity setting (full-speed or sub-capacity);
  - CP and zIIP counts;
  - LPAR weight and capping;
  - the WLM service classes used.

  These are needed both for honest comparison and for the planned MIPS calibration.

## Functional verification versus measurement
Porting work on a system that has not granted benchmark permission is **functional verification only**:
- compiles, installs and small pass/fail checks with tiny inputs (tens or hundreds of records, single transactions,
  one-shot jobs);
- no loops, ramps or concurrency, no overrides of site limits, one job at a time, and a stop on any refusal;
- **no performance data is recorded or published from such a system**, not even incidental elapsed times.

Every performance figure comes only from a system whose terms allow benchmarking, or from an agreed window with the
owner's written permission.

## What the system must provide
- Sanctioned access with agreed windows (CONTRIBUTING.md).
- **Products:** CICS TS, Db2 for z/OS (for the database variants and the TPC-derived modules), ICSF, zEDC enabled,
  Semeru Java, and Open XL C/C++ or equivalent build tools.
- **Systems programmer help:** a WLM service class (or classes) for the benchmark work, and access to the SMF and RMF
  records above. Both need authority that an ordinary user ID doesn't have.

## Suggested order
1. Java track (fastest route to equal-code z/OS numbers, plus the zIIP effect).
2. COBOL/CardDemo track.
3. Analytics on Db2 for z/OS, then the batch ETL module derived from TPC-DI.
4. The OLTP module derived from TPC-E.

Access and the availability of Db2 for z/OS are the gating items, not the code.
