# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.3.0] - 2026-03-25

### Changed
- **Skill Rename**: `openclaw-security-hardening` → `guomeiqing-security-audit` (package.json + SKILL.md name)
- **Skill Rename**: `safe-install` → `guomeiqing-safe-install` (SKILL.md name + package.json)

### Added
- **Article #4**: Are Your Installed Skills Safe? — Trust Chain & Model Security (text + images placeholder)
- **English translations** for all documentation and articles (`.en.md` files)
- Language switcher links in root README (🇬🇧/🇨🇳)

## [1.2.0] - 2026-03-25

### Added
- **guomeiqing-safe-install Skill**: Automated 7-point security review for third-party Skill installation
  - Downloads to isolated /tmp directory before any review
  - Checks: outbound data, credential access, dangerous exec, disguised behavior, privilege escalation, hidden instructions, script files
  - Generates human-readable audit report with ✅/⚠️/🔴 ratings
  - Auto-installs safe Skills, asks user confirmation for suspicious ones, rejects dangerous ones
- Updated README with guomeiqing-safe-install Skill description and roadmap

## [1.1.0] - 2026-03-24

### Added
- Repository restructured as security ecosystem (skills + articles + docs + resources)
- Article #1: Is Your Lobster Running Naked? (text + 5 illustrations)
- Article #2: Three Layers of Security Armor (text + 5 illustrations)
- Articles index (`articles/README.md`)
- Threat model placeholder (`docs/threat-model.md`)
- CONTRIBUTING.md with contribution guidelines
- CHANGELOG.md

### Changed
- Moved guomeiqing-security-audit skill to `skills/guomeiqing-security-audit/`
- New root README with full ecosystem overview, roadmap, and article index
- Updated series plan: articles 3-5 pivot to AI-native security topics (Prompt Injection, Trust Chain, Privacy)
- Package.json updated with `repository.directory` field

## [1.0.0] - 2026-03-22

### Added
- Initial release of guomeiqing-security-audit skill
- 7-dimension security scan (DM policy, Gateway, groups, tools, sandbox, file permissions, model security)
- Scoring system (0-100)
- Three-tier hardening plans (basic / standard / strict)
- Published to npm as `guomeiqing-security-audit`
