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
- **Batch and online together needs changes to CardDemo as shipped** (its CSD and JCL at commit 59cc6c2, 2025-10-16):
  - All 18 programs are defined CONCURRENCY(QUASIRENT), so a CICS region runs their code on its QR TCB, where "only
    one task at a time can be running" (IBM CICS documentation).
  - All 8 files are STRINGS(1), RECOVERY(NONE), DATABUFFERS(2) and INDEXBUFFERS(1), in LSR pool 1. For an LSR file,
    STRINGS "specifies the number of concurrent requests that can be made against that specific file".
  - The README's batch sequence starts with CLOSEFIL, which closes TRANSACT, CCXREF, ACCTDAT, CXACAIX and USRSEC in
    CICS (`CEMT SET FIL(...) CLO`), and reopens them with OPENFIL near the end (WAITSTEP and the optional CBPAUP0J
    follow). POSTTRAN's JCL allocates the VSAM clusters to its batch program (CBTRN02C) directly, and the clusters
    that ACCTFILE, CARDFILE, CUSTFILE, XREFFILE and TRANFILE define are SHAREOPTIONS(2 3): without RLS, one user at a
    time may open such a data set for read/write.
  - So running batch and online together on z/OS needs:
    - a way for batch to share the files with CICS, either:
      - VSAM record-level sharing (RLS), which needs a coupling facility lock structure (IGWLOCK00), and z/OS
        Transactional VSAM (TVS), a separately priced feature, if the files are made recoverable (LOG(UNDO) or
        LOG(ALL)); with LOG(NONE), TVS is not required; or
      - batch file access routed through CICS: through the external CICS interface (EXCI), programs running in z/OS
        can call CICS application programs in a CICS region, so the batch programs would call CICS programs to read
        and update the files instead of opening them;
    - threadsafe program definitions, or several CICS regions, so that application code is not limited to one QR
      TCB;
    - STRINGS and buffer counts sized to the measured concurrency.
  - Each change is part of the z/OS configuration and is disclosed with it (SPEC 4.3).
  - Sources (fetched 2026-11-25): CardDemo, https://github.com/aws-samples/aws-mainframe-modernization-carddemo
    (`app/csd/CARDDEMO.CSD`, `app/jcl/CLOSEFIL.jcl`, `app/jcl/ACCTFILE.jcl` and the other file-load jobs, and the
    README's "Running Batch Jobs" order); IBM CICS documentation on quasi-reentrant programs
    (https://www.ibm.com/docs/SSGMCP_5.5.0/fundamentals/threadsafe/quasi-reentrant.html), on STRINGS for LSR files
    (https://www.ibm.com/docs/en/SSJL4D_6.x/tuning/dfht33x.html) and on the RLS lock structure
    (https://www.ibm.com/docs/en/SSGMCP_5.4.0/installing/dfha1dd.html) and on EXCI
    (https://www.ibm.com/docs/en/SSGMCP_5.4.0/configuring/cics/dfha2_open_close_batch.html); IBM, "VSAM RLS/TVS
    Overview", on share options, RLS and TVS
    (https://www.ibm.com/support/pages/system/files/inline-files/RLS%20Overview%202025.pdf).
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

## Optional modules
Optional modules derived from industry-standard benchmark material are not described here until the
benchmark owners' permission is in place (SPEC 1.6).

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
- **Products:** CICS TS, Db2 for z/OS (for the database variants and the optional modules), ICSF, zEDC enabled,
  Semeru Java, and Open XL C/C++ or equivalent build tools.
- **Systems programmer help:** a WLM service class (or classes) for the benchmark work, and access to the SMF and RMF
  records above. Both need authority that an ordinary user ID doesn't have.

## Suggested order
1. Java track (fastest route to equal-code z/OS numbers, plus the zIIP effect).
2. COBOL/CardDemo track.
3. Optional modules, once permission is in place.

Access and the availability of Db2 for z/OS are the gating items, not the code.
