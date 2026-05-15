# Bug Report: Intermittent Character Dropping in LLM Responses

## Summary
The Kiro gateway backend is intermittently dropping characters from LLM-generated responses. The issue is non-deterministic but reproducible, affecting specific character sequences in certain contexts.

## Issue Description
When using the Kiro gateway with AWS SSO OIDC credentials (client_id/client_secret), responses from the backend LLM occasionally have characters or character sequences dropped. This appears to be a backend LLM generation issue rather than a transmission or display problem.

## Examples of Observed Drops

### Confirmed Drops (Session 2026-04-13)
1. "0.0.0.0" → "0.0.0" (dropped ".0" - 2 characters)
2. "Add" → "Ad" (dropped "d" - 1 character)
3. "FamousPeers" → "FamousPers" (dropped "ee" - 2 characters)

### Context
These drops were observed in list-formatted responses during diagnostic testing.

## Diagnostic Testing Results

### Test Results Summary
- **Test 5 (Initial)**: Drops observed in list format
  - Item 0.0.0.0 → Item 0.0.0
  - Item Add → Item Ad
  - Item FamousPeers → Item FamousPers
  - Item aaaa eeee 0000 → Item aaaa eeee 0000 (no drop)

- **Comprehensive Diagnostic Tests**: No drops observed
  - Repeated sequences: 0.0 0.0 0.0, 1.1 1.1 1.1, 0.0.0.0 0.0.0.0 (all intact)
  - Double letters: Add Add Add, See See See, Keep Keep Keep (all intact)
  - Double 'e': Peep, Beep, Jeep, FamousPeers (all intact)
  - Double 'd': Added, Addendum, Address, Add (all intact)
  - Mixed sentences with problematic patterns (all intact)

- **Test 5 (Recreated)**: No drops observed
  - All patterns came through correctly

### Key Finding
The issue is **intermittent and non-deterministic**. The same patterns that dropped in the initial test came through correctly in subsequent tests.

## Root Cause Analysis

### Hypothesis
This appears to be a **backend LLM generation issue**, not transmission or display-related, based on:

1. **Consistency**: The drops are specific to certain character sequences (e.g., ".0", "dd", "ee")
2. **Non-determinism**: Same patterns work sometimes but fail other times
3. **Pattern-specificity**: Not all repeated characters drop (e.g., "aaaa eeee 0000" came through intact)
4. **Context-dependency**: Drops occurred in list-formatted responses but not in other contexts

### Possible Causes
- Token generation variance in the LLM
- Streaming/buffering issues causing non-deterministic output
- Attention mechanism occasionally dropping certain token sequences
- Sampling/temperature effects causing inconsistent token selection
- Token boundary issues where certain character sequences are tokenized in a way that causes drops

## Steps to Reproduce

1. Use Kiro gateway with AWS SSO OIDC credentials (client_id/client_secret)
2. Generate responses containing patterns like:
   - IP addresses: "0.0.0.0"
   - Words with double letters: "Add", "FamousPeers"
   - Dot-separated sequences: "0.0.0.0"
3. Observe responses for missing characters
4. Note: Issue is intermittent - may not occur on every attempt

## Environment
- Kiro Gateway: Multi-account implementation with AWS SSO OIDC support
- Authentication: client_id + client_secret (not refresh_token)
- Deployment: Kubernetes with envFrom secret injection
- Testing Date: 2026-04-13

## Impact
- **Severity**: Medium
- **Frequency**: Intermittent (not every response)
- **Affected Content**: Specific character sequences in certain contexts
- **User Impact**: Potential for confusion or errors if critical information is dropped (e.g., IP addresses, configuration values)

## Recommendation
Investigate the LLM token generation pipeline for:
1. Non-deterministic token selection
2. Token boundary handling for certain character sequences
3. Streaming/buffering consistency
4. Attention mechanism behavior with repeated characters

## Additional Notes
- The issue appears to be backend-specific, not related to the multi-account gateway implementation
- Diagnostic testing suggests the issue is real but context-dependent
- More investigation needed to identify exact trigger conditions
