# GDPR — Transfers and Processors

> Art 28 processor contracts, sub-processor rules, Art 44–49 third-country transfers, and the
> Art 27 EU representative requirement. Vendor/adequacy facts below were current as of
> **September 2026** — confirm against the European Commission's current adequacy decisions list
> and the vendor's own current DPA/SCC documentation before citing one as fact; adequacy status
> and vendor terms both change without much notice.

## Table of Contents

1. [Processor contracts — Art 28](#processor-contracts--art-28)
2. [Sub-processors — Art 28(2) and (4)](#sub-processors--art-282-and-4)
3. [If the audited org is the processor](#if-the-audited-org-is-the-processor)
4. [The general principle for transfers — Art 44](#the-general-principle-for-transfers--art-44)
5. [Adequacy decisions — Art 45](#adequacy-decisions--art-45)
6. [Appropriate safeguards — Art 46 and Standard Contractual Clauses](#appropriate-safeguards--art-46-and-standard-contractual-clauses)
7. [Schrems II and the transfer impact assessment](#schrems-ii-and-the-transfer-impact-assessment)
8. [Derogations — Art 49](#derogations--art-49)
9. [The EU representative — Art 27](#the-eu-representative--art-27)
10. [A transfer is broader than a data copy](#a-transfer-is-broader-than-a-data-copy)

---

## Processor contracts — Art 28

Art 28(1) requires a controller use "only processors providing sufficient guarantees to
implement appropriate technical and organisational measures" — a due-diligence obligation before
engaging any processor. Art 28(3) requires the actual processing be governed by a contract or
other binding legal act specifying, at minimum, the subject-matter, duration, nature and purpose
of the processing, the type of data and categories of data subjects, and the controller's
obligations and rights — and binding the processor to (a) process only on documented instructions;
(b) ensure staff confidentiality; (c) implement Art 32 security measures; (d) respect the
sub-processor rules below; (e) assist the controller in responding to data subject rights
requests; (f) assist with Art 32–36 compliance (security, breach notification, DPIA); (g) delete
or return all personal data at the end of the service, unless law requires retention; and (h)
provide information demonstrating compliance and allow for audits.

This is the contract Gotcha 6 in `SKILL.md` refers to — any vendor receiving personal data on the
org's behalf (a hosting provider, an email/SMS provider, an LLM API, an analytics platform acting
in a processor capacity) needs one of these in place before data flows, not after. Absence is a
🔴 regardless of the vendor's general reputation or security posture.

## Sub-processors — Art 28(2) and (4)

Art 28(2) prohibits a processor from engaging another processor "without prior specific or
general written authorisation" from the controller, and requires informing the controller of any
intended change so the controller can object. Art 28(4) requires the same data-protection
obligations flow down to any sub-processor through a binding contract. In code, this shows up as
a vendor's own sub-processor list (most SaaS DPAs publish one) — check whether the org's own
outbound integrations (an LLM provider calling out to its own infra provider, an email service
using a third-party delivery network) are accounted for in that chain, not just the first hop.

## If the audited org is the processor

Everything above is written from the perspective of an org *hiring* a processor. When Gate 2
resolves the other way — the audited org processes personal data on behalf of its own customers,
who are the controllers — most of the checklist flips, and auditing it against the controller
article set (Art 6/9 lawful basis, Art 13/14 notices, Art 33(1)'s 72-hour authority-notification
clock) produces findings against obligations that aren't actually the audited org's to meet.

What a processor actually needs, each independently checkable:

- **Art 28(3)(a) — process only on documented instructions.** Look for processing that goes
  beyond what any single controller/customer configured — a feature that runs the same logic
  across all tenants' data for a purpose no customer opted into.
- **Art 28(3)(b) — staff confidentiality.** Access to customer data scoped to people under a
  confidentiality obligation, not open to the whole engineering org by default.
- **Art 28(3)(c) — Art 32 security measures.** Same checklist as `references/security-and-breach.md`,
  applied to the processor's own systems.
- **Art 28(2)/(4) — sub-processor authorization.** Covered above; check the org's own sub-processor
  list against what its contracts with its customers actually authorize.
- **Art 28(3)(e) — assisting the controller with data subject rights requests.** The concrete code
  check: is there a tenant-scoped export/delete capability a customer can actually invoke, or does
  a rights request have no path except an ad hoc engineering request?
- **Art 28(3)(f) and Art 33(2) — assisting with security/breach compliance, and notifying *the
  controller* without undue delay on a breach.** This is the processor's actual breach obligation —
  there's no Art 33(1) 72-hour authority-notification duty of its own; that belongs to each
  controller-customer, who the processor needs to be able to notify promptly enough for them to
  meet their own clock.
- **Art 28(3)(g) — deletion or return of data at contract end**, unless retention is legally
  required. Check for an actual tenant-offboarding data-purge job, not just a suspended account.
- **Art 30(2) — its own processing record**, covering the categories of processing performed on
  each controller's behalf — see `assets/ropa-template.md`'s processor-record section.

**The trap: using customer data for the processor's own purposes makes it a controller for that
processing.** Art 28(10) — if the org runs its own product analytics, or trains a model, over data
it holds only as a processor, that specific processing is outside any controller's instruction,
and the org becomes a controller for it, with its own Art 6 basis and Art 13/14 transparency
obligations attaching to that slice of processing (even though it remains a processor for
everything else). This is easy to miss because the org's *primary* role audit-wide is processor —
the analytics pipeline is where that flips for one specific activity.

## The general principle for transfers — Art 44

Art 44 states the umbrella rule plainly: a transfer to a third country or international
organisation may happen only if the conditions in this chapter are met, "including for onward
transfers... to another third country or another international organisation," and the whole
chapter exists to ensure "the level of protection of natural persons guaranteed by this
Regulation is not undermined" by the transfer. Onward transfers matter in practice: a US-based
processor that itself uses a sub-processor in a third country with no adequacy or safeguard needs
that leg covered too, not just the org's own direct transfer.

## Adequacy decisions — Art 45

Art 45(1): a transfer may take place without further safeguards where the European Commission has
decided the destination country, territory, sector, or international organisation "ensures an
adequate level of protection." The Commission's adequacy list changes over time — as of this
writing it includes the UK, Japan, South Korea, and a set of others, plus the EU-U.S. Data
Privacy Framework covering DPF-certified US organizations specifically (not the US generally).
**Always check the Commission's current list rather than relying on a prior audit's finding** —
this is exactly the kind of fact this skill's Gotcha 10 warns to date-stamp.

## Appropriate safeguards — Art 46 and Standard Contractual Clauses

Absent an adequacy decision, Art 46(1) allows a transfer where the controller or processor has
provided "appropriate safeguards" and enforceable data subject rights remain available. Art 46(2)
lists safeguards not requiring prior authorization from a supervisory authority, most commonly in
practice (c) the Commission's Standard Contractual Clauses (SCCs) — the modular 2021 SCC set
covering controller-to-controller, controller-to-processor, processor-to-processor, and
processor-to-controller transfers. Binding Corporate Rules (Art 47) are the alternative for
intra-group transfers at large multinationals; less common in a typical audit target's codebase.

Signed SCCs are necessary but, since *Schrems II*, not sufficient on their own — see the next
section.

## Schrems II and the transfer impact assessment

*Data Protection Commissioner v. Facebook Ireland Ltd (Schrems II)*, CJEU Case C-311/18 (2020),
invalidated the EU-U.S. Privacy Shield and held that SCCs alone don't guarantee adequate
protection if the destination country's own laws (e.g. government surveillance powers) could
override the contractual safeguards in practice. The practical consequence: signing SCCs with a
US-based vendor is not the end of the analysis. A transfer impact assessment (TIA) — evaluating
the destination country's laws and the actual risk of government access, and layering on
supplementary measures (strong encryption where the vendor has no access to keys, contractual
transparency commitments, etc.) where needed — is expected alongside the SCCs. A code audit
cannot perform a TIA, but it can flag when SCCs exist with no accompanying TIA documented anywhere
in the repo or its linked docs — grade that 🟡, and route the TIA's actual adequacy to a human via
`references/accountability-and-legal.md`.

## Derogations — Art 49

Where neither an adequacy decision nor Art 46 safeguards are in place, Art 49(1) provides narrow
derogations for specific situations: (a) explicit informed consent to the specific transfer;
(b) necessary for a contract with the data subject or pre-contractual steps at their request;
(c) necessary for a contract in the data subject's interest between the controller and someone
else; (d) important reasons of public interest; (e) legal claims; (f) vital interests where
consent can't be obtained; (g) a small carve-out for public registers. These are meant as
exceptions for occasional, non-systematic transfers — not a substitute for putting SCCs in place
for a transfer that happens on every request. A repeated, routine transfer relying solely on an
Art 49 derogation (e.g. "the user consented" as the sole basis for every API call to a
US-based vendor with no SCCs at all) is a 🟡 at minimum — flag that a stable safeguard (Art 46) is
the expected long-term basis, not a derogation used as a permanent workaround.

## The EU representative — Art 27

Where Art 3(2) applies (the org has no EU/EEA establishment but offers goods/services to, or
monitors, people in the Union), Art 27(1) requires designating in writing a representative
established in one of the Member States where the affected data subjects are located. Art 27(2)
exempts processing that is occasional, doesn't include large-scale special-category or criminal-
data processing, and is unlikely to result in a risk to data subjects' rights — or processing by
a public authority. Art 27(4) makes the representative addressable by supervisory authorities and
data subjects on all compliance matters, in addition to or instead of the controller/processor
itself. Check whether an org that cleared the "does GDPR apply" gate via Art 3(2) alone (no EU
establishment) has designated one — its absence is a concrete, checkable gap, typically visible
in a privacy policy's contact section or its absence from one.

## A transfer is broader than a data copy

The most common mistake auditing this area: treating "transfer" as meaning only "a dataset was
copied to a server in another country." Remote access counts too — a support engineer, a
contractor, or an outsourced ops team accessing production data from outside the EU/EEA is a
transfer under Art 44 the moment they view it, with no data ever physically relocated. Check who
has production database or admin-panel access, and from where, not just where servers or backups
are physically hosted.
