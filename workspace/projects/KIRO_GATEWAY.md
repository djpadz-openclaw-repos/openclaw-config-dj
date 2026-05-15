# KIRO_GATEWAY.md — Multi-Account Kiro AI Provider Gateway

**Repository:** https://github.com/djpadz-openclaw-gh/kiro-gateway

Multi-account support for the Kiro AI provider gateway, allowing multiple users/profiles to share a single Kiro instance while maintaining independent state and configuration.

## Architecture

**Single Gateway Instance:** One Kiro proxy at `http://localhost:9000` serves multiple accounts

**Account Isolation:**
- Each account has independent state and configuration
- Separate API tokens per account (stored in 1Password)
- Independent request routing and response handling
- No cross-account data leakage

## Configuration

**Accounts:** Configured via environment variables or configuration file
- `dj` — primary account (default)
- `sherra` — secondary account
- Additional accounts can be added following the same pattern

**Token Management:**
- Each account's Kiro API token stored in 1Password: item **"Kiro API"**, vault **OpenClaw**
- Tokens fetched at runtime via 1Password CLI
- Automatic token refresh on expiration

## API Endpoints

All endpoints support account selection via query parameter (defaults to 'dj'):

```
GET  /health?account=dj
POST /api/request?account=dj
GET  /api/status?account=dj
```

## Usage

**Python/TypeScript scripts:**
```python
from kiro_client import KiroClient
client = KiroClient(account="dj")  # or "sherra"
response = client.call_model("prompt", model="kiro/claude-haiku-4.5")
```

**Anthropic SDK with Kiro:**
```python
from anthropic import Anthropic
client = Anthropic(
    api_key="<token-for-account>",
    base_url="http://localhost:9000",
    default_headers={"Authorization": "Bearer <token>"}
)
```

## Deployment

- **Location:** k8s namespace `kiro` (or local service)
- **Port:** 9000 (internal only)
- **Docker:** Built and deployed via standard k8s manifests
- **Restart:** `systemctl restart kiro` or `kubectl rollout restart deployment/kiro -n kiro`

## Integration Points

- **Email automation:** Uses account-specific tokens for email classification
- **ScamBaiter:** Uses account-specific tokens for Beverly responses
- **Cron jobs:** Route through Kiro with appropriate account context
- **Scripts:** All Python/TypeScript scripts use Kiro instead of direct Anthropic API

## Cost Tracking

- Per-account usage can be tracked via Kiro logs
- Model selection rules apply per account (see `MODELS.md`)
- Cost multipliers consistent across all accounts

## Bug Fixes

### 429 Failover (Fixed)

**Problem:** When account 2 ran out of credits and returned a 429 (MONTHLY_REQUEST_COUNT), the gateway returned the error immediately without attempting failover to account 1.

**Solution:** Added failover logic in `routes_openai.py`:
1. Detects 429 status codes or MONTHLY_REQUEST_COUNT error reasons
2. Calls `account_manager.handle_billing_error()` to switch accounts
3. Retries the request with the new account
4. Only returns the error if all accounts fail

**Implementation:** New `_handle_kiro_error_with_failover()` helper function handles error detection and failover logic. Works for both streaming and non-streaming modes.

## Future Enhancements

- Per-account rate limiting
- Per-account usage quotas
- Account-specific model restrictions
- Audit logging per account
