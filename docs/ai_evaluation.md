# Integration notes (n8n + AI) — Email triage into a final table

## Goal
n8n fetches an email from Gmail, sends key fields to AI, AI returns a strict JSON (5 fields),
and n8n writes a full row into the final table.

## Final table columns (output)
- date (from Gmail)
- from (from Gmail)
- subject (from Gmail)
- category (from AI)
- importance (1–5) (from AI)
- Ai summary (from AI)
- action requiered (from AI)  ← keep this spelling if your sheet uses it
- status (from AI)

## What n8n sends to AI
Inject into the prompt:
- from: sender (e.g., "Name <email@...>")
- subject: subject line
- date: sent/received timestamp (ISO if possible)
- body: plain text body OR snippet (may be incomplete)

### Preprocessing (important)
- Strip HTML → plain text
- Truncate body to ~6000 characters (signatures/quoted threads can be huge)
- If body is empty → use snippet or subject

## Expected AI response
AI must return ONLY a JSON object matching `prompts/schema.json`:
- category: enum
- importance: integer 1..5
- ai_summary: short, no full URLs/codes/IDs
- action_required: boolean
- status: enum

## Parsing + validation in n8n
After the AI node:
1) Parse JSON
2) Validate:
   - category is in the allowed list
   - importance is within [1..5]
   - action_required is boolean
   - status is one of the allowed values
3) If parsing/validation fails → fallback below (do NOT stop the workflow)

## Fallback / safety (must-have)
If JSON parsing fails or required fields are missing, set:
- category: "other"
- importance: 3
- ai_summary: "Auto-classification failed for this email."
- action_required: true
- status: "needs_review"

Also:
- Log the raw AI output for debugging
- Do NOT store full email bodies in logs (privacy)

## Mapping into the final table row
Write one row as:
- date/from/subject → from Gmail trigger
- category/importance/ai_summary/action_required/status → from AI

Notes:
- Column “Ai summary” ← `ai_summary`
- Column “action requiered” ← `action_required` (sheet may contain a typo)

## Recommended status rules (aligned with the prompt)
- action_required = true → usually "waiting_user"
- importance <= 2 and action_required = false → "done"
- suspicious/phishing-like security messages → "needs_review" (highest priority)
