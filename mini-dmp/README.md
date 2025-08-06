# TECHNICAL INTERVIEW TASK: Event Ingestion + Identity Resolution for a Mini DMP

## Company Context  
At Blockchain Ads (BCA), we operate a programmatic ad platform where different JavaScript snippets (scripts) are embedded across various partner websites. These scripts include:

1. Analytics Script – tracks user pageviews, sessions, form submissions.  
2. Ad Script – tracks ad impressions and clicks from our real-time bidding network.

Each of these scripts independently sends user data to our backend. However:
- They have no shared user ID
- They each send whatever identifiers they have: `email`, `wallet_address`, `browser_id`, `ip_address`, etc.
- They may refer to the same user, but the backend must intelligently determine that.

Our goal is to unify these fragmented identifiers into a single internal user profile (`internal_user_id`), and store all associated events under that user.

## Your Task

Build a simplified backend service for our internal DMP (Data Management Platform) that:

1. Ingests events from multiple sources  
2. Resolves user identity across fragmented identifiers  
3. Stores events under the correct internal user profile  
4. Exposes endpoints for debugging and querying  
5. Uses clean modular structure

## System Overview

### Event Sources:
- Analytics Script
- Ad Script

### Identifiers:
- `email` (strong)
- `wallet_address` (strong)
- `browser_id` (weak)
- `ip_address` (weak)

We trust strong identifiers for resolving identities. Weak ones can help, but shouldn't merge two users on their own.

## What You Need to Build

### 1. POST /events

Ingest a batch of events from one of the scripts.

**Payload Example:**

```json
{
  "source": "analytics_script",
  "events": [
    {
      "identifiers": {
        "email": "a@b.com",
        "browser_id": "browser_001"
      },
      "event_type": "page_view",
      "timestamp": 1700000000
    },
    {
      "identifiers": {
        "email": "a@b.com"
      },
      "event_type": "form_submit",
      "timestamp": 1700000050
    }
  ]
}
```

**Another source:**

```json
{
  "source": "ad_script",
  "events": [
    {
      "identifiers": {
        "wallet_address": "0xABC123",
        "browser_id": "browser_001"
      },
      "event_type": "ad_click",
      "timestamp": 1700000060
    }
  ]
}
```

Logic:
- Each event contains identifiers.
- Your system should:
  - Match identifiers against existing users.
  - If one or more strong identifiers match an existing user → use that `internal_user_id`.
  - If no match, create a new internal user.
- Merge identifiers over time (build a profile).

### 2. GET /users/:internal_user_id

Returns:

```json
{
  "internal_user_id": "user_001",
  "identifiers": {
    "email": "a@b.com",
    "wallet_address": "0xABC123",
    "browser_ids": ["browser_001", "browser_456"]
  },
  "events": [
    {
      "event_type": "page_view",
      "source": "analytics_script",
      "timestamp": 1700000000
    },
    {
      "event_type": "ad_click",
      "source": "ad_script",
      "timestamp": 1700000060
    }
  ]
}
```

### 3. GET /debug/match?identifier=email&value=a@b.com

Useful for debugging — shows the resolved internal user and their identifiers.

### 4. (Optional) POST /merge

You may allow merging users manually if two `internal_user_id`s are confirmed to be the same.

## Tech Expectations

- Language: Python or JavaScript (Node.js + Express, or Next.js API routes).
- Database: 
  - Use any lightweight DB (SQLite, Mongo, Postgres) OR
  - Mock with in-memory structures + repository pattern.
- Project Structure:
  - Must be modular and organized — no dumping everything into one file.
- Matching logic:
  - Should be clear, configurable, and not hardcoded in the controller.

## Using AI / Vibe Coding

You are encouraged to use AI tools or co-pilots to scaffold, generate helpers, or boilerplate — but you must understand what the code is doing and ensure it's clean.

If we detect low-effort AI output, that's a red flag.  
If you vibe-code but then refactor, explain, and document, that's a big plus.

## Sample Data You Can Use

| Event Type | Source | Identifiers |
|------------|--------|-------------|
| page_view | analytics_script | email=a@b.com, browser_id=br1 |
| ad_click | ad_script | wallet_address=0xABC123, browser_id=br1 |
| page_view | analytics_script | email=a@b.com, ip=1.2.3.4 |
| ad_click | ad_script | wallet_address=0xXYZ999, browser_id=br2 |

## What We Expect

| Area | Expectation |
|------|-------------|
| Functionality | Should support all core endpoints correctly |
| Identity Logic | Handles matching/merging using strong identifiers only |
| Code Quality | Modular, clean, understandable |
| README | Clear setup, sample requests, short matching logic explanation |
| Understanding | If AI is used, you still know what's going on |
| Optional Add-ons | CLI tools, Swagger docs, tests, UI layer, etc. |

## Deliverables

- Share your GitHub repo (public or private invite).
- The repo should include:
  - Full working code
  - README.md with:
    - Setup instructions
    - Sample curl/Postman requests
    - Notes on how your matching logic works

## Time Estimate

This should take 4–8 focused hours, depending on experience and vibe-coding.
