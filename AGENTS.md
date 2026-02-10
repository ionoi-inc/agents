# Agent Team Roster

## Cloud Workforce (Nebula Platform)

### Nebula Core
**Role**: Main builder, automation orchestrator, cloud operations  
**Capabilities**: Web research, file operations, code execution, task coordination  
**When to use**: Default agent for new projects, general coordination, automation setup  
**Contact**: Via Nebula platform

### Specialized Nebula Sub-Agents
- **GitHub Agent**: Repository management, PR reviews, issue coordination
- **Gmail Agents**: Email operations, inbox monitoring
- **Google Suite Agents**: Docs, Sheets, Calendar, Drive, Tasks operations
- **Telegram Agents**: Messaging and channel management
- **Polymarket Edge Detector**: Trading signal detection
- **Trading Bots**: Automated position monitoring and trading
- **Crypto Analysis Agents**: Technical analysis, sentiment monitoring

## Local Machine Team

### Scrawl (Seafloor / Opus 4.5)
**Role**: NullPriest & Fathom orchestrator, autonomous operations  
**Capabilities**: Social media automation, treasury management, platform integrations, cron orchestration  
**When to use**: NullPriest/Fathom operations, social posting, autonomous agent tasks  
**Projects**: NULP Collective, Fathom  
**Contact**: helpfulnature234@agentmail.to  
**Location**: Seafloor cloud (bot-user7u0pc0)

### Seafloor (Opus 4.5)
**Role**: High-level strategic agent, primary decision maker  
**Capabilities**: Advanced reasoning, strategic planning, complex problem solving  
**When to use**: Major decisions, architecture planning, complex coordination  
**Location**: Local machine

### Claude
**Role**: Server management and deployment  
**Capabilities**: Server configuration, nginx/Apache setup, deployment orchestration, infrastructure documentation  
**When to use**: Server changes, deployments, infrastructure updates  
**Handoff protocol**: Documents changes in agents repo for Nebula reference  
**Location**: Local machine

### Cursor
**Role**: Active code development and editing  
**Capabilities**: Real-time code editing, feature implementation, debugging  
**When to use**: Active development sessions, feature building, code modifications  
**Location**: Local IDE

### Antigravity
**Role**: [Role TBD]  
**Capabilities**: [To be defined]  
**When to use**: [To be defined]  
**Location**: Local IDE

## Coordination Guidelines

### Delegation Hierarchy
1. **Seafloor**: Strategic decisions, architecture, complex problems
2. **Nebula**: Project coordination, automation, cloud operations
3. **Claude**: Server/deployment work
4. **Cursor**: Active development

### When to Delegate

**To Nebula**:
- Creating new repos in ionoi-inc
- Setting up automations and monitoring
- PR reviews and merges
- Research and documentation
- Cross-service coordination

**To Seafloor**:
- Major architectural decisions
- Complex problem solving
- Strategic planning
- Conflict resolution

**To Claude**:
- Server configuration changes
- Website deployments
- Infrastructure updates
- nginx/Apache configuration

**To Cursor**:
- Feature implementation
- Code modifications
- Bug fixes during development
- Real-time editing sessions

## Communication Protocols

### GitHub Issues
Primary communication layer for handoffs. Use tags from ORG.md.

### Documentation
- Claude documents server changes in agents repo
- Nebula documents automation and coordination decisions
- All agents update relevant procedure docs

### Handoff Template
```
**From**: [Agent name]
**To**: [Agent name/tag]
**Context**: [Brief background]
**Task**: [Specific work needed]
**Blockers**: [Any dependencies or issues]
**Acceptance**: [How to verify completion]
```

## Introductions

When joining a new project:
1. Read ORG.md for organizational context
2. Read this file (AGENTS.md) for team structure
3. Check procedures/ for relevant workflows
4. Review open issues in the project repo
5. Introduce yourself in an issue if coordination needed