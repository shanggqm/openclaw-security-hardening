# Contributing Guide

Thank you for your interest in OpenClaw Security! Contributions of any kind are welcome.

## How to Contribute

### 🐛 Report Bugs

1. Search [existing Issues](https://github.com/shanggqm/openclaw-security-hardening/issues) first to avoid duplicates
2. Create a new Issue with clear description:
   - What you did
   - What you expected to happen
   - What actually happened
   - Your environment (OpenClaw version, OS, etc.)

### 💡 Feature Suggestions

1. Create an Issue with the `feature` label
2. Describe clearly:
   - What problem you want to solve
   - Your proposed solution
   - Any alternative approaches

### 📝 Contribute Articles

Security series articles are stored in the `articles/` directory:

1. Fork this repository
2. Create a branch: `git checkout -b docs/add-article-xxx`
3. Create a new directory under `articles/`, containing `README.md` (article body) and `images/` (illustrations)
4. Submit a PR

### 🔧 Contribute Skills

New security-related Skills go in the `skills/` directory:

1. Fork this repository
2. Create a branch: `git checkout -b feat/new-skill-name`
3. Create a new directory under `skills/`, containing at minimum:
   - `SKILL.md` — Skill definition
   - `README.md` — Usage instructions
   - `package.json` — If publishing via npm
4. Submit a PR

## Development Guidelines

### Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(skill): add prompt injection detector
fix(audit): correct score calculation
docs(article): add article #3
```

### Branch Naming

```
feat/<description>      # New features
fix/<description>       # Bug fixes
docs/<description>      # Documentation
refactor/<description>  # Refactoring
```

### PR Requirements

- Title uses Conventional Commits format
- Description includes: What / Why / How
- One PR per concern — don't mix unrelated changes

## Code of Conduct

- Be kind, respectful, and inclusive
- Focus on issues, not people
- Welcome newcomers with patience

## License

Contributed code will be licensed under this project's [MIT License](LICENSE).
