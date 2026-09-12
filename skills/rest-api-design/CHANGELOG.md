# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-08-21

Initial release.

### Added
- Designs and reviews REST APIs: resource naming, HTTP methods and status codes, payload and RFC 9457 error formats, pagination, versioning, auth, rate limiting, and OpenAPI 3.1 specs.
- Includes an existing-API survey step so new endpoints match the conventions already in the codebase.
- Audits an existing API for consistency or security issues (IDOR, mass assignment).

### Config
- No config file.
