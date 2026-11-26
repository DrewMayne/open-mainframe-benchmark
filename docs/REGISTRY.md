# Registry: pre-registration, runs and tuning (draft for comment)

Status: **notes**. This page is the project's public record of what was registered and when, and of every kit run
started, and of every machine named before its runs. It supports proposed SPEC 4.7, 4.13 and 4.14, and
GOVERNANCE.md, "Pre-registration". No machine has been named and no kit run has been made.

## 1. Why the timestamp is external
Git history alone is weak evidence of when a hypothesis was written:
- a commit's author date is set by whoever makes the commit, and it can predate the commit date;
- rebasing and force pushes rewrite history, so the same change can appear under different commits;
- the date in a tag object can also be set by its author (`GIT_COMMITTER_DATE`) [S51].

So each registration is a signed tag, pushed to a branch and tag set protected against rewriting, and deposited in
an external archive whose files and identifier cannot be changed after publication [S48]. A record's metadata,
including its publication-date field, can still be edited after publication [S48]. The registration timestamp is
therefore the archive's own creation time of that version: for Zenodo, the `created` time of the record in the
archive's API, and the DOI's registration date at DataCite, recorded in UTC to the second (verify the API field
name); for OSF Registries, the time the registration was submitted, as the registry records it. It is never an
editable date field, and never a git date.

## 2. Procedure for each registration
**Before the tag `omb-v0.4.0-prereg`,** these steps are done in this order (proposed SPEC 10, preconditions 5 and 6):
1. **Machines.** Every machine that will score entry 1 is named in the machine registry (section 4), each at its own
   tag, before any kit run on it.
2. **The kit.** A tagged kit release passes its golden check (proposed SPEC 10, precondition 1).
3. **Classes and the mix table.** On a named x86 machine alone, the H1 workload classes are measured; the mix table,
   the classes, the request-stream values and H6's keyed store are recorded in an addendum entry at its own tag,
   before any kit run on IBM Z or LinuxONE (HYPOTHESES-ADDENDUM.md, "Fixed now").
4. **The choices tag.** Entry 1, with every choice except its repeat counts, is registered at the tag
   `omb-v0.4.0-choices` by the steps below.
5. **The pilot.** On the named machines, at least 6 runs over at least 3 boots or IPLs (proposed) on each side, at
   one point of each measurand, each entered in the run registry as kind "pilot" before it starts. Pilot runs score
   nothing, and are published with entry 1.
6. **The power analysis.** From the pilot, entry 1 records for every claim its planned Holm-adjusted level, its
   repeat count and the distance it resolves with 80% probability (proposed); after the pilot only repeat counts
   and boot counts may change.
7. **No open marks.** Every "(verify)" or "(source needed)" mark in the text of SPEC.md and the pages that entry 1
   cites is resolved, or the fact removed.
8. **The prereg tag.** Entry 1 is registered at `omb-v0.4.0-prereg` by the steps below.

**For every registration:**
1. **Protection.** The default branch is protected by a repository ruleset that blocks force pushes and deletion,
   and a tag ruleset restricts the deletion and update of tags matching `omb-*` [S50].
