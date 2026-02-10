# OAuth Integration Guide: Building Smooth Integrations Like Nebula

*A comprehensive guide for Seaflor to implement user-friendly OAuth flows and self-improvement patterns*

## Overview

Nebula uses **OAuth 2.0 Authorization Code Flow** with automatic token management to provide seamless "click to connect" experiences. Instead of asking users for API keys or tokens manually, users authenticate once through the service's official login page, and Nebula handles all token storage, refresh, and injection automatically.

**Key Principle**: Users should never see or handle API tokens. The system manages authentication lifecycle completely behind the scenes.

---

## How Nebula's Integration System Works

### Architecture Overview

```
User Request → Nebula Agent → OAuth Manager → Service API
                    ↓              ↓
              Token Check    Auto-Refresh
                    ↓              ↓
              Inject Auth    Return Data
```

### Core Components

**1. OAuth Authorization Flow**
- User clicks "Connect [Service]" button
- Redirect to service's authorization page (e.g., GitHub OAuth, Google consent screen)
- User authenticates with their service credentials
- Service redirects back with authorization code
- System exchanges code for access token + refresh token
- Tokens stored securely, associated with user account

**2. Token Management System**
- **Access Tokens**: Short-lived (1 hour typical), used for API calls
- **Refresh Tokens**: Long-lived (no expiry or 90+ days), used to get new access tokens
- **Auto-refresh**: System automatically refreshes expired access tokens using refresh token
- **Secure Storage**: Tokens encrypted at rest, never exposed to end users

**3. Agent Integration Layer**
- Each app has dedicated agent (e.g., `gmail-agent`, `github-agent`)
- Agent tools automatically inject auth headers: `Authorization: Bearer {access_token}`
- Agents don't handle token logic - delegated to OAuth manager
- Seamless multi-account support (user can connect multiple Gmail accounts)

---

## Implementation Guide for Seaflor

### Phase 1: OAuth Infrastructure (Foundation)

**Step 1: Set Up OAuth Application Registration**

For each service you want to integrate (GitHub, Google, Slack, etc.):

1. Register OAuth application with the service
   - GitHub: Settings → Developer Settings → OAuth Apps
   - Google: Google Cloud Console → APIs & Services → Credentials
   - Slack: api.slack.com/apps
   
2. Configure OAuth settings:
   ```
   Application Name: Seaflor
   Homepage URL: https://yourapp.com
   Authorization Callback URL: https://yourapp.com/oauth/callback/{service}
   ```

3. Note your credentials:
   - Client ID (public)
   - Client Secret (private, store securely)
   - Required Scopes (permissions needed)

**Step 2: Build Authorization Flow**

Implement OAuth 2.0 Authorization Code Flow:

```python
# Example: Initiating OAuth flow
from flask import Flask, redirect, request, session
import requests
import secrets

app = Flask(__name__)

# Configuration
GITHUB_CLIENT_ID = "your_client_id"
GITHUB_CLIENT_SECRET = "your_client_secret"
REDIRECT_URI = "https://yourapp.com/oauth/callback/github"

@app.route('/connect/github')
def connect_github():
    # Generate random state for CSRF protection
    state = secrets.token_urlsafe(32)
    session['oauth_state'] = state
    
    # Redirect to GitHub authorization page
    auth_url = (
        f"https://github.com/login/oauth/authorize"
        f"?client_id={GITHUB_CLIENT_ID}"
        f"&redirect_uri={REDIRECT_URI}"
        f"&scope=repo,user"
        f"&state={state}"
    )
    return redirect(auth_url)

@app.route('/oauth/callback/github')
def github_callback():
    # Verify state to prevent CSRF
    state = request.args.get('state')
    if state != session.get('oauth_state'):
        return "Invalid state", 400
    
    # Exchange authorization code for access token
    code = request.args.get('code')
    token_response = requests.post(
        'https://github.com/login/oauth/access_token',
        data={
            'client_id': GITHUB_CLIENT_ID,
            'client_secret': GITHUB_CLIENT_SECRET,
            'code': code,
            'redirect_uri': REDIRECT_URI
        },
        headers={'Accept': 'application/json'}
    )
    
    tokens = token_response.json()
    access_token = tokens['access_token']
    refresh_token = tokens.get('refresh_token')  # Not all services provide this
    
    # Store tokens securely (encrypted database)
    store_tokens(
        user_id=current_user.id,
        service='github',
        access_token=access_token,
        refresh_token=refresh_token
    )
    
    return "GitHub connected successfully!"
```

