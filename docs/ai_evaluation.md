# AI evaluation (plan + reporting)

## What AI automates
AI replaces manual work of:
- assigning a topic/category,
- scoring importance (1–5),
- writing a short summary,
- detecting whether user action is required,
- setting an initial status for tracking.

## AI artifacts in this repository
- prompts/main_prompt.md
- prompts/schema.json
- sample_data/test_emails.json (anonymized examples)
- sample_data/expected_rows.json (expected: category/importance/action_required/status)

## Quality metrics (for grading + presentation)
1) JSON validity rate:
   - % of emails where AI output is valid JSON matching the schema
   - target: 95–100%
2) Category accuracy:
   - category matches expected_rows
3) Importance error:
   - average absolute error |predicted - expected| for importance
4) Action_required accuracy:
   - boolean match vs expected_rows
5) Status consistency:
   - status follows the rules (e.g., action_required=true → waiting_user, etc.)

## Test plan
### Offline test (without Gmail)
- Use sample_data/test_emails.json as inputs
- Compare AI outputs to sample_data/expected_rows.json
- Record results (manual table or markdown is enough)

### Live test (Gmail + n8n)
- Run on 20–50 real emails (diverse types)
- Verify:
  - JSON parsing never breaks the workflow
  - category/importance look reasonable
  - action_required is not over-triggered on newsletters
- Collect 5 examples:
  - 3 correct classifications
  - 2 incorrect/edge cases + how to fix the prompt

## Common issues and prompt fixes
- Extra text outside JSON → strengthen “Return ONLY JSON” + include fallback JSON
- category outside enum → enforce allowed list and forbid new categories
- summary copies URLs/OTP codes → explicitly prohibit copying links/codes
- importance too high for newsletters → make the scale stricter (newsletters usually 1)

## Privacy policy (good for grading)
- Store only metadata + short summary in the table
- Avoid storing full email bodies in logs
- sample_data is anonymized
