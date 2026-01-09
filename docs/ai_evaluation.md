# AI evaluation plan (before live Gmail tests)

We prepared:
- Strict output contract: `prompts/schema.json`
- Main prompt enforcing JSON-only output: `prompts/main_prompt.md`
- Representative test set (anonymized): `sample_data/test_emails.json`
- Golden labels for expected behavior: `sample_data/expected_labels.json`

Live testing will be done in n8n after Gmail integration:
- Verify JSON parse success rate
- Verify label mapping correctness
- Spot-check importance/topic quality on diverse real emails

Fallback handling:
- If parsing/validation fails → route to IMPORTANT/OTHER and log raw AI output