**Step 3: Implement Token Storage & Encryption**

```python
from cryptography.fernet import Fernet
import os

# Generate encryption key (store securely, environment variable)
ENCRYPTION_KEY = os.environ.get('TOKEN_ENCRYPTION_KEY')
cipher = Fernet(ENCRYPTION_KEY)

def store_tokens(user_id, service, access_token, refresh_token=None):
    # Encrypt tokens before storage
    encrypted_access = cipher.encrypt(access_token.encode())
    encrypted_refresh = cipher.encrypt(refresh_token.encode()) if refresh_token else None
    
    # Store in database
    db.execute("""
        INSERT INTO oauth_tokens (user_id, service, access_token, refresh_token, created_at)
        VALUES (?, ?, ?, ?, ?)
        ON CONFLICT (user_id, service) DO UPDATE SET
            access_token = excluded.access_token,
            refresh_token = excluded.refresh_token,
            created_at = excluded.created_at
    """, (user_id, service, encrypted_access, encrypted_refresh, datetime.now()))

def get_access_token(user_id, service):
    row = db.execute(
        "SELECT access_token, refresh_token, created_at FROM oauth_tokens WHERE user_id = ? AND service = ?",
        (user_id, service)
    ).fetchone()
    
    if not row:
        raise Exception(f"No {service} account connected")
    
    encrypted_access, encrypted_refresh, created_at = row
    access_token = cipher.decrypt(encrypted_access).decode()
    
    # Check if token is expired (typically 1 hour)
    if datetime.now() - created_at > timedelta(hours=1):
        # Auto-refresh if expired
        access_token = refresh_access_token(user_id, service, encrypted_refresh)
    
    return access_token
```

**Step 4: Auto-Refresh Expired Tokens**

```python
def refresh_access_token(user_id, service, encrypted_refresh_token):
    refresh_token = cipher.decrypt(encrypted_refresh_token).decode()
    
    if service == 'github':
        # GitHub doesn't use refresh tokens - access tokens don't expire
        return access_token
    
    elif service == 'google':
        # Google uses refresh tokens
        response = requests.post(
            'https://oauth2.googleapis.com/token',
            data={
                'client_id': GOOGLE_CLIENT_ID,
                'client_secret': GOOGLE_CLIENT_SECRET,
                'refresh_token': refresh_token,
                'grant_type': 'refresh_token'
            }
        )
        
        new_access_token = response.json()['access_token']
        
        # Update stored token
        store_tokens(user_id, service, new_access_token, refresh_token)
        
        return new_access_token
```

### Phase 2: Agent Integration Layer

**Step 5: Create Service-Specific Agents**

Each service gets a dedicated agent that:
- Uses OAuth tokens automatically
- Provides high-level actions (not raw API calls)
- Handles errors and retries

```python
class GitHubAgent:
    def __init__(self, user_id):
        self.user_id = user_id
        self.base_url = "https://api.github.com"
    
    def _get_headers(self):
        access_token = get_access_token(self.user_id, 'github')
        return {
            'Authorization': f'Bearer {access_token}',
            'Accept': 'application/vnd.github+json',
            'X-GitHub-Api-Version': '2022-11-28'
        }
    
    def create_issue(self, repo, title, body):
        """High-level action: Create GitHub issue"""
        response = requests.post(
            f"{self.base_url}/repos/{repo}/issues",
            headers=self._get_headers(),
            json={'title': title, 'body': body}
        )
        
        if response.status_code == 401:
            # Token expired - refresh and retry
            refresh_access_token(self.user_id, 'github')
            return self.create_issue(repo, title, body)
        
        return response.json()
    
    def list_repositories(self):
        """High-level action: List user repositories"""
        response = requests.get(
            f"{self.base_url}/user/repos",
            headers=self._get_headers(),
            params={'sort': 'updated', 'per_page': 100}
        )
        return response.json()
```

