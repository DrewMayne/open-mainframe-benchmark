# C2 (derived from TPCx-V): compliance map

For each TPCx-V rule, this file says whether the C2 harness implements it, and how any rule it cannot meet is
disclosed. The rules are paraphrased; the clause numbers are for TPCx-V revision 2.1.10 (the PDF is kept outside this
repository).

Key:
- **auto:** `checks.py` scores it pass/fail from the kit's output.
- **kit:** the TPC kit enforces or reports it, and we keep its evidence.
- **op:** the operator states it in `notes.json` for the FDR.
- **dev:** a known deviation for this platform, always listed in the FDR.

| Rule (paraphrased) | Clause | C2, PC with 10 LU | C2, server with 50 LU | How |
| --- | --- | --- | --- | --- |
| Use the TPC kit as provided; only `vcfg.properties` and `testbed.properties` may change | 1.5 | met if no kit patch is needed | same | `build-info.txt` checksums of the kit tree, against the download |
| Any kit fix is reported to the TPC and listed in the FDR | 1.5 | **dev** if a patch is used | same | patch kept beside the kit; FDR lists file, reason, diff hash (the diff itself stays out of the repo) |
| VMMS commercially available; not static partitioning; I/O virtualised; one Node, one VMMS | 1.1 | met (KVM/libvirt or Hyper-V) | met | manifest: hypervisor, version, disk and network models |
| 3 VMs per Group, 4 Groups per Tile; Tier A has no database | 1.1, 4.3.4 | met (1 Tile) | met | manifest: VM roles |
| At least 5,000 customers per database; at least 50 LU per Tile; 10 LU steps; Groups at 1:2:3:4 | 2.4.1.2-3 | **dev** (10 LU: 1,000-4,000 per database) | met (50 LU) | `checks.py` from `runtime.properties` |
| Initial population by the kit's generator, with ITD 125 and SF 500 left at default | 2.4.1 | met | met | flat-file checksums; generator log |
| Configured LU equal to active LU | 2.4.1.7, 5.6.8.6 | met | met | auto |
| Clocks in sync, checked before and after | 4.3.2 | met | met | auto (kit check plus our own log) |
| Ramp-up of at least 12 min; Measurement Interval of exactly 2 h covering all 10 Phases | 5.6, 5.6.5.1 | met | met | auto from the mix log |
| Response-time limits at the 90th percentile per type; 99% of Market-Feed within its limit; Data-Maintenance within its limit | 5.5.1.2, 5.5.1.6 | measured; pass/fail reported | same | auto |
| Average response time not above the 90th percentile, per type | 5.5.1.4 | measured | same | auto |
| Transaction mix inside the required ranges | 5.3 | measured | same | auto (kit reporter, then our check) |
| Measured rate 80-102% of nominal; each Group 98-102% of its share | 5.7.1.2-3 | measured; a fail is a result, not hidden | same | auto |
| Throughput sustainable for a Business Day (8 h) from the start of Steady State | 5.6.3 | **dev** (2 h only on the PC; an 8 h run optional) | optional 8 h run | op |
| Database on Durable Media no more than 12 min older than any committed state | 5.6.5 | met through the checkpoint settings | same | manifest: `checkpoint_timeout` and WAL settings |
| ACID: the kit's xVAudit suite passes (all 11 reports) | 6, 9 | run after every measured run | same | kit; audit summary in the FDR |
| Durability: Loss of Processing of a VM (kill and restart), with throughput at least 95% of reported, no errors | 6.5 | can be run on our own host (`virsh destroy` of a Tier B VM mid-run) | same | op + raw logs |
| Pricing (TPC-Pricing), price-performance metric | 7 | **dev**: not produced | **dev** | SPEC FDR section 8 (list-price estimate only, our own metric name) |
| Independent audit | 9 | **dev**: none | **dev** | label: not comparable |
| Primary metric name | 5.7.1, Policies 8.1.4 | never used | never used | `fdr.py` refuses to print it |
