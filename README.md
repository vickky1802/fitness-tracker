# WhatsApp-Based Fitness Tracker (MVP)  
**Marathon + Strength + Mobility Tracking via WhatsApp & Google Sheets**

---

## 1. Goal of the System

Build a **personal WhatsApp-based fitness assistant** that allows you to:

- Track **runs, gym workouts, circuits, football**
- Query **historical performance** (weights, pace, volume)
- See **weekly progress & pending workouts**
- Log data **hands-free via chat**
- Store everything in **Google Sheets** (your existing workbook)

This will later evolve into a **RAG-powered personal coach**, but the MVP focuses on:
- Reliability
- Structured logging
- Simple conversational flows (MCQ-based)

---

## 2. High-Level Architecture (MVP)

WhatsApp
│
│ (Webhook via Twilio / Meta)
▼
FastAPI Backend
│
├── Intent Detection (Rule-based)
├── Conversation State Manager
├── Workout Logger
├── Query Engine
├── Google Sheets Adapter
│
▼
Google Sheets (Source of Truth)


---

## 3. Recommended Tech Stack

### Core
- **Backend**: Python + FastAPI
- **Hosting**: Railway / Fly.io / AWS Lightsail
- **WhatsApp API**:  
  - ✅ Twilio WhatsApp Sandbox (fastest)
  - or Meta WhatsApp Cloud API (later)

### Data Layer
- **Google Sheets API** (OAuth or Service Account)
- Your existing **Marathon_Training_Tracker** workbook

### Logic & State
- Redis (optional, for conversation state)
- In-memory dict (OK for MVP, single user)

### Future (not MVP)
- Vector DB (Chroma / FAISS)
- LLM (OpenAI / local)
- Auth + multi-user

---

## 4. Core Project Structure (Skeleton)

whatsapp-fitness-bot/
│
├── app/
│ ├── main.py # FastAPI entry
│ ├── config.py # Env vars, sheet IDs
│ │
│ ├── whatsapp/
│ │ ├── webhook.py # Incoming WhatsApp messages
│ │ ├── sender.py # Send replies
│ │
│ ├── intents/
│ │ ├── classifier.py # Rule-based intent detection
│ │ ├── schemas.py # Intent definitions
│ │
│ ├── conversations/
│ │ ├── state.py # Conversation memory
│ │ ├── flows/
│ │ │ ├── track_workout.py
│ │ │ ├── query_workout.py
│ │ │ ├── weekly_status.py
│ │
│ ├── sheets/
│ │ ├── client.py # Google Sheets auth
│ │ ├── readers.py # Fetch data
│ │ ├── writers.py # Append/update data
│ │
│ ├── domain/
│ │ ├── workout.py # Workout models
│ │ ├── run.py # Run models
│ │
│ └── utils/
│ ├── date.py
│ ├── pace.py
│
├── tests/
│ ├── test_intents.py
│ ├── test_sheets.py
│
├── requirements.txt
├── .env
└── README.md


---

## 5. Google Sheets as the Source of Truth

Your workbook already contains:
- Phase-wise plan
- Weekly schedule
- Strength tracking
- Running metrics
- Flags & pace calculators

### MVP Sheet Access Patterns
- **Read**:
  - Weekly workout schedule
  - Previous workout logs
- **Write**:
  - New workout entries
  - Run stats
  - Completion flags

No duplication. Sheets = database.

---

## 6. Intent Design (MVP)

### Intent 1: Weekly Status
**User**
What workouts do I have left this week?


**System**
- Detect intent: `WEEKLY_STATUS`
- Read weekly plan sheet
- Compare with logged workouts
- Respond with checklist

---

### Intent 2: Historical Query
**User**
What weight did I do on bench press last week?

**System**
- Intent: `QUERY_EXERCISE_HISTORY`
- Parse:
  - Exercise: Bench Press
  - Time range: Last week
- Query workout logs sheet
- Respond with last values

---

### Intent 3: Track Workout (Guided Flow)
**User**
Track workout

**System (MCQ-based)**
What did you do today?
1️⃣ Gym
2️⃣ Run
3️⃣ Circuit
4️⃣ Football

#### Example: Gym Flow
Which workout?
1️⃣ Upper
2️⃣ Lower
3️⃣ Push
4️⃣ Pull

Exercise: Squat
Enter weight (kg):
Enter reps:
Enter sets:

Add another exercise?
1️⃣ Yes
2️⃣ No


On completion → write to Google Sheets

---

## 7. Conversation State Management

Because WhatsApp is stateless:

```python
state = {
  "user_id": "varun",
  "current_intent": "TRACK_WORKOUT",
  "step": "ENTER_WEIGHT",
  "payload": {
     "exercise": "Squat",
     "sets": []
  }
}
```

8. MVP Build Plan (Step-by-Step)
Phase 1 – Foundations

 Setup FastAPI

 Configure Twilio WhatsApp webhook

 Echo message back (sanity test)

Phase 2 – Google Sheets Integration

 Authenticate via service account

 Read weekly plan

 Append workout logs

Phase 3 – Intent Engine (Rule-Based)

 Keyword matching

 Date parsing (last week, today)

 Exercise name normalization

Phase 4 – Core Flows

 Weekly status flow

 Track workout (gym + run)

 Query last workout stats

Phase 5 – Validation & Safety

 Confirm before write

 Handle partial inputs

 Error recovery

9. MVP Success Criteria

✅ You can log workouts fully via WhatsApp
✅ You can ask past performance questions
✅ Sheets update correctly
✅ No manual data entry needed

10. What Comes After MVP (Not Now)

LLM-powered summaries

RAG over fitness history

Auto insights & warnings

Multi-user support

Calendar-based reminders

11. Next Step (Recommended)

Next action:

Implement Phase 1 + Phase 2
Start with:

WhatsApp webhook

Google Sheets read/write test

When you’re ready, I’ll:

Write the FastAPI webhook code

Give you the exact Google Sheets schema mapping

Provide Twilio setup commands
