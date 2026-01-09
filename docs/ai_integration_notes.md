# Integration notes (n8n + AI)

## What n8n should pass into the prompt
Provide these variables (strings):
- from
- subject
- date (optional)
- body (plain text; if HTML, strip tags; can be snippet)

If body is long, truncate to ~4000–6000 chars.

## Expected AI response
AI must return ONLY a JSON object that matches `prompts/schema.json`.

## Parsing
After the AI node:
1) Parse JSON (or use a node that directly returns JSON).
2) Validate:
   - importance in ["important","not_important"]
   - topic in allowed list
   - suggested_label matches IMPORTANT/<TOPIC_UPPER> or NOTIMPORTANT/<TOPIC_UPPER>

## Fallback / safety (must-have)
If JSON parsing fails OR any required field missing:
- Set default:
  - importance: "important"
  - topic: "other"
  - subtopics: []
  - confidence: 0.3
  - reason: "Fallback: invalid AI output."
  - summary: "Nie udało się automatycznie sklasyfikować wiadomości."
  - suggested_label: "IMPORTANT/OTHER"
  - needs_follow_up: true
- Log the raw AI output for debugging (but do not store full email body if you want privacy).

## Recommended routing
- Apply Gmail label = suggested_label
- Optionally star important emails
- Save one row to report storage (Sheets/DB):
  date, from, subject, importance, topic, confidence, summary, suggested_label
