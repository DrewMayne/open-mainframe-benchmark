# Platform facts: LinuxONE Community Cloud, s390x toolchain and x86 accelerators

Desk research, 2026-11-19. Facts may change; each carries its source URL.
Marking: **fetched** means the page itself was retrieved and read. **snippet only** means the text came only from a search-engine snippet and the page was not retrieved.

---

## Q2. Terms of use: benchmarking, sustained load, publishing results

**Verdict: EXPRESSLY PROHIBITED.** Benchmark, performance and stress testing are banned, and so is publishing their results. This rests on a 2021 version of the terms that I could not fetch myself (see the evidence status below).

Clauses (Section 5 of "LinuxONE Community Cloud Terms & Conditions, Version 9.0, June 11, 2021", hosted by Marist at http://tc.l1cc.marist.edu/linuxone/TC.PDF). **Snippet only**, seen in three separate searches:

> "The LinuxONE Community Cloud may not be used for stress testing, performance testing or benchmark testing. Furthermore, publishing any stress testing, performance testing or benchmark testing results is strictly prohibited."

> "The services are available to develop, test, port or otherwise try open source software on the z System architecture hardware."

The same snippets also say the service is "as-is", that access is "up to 120 days", that no data backups are made, and that proprietary-software work is prohibited. All of this is snippet only.

Evidence status, in full:
- **http://tc.l1cc.marist.edu/linuxone/TC.PDF:** NOT fetched. WebFetch got ECONNREFUSED on 148.100.90.20:443, and a plain HTTP `curl` on port 80 timed out after 21 s. The only source for the text is the search snippets. The search index titles the document "Version 9.0 June 11, 2021 LinuxONE Community Cloud Terms & Conditions".
- **http://security.marist.edu/LinuxOne/TC.PDF** (the older link, cited from an Open Mainframe Project page): fetched. It gives a 302 redirect to https://www.marist.edu/information-security, a general page, so the document is no longer there.
- **https://www.ibm.com/community/z/ibm-linuxone-community-cloud-terms-and-conditions/:** fetched. It gives a 302 redirect to https://community.ibm.com/zsystems/l1cc/terms/. I fetched that page too, but it renders only its title ("Terms and Conditions: IBM LinuxONE Community Cloud"), because the body is loaded by JavaScript. I could not read any clause there. A snippet of the IBM page describes the service as free access "to virtual servers and services for testing and piloting emerging applications for evaluation purposes" and says it links to separate "Terms for accessing the Community Cloud hosted at Marist College" (snippet only).
- **No newer version found, but a 2026 copy may exist.** Several searches for a 2022-2026 version of the terms found nothing. However, a fetched IBM Community thread (https://community.ibm.com/community/user/question/problem-with-starting-the-ibm-linuxone-community-cloud-instance-again-please-help) shows a library entry titled "The LinuxONE Community Cloud Terms and Conditions", added 02/10/26 by Bruce Gilkes (the IBM "LinuxONE Community Cloud Architect and Project Leader", per https://community.ibm.com/community/user/discussion/taking-the-lnx1-linux1-course-where-can-one-join-linuxone-community-cloud-to-request-virtual-server). I could not read its contents: the group library (https://community.ibm.com/community/user/groups/community-home/librarydocuments?CommunityKey=8c2c15eb-f059-4e7c-8cb6-5fb713a7806c) does not list documents without a login. That 2026 document may restate or revise Section 5, and only someone logged in can tell which.
- The same January 2026 thread (fetched) reports that the Community Cloud support address "linuxone.community@us.ibm.com** doesn´t seem to exist anymore". The thread was closed by an administrator with no IBM reply.
- **Older conflicting statement:** a 2015 Enterprise Times article said partners could "port, test and benchmark" on the cloud (snippet only, https://www.enterprisetimes.co.uk/2015/08/17/linuxone-the-first-linux-only-mainframe/). It predates the 2021 terms and is superseded by them.
- **Other limits, from a 2019 IBM Japan deck (fetched, https://speakerdeck.com/sierraone47/try-linuxone-community-cloud):** slide 11 says production workloads are not allowed ("本番ワークロードは稼働できません (開発・評価用なので)") and that VMs may be deleted after 30 days of inactivity. The deck does not mention benchmarking.
- **Enforcement:** an Open Mainframe Project FAQ, quoted in a search summary, says Marist may change or shut down an instance that violates the terms (snippet only; the page https://openmainframeproject.org/?p=9479 now returns 404).

**What this means for the project:** the Community Cloud cannot be the s390x leg of a published benchmark. Even running the workload to measure it breaks the clause, and publishing the numbers is "strictly prohibited". If a newer version of the terms exists, it sits behind the JS-rendered IBM page or the unreachable Marist host. Anyone relying on a change must read the current terms, or get written permission from the Community Cloud project, first.

---

## Q1. LinuxONE Community Cloud today (2026)

**Who runs it.** It is a joint IBM and Marist service, hosted at Marist College. The Open Mainframe Project only lists it; OMP does not run it.
- The 2021 IBM deck (fetched, text extracted, https://www.sluug.org/resources/presentations/media/2021/STLLINUX/2021-04-22_STLLINUX.pdf) says it was "Developed as a joint project between IBM and Marist College, starting in 2015".
- SHARE New Orleans 2023 (fetched, https://www.share.org/Events/Past-Events/Proceedings/Proceeding-Details/15614-linuxone-community-cloud-using-ibm-cloud-infrastructure-center-icic-to-bring-zlinux-to-the-world) says: "Since 2016, IBM and Marist College have worked jointly to provide a zLinux testing platform".
- The IBM project lead is Bruce Gilkes, "LinuxONE Community Cloud Architect and Project Leader", IBM (fetched, 2024 thread: https://community.ibm.com/community/user/discussion/taking-the-lnx1-linux1-course-where-can-one-join-linuxone-community-cloud-to-request-virtual-server).
- OMP lists it among third-party resources (fetched, page updated 2025-09-03: https://openmainframeproject.org/news/developer-resources-for-linux-on-s390x/).

**Sign-up.** The entry point is https://community.ibm.com/zsystems/l1cc/. The page is JS-rendered, so a fetch returned only its title.
- The steps come from a search snippet (https://community.ibm.com/zsystems/l1cc/, snippet only): "Try Virtual Machines", then "Request your trial" (with an optional event code), then a confirmation email carrying the portal user ID and password.
- The same snippet notes a known registration problem with outlook.com and hotmail.com addresses.
- Deployment, from the fetched guide (https://github.com/linuxone-community-cloud/technical-resources/blob/master/faststart/deploy-virtual-server.md): Service Catalog, then Virtual Servers, then "General purpose VM".
- The 2021 IBM deck (fetched) says a "valid email address is required, plus affirm standards of use agreement".

**Eligibility and cost.** It is free and open to anyone; there is no academic gate.
- The 2023 IBM blog (fetched, https://community.ibm.com/community/user/blogs/bruce-gilkes1/2023/03/28/get-access-to-linuxone) says "Open to users worldwide" and "no cost, no credit card".
- An IBM snippet describes it as for "developers, students, and entrepreneurs ... for testing and piloting emerging applications for evaluation purposes" (snippet only, https://www.ibm.com/community/z/ibm-linuxone-community-cloud-terms-and-conditions/).
- The 2021 terms restrict use to open-source work (Q2).

**VM size.** No current published figure.
- The fetched deploy guide says only "There is only one at this time" for the flavor and gives no numbers.
- The only numbers found are from 2019: "VCPU 2 RAM 4GB DISK 50GB" (fetched, slide 6, https://speakerdeck.com/sierraone47/try-linuxone-community-cloud).
- Treat the size as UNKNOWN until a VM is provisioned.

**Term and renewal.** The sources conflict.
- "Get up to 120 days of free access" (2023 IBM blog, fetched).
- "The initial trial lasts 60 days, with the option to renew for another 60 days" (OMP, updated 2025-09-03, fetched).
- The 2021 IBM deck (fetched) says "120 days at a time".
- Expired servers are deleted, and a moderator said there is "no mechanism to restore servers" (Jan 2022 thread, snippet only, https://community.ibm.com/community/user/discussion/trial-expired).
- The 2019 deck (fetched) adds possible deletion after 30 days of inactivity.
- "There is no backup for your virtual server" (deploy guide, fetched).

**Hardware generation.** Not documented for 2026.
- Latest documented: "Current L1CC resides on LinuxONE Emperor III (z15 technology)" (2021 IBM deck, fetched).
- Before that: Marist installed the "world's first" LinuxONE III in October 2019 to power the cloud (fetched, https://www.marist.edu/-/marist-first-linuxone-iii).
- 2019: Emperor II (z14) and Rockhopper II (z14 ZR1) (fetched, 2019 deck slide 7).
- I found NO source saying it moved to LinuxONE 4 (z16) or LinuxONE 5 (z17), so whether it has Telum or an AIU is unknown. Running `lscpu` or reading `/proc/cpuinfo` on a VM would show the machine type (8561 = z15, 3931 = z16, 9175 = z17).

**Shared vs dedicated, SMT.** Not documented anywhere I found. Being a z/VM guest in a "shared public cloud" (2023 blog, fetched) implies shared IFLs, but that is inference, not a source.

**Hypervisor: z/VM.**
- "The latest iteration of the cloud is based on ICIC (IBM Cloud infrastructure Center) on z/VM" (SHARE 2023, fetched).
- 2021: "Three hypervisors running z/VM 7.1", with ICIC 1.1.1 in production since August 2020 (2021 IBM deck, fetched).

**OS images.** RHEL, SLES and Ubuntu.
- The 2023 IBM blog (fetched) lists "Red Hat Enterprise Linux 9", "SUSE Linux Enterprise Server 15" and Ubuntu without a version, plus OpenShift with "additional registration required".
- A search snippet mentions Ubuntu 24.04 and RHEL 9.8 guides (snippet only, from the first search; source page not identified).
- No authoritative current list was found.

**Root, packages, network.** Root login itself is off, but sudo works.
- From the fetched deploy guide: "The user 'root' login is disabled for security reasons. No modification is allowed." You log in as `linux1` with an SSH key, and "You can use 'sudo' to execute commands that require root authority."
- Password authentication is not allowed. "Only the SSH port is open by default"; other ports are opened with iptables.
- The 2023 blog says "full root access". OMP says "root access to a Linux-based virtual machine".
- So package installs through sudo should work. Whether you can set cgroups inside the guest is not documented (cgroup v2 depends on the distro, so see Q4).
- No published network bandwidth limit was found.

**Service health, 2026.** A Jan 2026 user post (fetched, https://community.ibm.com/community/user/question/problem-with-starting-the-ibm-linuxone-community-cloud-instance-again-please-help) reports an instance that would not restart, and says the support email "doesn't seem to exist anymore". The thread was closed with no IBM reply.

**What this means for the project.** Sign-up is easy and free, and the VM is fine for porting and functional tests of the kit. But:
- size, hardware generation, SMT and dedication are all unknown;
- the hardware was z15 as of 2021, which means no AIU;
- the VM is a shared z/VM guest, which is the wrong basis for a fair performance number even before the terms (Q2) rule benchmarking out.

---

## Q3. Alternatives

**IBM Cloud LinuxONE Virtual Server for VPC (s390x), being withdrawn.**
- From the fetched profiles page (https://github.com/ibm-cloud-docs/vpc/blob/master/s390x-vsi-profiles.md, updated 2026-08-14): "As of 28 February 2026, you can't create new instances. Existing instances are supported until 20 February 2027." Any instance still existing then is deleted.
- The VPC release notes announcing it on 20 Feb 2026 also deprecate Dedicated Host for VPC (s390x), Hyper Protect Virtual Server for VPC and Wazi as a Service (snippet only, https://github.com/ibm-cloud-docs/vpc/blob/master/release-notes.md; my fetch was truncated before February).
- Profiles, from the same fetched page:
  - bz2 is 1 vCPU : 4 GiB, up to bz2-16x64;
  - cz2 is 1 : 2, up to cz2-16x32;
  - mz2 is 1 : 8, up to mz2-16x128;
  - Secure Execution variants are bz2e, cz2e and mz2e;
  - 2 vCPUs per core (that is, SMT-2 pairs), with bandwidth up to 32 Gbps.
- The hardware generation is not named on the page. A SUSE guide says Hyper Protect needs "IBM z16 or IBM LinuxONE 4" (snippet only, https://documentation.suse.com/trd/ibm/html/gs_sles_ibm-hpvs/index.html).
- I did not look up prices, because the service is closed to new orders.
- Hyper Protect Virtual Servers for VPC are deprecated in the same notice.
- **Not usable for a new Phase 1.**

**IBM Z / LinuxONE Partner Network developer access.** There is a page at https://zlpn.ibm.com/public/ ("Developer System Access IBM Z and IBM LinuxONE Partner Network", snippet only). It is aimed at ISV partners. Terms not checked.

**IBM SkillsBuild Technology Access (formerly the Academic Initiative).** A third-party 2026 guide says access needs an institutional email and is "limited to classroom, learning, and non-commercial research purposes only" (snippet only, https://aicreditmart.com/ai-credits-providers/ibm-academic-initiative-free-ai-training-credits-2026-guide/). I found no evidence that it gives LinuxONE VMs. It covers software and cloud credits.

**Open Mainframe Project mainframe access program.** New, announced about 2026-11-05.
- "Apply to the TAC". The work must be "an open source project, not commercial work", and should benefit the mainframe ecosystem. IBM handles setup. Hardware generation and benchmark rules are not stated. (Fetched: https://tfir.io/mainframe-ci-access-open-source-linux-foundation/)
- A related "s390x Compute Commons" proposal was closed 2026-09-22, with the reply "we already have the mainframe access program" (fetched, https://github.com/openmainframeproject/tac/issues/1129).

**IBM LinuxONE Open Source Software Cloud.** Offers "longer-lived VMs for continued development or for use in your CI system", by request form; the project must be open source (fetched, OMP page above).

**CI services.** OSU Open Source Lab IBM Z Jenkins (https://osuosl.org/services/ibm-z/), Travis CI s390x, and IBM-hosted GitHub Actions runners (all fetched via the OMP page). These are CI services, not benchmark hosts.

**University programmes.** Marist itself hosts the Community Cloud. Separately, the IBM Z Datathon 2026 gave students IBM Z/LinuxONE access, but registration closed 16 Oct 2026 (snippet only, https://hack2skill.com/event/datathon-2026). No other university LinuxONE benchmark programme was found.

**What this means for the project.**
- IBM's only pay-as-you-go s390x cloud stopped taking new instances on 28 Feb 2026.
- The realistic routes to sanctioned s390x time are:
  - the OMP access program, applying with the kit as an open-source project;
  - the LinuxONE Open Source Software Cloud;
  - a direct, written arrangement with IBM or Marist.
- With each one, get **written permission to run and publish performance numbers** before running anything. None of the program pages says benchmarking is allowed.

---

## Q4. The s390x toolchain

**GnuCOBOL.** Available as distro packages on s390x.
- Latest upstream release: 3.2, released 2023-07-28. No later release appears in the project news (fetched, https://sourceforge.net/p/gnucobol/news/ and https://sources.debian.org/src/gnucobol3/3.2-4/NEWS).
- Debian trixie ships `gnucobol3` 3.2-3 with an s390x build (fetched, https://packages.debian.org/trixie/gnucobol3).
- Ubuntu 24.04 ships 3.1.2-5.1ubuntu1 for s390x, in universe (fetched, https://packages.ubuntu.com/noble/gnucobol3).
- The Debian runtime `libcob4t64` depends on `libdb5.3t64`, so the distro build uses the **Berkeley DB** indexed backend (fetched, https://packages.debian.org/trixie/libcob4t64).
- 3.2 also offers `--with-vbisam`, `--with-cisam` and `--with-disam` (snippet only, https://sourceforge.net/projects/gnucobol/files).
- An LMDB handler shows up only in 2019 development-tree notes, not in 3.2 (snippet only, https://lists.endsoftwarepatents.org/archive/html/bug-gnucobol/2019-12/txtuloaiL4Lvj.txt). Treat LMDB as unavailable in a release.
- No RHEL/SLES package was found. On those distros you would build from the 3.2 tarball.

**PostgreSQL.** Available.
- "The s390x architecture is now fully supported for Debian and Ubuntu on apt.postgresql.org" (2023-11-23, fetched, https://postgresql.org/about/news/postgresql-on-s390x-on-debian-and-ubuntu-2752).
- A Sept 2025 report says the PGDG jammy (Ubuntu 22.04) s390x InRelease file omitted s390x (snippet only, https://www.postgresql.org/message-id/301fd5da-f14f-44ff-b4cc-7d5b0a393c63%40arenadata.io). Prefer Ubuntu 24.04+ or Debian.

**GNU sort.** Part of coreutils, which every distro ships; Debian trixie has coreutils 9.7-3 for s390x (fetched, https://packages.debian.org/trixie/coreutils). Note that GNU sort has no hardware assist on Z (the z/OS DFSORT/SORTL acceleration does not apply), so it is a plain-CPU comparison.

**zlib with DFLTCC.**
- **Hardware:** z15 or later. "SystemZ deflate hardware acceleration is available on IBM z15 and newer machines" (fetched, https://raw.githubusercontent.com/zlib-ng/zlib-ng/develop/arch/s390/README.md).
- **Upstream zlib does not have it.** madler/zlib PR #410 (`./configure --dfltcc`) is still Open (fetched, https://github.com/madler/zlib/pull/410). Distros carry the IBM patch instead: for example, AlmaLinux/RHEL `zlib-1.2.12-IBM-Z-hw-accelerated-deflate.patch` (snippet only, https://git.almalinux.org/rpms/zlib/blame/commit/872735211ab9f1436eaf4567b3047b25063f1a9c/zlib-1.2.12-IBM-Z-hw-accelerated-deflate.patch).
- **Levels and the environment variable:** built with only the flag, the PR says "zlib would compress in hardware on level 1, and in software on all other levels". Levels 1-6 can use hardware via `--dfltcc-level-mask=0x7e` at build time or "`export DFLTCC_LEVEL_MASK=0x7e` at run time" (PR #410, fetched).
- **zlib-ng:** `cmake -DWITH_DFLTCC_DEFLATE=1 -DWITH_DFLTCC_INFLATE=1`. Decompression always uses hardware; add `-DDFLTCC_LEVEL_MASK=0x7e` for levels 1-6. `DFLTCC=0` disables it, but only if built with `-DWITH_DFLTCC_ENV=1` (fetched, zlib-ng README above).
- **Ubuntu:** on by default since 20.10. The zlib changelog reads "Enable hardware compression on s390x at level 6", and the gzip changelog reads "Enable DFLTCC optimisations on s390x by default" (Fix Released 2021-01-29, fetched, https://bugs.launchpad.net/ubuntu-z-systems/+bug/1884514). The bug notes the gzip build lacked the 0x7e mask, and whether it was added later is not confirmed.
- **RHEL 8:** "Enabled HW compression for compression levels 1 through 6 (#1847438)" (Oracle Linux 8 changelog, snippet only, https://yum.oracle.com/repowatch/oraclelinux8.x86_64/ol8_baseos_latest/633f24f59f6aa9b25f172baec7085951b30183c13337f6d564972194984eae05).
- **RHEL 10:** the `zlib-ng-compat` entry reads "Enable DFLTCC on s390x - Resolves: RHEL-135504" (snippet only, https://yum.oracle.com/repowatch/oraclelinux10.x86_64/ol10_baseos_latest/e2a30b9bd1e752c7db640693c8c3544eb88a0a4f).
- **cpuinfo flag:** "Reported with new feature flag in /proc/cpuinfo: dflt" (2021 IBM deck, fetched, https://www.sluug.org/resources/presentations/media/2021/STLLINUX/2021-04-22_STLLINUX.pdf).
- **Reproducibility trap:** hardware deflate output differs from software output and may differ between calls. Red Hat closed the issue NOTABUG: "zlib does not guarantee a particular compressed data structure" (fetched, https://bugzilla.redhat.com/show_bug.cgi?id=1974658). **Golden files must compare decompressed bytes, never compressed bytes.**

**OpenSSL with CPACF.**
- OpenSSL PR #9348 added "accelerate ECDSA for NIST P-256, P-384 and P-521 using KDSA instruction" and PCC scalar multiplication, gated on the MSA9 facility bit (fetched, https://github.com/openssl/openssl/pull/9348). PR #11188 backported this to 1.1.1 and fixed a bug where `OPENSSL_s390xcap=z15` disabled the hardware support (snippet only, https://github.com/openssl/openssl/pull/11188).
- MSA9 ECDSA P256/P384/P521 and Ed25519/Ed448 are z15 features: "IBM z15 Support: CPACF ... New Message Security Assist MSA9", "Up to 38x sign operations" (2021 IBM deck, fetched).
- **So KDSA ECDSA needs z15 or later.**
- AES, GHASH and SHA in Linux use CPACF (fetched, https://www.ibm.com/docs/en/linuxonibm/lddd/lddd_r_crypt_cpacf_reqs.html). Hashes are enabled on every machine. AES and other ciphers need the no-charge feature code 3863 activated on the machine: "Without 3863, only SHA is supported" (snippet only, IBM Client Center paper as summarised by search; https://www.ibm.com/docs/en/linuxonibm/lxos/lxos_hw_cpacf.html).
- `openssl info -cpusettings` shows the capabilities in use (snippet only, https://github.com/golang/go/issues/82520).

**IBM zDNN on Linux.** Packaged in Ubuntu for s390x only (fetched, https://packages.ubuntu.com/search?keywords=libzdnn&searchon=names&suite=all&section=all):
- 22.04: 0.4.0
- 24.04: 1.0.1
- 26.04 LTS: 1.1.2
- The packages are `libzdnn0` and `libzdnn-dev`.
- Fedora packages it as well (snippet only, https://bugzilla.redhat.com/show_bug.cgi?id=2008090).
- The README (zDNN 1.2.0) says "zDNN API 1.0.x indicates the API requires Telum I or greater" and "1.1.x ... requires Telum II or greater". INT8 is "supported with Telum II". The NNPA check is `zdnn_is_nnpa_installed()` (fetched, https://raw.githubusercontent.com/IBM/zDNN/main/README.md).
- **So the hardware floor is z16 / LinuxONE 4, and z17 / LinuxONE 5 for INT8.**
- llama.cpp's zDNN backend supports z17 / LinuxONE 5 only and needs libzdnn >= 1.1.2 (fetched, https://raw.githubusercontent.com/ggml-org/llama.cpp/HEAD/docs/backend/zDNN.md).

**ONNX Runtime and the IBM Z Deep Learning Compiler (zDLC).**
- ONNX Runtime's build matrix lists S390X as supported on Linux, as a source build (fetched, https://onnxruntime.ai/docs/build/inferencing.html). No official PyPI s390x wheel was found.
- Ubuntu has an `onnxruntime` 1.13.1 s390x build (snippet only, https://launchpad.net/ubuntu/+source/onnxruntime/1.13.1+dfsg-1/+build/28445852).
- I found **no zDNN/NNPA execution provider for ONNX Runtime**.
- IBM's NNPA route for ONNX is zDLC, which uses ONNX-MLIR to compile `.onnx` models to shared libraries. The container is `icr.io/ibmz/zdlc:5.1.0`, and the license is Apache-2.0 (fetched, https://github.com/IBM/zDLC).
- zDLC CPU code uses SIMD on z13+. NNPA use needs "the Integrated Accelerator for AI available on IBM z16 and z17", with `-march=z16`/`-march=z17` (same page).
- Upstream: https://github.com/onnx/onnx-mlir (`-DONNX_MLIR_ACCELERATORS=NNPA`, snippet only).

**IBM Semeru / OpenJDK.**
- Semeru Open Edition 27.0.1.0 is offered for Linux s390x (tar.gz and rpm). The download page lists Java 8, 11, 17, 21, 25, 26 and 27, but the per-version s390x rows were truncated in my fetch (fetched, https://developer.ibm.com/languages/java/semeru-runtimes/downloads/).
- Red Hat ships `java-21-ibm-semeru-certified-jdk` 21.0.11 for s390x (snippet only, https://access.redhat.com/errata/RHSA-2026:22328).
- Distro OpenJDK exists on all s390x distros; I did not check a specific version.

**cgroups v2 and cpu.weight.**
- RHEL 9: "by default, mounts and uses cgroups-v2" (snippet only, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/considerations_in_adopting_rhel_9/assembly_kernel_considerations-in-adopting-rhel-9; the fetch got 403).
- Ubuntu since 21.10: systemd is switched to the unified cgroup hierarchy (cgroup v2) by default (snippet only, https://discourse.ubuntu.com/t/impish-indri-release-notes).
- `cpu.weight` is a standard cgroup v2 controller file and does not depend on the architecture. Inside a z/VM guest it only shares the guest's own virtual CPUs; z/VM SHARE settings decide what the guest gets.

**perf/PMU in a guest: none under z/VM.** "On z/VM the CPU Measurement facilities do not exist", and `/sys/devices/cpum_cf/events` is missing (Thomas Richter, IBM, LKML 2025-11-13, fetched, https://lkml.iu.edu/hypermail/linux/kernel/2511.1/07661.html). A 2026 patch skips perf stat tests on z/VM because cycles and instructions events are unsupported (snippet only, https://ratatoskr.run/linux-s390/2026/09/17502879/t).

**What this means for the project.**
- COBOL (with BDB), PostgreSQL, sort, OpenSSL and Java are all straightforward on s390x Debian or Ubuntu.
- Compression offload needs z15+.
- ECDSA offload needs z15+ plus CPACF enablement.
- AI offload needs z16+ (and z17 for INT8), and goes through zDNN or zDLC, not ONNX Runtime.
- On a z/VM guest such as the Community Cloud there are no hardware counters. The kit must measure with wall-clock time and `/proc` or `getrusage` CPU time, not `perf stat`.
- For DFLTCC, compare golden output after decompression.

---

## Q5. The x86 side, for fairness

**Ryzen 7 5800X (Zen 3).** AVX2, FMA3, AES-NI, VAES (256-bit) and SHA extensions; **no AVX-512**, no AMX, no QAT.
- "Zen3 supports all modern instruction sets including AVX2, FMA3 and even more like SHA HWA but not AVX-512", plus "VAES 256-bit (vs. AES HWA 128-bit)" (fetched, https://www.sisoftware.co.uk/?p=2946).
- TechPowerUp likewise lists AES, AVX and AVX2 and notes no AVX-512 (snippet only, https://techpowerup.com/review/amd-ryzen-7-5800x/3.html).
- So the matching mainframe offloads have only software counterparts on this PC: zlib and zlib-ng with AVX2 for compression, AES-NI/SHA-NI for crypto, and AVX2 int8 or FP32 for inference.

**Intel QAT, built in from 4th Gen Xeon (Sapphire Rapids) onwards, but only on some SKUs.**
- "all 4th Gen Intel Xeon Scalable Processors support the Intel QAT Acceleration Technology", but "depending on the Processor SKU, there may or may not be QAT End Points available". The example given is that the Xeon Gold 6430 has none, and the advice is to check ARK under "Advanced Technologies" (fetched, Intel, 2023-06-02, https://community.intel.com/t5/Intel-QuickAssist-Technology/All-Sapphire-Rapids-supports-QAT/m-p/1491793).
- Sapphire Rapids parts have "0, 1, 2 or 4 QAT accelerators, depending on the processor SKU" (snippet only, https://lenovopress.lenovo.com/lp1923.pdf).
- 5th Gen (Emerald Rapids): "QAT is only active on half of the new SKUs" (fetched, ServeTheHome, 2023-12-14, https://www.servethehome.com/5th-gen-intel-xeon-scalable-emerald-rapids-resets-servers-by-intel/5/).
- Xeon 6 (Granite Rapids): supported by the QAT driver; per-SKU endpoints are unconfirmed (snippet only, https://www.intel.com/content/www/us/en/support/articles/000094285/technologies/intel-quickassist-technology-intel-qat.html).
- Intel's AI tuning guide: "Up to 4x cryptography and data compression workloads acceleration with Integrated Intel QuickAssist Technology (QAT)" (fetched, https://intel.com/content/www/us/en/developer/articles/technical/tuning-guide-for-ai-on-the-4th-generation.html).

**AMX and AVX-512 VNNI.**
- 4th Gen Xeon "provides two instruction sets viz. AMX_BF16 and AMX_INT8" (fetched, Intel guide above).
- AVX-512 VNNI fuses vpmaddubsw, vpmaddwd and vpaddd into vpdpbusd, and is on 2nd-4th Gen Xeon (same page).
- **AMX is the closest x86 counterpart to the Telum AIU for INT8 inference.**

**AMD EPYC.**
- Zen 4 (Genoa) added AVX-512 including VNNI and BF16, "double-pumped" over 256-bit units (snippet only, https://www.tomshardware.com/reviews/amd-4th-gen-epyc-genoa-9654-9554-and-9374f-review-96-cores-zen-4-and-5nm-disrupt-the-data-center/2; https://www.phoronix.com/review/amd-zen4-avx512/1).
- I found no on-die compression or crypto offload engine comparable to QAT on EPYC. QAT is Intel's product. That is an absence of evidence from my searches, not a sourced statement.

**Cloud bare metal with QAT/AMX (rough on-demand prices).**
- **AWS:**
  - M7i is "powered by custom 4th Generation Intel Xeon Scalable processors (code named Sapphire Rapids)", and AMX is available on M7i and M7i-flex. DSA, IAA and QAT are "available only on M7i bare metal sizes": m7i.metal-24xl and m7i.metal-48xl (fetched, https://aws.amazon.com/ec2/instance-types/m7i/).
  - m7i.metal-24xl has 96 vCPU / 384 GiB (Xeon 8488C) at **$4.8384/h** on-demand in us-east-1, with spot around $1.18/h (fetched, https://sparecores.com/server/aws/m7i.metal-24xl; third-party aggregator).
- **GCP:**
  - C3 bare metal instances "exclusively provide access to Intel integrated CPU features such as Intel Quick Assist (QAT), Intel Data Streaming Accelerator (DSA), and Intel In-Memory Analytics Accelerator (IAA)" (fetched, Google blog 2024-08-01, https://cloud.google.com/blog/products/compute/compute-engine-c3-bare-metal-and-x4-machine-types-now-ga).
  - C4 metal on Xeon 6 can use QAT, per Intel (snippet only, https://community.intel.com/t5/Blogs/Tech-Innovation/Cloud/Google-Cloud-C4-VMs-Intel-Xeon-6-Delivers-Instant-AI-ROI-and/post/1726843; the fetch redirected).
  - c4-standard-288-metal has 288 vCPU / 1080 GiB at about **$14.23/h** (fetched, https://instances.vantage.sh/gcp/c4-standard-288-metal).
- **Azure:** Dv6/Ev6 run on 5th Gen Xeon Platinum 8573C with AMX (snippet only, https://mc.merill.net/message/AZ478996). I found no Azure source exposing QAT, and no Azure bare-metal QAT offering.
- **IBM Cloud:** has 4th Gen Xeon bare metal (8474C, 8490H) on Classic and VPC (snippet only, https://community.ibm.com/community/user/blogs/laurentiu-pana/2025/04/03/power-real-time-business-with-high-end-servers-cer). I found no published price or QAT statement; pricing is quoted in the catalog.

**What this means for the project.**
- A fair x86 comparison needs two rows:
  - a "no-accelerator" row: the 5800X as-is, and EPYC;
  - an "accelerator" row: a Sapphire Rapids or newer bare-metal host with QAT endpoints and AMX. AWS m7i.metal-24xl at about $4.84/h is the cheapest documented one. A few hours per run costs tens of dollars.
- Check that QAT endpoints are present (for example, `lspci` showing QAT devices) before trusting any "QAT" row, because many SKUs have none.

---

## Q6. Existing open benchmark kits

**BenchBase** (CMU, successor to OLTPBench).
- A JDBC "Multi-DBMS SQL Benchmarking Framework" with TPC-C, TPC-H, TATP, YCSB, SmallBank, Voter and others (snippet only, https://www.github.com/cmu-db/benchbase).
- It is pure Java, so it should run on s390x under Semeru or OpenJDK, but I found no s390x-specific statement. Inference.

**HammerDB** (TPC Council).
- Ships x86-64 binaries only: "there are no alpha, beta or pre-release versions of HammerDB for ARM64 at present" (2025-03-18). Arm work started in 2026, and s390x is not mentioned anywhere (fetched, https://github.com/TPC-Council/HammerDB/discussions/767).
- Because it ships x86-64 binaries only, testing PostgreSQL on s390x with it means running its client on x86.

**DBT-5 (TPC-E) and DBT-2 (TPC-C)**, from OSDL DBT.
- DBT-5 is an "open source fair-use implementation of the TPC Benchmark(TM) E (TPC-E) specification". "The TPC's TPC-E Tools cannot be redistributed with DBT-5 and must be downloaded separately". AppImages are Linux-only, with the architecture unstated (fetched, https://osdldbt.github.io/dbt5).
- They are C/C++ against PostgreSQL, so an s390x source build is plausible, but no s390x statement was found.
- DBT-2 results "do not constitute a TPC-C result, and are incomparable with any TPC-C benchmark" (snippet only, https://osdldbt.github.io/dbt2).
- **Caution:** fair-use kits restrict how results may be labelled.

**AWS CardDemo.** COBOL, CICS, VSAM (KSDS with AIX), JCL, RACF, plus optional Db2, IMS and MQ. Apache 2.0. It has about 20 batch jobs (POSTTRAN, INTCALC, TRANEXTR, COMBTRAN, CREASTMT and others) and lists "performance testing" as a use case (fetched, https://github.com/aws-samples/aws-mainframe-modernization-carddemo, last updated April 2025).
- The prerequisite is a "Mainframe environment with CICS, VSAM, and JCL support". It does not target GnuCOBOL.
- It is the **closest existing COBOL online+batch application to reuse** as the workload design, but porting it to GnuCOBOL means replacing CICS and VSAM.

**COBOL performance micro-benchmarks.**
- The GnuCOBOL FAQ points to the speleotrove "telco" decimal benchmark (http://speleotrove.com/decimal/telco.html), with a GnuCOBOL-tuned version (snippet only, https://gnucobol.sourceforge.io/faq/index.html).
- I found no open COBOL/CICS-style online+batch benchmark kit.

**What this means for the project.**
- No kit does "COBOL online + batch + compression + signing + inference" out of the box. Reuse pieces:
  - CardDemo's data model and batch flow for the COBOL side;
  - telco for decimal arithmetic.
- The OLTP kits above (BenchBase, HammerDB, DBT-2, DBT-5) are surveyed here as third-party work only; the
  specification does not use them.

---

## Blockers and risks

1. **The Community Cloud terms ban benchmarking and publishing results.** This is Section 5 of v9.0 (2021), snippet only. A newer 2026-02-10 terms document exists, and I could not read it. Do not run the kit there without written permission.
2. **IBM Cloud s390x VSIs are closed to new orders** since 2026-02-28, and existing ones are deleted on 2027-02-20. Hyper Protect VPC and Dedicated Host s390x are deprecated too. No pay-as-you-go s390x cloud remains that I could find.
3. **The Community Cloud hardware is undocumented since 2021** (z15 then). There may be no AIU, so the zDNN/zDLC inference step may only run in CPU mode there.
4. **No hardware counters in z/VM guests.** `perf stat` cycles and instructions are unavailable, so measure with wall-clock and OS CPU time.
5. **The Community Cloud is a shared z/VM guest** with unknown SMT and IFL dedication, and noisy neighbours. Results would not be reproducible even if they were allowed.
6. **The VM size and term are unknown or conflicting:** 60+60 days vs 120 days, possible deletion after 30 idle days, and no backups. The support email was reported dead in January 2026.
7. **DFLTCC output differs from software output.** Golden-file checks must compare decompressed data.
8. **AES and other ciphers in CPACF depend on the host having feature 3863 enabled.** KDSA ECDSA needs z15+.
9. **Fair-use and TPC naming rules apply** to DBT-2, DBT-5 and HammerDB results.
10. **QAT exists only on some Xeon SKUs.** Cloud "QAT" claims must be checked on the instance itself.
11. **Not fetched:** the Community Cloud terms PDF (host unreachable), its JS-rendered pages and the logged-in library. Every terms quote here is snippet only.

## Choices I made

- I downloaded the terms PDF with one plain-HTTP `curl` attempt (Bash) to tc.l1cc.marist.edu after WebFetch failed. It timed out and nothing was retrieved. This was a public web server, not a mainframe, but the brief named only WebSearch and WebFetch.
- I extracted text locally (`pdftotext`) from two PDFs that WebFetch had saved (the 2021 SLUUG/IBM deck and the 2017 Marist ECC deck), to read content WebFetch could not decode. I count these as fetched.
- I sent the Q2 verdict to "main" twice: the verdict, then an addendum about the 2026 terms document.
- I did not look up IBM Cloud s390x prices, because the service no longer accepts new instances.
