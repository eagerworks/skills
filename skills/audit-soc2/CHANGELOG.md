# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-12

Initial release.

### Added
- Plans and runs a SOC 2 readiness effort: an engagement-framing step (own company vs. client assessment, Type I vs. Type II, current stage), a batched intake interview, and a fixed plan output shape (scope statement → gap matrix → five-phase roadmap → top-5 priorities → stated assumptions).
- Eight `references/*.md` files: Trust Services Criteria, full intake rationale, scoping (system boundary, subservice orgs), required policies, technical controls, DIY evidence collection, the CPA audit process, and tooling (DIY-first, compliance platforms as a closing note).
- `assets/`: a fillable intake questionnaire, a `gap-matrix.csv`, a `readiness-roadmap.md` template with a worked example, and nine policy skeletons (infosec, access control, incident response, vendor risk, BCDR, SDLC/change management, risk assessment, data classification, acceptable use).

### Config
- No config file.
