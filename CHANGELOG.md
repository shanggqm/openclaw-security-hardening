# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.1.0] - 2026-03-24

### Added
- Repository restructured as security ecosystem (skills + articles + docs + resources)
- Article #1: 你的龙虾正在裸奔吗？ (text + 5 illustrations)
- Article #2: 安全加固的三层盔甲 (text + 5 illustrations)
- Articles index (`articles/README.md`)
- Threat model placeholder (`docs/threat-model.md`)
- CONTRIBUTING.md with contribution guidelines
- CHANGELOG.md

### Changed
- Moved security-audit skill to `skills/security-audit/`
- New root README with full ecosystem overview, roadmap, and article index
- Updated series plan: articles 3-5 pivot to AI-native security topics (Prompt Injection, Trust Chain, Privacy)
- Package.json updated with `repository.directory` field

## [1.0.0] - 2026-03-22

### Added
- Initial release of security-audit skill
- 7-dimension security scan (DM policy, Gateway, groups, tools, sandbox, file permissions, model security)
- Scoring system (0-100)
- Three-tier hardening plans (basic / standard / strict)
- Published to npm as `openclaw-security-hardening`
