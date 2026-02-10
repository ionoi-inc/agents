# Agent Coordination & Workflow Guide

Central hub for coordinating between cloud-based Nebula agents and local AI assistants (Claude, Cursor, Antigravity, OpenClaw) in the dutchiono organization.

## The Team

### Cloud Workforce (Nebula)
- **Nebula Core**: Task orchestration, research, automation
- **GitHub Agent**: Repository management, PR reviews, issue tracking
- **Gmail Agents**: Email processing and management
- **Google Suite Agents**: Calendar, Docs, Sheets, Drive, Tasks
- **Telegram Agents**: Messaging and channel management
- **Specialized Agents**: Polymarket, crypto analysis, trading automation

### Local Machine Team
- **Claude**: Server management, user configuration, website deployment
- **Cursor**: Code editing and development
- **Antigravity**: [Role TBD]
- **OpenClaw**: [Role TBD]

## Standard Operating Procedures

### 1. Repository Management
**Owner**: GitHub Agent (Nebula)
- Creating new repositories
- Managing organization settings
- Transferring repositories between accounts
- Issue and PR automation

**Handoff to Claude**: When repositories need server deployment or DNS configuration

### 2. Website Deployment
**Owner**: Claude (Local)
- SSH access and server management
- Nginx/Apache configuration
- SSL certificate management
- Username and permission setup
- Domain and subdomain routing

**Handoff from Nebula**: After code is pushed to GitHub, Claude handles server deployment

### 3. Code Development
**Owner**: Cursor (Local) for active development
**Owner**: Nebula for automated fixes and updates

**Coordination**: 
- Nebula can make automated commits for bug fixes, dependency updates
- Cursor handles feature development and refactoring
- Use branch protection to prevent conflicts

### 4. Documentation
**Owner**: Shared responsibility
- **Nebula**: Auto-generates docs, API references, workflow guides
- **Claude**: Server-specific documentation, deployment guides
- **Location**: Store in respective repo `/docs` folders or this agents repo

## Communication Protocols

### For Nebula → Claude Handoffs
Create an issue in the target repository with:
```
Title: [DEPLOY] Project Name - Ready for Server Setup
Labels: deployment, needs-claude

Body:
- Repository: [link]
- Type: [static site / node app / python service]
- Domain: [desired domain/subdomain]
- Environment variables: [list if needed]
- Special requirements: [ports, dependencies, etc]
```

### For Claude → Nebula Handoffs
**Method 1**: GitHub issue in agents repo
**Method 2**: Document in `/deployments/[project-name].md` with status updates

### For General Coordination
- Use this agents repo's Issues for cross-platform discussions
- Tag with appropriate labels: `nebula`, `claude`, `cursor`, `urgent`, `blocked`

## Workflow Examples

### New Website Project
1. **Nebula**: Creates repository, initializes structure
2. **Cursor**: Develops features locally
3. **Nebula**: Reviews PRs, runs tests, merges to main
4. **Claude**: Receives deployment notification, sets up server
5. **Claude**: Confirms deployment, provides live URL

### Bug Fix Flow
1. **User reports issue** (via any channel)
2. **Nebula**: Investigates, creates GitHub issue
3. **Cursor or Nebula**: Implements fix
4. **GitHub Agent**: Merges PR after review
5. **Claude**: Auto-deploys if CI/CD configured, or deploys manually

### Server Configuration Change
1. **Claude**: Makes server changes (nginx, SSL, users)
2. **Claude**: Documents in `/server-config/[date]-changes.md`
3. **Nebula**: Reads documentation for future reference

## Repository Structure
```
agents/
├── README.md (this file)
├── procedures/
│   ├── deployment.md
│   ├── github-workflow.md
│   ├── server-management.md
│   └── emergency-protocols.md
├── deployments/
│   ├── active-sites.md
│   └── [project-name].md
├── server-config/
│   └── [date]-changes.md
└── templates/
    ├── deployment-request.md
    └── handoff-issue.md
```

## Quick Reference

### Nebula Capabilities
- Web research and scraping
- API integrations (100+ apps via OAuth)
- Code execution (Python, Bash)
- File operations
- Automated task scheduling
- Multi-agent delegation

### Claude Capabilities
- SSH server access
- System administration
- Web server configuration
- Database management
- User and permission management

### Cursor Capabilities
- Local development environment
- Code editing with AI assistance
- Git operations
- Real-time code suggestions

## Emergency Protocols

### Production Site Down
1. **Claude**: Investigate server status, logs
2. **Claude**: Attempt immediate fix (restart services, check configs)
3. **Nebula**: Create incident issue, notify stakeholders
4. **Both**: Document root cause and resolution

### Deployment Rollback Needed
1. **Claude**: Revert to previous working deployment
2. **Nebula**: Create GitHub issue with failure details
3. **Cursor/Nebula**: Fix issue in development
4. **Claude**: Re-deploy after verification

---

**Last Updated**: 2026-02-10
**Maintained By**: All agents (update as procedures evolve)
**Questions**: Create an issue with label `question`