# 🏠 AI Property Assistant

An intelligent n8n workflow that helps users search for properties, check weather, and engage in general conversation — all through a single chat interface powered by Groq LLM and Supabase.

---

## 📋 Overview

The AI Property Assistant is a multi-agent n8n workflow that:

- Understands natural language messages from users
- Detects intent (property search, weather query, or general chat)
- Maintains session state across a conversation
- Searches a Supabase property database with dynamic filters
- Fetches real-time weather data via WeatherAPI
- Responds in a friendly, human-readable format

---

## 🔧 Tech Stack

| Component        | Technology                          |
|-----------------|--------------------------------------|
| Workflow Engine  | [n8n](https://n8n.io)               |
| LLM              | Groq (`openai/gpt-oss-120b`)        |
| Database         | Supabase (`properties` table)       |
| Session Storage  | n8n DataTable                       |
| Weather API      | [WeatherAPI](https://weatherapi.com) |

---

## 🗂️ Workflow Architecture

The workflow is composed of **19 nodes** organized into the following stages:

```
Chat Trigger
    └── Load State
        └── AI Agent 1 (Intent Detector)
            └── Check Session Row
                ├── [New Session] Insert Row → Action Router
                └── [Existing]              → Action Router
                                                ├── search_db  → Property Agent → Supabase
                                                ├── get_weather → Weather Agent → WeatherAPI
                                                └── general_chat → Chat Agent
```

### Node-by-Node Breakdown

| # | Node | Description |
|---|------|-------------|
| 1 | **Chat Trigger** | Entry point — receives incoming chat messages |
| 2 | **Load State** | Creates or loads session state and cache by `sessionId` |
| 3 | **AI Agent 1** | Detects intent (`search_db`, `get_weather`, `general_chat`) and extracts property filters |
| 4 | **Structured Output Parser** | Enforces structured JSON output from AI Agent 1 |
| 5 | **Groq LLM 1** | Powers AI Agent 1 |
| 6 | **If Row Exists** | Checks if the session already has a row in the DataTable |
| 7 | **If (Session Gate)** | Routes to Switch if session exists, or Insert Row if new |
| 8 | **Insert Row** | Creates a new empty session row in the DataTable |
| 9 | **Switch (Action Router)** | Routes to Property, Weather, or Chat agent based on detected action |
| 10 | **HTTP Request** | Calls WeatherAPI with the extracted location |
| 11 | **AI Agent 4 (Weather)** | Formats the weather API response into a readable report |
| 12 | **Groq LLM 4** | Powers the Weather Agent |
| 13 | **Code (Filter State)** | Strips empty, null, and zero values from the updated state |
| 14 | **Update Row(s)** | Saves updated property filter values back to the DataTable |
| 15 | **AI Agent 5 (Property)** | Validates required fields, queries Supabase, and formats property listings |
| 16 | **Groq LLM (Property)** | Powers the Property Agent |
| 17 | **Supabase Tool** | Fetches up to 3 matching properties from the `properties` table |
| 18 | **AI Agent 2 (Chat)** | Handles greetings and guides users toward property search |
| 19 | **Groq LLM 2** | Powers the Chat Agent |

---

## 🧠 Intent Detection

AI Agent 1 classifies every user message into one of three actions:

| Action | Triggered When |
|--------|---------------|
| `search_db` | User mentions property types, locations, prices, bedrooms, or availability |
| `get_weather` | User asks about weather, temperature, or climate conditions |
| `general_chat` | User sends greetings, thanks, or any off-topic message |

> **Note:** A query like *"warm apartment"* is treated as `search_db`, not `get_weather`.

---

## 🏘️ Property Search

### Searchable Fields

| Field | Type | Example |
|-------|------|---------|
| `property_type` | string | Apartment, Villa, Studio, Condo, House |
| `city` | string | Lahore, Islamabad, Karachi |
| `state` | string | Punjab, Sindh |
| `min_price` | number | 20000 |
| `max_price` | number | 50000 |
| `bedrooms` | number | 2 |
| `availability` | string | true / false |

### Required Fields

Before querying Supabase, the Property Agent validates that all four of these fields are present:

- `property_type`
- `city`
- `min_price`
- `max_price`

If any are missing, the agent asks the user for the missing information before proceeding.

### Example Response

```
Great news! I found 2 Apartments in Islamabad:

Option 1
--------------------
Type       : Apartment
Price      : PKR 25,000 / month
City       : Islamabad
Bedrooms   : 2 Bedrooms
Available  : Yes
Amenities  : Gym | Pool
--------------------
```

---

## 🌤️ Weather Feature

The Weather Agent accepts any city or country name and returns a full report including:

- Temperature (°C / °F) and Feels Like
- Condition, Humidity, Wind, Visibility
- UV Index, Cloud Cover, Precipitation
- Air Quality Index (EPA scale)
- A friendly tip based on current conditions

---

## 💬 Session Management

Each user conversation is tracked by a unique `sessionId` stored in an n8n DataTable (`cache`). This allows the assistant to:

- Remember property filter preferences across messages
- Avoid re-asking for information already provided
- Accumulate filters incrementally across the conversation

---

## ⚙️ Setup & Configuration

### Prerequisites

- n8n instance (self-hosted or cloud)
- Groq API account
- Supabase project with a `properties` table
- WeatherAPI key

### Required Credentials

| Credential | Used By |
|-----------|---------|
| `Groq API` | All AI Agents (LLM) |
| `Supabase API` | Property search |
| WeatherAPI key (hardcoded in system prompt) | Weather Agent |

### Supabase Table Schema

The `properties` table should include at minimum:

```sql
CREATE TABLE properties (
  id              SERIAL PRIMARY KEY,
  property_type   TEXT,
  city            TEXT,
  state           TEXT,
  price           NUMERIC,
  bedrooms        INTEGER,
  availability    BOOLEAN,
  amenities       TEXT
);
```

### DataTable Schema

The n8n DataTable (`cache`) used for session storage requires:

| Column | Type |
|--------|------|
| `sessionid` | string |
| `property_type` | string |
| `city` | string |
| `state` | string |
| `min_price` | number |
| `max_price` | number |
| `bedrooms` | number |
| `availability` | string |

---

## 🚀 How to Import

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Select `AI_Property_Assistant.json`
4. Configure your credentials (Groq, Supabase)
5. Update the DataTable ID references if needed
6. Activate the workflow

---

## 📌 Notes

- The workflow is currently set to **inactive** — activate it after configuring credentials.
- The WeatherAPI key is embedded in the Weather Agent's system prompt — replace it with your own key before deploying to production.
- All AI agents use the `openai/gpt-oss-120b` model via the Groq API.
- The `search_db` path cleans filter state before saving, ensuring only meaningful values are persisted.
