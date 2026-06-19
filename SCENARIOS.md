# Conversation Scenarios & Interaction Design (Task 3)

This document illustrates the chatbot's full dialogue flows across three contrasting
travel scenarios and explains the branching logic that adapts to each user's
sustainability preference and budget. The matching flow diagrams are in `diagrams/`
(`flow_city_break.mermaid`, `flow_rural_ecotour.mermaid`, `flow_business_trip.mermaid`)
and the overall architecture / integration points are in
`diagrams/architecture_and_integration.mermaid`.

The UI elements referenced (quick-reply buttons, the colour-coded hotel **carousel**,
the high-emission **alert** message, and the human-**handover** banner) are all wired to
live bot data in `frontend/index.html`.

---

## Branching logic (shared across scenarios)

The intake form collects four slots — destination, dates, budget and sustainability
preference — then `action_trip_recommendation` ranks options with a weighted scoring
function whose weights shift with the stated sustainability priority:

| Sustainability priority | Carbon weight | Price weight | Certification weight |
|---|---|---|---|
| High | 0.6 | 0.2 | 0.2 |
| Medium | 0.4 | 0.4 | 0.2 |
| Low | 0.2 | 0.6 | 0.2 |

So a sustainability-led user is shown the lowest-carbon stays first, while a
budget-led user sees cheaper options ranked higher — the same data, re-ordered by
preference. Carbon bands drive the card colour (🟢 ≤5 kg, 🟠 ≤10 kg, 🔴 >10 kg CO₂e),
and any red option triggers an explicit alert.

---

## Scenario 1 — Short city break (sustainability-led)

**Persona:** weekend traveller to Lisbon who prioritises low environmental impact.

```
User: help me plan a sustainable trip
Bot:  Which destination are you considering?           [Lisbon] [Copenhagen] [Kyoto] [Costa Rica]
User: Lisbon
Bot:  When are you planning to travel?
User: 12 august
Bot:  Roughly what is your budget?                      [Budget] [Mid-range] [Premium]
User: mid-range
Bot:  How important is sustainability?                  [Very important] [Somewhat] [Not a priority]
User: very important
Bot:  (ranks with carbon-heavy weights)
      🟢 Tejo Sustainable Hostel — EU Ecolabel, €40, 4.1 kg CO₂e/night
      🟠 Casa Verde Eco-Suites — Green Key, €95, 6.2 kg CO₂e/night
      🚆 Low-carbon transport: tram, train
      🎭 Community Fado evening, urban organic farm tour
      ♻️ Offset: Atlantic Reforestation Fund (Gold Standard)
```

**Branching point:** because sustainability = high, the greenest hostel ranks first
even though pricier eco-hotels exist. With sustainability = low, the ordering would
favour price.

---

## Scenario 2 — Eco-tour in a rural area

**Persona:** traveller wanting an immersive, community-supporting trip to Costa Rica.

```
User: I want a rural eco-tour in Costa Rica
Bot:  (form completes the remaining slots)
Bot:  🟢 Pacuare Eco Camp — Rainforest Alliance, off-grid solar
      🟠 Selva Verde Lodge — CST certified, community employment
      ✈️ Live check (Amadeus): N hotels currently listed in Costa Rica
      🚆 Lower-impact transport: shared shuttle preferred over domestic flight
      🎭 Indigenous-led rainforest tour (supports local community)
      ♻️ Offset: Costa Rica Reforestation / FONAFIFO
```

**Branching point:** rural destinations have weaker public transport, so the bot
recommends shared shuttles over domestic flights and foregrounds community-led
activities — demonstrating context-aware suggestions that balance impact with
practical constraints.

---

## Scenario 3 — Carbon-neutral business trip (escalation path)

**Persona:** corporate traveller with a complex, multi-city itinerary.

```
User: I need a carbon-neutral business trip, it's complicated
Bot:  (ranks options, flags a high-emission flight)
      🔴 high-emission flight detected
      ⚠️ Alert: this is a high-emission option — consider rail or offsetting.
User: this is too complex, connect me to a human advisor
Bot:  🤝 Connecting you with a human travel specialist (full context shared).
      [orange handover banner appears]
```

**Branching point:** two routes reach the human advisor — (a) the user explicitly
requests it, or (b) the **two-stage fallback** fires: after repeated low-confidence
messages the bot first re-prompts with constrained quick-reply buttons, and only then
escalates to `action_human_handover`, which packages the collected slots and recent
messages for the specialist.

---

## Mapping to assignment UI requirements

| Required UI element | Implementation |
|---|---|
| Quick-reply buttons for destinations | `utter_ask_destination` buttons → rendered in frontend |
| Carousel displaying eco-friendly hotels | horizontal scrolling `.carousel` in `index.html` |
| Alert messages highlighting high-emission options | red `.alert` triggered on `band === "red"` |
| Clear indication of human handover | orange handover banner + `handover` json payload |