**Step 6: Build Unified Agent Interface**

```python
class IntegrationManager:
    """Central hub for all service integrations"""
    
    def __init__(self, user_id):
        self.user_id = user_id
        self.agents = {}
    
    def get_agent(self, service):
        """Lazy-load service agents"""
        if service not in self.agents:
            if service == 'github':
                self.agents[service] = GitHubAgent(self.user_id)
            elif service == 'gmail':
                self.agents[service] = GmailAgent(self.user_id)
            # ... more services
        
        return self.agents[service]
    
    def execute_task(self, service, action, **params):
        """Universal task execution"""
        agent = self.get_agent(service)
        method = getattr(agent, action)
        return method(**params)

# Usage
manager = IntegrationManager(user_id=123)
manager.execute_task('github', 'create_issue', 
                    repo='user/repo', 
                    title='Bug found', 
                    body='Description here')
```

### Phase 3: User Experience

**Step 7: Connection Status UI**

Show users which services are connected:

```python
@app.route('/integrations')
def integrations_page():
    connected_services = db.execute(
        "SELECT service, created_at FROM oauth_tokens WHERE user_id = ?",
        (current_user.id,)
    ).fetchall()
    
    available_services = [
        {'name': 'GitHub', 'slug': 'github', 'icon': 'github.svg'},
        {'name': 'Google Calendar', 'slug': 'google_calendar', 'icon': 'google.svg'},
        {'name': 'Slack', 'slug': 'slack', 'icon': 'slack.svg'},
    ]
    
    for service in available_services:
        service['connected'] = any(s[0] == service['slug'] for s in connected_services)
    
    return render_template('integrations.html', services=available_services)
```

**HTML Template Example**:
```html
<div class="integrations-grid">
  {% for service in services %}
  <div class="integration-card">
    <img src="/static/icons/{{ service.icon }}" />
    <h3>{{ service.name }}</h3>
    
    {% if service.connected %}
      <span class="badge-connected">Connected ✓</span>
      <button onclick="disconnect('{{ service.slug }}')">Disconnect</button>
    {% else %}
      <button onclick="connect('{{ service.slug }}')">Connect</button>
    {% endif %}
  </div>
  {% endfor %}
</div>

<script>
function connect(service) {
  window.location.href = `/connect/${service}`;
}

function disconnect(service) {
  if (confirm('Disconnect from ' + service + '?')) {
    fetch(`/disconnect/${service}`, {method: 'POST'})
      .then(() => location.reload());
  }
}
</script>
```

---

## Best Practices from Nebula's Implementation

### 1. **Never Expose Tokens to Users**
❌ Bad: "Enter your GitHub Personal Access Token"
✅ Good: "Click to connect GitHub" → OAuth flow

### 2. **Handle Token Refresh Transparently**
- Check token expiry before every API call
- Auto-refresh expired tokens without user intervention
- Retry failed requests after refresh
- Never show "token expired" errors to users

### 3. **Support Multiple Accounts Per Service**
```python
# User can connect multiple Gmail accounts
db schema:
  oauth_tokens (
    user_id,
    service,
    account_identifier,  # e.g., email address
    access_token,
    refresh_token,
    PRIMARY KEY (user_id, service, account_identifier)
  )
```

### 4. **Graceful Degradation**
```python
def execute_task(service, action, **params):
    try:
        return agent.action(**params)
    except TokenExpiredError:
        refresh_and_retry()
    except NoConnectionError:
        return {
            'error': f'No {service} account connected',
            'action_required': f'connect_{service}',
            'connect_url': f'/connect/{service}'
        }
```

### 5. **Security Considerations**
- **State parameter**: Prevent CSRF attacks
- **Token encryption**: Encrypt tokens at rest (Fernet, AES-256)
- **HTTPS only**: Never send tokens over HTTP
- **Minimal scopes**: Request only necessary permissions
- **Token rotation**: Support refresh token rotation
- **Audit logging**: Log all token access and API calls

