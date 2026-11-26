# Governance

## Maintainer
- **Andrew Mayne** (Durham College) maintains the project. It is an independent project, not a Durham College
  publication.
- The maintainer decides on spec changes and merges, after public discussion in issues, pull requests or GitHub
  Discussions, and enforces the [code of conduct](CODE_OF_CONDUCT.md).

## Competing interests
The maintainer teaches mainframe development at Durham College and has a relationship with IBM through its academic initiatives, which provide the IBM Z academic system used for teaching.

## Spec changes
- Proposed as issues or pull requests, discussed in public, and recorded in the change note of [SPEC.md](SPEC.md)
  and in [CHANGELOG.md](CHANGELOG.md). Versions follow semantic versioning.
- Every result names the spec version and track it ran against.

## Pre-registration
- The registered text in [HYPOTHESES.md](HYPOTHESES.md) is never edited. H1j and H6 were added to it on 2026-11-19
  as new, dated entries. From spec v0.4.0, the new, dated entries that HYPOTHESES.md provides for ("Revised
  predictions are added as new, dated entries") are made in
  [docs/HYPOTHESES-ADDENDUM.md](docs/HYPOTHESES-ADDENDUM.md) instead, so that HYPOTHESES.md no longer changes:
  clarifications, scoring configurations and new or revised predictions go there, each registered before the first
  kit run it affects.
- **Registration is external.** Git commit dates alone are weak evidence of when something was written: the author
  date of a commit is set by whoever makes it, it can predate the commit date, rebasing rewrites history, and the
  date of a tag can be set too. Each addendum entry is therefore registered by a signed tag on the default branch,
  and the tagged release is deposited in an external archive that issues a DOI (Zenodo; or OSF Registries). The
  timestamp is the archive's own creation time of that version, because a record's metadata, its date fields
  included, stays editable after publication. The procedure, the tag names and the DOIs are in
  [docs/REGISTRY.md](docs/REGISTRY.md).
- **Machines and the pilot.** Every machine that scores a registered hypothesis is named in the machine registry
  before any kit run on it, and a pilot and a power analysis precede the registration tag (SPEC 10;
  docs/REGISTRY.md).
- **Run registry and tuning log.** From the first kit run, every kit run started is entered in the public run
  registry before it starts, including runs that fail or are abandoned, and every configuration change is entered
  in the series' tuning log before the run it affects (SPEC 4.13 and 4.14; docs/REGISTRY.md).
- Every registered verdict is published, VERIFIED or DIFFERS, whichever platform it favours, and it comes first in
  the FDR summary (SPEC 4.12).
- No result is called a headline result until the preconditions of SPEC section 10 hold.

## Results
- Anyone may publish results from this kit, if they include the FDR, the manifest, the raw output and the run
  registry entries of the series, and name the spec version. Results that don't follow the run rules must say so.
- No result from a licensed program, IBM's or another vendor's (for example Db2, z/OS, CICS TS, a Java runtime, a
  commercial Linux distribution, a hypervisor or a vendor accelerator library), is published until its licence
  terms have been checked for conditions on benchmarking or publishing results, and the check is recorded in the
  FDR (docs/PLAYBOOK.md, section 1).
- Our workloads are never described with another benchmark's name, including any TPC benchmark name, or with
  another benchmark's metric names (SPEC 5 and 6).

## Independence
- No vendor funds, approves or vetoes results. A platform owner's permission to benchmark and publish (SPEC 10,
  precondition 2) is obtained in writing before the first run of a series, and its terms are recorded in FDR 9. A
  permission that depends on the outcome, or that lets the owner see results before deciding whether they may be
  published, is not accepted, and that platform is not used for counted runs. The vendor review window (below) is
  for errors of fact, not consent.
- Support from any party (system access, hardware, advice, configuration work) is disclosed in full in the FDR. If
  the party asks not to be named, the support is still disclosed without the name, together with whether the party
  is, or is affiliated with, a vendor of a compared platform.
- The FDR names who configured each side, and their affiliation, including when a platform provider's staff did the
  configuring. Both sides start from the kit's configuration checklist and have the same tuning budget (SPEC 4.18,
  "Tuning parity").
- Contributors who are employed or paid by a vendor of a compared platform say so in the pull request or issue in
  which they propose a spec change.

## Vendor review and right of reply
- **Who counts as a vendor.** For a result, the vendors are: the maker of each compared processor (for example IBM,
  Intel or AMD); the maker of each system, or the cloud provider, the runs used; the owner of each platform the runs
  used; and the vendor or project of each licensed software product in the run.
- **Review window.** Before a result that names a vendor's product is published, the maintainer offers each of these
  vendors a factual review of the draft FDR, sent to a contact the vendor publishes. The window is 14 days
  (proposed) from the offer. Results are not held back after the window closes.
- **Factual review only.** Reviewers may point out errors of fact, configuration or method, with evidence.
  Corrections of fact are made, and recorded in FDR 9 with the date. Reviewers do not approve results, and a review
  is not an endorsement.
- **Right of reply.** A vendor may reply to any published result, at any time, in public: in an issue with the label
  `vendor-reply`, which the vendor may open, or which the maintainer opens with the reply as sent. A reply received
  within the review window is published with the result; a later reply is added to it with its date. The reply is
  published as sent, and is not edited except to remove credentials or private data.

## Contact
- **Questions and discussion:** GitHub Discussions on this repository, issues and pull requests.
- **Vendor replies:** an issue with the label `vendor-reply` (see above).
- **Platform access offers:** an issue from the platform-access template, with no private details; the maintainer
  replies there with a private contact for the details.
- **Conduct reports:** GitHub's content reporting, from the "Report content" menu on the issue, pull request,
  discussion or comment concerned: "Report to repository admins", which this repository accepts and which goes to
  the maintainer, or "Report abuse", which goes to GitHub Support and is handled under GitHub's own policies
  ([CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)). A report about the maintainer goes to GitHub Support. Questions about
  the code of conduct, and concerns not tied to a piece of GitHub content, go to GitHub Discussions; do not post
  private details there.
- **Security issues only:** GitHub's private vulnerability reporting on this repository ([SECURITY.md](SECURITY.md)).

## Future
- If the project gains contributors from more than one organisation, it may seek a neutral home (for example the
  Open Mainframe Project). It is not a project of the Open Mainframe Project or The Linux Foundation, and is not
  affiliated with either.
