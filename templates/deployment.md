# Deployment Request Template

## Project Information
**Project Name**: [Name]
**Repository**: [Link to GitHub repo]
**Type**: [static site / node app / python service / other]

## Deployment Details

### Domain
**Primary Domain**: [e.g., example.ionoi.com]
**Additional Domains/Subdomains**: [if any]

### Technical Requirements
**Framework/Platform**: [e.g., Next.js, Express, Flask, static HTML]
**Build Command**: [e.g., `npm run build`, `python setup.py`, or "N/A" for static]
**Start Command**: [e.g., `npm start`, `python app.py`, or "N/A" for static]
**Port**: [e.g., 3000, 8080, or "N/A" for static]

### Environment Variables
```
VARIABLE_NAME=value_description
ANOTHER_VAR=another_description
```
[Or write "None" if no environment variables needed]

### Dependencies
- **System packages**: [e.g., Node.js 18+, Python 3.11, nginx]
- **Database**: [PostgreSQL, MongoDB, none, etc.]
- **External services**: [APIs, third-party integrations]

### Special Requirements
- [ ] SSL certificate needed
- [ ] Database setup required
- [ ] Scheduled tasks/cron jobs
- [ ] Webhook endpoints
- [ ] File upload/storage
- [ ] Other: [specify]

## Current Status
**Branch Ready for Deployment**: [branch name, typically `main`]
**Last Commit**: [commit SHA or link]
**Tested Locally**: [Yes/No - describe any testing done]

## Timeline
**Target Deployment Date**: [date or "ASAP"]
**Priority**: [Low / Medium / High / Urgent]

## Post-Deployment
**Monitoring**: [What metrics or logs should be watched]
**Success Criteria**: [How to verify deployment worked]

## Notes
[Any additional context, warnings, or instructions for the deployment team]

---

**Handoff to**: @claude
**Labels**: `deployment`, `needs-claude`