2. **The commit.** The entry (an addendum entry, or a series' run plan) is merged to the default branch.
3. **The tag.** The maintainer creates an annotated, signed tag on that merge commit (`git tag -s <name>`) and pushes
   it [S51]. Anyone can check the signature (`git tag -v <name>`) [S51].
4. **The deposit.** The maintainer publishes a GitHub release from the tag. With the repository connected to Zenodo,
   the release is archived and receives a DOI [S48]. The files and the identifier of a published Zenodo record
   cannot be modified [S48]. If Zenodo is not used, the tagged tree is registered on OSF Registries instead, where a
   submitted registration and its files cannot be edited [S49].
5. **The record.** In the next commit, the DOI and the archive's creation time (UTC) are added to the table below,
   to the README's status section, and to CITATION.cff.
6. **Before the first kit run.** A registration counts only for kit runs whose registry start time (UTC) is later
   than its registration timestamp.

Tag names: `omb-v0.4.0-choices` and then `omb-v0.4.0-prereg` for entry 1 of
[HYPOTHESES-ADDENDUM.md](HYPOTHESES-ADDENDUM.md); later entries `omb-v<spec version>-prereg-<n>`; a machine
registry entry `omb-machine-<machine id>`; a series' run plan `omb-series-<series id>`.

## 3. Registrations
| Tag | What it registers | Commit | Archive record (DOI) | Archive creation time (UTC) |
| --- | --- | --- | --- | --- |
| omb-v0.4.0-choices | HYPOTHESES-ADDENDUM.md, entry 1, every choice except its repeat counts | not yet created | not yet deposited | not yet deposited |
| omb-v0.4.0-prereg | HYPOTHESES-ADDENDUM.md, entry 1, with the pilot and power analysis | not yet created | not yet deposited | not yet deposited |

## 4. Machine registry
Every machine that gives a counted result, or scores a registered hypothesis, is named here, in a dated entry
registered at its own tag (section 2), before any kit run on it, calibration and pilot included. Machines that will
score entry 1 of the addendum are all named before its first counted run; one Holm family covers every pair of them
(HYPOTHESES-ADDENDUM.md, "Multiplicity"). A named machine that is not run stays in the table as "named, not run",
with the reason. Entries are never deleted.

| Machine ID | Named (UTC) | Tag | Side | IBM Z: machine type and model, capacity setting, CPC drawers, engine mode | x86: part, class, server model, memory population | Reason chosen | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |

No machine has been named.

## 5. Run registry
From the first kit run, every kit run started on any system is entered here **before it starts**: scored and
unscored runs, calibration, exploratory and tuning runs, and runs that later fail or are abandoned. Entries are
never deleted or edited; a correction is a new row that names the row it corrects. A run that is not in this
registry is not a result of this benchmark (proposed SPEC 4.13).

| Run ID | Start (UTC) | Series | Run plan (tag) | Platform | Kind | Status | Reason, if failed or abandoned | FDR |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

No kit run has been made.

- **Kind:** pilot, classification, selection, calibration, scored, counted, exploratory, tuning, self-test,
  cross-check or accounting validation.
- **Platform:** the machine ID from the machine registry.
- The registry records what is entered in it, and cannot show that no other run was made. FDR 9 states, for each
  machine in a result, whether any kit run on it is missing from the registry (proposed SPEC 4.13).
- **Status:** started, completed, invalid, failed or abandoned. Every series ends in a published FDR, or in a
  published withdrawal notice that gives the reason and includes all raw data gathered.

## 6. Tuning logs
Each series keeps a tuning log, published with its FDR. Every change to the configuration of the system under test
or the driver (a setting, a version, a placement, a limit) is entered with its time (UTC), what changed, why, and
the run ID of the first run it affects, **before** that run starts (proposed SPEC 4.14). A value of the run plan, or
a limit of the frozen configuration, chosen after looking at decision-point data starts a new, labelled series, and
the results under the earlier plan are still published in full. A limit raised after a refusal before any
decision-point run of the series is tuning, and the series continues (proposed SPEC 1.8.2).

## Sources
Fetched 2026-11-26. The numbering matches [ARCHITECTURE.md](ARCHITECTURE.md) section 12.
- [S48] Zenodo help, GitHub integration ("new releases from the repository will be automatically ingested and
  archived"): https://help.zenodo.org/docs/github/enable-repository/ ; and About records ("Files and the persistent
  identifier CANNOT be modified"; "Metadata CAN be modified"): https://help.zenodo.org/docs/deposit/about-records/
- [S49] OSF help, Welcome to Registrations ("Once you submit a registration, you will not be able to edit or make
  changes to it or any associated files"): https://help.osf.io/article/330-welcome-to-registrations
- [S50] GitHub Docs, About rulesets, and Available rules for rulesets (rulesets for branches and tags; "Restrict
  deletions", "Block force pushes", "Restrict updates"):
  https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets
- [S51] Git, git-tag ("A cryptographically signed tag object will be created when -s or -u <key-id> is used"; "To
  set the date used in future tag objects, set the environment variable GIT_COMMITTER_DATE"):
  https://git-scm.com/docs/git-tag ; GitHub Docs, Signing tags:
  https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-tags
