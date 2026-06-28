# Tool-Using Travel Assistant

> An LLM travel agent that plans trips by orchestrating **7 real tools** —
> flights, hotels, restaurants, weather, currency, an FAQ knowledge base, and a
> day-by-day itinerary planner — through structured tool calling with robust
> fallbacks.

![Python](https://img.shields.io/badge/python-3.10+-blue)
![LangGraph](https://img.shields.io/badge/agent-LangGraph-orange)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

---

## Overview

A single chat model is given a toolbox and a system policy, then left to decide
*which* tool to call and *with what arguments* to satisfy a travel request. The
agent parses natural-language dates, maps city names to IATA / currency codes,
calls live travel APIs, and falls back to public datasets or cached data when an
external service is unavailable — so it degrades gracefully instead of failing.

**The 7 tools**
1. `flight_search(origin, destination, date)` — flights (≥3 options, Amadeus API)
2. `hotel_search(destination, check_in, check_out, budget)` — hotels (≥3 options)
3. `restaurant_search(destination)` — top local restaurants/foods
4. `get_weather_info(destination, date)` — weather forecast
5. `get_currency_info(origin_country, destination_country, amount)` — FX conversion
6. `faq_search(query)` — RAG over a travel-agency FAQ (LanceDB vector search)
7. `trip_planning(destination, arrival, departure, interests)` — RAG-grounded,
   day-by-day itinerary (Morning/Afternoon/Evening + practical tips)

---

## Results

A lightweight evaluation suite of **14 scenarios** (two per tool) checks both
that the agent routes to the correct tool and that the response contains the
expected fields/keywords.

| Aspect | Result |
|---|---|
| Correct tool routing | **14 / 14 (100%)** |
| Itinerary quality | Well-structured multi-day plans aligned to stated interests |

The notebook records the full output and a short analysis for every scenario,
including honest notes on weaker spots (e.g. some weather fields returning N/A).

---

## Architecture

```
User message
   │
   ▼
ChatOpenAI (gpt-4o-mini) .bind_tools(TOOLS)   ◄── system policy: when to use each tool,
   │                                              date normalization, output rules
   ▼
LangGraph: agent node ⇄ ToolNode(TOOLS)
   │            ▲   │
   │            └───┘  (loop until the model stops requesting tools)
   ▼
Final grounded answer

Data/APIs: Amadeus (flights, hotels) · Kaggle datasets (IATA airport codes,
country→currency) with public GitHub fallbacks · LanceDB FAQ index · web-search
fallback
```

---

## Tech Stack

- **Language:** Python 3.10+
- **Agent:** LangChain + LangGraph (`bind_tools`, `ToolNode`)
- **LLM:** `gpt-4o-mini` via an OpenAI-compatible endpoint (configurable)
- **Travel data:** Amadeus API; Kaggle datasets (IATA, currency) via `kagglehub`
- **Retrieval:** LanceDB + sentence-transformers embeddings (FAQ / itinerary RAG)

---

## Getting Started

> Written for Google Colab / Jupyter. The notebook installs its own
> dependencies in the first cells.

1. Open `NLP_CA5_Q2_Harandi_810199596.ipynb`.
2. Set the required keys as environment variables / Colab secrets:
   - `OPENAI_API_KEY` (+ optional `OPENAI_BASE_URL`, `LLM_MODEL`)
   - Amadeus API credentials (for live flight/hotel search)
   - Kaggle credentials (optional — public fallbacks are used otherwise)
3. Run the tool-definition cells, then the **14-test** evaluation cell, or chat
   with the agent interactively.

---

## Design Notes

- **Structured tool calling** with a strict system policy keeps the model from
  free-forming answers to factual/time-sensitive questions.
- **Fallback-first data layer:** every external dependency (Amadeus, Kaggle) has
  a public or cached fallback, so the demo runs even without credentials.
- **RAG where it helps:** FAQ answers and itineraries are grounded in a LanceDB
  index rather than hallucinated.

---

## Limitations & Next Steps

- [ ] Fill missing weather fields (humidity, precipitation, clothing tips).
- [ ] Add multi-tool scenarios (e.g. flight + hotel + itinerary in one request).
- [ ] Cache API responses to make the test suite fully offline-reproducible.

---

## License

MIT — see [LICENSE](LICENSE).

## Contact

**Mahdiar Harandi** — harandimahdiar@gmail.com
[GitHub](https://github.com/MahdiarHarandi) · [LinkedIn](https://www.linkedin.com/in/mahdiarharandi)
