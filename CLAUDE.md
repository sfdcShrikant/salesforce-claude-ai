# CLAUDE.md

## PROJECT CONTEXT
- Salesforce Edition: Enterprise / Developer
- Primary Objects: Account, Contact, Case, Opportunity
- Coding Standards: trigger handler pattern, separation of concerns, bulkification
- Naming: [Object]TriggerHandler, [Object]Service, [Object]Test
- Test Coverage Target: 85%+ with assertions
- No hardcoded IDs or record type names
- Use Custom Labels for user-facing strings
- Use Custom Metadata for configuration values

## OPERATING RULES
- Plan mode first, edit mode second. Always.
- Verify field API names from metadata before writing formulas or flow conditions.
- Deploy report is the source of truth, not the initial noisy CLI output.
- Narrow prompts: one class, one flow, one object. Whole-repo prompts waste tokens and reduce quality.

## AGENT DEFINITIONS

### ADMIN_AGENT
Role: Declarative configuration assistant
Prompt: Suggest declarative solutions only. Reference Setup navigation paths. No code.
Max Tokens: 512

### DEV_AGENT
Role: Apex, LWC, SOQL development
Prompt: Production-grade code. Trigger handler pattern. CRUD/FLS. Always include test class.
Max Tokens: 2048

### TEST_AGENT
Role: Test automation and QA
Prompt: Test classes, data factories, bulk (200+), negative tests, permission checks. 85%+ coverage.
Max Tokens: 1024

## GUARDRAILS
- Never include PII in API calls or responses
- Add disclaimer on advisory content
- Rate limit: 50 calls/user/hour
- On API timeout: return user-friendly error message
