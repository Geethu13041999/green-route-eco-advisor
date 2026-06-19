# GREEN ROUTE🌿 — Eco-Travel Advisor

A conversational agent for **sustainable tourism planning**, built with **Rasa Open Source** (NLU + Core). It helps travellers plan low-carbon trips by recommending eco-certified accommodation, low-emission transport, carbon-offset programmes and community-supporting cultural experiences, calculating approximate carbon footprints, and escalating complex cases to a human advisor with full context.

> MSc Advanced Conversational UI Design and Chatbot Development — Set Exercise.

## Features (mapped to the assignment brief)

| Capability | Where |
|---|---|
| Adaptive multi-turn trip intake (destination, dates, budget, sustainability) | `trip_intake_form` in `domain.yml` |
| Carbon footprint calculation (Climatiq API + offline fallback) | `action_calculate_carbon` |
| Eco-hotel retrieval + **weighted scoring** (carbon + price + preference) | `action_trip_recommendation` |
| Colour-coded result cards (🟢 green / 🟠 amber / 🔴 red) | `json_message` payloads → frontend |
| Human handover with **full context packaging** | `action_human_handover` |
| **Two-stage** fallback / clarification then escalation | `action_default_fallback` |
| Curated verified data (no reliable free API) | `data/mock_db.json` |

## Project structure

```
config.yml              # DIETClassifier pipeline + policies
domain.yml              # intents, entities, slots, responses, form, actions
data/nlu.yml            # training examples
data/stories.yml        # conversation training stories
data/rules.yml          # rules incl. two-stage fallback
data/mock_db.json       # curated eco-certified hotels, transport, offsets
actions/actions.py      # custom actions (Climatiq, scoring, handover, fallback)
tests/test_stories.yml  # test stories for `rasa test core`
endpoints.yml           # action server endpoint (local)
credentials.yml         # REST + socket.io channels
Dockerfile.actions      # action server container
docker-compose.yml      # full local / cloud deployment
.env.example            # API key template (copy to .env)
```

## Quick start (local)

```bash
# 1. Create environment
python -m venv venv && source venv/bin/activate     # Windows: venv\Scripts\activate
pip install rasa==3.6.* rasa-sdk==3.6.*
pip install -r requirements-actions.txt

# 2. Add your API keys (optional - bot works without them via fallback)
cp .env.example .env        # then edit .env
# Load them into your shell so the action server can read them:
export $(grep -v '^#' .env | xargs)     # macOS/Linux

# 3. Train the model
rasa train

# 4. Run the action server (terminal 1)
rasa run actions

# 5. Talk to the bot (terminal 2)
rasa shell
```

### API keys (both have free tiers)
- **Climatiq** — 500 free calls/month — https://www.climatiq.io/
- **Amadeus for Developers** — free sandbox — https://developers.amadeus.com/

Without keys the bot automatically uses indicative emission factors and the curated mock database, so it remains fully demonstrable.

## NLU pipeline note (Task 4)
The default **DIETClassifier** pipeline is used for the best accuracy/latency balance on CPU. An optional DistilBERT pipeline can be enabled by adding the `LanguageModelFeaturizer` (HuggingFace `distilbert-base-uncased`) to `config.yml` — only do this if inference stays under 3 seconds in your environment.

## Testing (Task 5)
```bash
rasa test nlu --nlu data/nlu.yml --cross-validation   # intent/entity accuracy + confusion matrix
rasa test core --stories tests/test_stories.yml        # dialogue / story transitions
```
Results (confusion matrix, intent report, failed stories) are written to `results/`.

## Frontend (Task 4)
The bot emits structured `json_message` payloads (`hotel_cards`, `carbon_card`, `handover`) that a **Rasa Webchat** widget or **ReactJS** client renders as colour-coded cards, quick-reply buttons and a handover indicator. Point the widget at the REST/socket.io endpoint on port 5005.

## Deployment (Task 6)
```bash
docker compose up --build
# Rasa API: http://localhost:5005   |   Action server: internal only (5055)
```
Recommended zero-cost host: **HuggingFace Spaces (Docker SDK)**. Never commit `.env`.

## Ethics & privacy
Environmental figures are presented as *approximate* with their source labelled to avoid greenwashing. No personal data is persisted (in-memory tracker store); the design targets GDPR compliance, screen-reader-friendly text, and sub-3-second responses for critical interactions.
