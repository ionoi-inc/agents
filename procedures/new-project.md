# New Project Setup Procedure

Standard operating procedure for creating and initializing new projects in ionoi-inc.

## Overview

**Owner**: Nebula (with GitHub Agent)
**When to use**: Starting any new project, repository, or major feature initiative
**Estimated time**: 15-30 minutes

## Prerequisites

- [ ] Project purpose and scope defined
- [ ] Technology stack decided
- [ ] Repository name chosen (follow naming conventions)
- [ ] Initial team members identified

## Step-by-Step Process

### 1. Repository Creation

**Agent**: GitHub Agent (@github-agent)

```
Create repository in ionoi-inc:
- Name: [project-name]
- Description: [clear one-line description]
- Visibility: Private (default) or Public if applicable
- Initialize with README: Yes
- Add .gitignore: [select appropriate template]
- License: [if applicable]
```

### 2. Initial Repository Structure

**Agent**: Nebula Core or Cursor (depending on complexity)

Create standard directory structure:
```
project-name/
├── README.md           # Project overview and setup instructions
├── .gitignore          # Language/framework specific
├── docs/               # Documentation
│   ├── api.md         # API documentation (if applicable)
│   ├── architecture.md # System design
│   └── setup.md       # Development environment setup
├── src/                # Source code
├── tests/              # Test files
├── .github/            # GitHub workflows and templates
│   └── workflows/     # CI/CD pipelines
└── [additional folders based on stack]
```

### 3. Documentation Setup

**Agent**: Nebula Core

Create/update essential documentation:

**README.md** must include:
- Project description and purpose
- Key features
- Technology stack
- Setup instructions
- Usage examples
- Contributing guidelines
- Links to ORG.md and AGENTS.md in agents repo

**docs/setup.md** should cover:
- Prerequisites (Node.js version, Python version, etc.)
- Installation steps
- Environment variables
- Database setup (if applicable)
- Running locally
- Common issues and troubleshooting

**docs/architecture.md** should explain:
- High-level system design
- Component relationships
- Data flow
- External dependencies
- Deployment architecture

### 4. Development Environment Configuration

**Agent**: Cursor or Claude (depending on local vs. server setup)

Set up development tooling:
- [ ] Install dependencies
- [ ] Configure linting (ESLint, Pylint, etc.)
- [ ] Configure formatting (Prettier, Black, etc.)
- [ ] Set up pre-commit hooks
- [ ] Configure testing framework
- [ ] Set up CI/CD pipeline basics

### 5. Initial Commit and Branch Protection

**Agent**: GitHub Agent

- [ ] Create initial commit with structure
- [ ] Set up branch protection rules for `main`:
  - Require pull request reviews
  - Require status checks to pass
  - Enforce linear history (optional)
- [ ] Create `develop` branch if using Git Flow

### 6. Issue Labels and Project Board

**Agent**: GitHub Agent

Create standard labels:
- `bug` - Something isn't working
- `enhancement` - New feature or request
- `documentation` - Documentation updates
- `good first issue` - Good for newcomers
- `help wanted` - Extra attention needed
- `wip` - Work in progress
- Agent tags: `nebula`, `seafloor`, `claude`, `cursor`, `antigravity`
- Priority: `urgent`, `high`, `medium`, `low`
- Status: `blocked`, `needs-review`, `ready-to-merge`

Optional: Create project board for task tracking

### 7. Team Onboarding

**Agent**: Nebula Core

Create onboarding issue:
- [ ] Link to ORG.md in agents repo
- [ ] Link to AGENTS.md in agents repo
- [ ] Link to project-specific documentation
- [ ] List key contacts and responsible agents
- [ ] Outline current priorities

### 8. Integration Setup

**Agent**: Varies by integration

Configure necessary integrations:
- [ ] CI/CD (GitHub Actions)
- [ ] Code quality tools (SonarCloud, CodeClimate)
- [ ] Deployment pipelines
- [ ] Monitoring and logging
- [ ] Environment variable management

## Post-Setup Checklist

- [ ] All team members have appropriate access
- [ ] Documentation is complete and accurate
- [ ] CI/CD pipeline runs successfully
- [ ] Project appears in ionoi-inc organization
- [ ] Initial issue created for first milestone
- [ ] Handoff communication sent if applicable

## Common Issues

**Issue**: Repository created in wrong organization
**Solution**: Use GitHub Agent to transfer repository to ionoi-inc

**Issue**: Branch protection preventing initial commits
**Solution**: Temporarily disable protection, make commits, re-enable

**Issue**: CI/CD pipeline failing on first run
**Solution**: Check secrets/environment variables are configured in repo settings

## References

- [ORG.md](https://github.com/ionoi-inc/agents/blob/main/ORG.md) - Organizational standards
- [AGENTS.md](https://github.com/ionoi-inc/agents/blob/main/AGENTS.md) - Team roster
- [GitHub Best Practices](https://docs.github.com/en/repositories/creating-and-managing-repositories/best-practices-for-repositories)

---

**Last Updated**: 2026-02-10
**Maintained By**: GitHub Agent