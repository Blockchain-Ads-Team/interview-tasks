# TECHNICAL INTERVIEW TASK: Ad Budget Manager with Daily Budget and Rollover

## Company Context

At Blockchain Ads (BCA), advertisers run paid campaigns that spend real ad dollars. Each ad campaign has an associated cost, and advertisers must **top up their account balance** in order to run campaigns.

The system enforces:
- A **$5,000/day default daily ad budget** for each advertiser
- **Unused daily budget rolls over** and adds to the next day’s available balance
- Advertisers can manually **top up their balance**, minimum $10,000 per top-up
- If there isn’t enough available budget for a campaign, the campaign is **deferred** until there is

This task is about building a simplified backend system to manage ad spend, daily budgets, rollover, and top-ups.

## Your Task

Build a simplified backend service that:

1. Accepts campaign submissions with a cost
2. Tracks each advertiser’s available balance
3. Schedules or defers campaigns based on available balance
4. Supports manual top-ups
5. Simulates the passing of days (e.g. nightly system tick)
6. Supports other non-campaign spend (optional)

## What You Need to Build

### 1. POST /campaigns

Submits a campaign to be scheduled.

**Request:**

```json
{
  "advertiser_id": "adv_001",
  "campaign_name": "My Retargeting Campaign",
  "cost": 4000
}
```

**Response (scheduled):**

```json
{
  "status": "scheduled",
  "scheduled_for": "2024-08-05",
  "balance_remaining": 1000
}
```

**Response (deferred):**

```json
{
  "status": "deferred",
  "scheduled_for": "2024-08-06",
  "reason": "insufficient_balance"
}
```

### 2. POST /topup

Top up the advertiser's account.

**Request:**

```json
{
  "advertiser_id": "adv_001",
  "amount": 10000
}
```

**Rules:**
- Minimum top-up: $10,000
- Top-up is added to the advertiser's **rollover balance**

### 3. POST /simulate-day

Simulates passage of one day.

Logic to apply:
- Add $5,000 to daily budget
- Add unused budget to rollover
- Attempt to reschedule previously deferred campaigns

### 4. GET /budgets/:advertiser_id

Returns advertiser’s full budget state.

**Response:**

```json
{
  "advertiser_id": "adv_001",
  "current_day": "2024-08-05",
  "daily_budget": 5000,
  "rollover_balance": 10000,
  "total_available": 15000,
  "used_today": 4000,
  "remaining_today": 11000
}
```

### 5. GET /campaigns

Returns all campaign submissions and their statuses (scheduled, deferred, completed).

### 6. POST /spend (optional)

Simulates spending for non-campaign activities (e.g., analytics, reports).

**Request:**

```json
{
  "advertiser_id": "adv_001",
  "amount": 3000,
  "reason": "data export"
}
```

**Response:**

```json
{
  "status": "success",
  "remaining_today": 2000
}
```

Or:

```json
{
  "status": "rejected",
  "reason": "insufficient_balance"
}
```

## Sample Scenarios

### Scenario 1 – Single Day Spend
Day 1:  
- $5,000 daily budget  
- Campaign A ($3,000) → scheduled  
- Campaign B ($4,000) → deferred  
- Remaining balance: $2,000

### Scenario 2 – Top-Up & Rollover
Advertiser tops up $10,000.  
Next day:  
- Daily budget: $5,000 + Rollover: $2,000 + Top-Up: $10,000  
- Campaign B ($4,000) now fits → scheduled

### Scenario 3 – Overspend via /spend
Advertiser has $6,000.  
Spends $5,000 on a report → allowed  
Attempts to launch $2,000 campaign → deferred

## Expectations

This task is about backend logic and architecture. UI and polish are not necessary.

You should:
- Use Python or Node.js (JavaScript/TypeScript)
- Use a real database or mock with repository abstraction
- Separate business logic from routing
- Cleanly simulate budget rollovers, campaign queueing, and top-ups

## Using AI / Vibe Coding

You can use AI tools to help scaffold, write boilerplate, and assist you.

However:
- You must understand and verify what’s written
- If you use AI, your code should be **cleaner and better**, not sloppier

## What We’re Evaluating

| Area            | Expectation                                                      |
|------------------|-------------------------------------------------------------------|
| Architecture     | Logical separation of concerns (campaigns, budgets, time)        |
| Code Quality     | Readable, modular, testable                                      |
| Budget Logic     | Rollover, top-up enforcement, correct deferring behavior         |
| Scheduling       | Campaigns are scheduled or deferred correctly                    |
| Clarity          | Clean README, sample requests, setup instructions                |

## Deliverables

- GitHub repo (public or private invite)
- README.md with:
  - Setup instructions
  - Sample curl/Postman commands
  - Notes on how budget rollover and top-ups work

## Time Estimate

4–6 hours, especially with effective use of AI tools.
