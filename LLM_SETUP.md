# Enabling the LLM "advisor" (advanced conversational intelligence)

The bot now has an **LLM layer** that answers open-ended, free-form sustainable-travel
questions naturally (e.g. "is flying worse than the train?", "how does carbon
offsetting work?") and powers an intelligent first-stage fallback. It is grounded with
the live trip context and the eco-database, and it's **defensive**: with no key, the bot
still works using its rule-based responses.

Pick ONE provider. **Groq is recommended** (free, instant, very fast — well under the
3-second latency requirement).

## Option A — Groq (free cloud, recommended)

1. Go to **https://console.groq.com** and sign up (free, Google/email).
2. Open **API Keys → Create API Key**, copy it.
3. In `.env`, set:
   ```
   LLM_API_KEY=gsk_your_key_here
   LLM_BASE_URL=https://api.groq.com/openai/v1
   LLM_MODEL=llama-3.1-8b-instant
   ```
4. Reload + restart the action server (must run `export` in that tab):
   ```bash
   export $(grep -v '^#' .env | xargs)
   rasa run actions
   ```

## Option B — Ollama (local, no key, runs on your Mac)

1. Install from **https://ollama.com** (the app starts a local server automatically).
2. Pull a small model (good for an M1): `ollama pull llama3.2`
3. In `.env`, set:
   ```
   LLM_API_KEY=ollama
   LLM_BASE_URL=http://localhost:11434/v1
   LLM_MODEL=llama3.2
   ```
4. Reload + restart the action server as above.

## Test it
- "is flying worse than taking the train?"  → natural LLM answer (eco_question intent)
- "how can I travel more sustainably?"       → LLM advice
- anything off-script                        → intelligent grounded fallback (not "Sorry I didn't catch that")
- "carbon footprint of flying to Tokyo"      → carbon card **+ comparison bar chart** across modes

## Report note
This demonstrates LLM integration in a conversational agent (LO1/LO2) with a
retrieval-grounded prompt (trip context + verified eco-data injected), and graceful
degradation when the LLM is unavailable — a robust, modern design.