### 6. **Error Handling & User Communication**
```python
# Good error messages
if not has_github_connection:
    return "I need access to your GitHub to create issues. [Connect GitHub](/connect/github)"

# Not: "Error: No OAuth token found for service 'github' in database"
```

---

## Self-Improvement Patterns

### Pattern 1: Monitor API Usage & Add Integrations Proactively

```python
# Track which API calls users attempt
class UsageAnalytics:
    def log_attempt(self, user_id, service, action, success):
        db.execute("""
            INSERT INTO api_usage_log (user_id, service, action, success, timestamp)
            VALUES (?, ?, ?, ?, ?)
        """, (user_id, service, action, success, datetime.now()))
    
    def suggest_new_integrations(self):
        # Find frequently requested but unavailable services
        top_missing = db.execute("""
            SELECT service, COUNT(*) as attempts
            FROM api_usage_log
            WHERE success = FALSE
            GROUP BY service
            ORDER BY attempts DESC
            LIMIT 10
        """).fetchall()
        
        return [f"Add {service} integration (requested {count}x)" 
                for service, count in top_missing]
```

**Action**: When enough users request a service, prioritize building that integration.

### Pattern 2: Learn from Failed Requests

```python
def handle_api_error(service, action, error_response):
    # Parse error to understand what went wrong
    if 'insufficient permissions' in error_response.text:
        # Log that we need additional scopes
        db.execute("""
            INSERT INTO scope_gaps (service, action, missing_scope, count)
            VALUES (?, ?, ?, 1)
            ON CONFLICT (service, action) DO UPDATE SET count = count + 1
        """, (service, action, extract_scope(error_response)))
        
        # Suggest user to reconnect with expanded permissions
        return f"I need additional {service} permissions. [Reconnect with expanded access](/reconnect/{service})"
```

### Pattern 3: Optimize Token Refresh Strategy

```python
# Predictive refresh: refresh before expiry
def smart_get_token(user_id, service):
    token_data = get_token_data(user_id, service)
    time_until_expiry = token_data.expires_at - datetime.now()
    
    # Refresh if less than 5 minutes remaining
    if time_until_expiry < timedelta(minutes=5):
        return refresh_access_token(user_id, service)
    
    return token_data.access_token
```

### Pattern 4: User-Driven Feature Discovery

```python
# Let AI discover what's possible
class CapabilityDiscovery:
    def explore_api(self, service):
        """Automatically discover available API endpoints"""
        agent = self.get_agent(service)
        
        # Try common patterns
        endpoints_to_try = [
            '/user/repos', '/user/issues', '/user/notifications',
            '/search/repositories', '/gists', '/teams'
        ]
        
        available = []
        for endpoint in endpoints_to_try:
            try:
                response = agent.request('GET', endpoint)
                if response.status_code == 200:
                    available.append(endpoint)
            except:
                pass
        
        # Store discovered capabilities
        update_agent_capabilities(service, available)
```

### Pattern 5: Automatic Documentation Generation

```python
def document_new_integration(service, agent_class):
    """Auto-generate docs from agent code"""
    methods = [m for m in dir(agent_class) if not m.startswith('_')]
    
    doc = f"# {service.title()} Agent\n\n"
    doc += "## Available Actions\n\n"
    
    for method_name in methods:
        method = getattr(agent_class, method_name)
        docstring = method.__doc__ or "No description"
        params = inspect.signature(method).parameters
        
        doc += f"### `{method_name}`\n"
        doc += f"{docstring}\n\n"
        doc += "**Parameters:**\n"
        for param in params:
            if param != 'self':
                doc += f"- `{param}`\n"
        doc += "\n"
    
    # Save to docs folder
    with open(f'docs/agents/{service}.md', 'w') as f:
        f.write(doc)
```

---

## Migration Path: From API Keys to OAuth

If Seaflor currently uses API keys/tokens, here's the transition strategy:

### Phase 1: Parallel Support (Months 1-2)
- Keep existing API key input working
- Add OAuth option as alternative
- Mark OAuth as "Recommended" in UI

