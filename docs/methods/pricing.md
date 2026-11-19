# Pricing method: how we borrow from TPC-Pricing

Status: draft for comment, 2026-11-19. Applies to the optional pricing appendix of an FDR
([FDR-TEMPLATE.md](../FDR-TEMPLATE.md), section 8) and to the open cost model (ROADMAP M7). See SPEC sections 1.7, 3
and 6.

Source: TPC Pricing Specification, version 2.9.0, February 2023 ("TPC-Pricing 2.9.0" below), read in full from the PDF
listed under Sources. Clause numbers refer to that version. **No TPC text is reproduced here**: everything is our own
summary, and readers who need the exact rule should read the clause itself (SPEC section 6).

## 1. What this is, and what it is not
- TPC-Pricing is a method for turning a benchmarked system into a defensible price. We use it because it already
  answers the hard questions (what to include, which prices count, how to stop "benchmark specials").
- **Our prices are not TPC prices.** No TPC auditor or Pre-Publication Board reviews them (TPC-Pricing 2.9.0,
  clauses 1.7.6 and 6), and no TPC benchmark result sits underneath them.
- Every pricing appendix and every M7 output carries the label **"derived from TPC-Pricing list-price estimate; not
  comparable to published TPC results"**.
- We never use a TPC price/performance name (TPC-Pricing 2.9.0 defines "Price Performance" as total price over the
  benchmark's primary metric, clause 0.1.2). Ours is written out in plain units, e.g. "3-year list-price estimate,
  CAD per (transactions/s)".

## 2. Element by element: adopt, adapt or drop

### 2.1 Priced configuration
TPC-Pricing 2.9.0 leaves the list of items to the benchmark (definition of Priced Configuration, clause 0.1.2;
clause 1.3.10; Appendix A), and sets the rules for how they are priced (clause 3).
- **Adopt:**
  - Everything in the system under test is priced: hardware, the software used in the run, storage for data and logs,
    and maintenance (clauses 1.3.6, 3.3.5).
  - The load driver is not priced, unless it does work beyond driving load (clause 3.3.1).
  - Links from users into the data centre are not priced; links between priced components are (clause 3.3.2).
  - Hardware is priced with the activation licences needed for the resources used (clause 3.1.1).
  - A unit is priced whole: no price reduction because only part of it was used or capped (clause 3.1.2).
  - Resource-based software is licensed for every resource it could run on in the priced box (clause 3.2.3).
  - The measured and priced configurations must perform the same, and both are described with their differences
    (clauses 1.3.11, 5.1.3). The usage level each item was priced at is disclosed (clause 5.4.11).
- **Our definition (proposed):** the priced configuration is the SUT of the FDR (section 2) plus the software of
  FDR section 3, with the run's storage, and 3 years of maintenance. Our SPEC section 1 is the "benchmark standard"
  for this purpose.
- **Adapt:** we pin 2 or 4 cores on bigger machines. Under clauses 3.1.2 and 3.2.3 the appendix still prices the
  whole box and licenses all its cores. A per-core share of that price is not a TPC-style figure, so it may appear
  only in M7, as a visible assumption ("apportioned by cores used: N of M").

### 2.2 Three-year cost of ownership
- **Adopt** the default 3-year method (clauses 1.7.1, 1.7.2):
  - hardware at purchase price, software licence charges, Licensed Compute Services (cloud) and maintenance and
    software updates, over 36 months;
  - for anything priced by the hour or month, scale to 26,280 hours (clause 1.3.3);
  - a cloud service priced only for 1 year may be multiplied by 3 (comment to clause 1.7.1).
- **Excluded by TPC-Pricing, and by us in the appendix:**
  - domestic taxes and shipping (clause 1.3.20; import duties and shipping to the port of entry stay in);
  - the driver and user connectivity (clauses 3.3.1, 3.3.2);
  - tools used only for the FDR or data generation (Appendix C, clause 7.1.2.2 of the example, which is
    non-binding).
- **Not in the TPC cost at all:** the Pricing Period Cost covers only hardware, software, cloud services and
  maintenance (definition, clause 0.1.2). Power, cooling, floor space and staff have no line.
- **Adapt for M7:** M7 may add energy (from our energy method, [energy.md](energy.md)), facilities and staff as
  separate, editable operating-cost inputs. They are shown beside the derived from TPC-Pricing total, never inside it.
- **Drop** the 1-year, 1-month and pay-as-you-go methods (clauses 1.8 to 1.10). One method keeps results comparable
  (clause 1.2 says the methods are not comparable with each other).

### 2.3 List prices and their sources
- **Adopt:**
  - The list price is what any customer would pay for one unit from a sales channel, without other purchases
    (definition, clause 0.1.2; clauses 0.1.3, 0.2.1).
  - New products only, at full price, bought once and stand-alone; no trade-ins, refurbished or used parts, and no
    credit for past or future purchases (clauses 1.3.4, 1.3.5, 1.3.18).
  - Line-item detail, as on an itemised invoice (clause 1.3.19).
  - The Pricing Source of every line is named (clause 1.4.1).
  - A third-party source above 2% of the total must be the OEM or an authorised reseller (clause 1.4.5). We apply
    this to every line: no marketplace or grey-import sellers.
  - A public web price counts as the quote (comment to clause 1.4.4).
  - The benchmark-special tests of clause 0.2.2 are used as a checklist when a price looks too good.
- **Adapt:**
  - We cannot get written quotes guaranteed for 90 days (clause 1.4.3). Each price is a dated, archived capture of the
    public page (URL, date, and a saved copy or web-archive link). That capture replaces the quote.
  - TPC-Pricing requires revisions when prices rise 2% or more, or a product is withdrawn (clause 5.6.1). We have
    no ongoing duty: the appendix is dated, and a republished result is re-priced in full.
- **Never used** (TPC-Pricing bans most of them; we ban all of them): negotiated or confidential prices (clause
  1.5.7), prices that an NDA stops any customer from ordering (clause 1.3.13), educational, government or partner
  schedules (clauses 0.2.2 item 3, 1.5.8), and promotions, closeouts or limited-time offers (comment to clause 1.5.2).
  The educational ban matters to us: academic programme pricing is never used, even when we have it.

### 2.4 Availability
- **TPC-Pricing:**
  - "Available", "Generally Available" and "Availability Date" are defined in clause 0.1.2.
  - Every line must be orderable by any customer on the Availability Date (clause 1.3.12).
  - That date must fall within 185 days of the FDR submission and be reported to the day (clauses 5.4.6, 5.4.7).
  - Announced but unshipped products may be priced under conditions (clauses 1.6, 5.4.8).
  - A slip past the 185-day window means withdrawal (clause 5.6.1.2).
- **Adapt:** we price only items orderable on the pricing date, so our Availability Date is the pricing date. We drop
  the pre-announcement route (clauses 1.6, 5.4.8) and the 185-day window: we cannot commit a vendor's ship dates.
- **Superseded hardware.** A machine we measured may no longer be sold, and clause 1.3.18 rules out pricing it used.
  - We then price the nearest orderable equivalent and mark the line "S", as clause 5.2.2 item 5 does.
  - We use the middle-tier criteria of clause 5.7.3.2 as the test. The substitute may not reduce bus speed, any cache
    level, core or thread count or clock, and may not change architecture. Memory may not lose capacity or clock, or
    gain latency.
  - The test is passed on spec sheets alone, with no auditor, so the appendix says so. If nothing passes, the line has
    no price and the appendix is marked incomplete (section 3.5).

### 2.5 Maintenance and support
- **TPC-Pricing:**
  - Hardware, software and cloud services carry support that is available 24x7 (clause 4.1).
  - A problem must be recognised within 4 hours. A probable fix or part must be made available within 4 hours once
    identified (clause 4.1, clause 4.3 for hardware).
  - Customer-replaceable spares are allowed in place of on-site service: 10% extra, minimum 2, with a 7-day
    replenishment service (clause 4.3.2).
  - Software defects get a 4-hour response (clause 4.4), and software updates are included (clause 4.5).
  - The terms must be public (clause 4.1).
- **Adopt** that level as the target, priced from public support offerings, for 3 years.
- **Adapt:** consumer and small-business PCs are often not sold with 24x7 4-hour cover.
  - We price the best publicly sold tier, and state the gap per line, e.g. "next business day; below the TPC-Pricing
    level of clause 4.3".
  - We do not use the spares route (clause 4.3.2) unless the vendor sells spares as a standard offering.
- **Open-source software** (Linux, PostgreSQL, GnuCOBOL, OpenJDK builds) has no licence charge.
  - Where a vendor publishes a commercial support subscription for the exact build used, that price is the line item.
  - Otherwise the line is priced at zero and noted as "community software, no priced support". That is below TPC's
    software support rule (clauses 4.4, 4.5), and the note says so.

### 2.6 Discounts
- **TPC-Pricing:**
  - Generally available discounts are allowed (clause 1.5.1). Discounts cannot depend on other suppliers
    (clause 1.4.2), on payment faster than 30 days (clause 1.5.5), or on only part of the configuration (clause 1.5.4).
  - Each discount's percentage, amount and basis is disclosed (clauses 5.2.4, 5.4.2), and discounts round down
    (clause 5.3.6).
- **Adapt: we take no discounts at all.** List price only. This removes the need to prove a discount is general.
- **Not a discount:** a published fixed-term commitment price (for example a cloud provider's public 3-year rate) is
  a separate list price for a separate offer. It may be used if it is public and open to any customer, with its term
  and payment option stated.

### 2.7 Currency, locale and rounding
- **Adopt:**
  - One Priced Locale and one Priced Currency per appendix, named with its ISO 4217 code (clauses 1.3.7, 5.1.4,
    5.3.3). Default (proposed): Canada and CAD, where the maintainer can check prices. USD is used when an item has no
    Canadian list price.
  - Totals are rounded up to whole currency units (clauses 5.3.4, 5.3.5).
  - Where conversion is unavoidable: convert per-currency totals rather than single lines (clause 1.3.16), all on one
    date and from one source, and disclose both (clauses 1.3.17, 5.1.6, 5.4.5).
- **Adapt:** TPC fixes the conversion source through its administrator (definition of Currency Conversion Source,
  clause 0.1.2). We use the Bank of Canada daily rate and name the date.

### 2.8 Pricing models (cloud regions)
- **Adopt** the default model: driver and SUT in the same region, measured and priced (clause 2.1.1).
- A run measured within one region is never priced across regions (clause 2.2).

### 2.9 Disclosure and audit
- **Adopt** the pricing spreadsheet content of clause 5.2.2:
  - description, part number and Pricing Source per line;
  - list price with ISO code, quantity and extended price;
  - maintenance price per line, or a note that a group price covers it;
  - the "S" and "not orderable" markers.
- **Adopt** the subtotals by category (clause 5.2.3): general hardware or cloud services, storage, and software.
- **Drop** the TPC Executive Summary and the TPC advisory notice (clauses 5.1.2, 5.2.6). They are TPC text and would
  imply a TPC result. We use our own notice (section 4).
- **Drop** the audit (clauses 1.7.6, 6). In its place, every source capture is published with the result, so any
  reader can check every line. The clause 6.2.1 checklist is used as our self-check.

## 3. What we can and cannot price, per platform

### 3.1 x86 PC or server on bare metal
- **Can:** OEM and authorised-reseller web prices for the system or its parts, OS subscriptions where they exist,
  and published warranty extensions.
- **Limits:**
  - Support below the clause 4.3 level is usual for PCs, and is stated per line (section 2.5).
  - A self-built PC is priced as its parts, each from its own Pricing Source. There is no system-level warranty.

### 3.2 Cloud instances (x86 bare metal or VM)
- These are Licensed Compute Services (definition, clause 0.1.2).
- **Pricing:**
  - From the provider's public price page for the region used, either the on-demand hourly rate x 26,280
    (clause 1.3.3), or a published 3-year commitment rate (section 2.6).
  - A 1-year commitment rate may be multiplied by 3 (comment to clause 1.7.1).
  - The appendix states which one was used. Where more than one applies, M7 shows each.
- **Storage and network:** block storage and any priced interconnect between priced instances are included
  (clause 3.3.2). User-facing egress is not.
- **Support:** hardware upkeep is part of the service if it meets clause 4.3 (clause 4.2). Response times are met
  by the provider's published support plan, priced as its own line. The plan's response times are quoted from the
  provider's page.

### 3.3 LinuxONE (Linux on IBM Z hardware)
- **Hardware:** we found no public list price for LinuxONE systems or IFL features. The IBM Cloud LinuxONE virtual
  servers closed to new orders in February 2026 ([platforms.md](../platforms.md), Q3).
- **Software:** Linux distribution subscriptions for s390x may have public prices; not checked yet.
- **Therefore:** no pricing appendix for LinuxONE hardware. In M7, the hardware and maintenance lines are empty,
  labelled assumptions (section 3.5).

### 3.4 z/OS
- **Software:**
  - IBM's z/OS pricing documentation is an index of announcement letters (Workload License Charges, sub-capacity
    pricing for IPLA products and others). It points to those letters for terms and lists no prices.
  - IBM's public pages describe the Tailored Fit Pricing options: usage measured in MSUs consumed, or a full-capacity
    option. They offer them as alternatives to the rolling four-hour average.
  - These pages publish no rate card. Each option is set up with IBM sales, and the development and test option
    refers to an agreed price (fetched 2026-11-19; see Sources).
  - We have not read IBM's MLC or IPLA metric pages themselves; their details are not used here.
- **Hardware:** we found no public list prices for IBM Z machines, engines or MSU capacity.
- **Therefore:** no pricing appendix for z/OS. Every z/OS cost in M7 is an empty, labelled assumption.

### 3.5 Rules for figures we cannot source publicly
- **Never used:** confidential or negotiated prices, prices given to us under an NDA, and prices recalled from
  memory or taken from secondary write-ups.
- **Vendor quotes (proposed):**
  - A vendor quote may stand in for a public page only if three things hold: the vendor agrees in writing that it can
    be published in full; it states that it is the list price open to any customer; and it is published with the
    result.
  - Anything else is not a source.
- **In M7, an unsourced figure is an editable assumption with no default value.**
  - It is labelled "ASSUMPTION: no public source" and named in the output.
  - M7 computes nothing that depends on it until the user enters a value. The output then carries the user's value
    and the label.
- **Labels on any result that carries a price:**
  - "list-price estimate": every line has a public source.
  - "list-price estimate, incomplete: <components> not priced (no public list price)": some lines are missing. The
    total is shown with the missing components named, and no price/performance figure is derived.
  - "assumption-based cost model (M7): <inputs> supplied by the user": the figure comes from M7 with user inputs.

## 4. Worked template: an x86 pricing appendix
Placeholders only. `<...>` marks a value to fill in; no real prices.

Header lines:
- Label: derived from TPC-Pricing list-price estimate; not comparable to published TPC results.
- Result ID, spec version, kit commit: `<...>`
- Priced Locale / Priced Currency: `<Canada / CAD>`
- Pricing date (= Availability Date): `<YYYY-MM-DD>`
- Measured vs priced configuration: `<same | differences listed>`. Cores used: `<N>` of `<M>` priced.

| # | Category | Description and revision | Part number | Source | Unit list price | Qty | Extended | 3-yr support | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Hardware | `<system or chassis, PSU>` | `<pn>` | [1] | `<CAD>` | 1 | `<CAD>` | in line 9 | |
| 2 | Hardware | `<CPU model, cores, clock>` | `<pn>` | [1] | `<CAD>` | `<n>` | `<CAD>` | in line 9 | `<S if substituted>` |
| 3 | Hardware | `<memory, size, speed>` | `<pn>` | [2] | `<CAD>` | `<n>` | `<CAD>` | in line 9 | |
| 4 | Storage | `<data drive, type, interface, size>` | `<pn>` | [2] | `<CAD>` | `<n>` | `<CAD>` | in line 9 | |
| 5 | Storage | `<log or second drive, if used>` | `<pn>` | [2] | `<CAD>` | `<n>` | `<CAD>` | in line 9 | |
| 6 | Hardware | `<NIC, only if it links priced parts>` | `<pn>` | [2] | `<CAD>` | `<n>` | `<CAD>` | in line 9 | |
| 7 | Software | `<OS and version, subscription term>` | `<pn>` | [3] | `<CAD>` | 1 | `<CAD>` | incl. | |
| 8 | Software | `<PostgreSQL / JVM / GnuCOBOL version>` | n/a | [4] | 0 | 1 | 0 | `<CAD or none>` | community software, no priced support |
| 9 | Support | `<3-yr hardware warranty, level>` | `<pn>` | [1] | `<CAD>` | 1 | `<CAD>` | `<CAD>` | `<level; gap vs clause 4.3>` |

Totals and derived figures:
- Subtotals: hardware `<CAD>`, storage `<CAD>`, software `<CAD>`, support `<CAD>`.
- 3-year total, rounded up: `<CAD>`.
- Derived figure: `<CAD>` per (transactions/s), 3-year list-price estimate, from FDR section 6 throughput
  `<value>`.
- Sources: [1] `<URL, captured YYYY-MM-DD, archive link>`; [2] `<...>`; [3] `<...>`; [4] project download page.
- Excluded: taxes, shipping, load driver, user network, power and facilities (see M7 for the last two).
- Notice (ours): "List prices from public sources on the pricing date, for a one-time purchase of new items. No
  discounts, negotiated or special prices. Not audited. Report errors as an issue on this repository."

## 5. Sources
- TPC Pricing Specification, version 2.9.0, February 2023 (PDF):
  https://www.tpc.org/TPC_Documents_Current_Versions/pdf/TPC-Pricing_v2.9.0.pdf. Listed at version 2.9.0, with no
  kit, on TPC's current specifications page: https://www.tpc.org/tpc_documents_current_versions/current_specifications5.asp
  (fetched 2026-11-19). Read from a local copy of the same PDF.
- IBM, "Tailored Fit Pricing for IBM Z" content solution: https://www.ibm.com/support/z-content-solutions/tailored-fit-pricing
  (fetched 2026-11-19).
- IBM, "Pricing for IBM Z": https://www.ibm.com/it-infrastructure/z/software/pricing-resources (fetched 2026-11-19).
- IBM, z/OS 2.4 "Detailed pricing information" (index of pricing announcements):
  https://ibm.com/docs/SSLTBW_2.4.0/com.ibm.zos.v2r4.e0zi100/e0zi113.htm (fetched 2026-11-19).
- IBM Cloud LinuxONE withdrawal: see [platforms.md](../platforms.md), Q3, and its sources.
- Not yet checked: public s390x Linux subscription prices; any IBM public LinuxONE hardware price list.
