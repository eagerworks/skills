# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-12

Initial release.

### Added
- Audits a codebase and its infrastructure config against the GDPR (Regulation (EU) 2016/679):
  territorial scope (Art 3) and controller/processor/joint-controller classification (Art 4(7)/(8), Art 26).
- Personal data identification — special categories (Art 9), pseudonymisation vs. anonymisation, online identifiers.
- Lawful basis and consent — Art 6 bases, Art 7 consent conditions, Art 9(2) special-category conditions, Art 13/14 notices, the ePrivacy cookie interaction.
- Data subject rights mapped to code — Art 12–22, including the access-vs-portability distinction and what a real erasure flow has to reach beyond the primary database.
- The highest-yield audit surface — logs, error trackers, analytics/telemetry SDKs, LLM prompts, seeds and fixtures.
- Security and breach — Art 32 security of processing, Art 25 data protection by design/default, Art 33/34 breach notification, Art 35/36 DPIA triggers.
- Transfers and processors — Art 28 processor contracts and sub-processors from both sides (hiring a processor and being one, including the Art 28(10) trap where a processor's own-purpose use of customer data makes it a controller for that processing), Art 44–49 third-country transfers, Standard Contractual Clauses, the Schrems II transfer-impact-assessment expectation, Art 27 EU representative.
- Accountability items routed to a human — ROPA (Art 30), DPO designation (Art 37–39), the DPIA and breach-notification processes, Art 83 fine tiers.
- Outputs a severity-graded audit report, a gap-matrix CSV, and a fillable Art 30 ROPA template, written to a dated file in the audited repo.

### Config
- No config file.