### Phase 2: Encourage Migration (Months 3-4)
- Add banner: "Switch to secure OAuth connection"
- Show benefits: "No need to manage tokens", "Auto-refresh", "Revoke anytime"
- Provide one-click migration for existing users

### Phase 3: Deprecate API Keys (Month 5+)
- Set deadline for API key sunset
- Email users with migration instructions
- Keep read-only access for API keys, OAuth required for write operations

---

## Quick Wins: Implement These First

### 1-Week Implementation Plan

**Day 1-2**: GitHub OAuth
- Easiest to implement (no refresh tokens needed)
- High user demand (code operations)
- Simple API

**Day 3-4**: Google OAuth (Gmail or Calendar)
- Teaches refresh token handling
- High value for productivity use cases

**Day 5**: Token management infrastructure
- Encryption, storage, auto-refresh
- Foundation for all future integrations

**Day 6-7**: UI for connection management
- Show connected accounts
- "Connect" buttons with OAuth flow
- Disconnect functionality

### After First Week: Scale Rapidly

Once infrastructure is built, adding new services is fast:
- **Slack**: 2-3 hours (webhooks, messaging, channels)
- **Notion**: 2-3 hours (databases, pages, blocks)
- **Linear**: 1-2 hours (issues, projects, teams)
- **Stripe**: 2-3 hours (payments, customers, subscriptions)

---

## Resources & References

### OAuth Libraries
- **Python**: `Authlib`, `OAuthlib`, `requests-oauthlib`
- **Node.js**: `passport.js`, `simple-oauth2`, `grant`
- **Ruby**: `omniauth`, `doorkeeper`

### Service-Specific OAuth Guides
- GitHub: https://docs.github.com/en/apps/oauth-apps/building-oauth-apps
- Google: https://developers.google.com/identity/protocols/oauth2
- Slack: https://api.slack.com/authentication/oauth-v2
- Microsoft: https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow

### Security Standards
- OAuth 2.0 RFC: https://datatracker.ietf.org/doc/html/rfc6749
- OAuth 2.0 Security Best Practices: https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics
- PKCE (for mobile/SPA): https://datatracker.ietf.org/doc/html/rfc7636

---

## Support & Iteration

### How Nebula Iterates

1. **User request tracking**: Log every failed API call or missing integration
2. **Weekly review**: Team reviews top requested features
3. **Rapid deployment**: New integrations shipped within days
4. **User testing**: Beta users try new integrations before general release
5. **Continuous monitoring**: Error tracking, usage analytics, performance metrics

### Suggested Seaflor Roadmap

**Month 1**: Core OAuth infrastructure + GitHub + Google
**Month 2**: Add 3-5 high-demand services (Slack, Notion, Linear)
**Month 3**: Webhook support for real-time updates
**Month 4**: Multi-account management UI improvements
**Month 5**: Self-service integration requests (users can request new services)
**Month 6**: Custom OAuth apps (enterprise users can use their own OAuth credentials)

---

## Questions to Consider

Before implementing, decide:

1. **Token storage**: Database (PostgreSQL, MySQL) or secure vault (HashiCorp Vault, AWS Secrets Manager)?
2. **Refresh strategy**: Predictive (refresh 5 min before expiry) or reactive (refresh on 401 error)?
3. **Multi-account support**: Do users need multiple accounts per service?
4. **Revocation**: How do users disconnect services? Immediate or delayed?
5. **Audit logging**: Track all API calls for security/debugging?
6. **Rate limiting**: How to handle API rate limits per service?

---

**Last Updated**: 2026-02-10
**Maintained By**: Nebula Team
**Questions**: Create issue in this repo with label `oauth-integration`

---

## Appendix: Complete Example (GitHub Integration)

See `/integrations/examples/github-oauth-complete.py` for a fully working example including:
- Flask OAuth flow
- Token encryption
- Auto-refresh
- Agent implementation
- Error handling
- Multi-account support

---

*This guide is based on Nebula's production OAuth implementation powering 100+ app integrations. Adapt patterns to fit your architecture and use cases.*