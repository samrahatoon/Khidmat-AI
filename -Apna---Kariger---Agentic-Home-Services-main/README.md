# Apna Kariger - Agentic Home Services

Apna Kariger is an intelligent, conversational home services platform tailored for Pakistan. It disrupts traditional manual-form booking systems by allowing users to request services via natural language—in English, Roman Urdu, or native Urdu script.

## System Architecture & Orchestration

The system relies on an agentic workflow orchestrated by the **Antigravity** framework. The pipeline is defined in `agent.yaml` and executes sequentially, processing raw user input and driving it through to a completed booking.

The core orchestration passes context linearly through 4 specialized AI agents:

### 1. IntentAgent
- **Responsibility:** Parses free-form user queries. It detects the language and extracts the core parameters: the required service (Intent), the Location (sector/area), and the Time of service.
- **Data Source:** Matches against configured domain dictionaries in `data/intent_keywords.json`.
- **Output:** Structured parameters (e.g., Service: `plumber`, Location: `F-7`, Time: `afternoon`).

### 2. DiscoveryAgent
- **Responsibility:** Fetches potential service providers based on the exact intent and location extracted by the IntentAgent.
- **Data Source:** Queries the mock database at `data/providers_mock_db.json`.
- **Output:** An unranked subset of eligible providers within the given sector offering the requested service.

### 3. RankingAgent
- **Responsibility:** Evaluates the subset of providers from the DiscoveryAgent against dynamic constraints—primarily checking time availability against the user's requested slot.
- **Logic:** Filters out any providers unavailable at the requested time (e.g., matching "evening" to slot arrays). Ranks the remaining providers based on their rating and overall fit.
- **Output:** The single best-matched provider.

### 4. BookingAgent
- **Responsibility:** Finalizes the booking with the top-ranked provider.
- **Outputs:** 
  - Appends the final booking payload to the transaction ledger (`data/bookings.json`).
  - Generates a human-readable confirmation ticket (`data/booking_receipt.txt`).

## Observability

The entire reasoning chain, including language detection, intent matching, filtering logic, and booking states, is continuously logged to `data/agent_trace.log` to ensure the AI's decision-making is fully transparent.

## Data APIs & Storage

Since this is an offline demonstration of the agent architecture, the system relies on local JSON APIs to mock backend operations:
- **`data/providers_mock_db.json`**: Acts as the Provider API/Database. Contains schema data for providers, ratings, verification status, and availability slots.
- **`data/intent_keywords.json`**: Acts as the NLP Configuration API.
- **`data/bookings.json`**: Acts as the transactional database storing confirmed records.

## Assumptions & Limitations

1. **Sequential Linearity:** The pipeline assumes that a user provides all necessary details (service, location, time) in a single shot. There is currently no multi-turn conversational loop to ask clarifying questions if parameters are missing.
2. **Simplified Time Mapping:** Natural language terms like "aaj shaam" (today evening) or "kal subah" (tomorrow morning) are mapped directly to general buckets like "morning" or "evening", or matched explicitly against hardcoded time slots.
3. **Exact Location Matching:** Location matching works strictly against predefined sector codes (e.g., G-13, F-7, G-9).
4. **Availability Collisions:** The mock database does not check for real-time calendar conflicts. If a slot is listed in the provider's array, it is assumed completely available.
