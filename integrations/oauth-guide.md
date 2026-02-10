# OAuth Integration Guide: Building Smooth Integrations Like Nebula

## Overview

This guide documents how to build seamless OAuth integrations inspired by Nebula's approach: **users click to connect, no manual API key hunting required**. By the end, you'll understand OAuth flows, token management, and self-improvement patterns that make integrations delightful.

---

## Table of Contents

1. [Why OAuth Over API Keys](#why-oauth-over-api-keys)
2. [OAuth 2.0 Authorization Code Flow](#oauth-20-authorization-code-flow)
3. [How Nebula's Integration System Works](#how-nebulas-integration-system-works)
4. [Implementation Guide](#implementation-guide)
5. [Best Practices from Nebula](#best-practices-from-nebula)
6. [Self-Improvement Patterns](#self-improvement-patterns)
7. [Quick Wins: 1-Week Implementation Plan](#quick-wins-1-week-implementation-plan)
8. [Migration Path: API Keys → OAuth](#migration-path-api-keys--oauth)

---

## Why OAuth Over API Keys

**User Experience:**
- ✅ **OAuth**: Click button → authorize → done (30 seconds)
- ❌ **API Keys**: Find settings → generate key → copy → paste → validate (5+ minutes, error-prone)

**Security:**
- ✅ **OAuth**: Scoped permissions, automatic expiry, revocable by user
- ❌ **API Keys**: Often full access, no expiry, hard to rotate

**Developer Experience:**
- ✅ **OAuth**: Standard flow across all services
- ❌ **API Keys**: Every service has different auth patterns

**Key Insight**: OAuth moves complexity from the user to the platform. Build it once, win forever.

---

## OAuth 2.0 Authorization Code Flow

### High-Level Steps

```
1. User clicks "Connect GitHub" button
   ↓
2. Redirect to GitHub authorization page
   https://github.com/login/oauth/authorize?
     client_id=YOUR_CLIENT_ID&
     redirect_uri=YOUR_CALLBACK_URL&
     scope=repo,user:email
   ↓
3. User authorizes on GitHub
   ↓
4. GitHub redirects back with authorization code
   https://your-app.com/callback?code=AUTH_CODE
   ↓
5. Exchange code for access token (server-side)
   POST https://github.com/login/oauth/access_token
   ↓
6. Store encrypted token, associate with user
   ↓
7. Use token for API calls on user's behalf
```

### Security Critical Points

1. **Authorization code exchange must happen server-side** (never expose client secret)
2. **Store tokens encrypted** (use Fernet, AWS KMS, or equivalent)
3. **Use HTTPS for all OAuth redirects**
4. **Validate `state` parameter** to prevent CSRF attacks

---

## How Nebula's Integration System Works

### 1. Click-to-Connect UI

```python
# When user requests a GitHub action without connection
if not user.has_github_connection():
    return {
        "status": "auth_required",
        "message": "Connect your GitHub account to continue",
        "auth_url": generate_oauth_url(service="github", user_id=user.id)
    }
```

**Key Features:**
- Detects missing auth automatically
- Generates secure OAuth URL with state parameter
- Returns user-friendly prompt with clickable link

### 2. Token Storage & Encryption

```python
from cryptography.fernet import Fernet
import os

class TokenManager:
    def __init__(self):
        # In production: use AWS KMS, Azure Key Vault, etc.
        self.cipher = Fernet(os.environ['ENCRYPTION_KEY'])
    
    def store_token(self, user_id, service, token_data):
        encrypted = self.cipher.encrypt(json.dumps(token_data).encode())
        db.tokens.insert({
            'user_id': user_id,
            'service': service,
            'access_token': encrypted,
            'refresh_token': self.cipher.encrypt(token_data['refresh_token'].encode()),
            'expires_at': datetime.now() + timedelta(seconds=token_data['expires_in']),
            'created_at': datetime.now()
        })
    
    def get_token(self, user_id, service):
        record = db.tokens.find_one({'user_id': user_id, 'service': service})
        if not record:
            return None
        
        # Auto-refresh if expired
        if record['expires_at'] < datetime.now():
            return self.refresh_token(user_id, service, record)
        
        return json.loads(self.cipher.decrypt(record['access_token']).decode())
```

### 3. Service-Specific Agents with Auto-Auth

```python
class GitHubAgent:
    def __init__(self, user_id):
        self.user_id = user_id
        self.token_manager = TokenManager()
    
    def create_issue(self, repo, title, body):
        token = self.token_manager.get_token(self.user_id, 'github')
        if not token:
            raise AuthRequiredError("Please connect your GitHub account")
        
        response = requests.post(
            f'https://api.github.com/repos/{repo}/issues',
            headers={'Authorization': f'Bearer {token}'},
            json={'title': title, 'body': body}
        )
        
        if response.status_code == 401:
            # Token invalid - trigger re-auth
            self.token_manager.invalidate_token(self.user_id, 'github')
            raise AuthRequiredError("GitHub connection expired, please reconnect")
        
        return response.json()
```

**Benefits:**
- Agents automatically retrieve tokens
- Handle token expiry gracefully
- Trigger re-auth when needed

---

## Implementation Guide

### Step 1: Register OAuth App

**GitHub Example:**
1. Go to GitHub Settings → Developer Settings → OAuth Apps
2. Click "New OAuth App"
3. Fill in:
   - **Application name**: Your App Name
   - **Homepage URL**: https://your-app.com
   - **Callback URL**: https://your-app.com/oauth/callback/github
4. Save `client_id` and `client_secret` to environment variables

### Step 2: Implement Authorization Flow

```python
from flask import Flask, request, redirect, session
import requests
import secrets

app = Flask(__name__)
app.secret_key = os.environ['FLASK_SECRET_KEY']

OAUTH_CONFIGS = {
    'github': {
        'authorize_url': 'https://github.com/login/oauth/authorize',
        'token_url': 'https://github.com/login/oauth/access_token',
        'client_id': os.environ['GITHUB_CLIENT_ID'],
        'client_secret': os.environ['GITHUB_CLIENT_SECRET'],
        'scope': 'repo,user:email'
    }
}

@app.route('/connect/<service>')
def connect_service(service):
    config = OAUTH_CONFIGS.get(service)
    if not config:
        return {"error": "Service not supported"}, 400
    
    # Generate state parameter for CSRF protection
    state = secrets.token_urlsafe(32)
    session[f'{service}_oauth_state'] = state
    
    auth_url = (
        f"{config['authorize_url']}?"
        f"client_id={config['client_id']}&"
        f"redirect_uri={request.url_root}oauth/callback/{service}&"
        f"scope={config['scope']}&"
        f"state={state}"
    )
    
    return redirect(auth_url)

@app.route('/oauth/callback/<service>')
def oauth_callback(service):
    config = OAUTH_CONFIGS.get(service)
    if not config:
        return {"error": "Service not supported"}, 400
    
    # Validate state parameter
    state = request.args.get('state')
    if state != session.get(f'{service}_oauth_state'):
        return {"error": "Invalid state parameter"}, 400
    
    # Exchange authorization code for access token
    code = request.args.get('code')
    token_response = requests.post(
        config['token_url'],
        data={
            'client_id': config['client_id'],
            'client_secret': config['client_secret'],
            'code': code,
            'redirect_uri': f"{request.url_root}oauth/callback/{service}"
        },
        headers={'Accept': 'application/json'}
    )
    
    token_data = token_response.json()
    
    # Store token (encrypted)
    user_id = session.get('user_id')  # Assumes user is logged in
    token_manager = TokenManager()
    token_manager.store_token(user_id, service, token_data)
    
    return redirect('/dashboard?connected=' + service)
```

### Step 3: Auto-Refresh Token Flow

```python
class TokenManager:
    def refresh_token(self, user_id, service, token_record):
        config = OAUTH_CONFIGS[service]
        refresh_token = self.cipher.decrypt(token_record['refresh_token']).decode()
        
        response = requests.post(
            config['token_url'],
            data={
                'client_id': config['client_id'],
                'client_secret': config['client_secret'],
                'refresh_token': refresh_token,
                'grant_type': 'refresh_token'
            },
            headers={'Accept': 'application/json'}
        )
        
        new_token_data = response.json()
        
        # Update stored token
        db.tokens.update_one(
            {'user_id': user_id, 'service': service},
            {'$set': {
                'access_token': self.cipher.encrypt(json.dumps(new_token_data).encode()),
                'expires_at': datetime.now() + timedelta(seconds=new_token_data['expires_in']),
                'updated_at': datetime.now()
            }}
        )
        
        return new_token_data
```

### Step 4: Graceful Degradation

```python
def execute_action(user_id, action_type, params):
    """
    Executes an action, gracefully handling auth failures.
    """
    try:
        if action_type == 'github_create_issue':
            agent = GitHubAgent(user_id)
            return agent.create_issue(**params)
    
    except AuthRequiredError as e:
        return {
            "status": "auth_required",
            "message": str(e),
            "auth_url": url_for('connect_service', service='github', _external=True)
        }
    
    except Exception as e:
        log_error(e, user_id=user_id, action=action_type)
        return {
            "status": "error",
            "message": "Something went wrong. Please try again."
        }
```

---

## Best Practices from Nebula

### 1. Show Connection Status Upfront

```python
# Dashboard endpoint
@app.route('/dashboard')
def dashboard():
    user_id = session.get('user_id')
    connections = {
        'github': token_manager.has_valid_token(user_id, 'github'),
        'gmail': token_manager.has_valid_token(user_id, 'gmail'),
        'slack': token_manager.has_valid_token(user_id, 'slack')
    }
    return render_template('dashboard.html', connections=connections)
```

**UI:**
```
✅ GitHub - Connected
❌ Gmail - Not connected [Connect]
✅ Slack - Connected
```

### 2. Smart Re-Authentication

```python
class TokenManager:
    def invalidate_token(self, user_id, service):
        """Mark token as invalid and trigger re-auth notification."""
        db.tokens.update_one(
            {'user_id': user_id, 'service': service},
            {'$set': {'status': 'invalid', 'invalidated_at': datetime.now()}}
        )
        
        # Send notification
        notify_user(user_id, 
            f"Your {service} connection has expired. "
            f"Click here to reconnect: {generate_oauth_url(service, user_id)}"
        )
```

### 3. Multi-Account Support

```python
# Allow users to connect multiple accounts per service
class TokenManager:
    def store_token(self, user_id, service, token_data, account_label=None):
        db.tokens.insert({
            'user_id': user_id,
            'service': service,
            'account_label': account_label or 'default',
            'access_token': self.cipher.encrypt(json.dumps(token_data).encode()),
            # ... rest of token data
        })
    
    def get_token(self, user_id, service, account_label='default'):
        record = db.tokens.find_one({
            'user_id': user_id,
            'service': service,
            'account_label': account_label
        })
        # ... decrypt and return
```

### 4. Audit Logging

```python
def log_oauth_event(user_id, service, event_type, metadata=None):
    db.oauth_audit_log.insert({
        'user_id': user_id,
        'service': service,
        'event_type': event_type,  # 'connected', 'refreshed', 'revoked', 'expired'
        'metadata': metadata,
        'timestamp': datetime.now(),
        'ip_address': request.remote_addr
    })

# Usage
log_oauth_event(user_id, 'github', 'connected', {'scopes': 'repo,user:email'})
```

---

## Self-Improvement Patterns

### 1. Track Connection Funnel

```python
# Track where users drop off in OAuth flow
class OAuthMetrics:
    def track_step(self, user_id, service, step):
        """
        Steps: initiated, authorized, completed, failed
        """
        db.oauth_metrics.insert({
            'user_id': user_id,
            'service': service,
            'step': step,
            'timestamp': datetime.now()
        })
    
    def get_conversion_rate(self, service, days=7):
        since = datetime.now() - timedelta(days=days)
        
        initiated = db.oauth_metrics.count_documents({
            'service': service,
            'step': 'initiated',
            'timestamp': {'$gte': since}
        })
        
        completed = db.oauth_metrics.count_documents({
            'service': service,
            'step': 'completed',
            'timestamp': {'$gte': since}
        })
        
        return (completed / initiated * 100) if initiated > 0 else 0
```

**Insights:**
- If conversion rate < 70%, investigate redirect issues or confusing UI
- High "authorized" but low "completed" = token exchange failing

### 2. Auto-Detect Required Scopes

```python
class ScopeManager:
    def detect_required_scopes(self, action_type):
        """
        Automatically determine minimum scopes needed for an action.
        """
        scope_map = {
            'github_create_issue': ['repo'],
            'github_list_repos': ['repo', 'read:org'],
            'github_create_pr': ['repo', 'workflow']
        }
        return scope_map.get(action_type, [])
    
    def validate_scopes(self, user_id, service, required_scopes):
        token_record = db.tokens.find_one({'user_id': user_id, 'service': service})
        granted_scopes = token_record.get('scopes', [])  # Store during OAuth
        
        missing = set(required_scopes) - set(granted_scopes)
        if missing:
            raise InsufficientScopesError(
                f"This action requires additional permissions: {', '.join(missing)}. "
                f"Please reconnect your {service} account."
            )
```

**Benefit:** Users only grant minimum necessary permissions, improving trust.

### 3. Failure Analysis

```python
def analyze_oauth_failures():
    """
    Run daily to identify patterns in OAuth failures.
    """
    failures = db.oauth_metrics.aggregate([
        {'$match': {'step': 'failed', 'timestamp': {'$gte': datetime.now() - timedelta(days=1)}}},
        {'$group': {
            '_id': {'service': '$service', 'error': '$metadata.error'},
            'count': {'$sum': 1}
        }},
        {'$sort': {'count': -1}}
    ])
    
    for failure in failures:
        if failure['count'] > 10:
            alert_devs(
                f"OAuth failure spike for {failure['_id']['service']}: "
                f"{failure['_id']['error']} ({failure['count']} occurrences)"
            )
```

---

## Quick Wins: 1-Week Implementation Plan

### Day 1-2: Core OAuth Flow
- Register OAuth apps (GitHub, Google, Slack)
- Implement authorization redirect
- Build callback handler with code exchange
- Set up encrypted token storage

### Day 3: Token Management
- Implement auto-refresh logic
- Add token validation checks
- Build graceful auth failure handling

### Day 4: User Experience
- Create "Connect Account" UI components
- Show connection status in dashboard
- Add reconnect prompts for expired tokens

### Day 5: Service Agents
- Build agent classes (GitHubAgent, GmailAgent, etc.)
- Inject auto-auth into agent methods
- Handle scope validation

### Day 6: Monitoring & Logging
- Add OAuth event logging
- Set up conversion rate tracking
- Create failure alert system

### Day 7: Testing & Polish
- Test full OAuth flow for each service
- Verify token refresh works
- Check graceful degradation
- Document for users

---

## Migration Path: API Keys → OAuth

### Phase 1: Support Both (Weeks 1-4)

```python
class HybridAuthAgent:
    def __init__(self, user_id):
        self.user_id = user_id
        self.token_manager = TokenManager()
    
    def get_credentials(self, service):
        # Try OAuth first
        oauth_token = self.token_manager.get_token(self.user_id, service)
        if oauth_token:
            return {'type': 'oauth', 'token': oauth_token}
        
        # Fall back to API key
        api_key = db.api_keys.find_one({'user_id': self.user_id, 'service': service})
        if api_key:
            return {'type': 'api_key', 'key': api_key['key']}
        
        raise AuthRequiredError(f"Please connect your {service} account")
```

### Phase 2: Encourage Migration (Weeks 5-8)

```python
@app.route('/dashboard')
def dashboard():
    user_id = session.get('user_id')
    
    # Show migration prompt for users still on API keys
    api_key_services = db.api_keys.find({'user_id': user_id})
    migration_needed = []
    
    for record in api_key_services:
        if not token_manager.has_valid_token(user_id, record['service']):
            migration_needed.append(record['service'])
    
    return render_template('dashboard.html', 
        migration_needed=migration_needed,
        migration_benefits=[
            "No more manual key management",
            "Automatic token refresh",
            "Better security with scoped permissions"
        ]
    )
```

### Phase 3: Deprecate API Keys (Week 9+)

1. Announce deprecation with 30-day notice
2. Send email reminders to users still on API keys
3. Disable API key auth, require OAuth
4. Clean up API key storage code

---

## Advanced: Self-Improving OAuth System

### Dynamic Scope Expansion

```python
class SmartOAuthManager:
    def request_additional_scopes(self, user_id, service, new_scopes, reason):
        """
        Request additional scopes mid-session when needed.
        """
        current_scopes = self.get_granted_scopes(user_id, service)
        all_scopes = list(set(current_scopes + new_scopes))
        
        # Generate re-auth URL with updated scopes
        auth_url = generate_oauth_url(
            service=service,
            user_id=user_id,
            scopes=all_scopes,
            prompt='consent'  # Force consent screen to show new permissions
        )
        
        return {
            "status": "additional_scopes_required",
            "message": f"This feature requires additional permissions: {reason}",
            "new_scopes": new_scopes,
            "auth_url": auth_url
        }
```

### Auto-Healing Connections

```python
def auto_heal_connections():
    """
    Background job that proactively refreshes expiring tokens.
    """
    expiring_soon = db.tokens.find({
        'expires_at': {
            '$gte': datetime.now(),
            '$lte': datetime.now() + timedelta(hours=24)
        },
        'status': 'active'
    })
    
    for token in expiring_soon:
        try:
            token_manager = TokenManager()
            token_manager.refresh_token(
                token['user_id'],
                token['service'],
                token
            )
            log_event('token_refreshed_proactively', token['user_id'], token['service'])
        except Exception as e:
            log_error(f"Failed to refresh token: {e}")
            notify_user(token['user_id'], 
                f"We couldn't refresh your {token['service']} connection. "
                f"Please reconnect: {generate_oauth_url(token['service'], token['user_id'])}"
            )
```

---

## Summary: The Nebula OAuth Philosophy

1. **User-first**: Click to connect, zero manual config
2. **Secure by default**: Encrypted storage, auto-refresh, scoped permissions
3. **Self-healing**: Proactive token refresh, graceful failure handling
4. **Transparent**: Show connection status, explain permission needs
5. **Self-improving**: Track metrics, detect failures, optimize flow

**Result**: Users trust your integrations because they "just work." You spend less time on auth bugs and more time building features.

---

## Resources

- [OAuth 2.0 Specification (RFC 6749)](https://tools.ietf.org/html/rfc6749)
- [GitHub OAuth Documentation](https://docs.github.com/en/developers/apps/building-oauth-apps)
- [Google OAuth 2.0 Guide](https://developers.google.com/identity/protocols/oauth2)
- [OWASP OAuth Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html)

---

**Questions or suggestions?** Open an issue or PR in this repo. Let's build better integrations together! 🚀