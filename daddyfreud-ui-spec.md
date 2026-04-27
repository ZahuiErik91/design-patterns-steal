# daddyfreud.com — UI & UX Specification

**Version:** 1.0  
**Date:** 2026-04-27  
**Author:** Zahui Varga Erik / Mr.Sun  
**Design Aesthetic:** Brutalist white, 80% whitespace, IBM Plex + SF Mono, thin dividers, no shadows/colors/gradients

---

## Table of Contents

1. Onboarding Flow
2. Logged-In Experience
3. Task Resolver (AI)
4. Timer System
5. System Popup
6. Data Model
7. Architecture & Integration Points

---

## 1. Onboarding Flow

### Screen State Machine

```
┌────────────────────────────────┐
│    [email input box]           │  ← Step 1: Enter email
│    [Get Access button]         │
│    ───────────────────         │
│    [Enter as Guest]            │  ← Step 3 alternative
│    Ⓣ (Telegram logo)          │  ← Step 3 alternative
└────────────────────────────────┘
         │ user types email + clicks "Get Access"
         ▼
┌────────────────────────────────┐
│    [code input box]            │  ← Step 2: Enter verification code
│    [Enter button]              │
│    ───────────────────         │
│    [Enter as Guest]            │
│    Ⓣ (Telegram logo)          │
└────────────────────────────────┘
```

### Flow Paths

**Path A: Email Login**
1. User enters email → clicks "Get Access"
2. Input box transforms to "Enter Code" (email sent with verification code)
3. User enters code → clicks "Enter"
4. → Logged in

**Path B: Guest Access**
1. User clicks "Enter as Guest" at any step
2. → Authenticated via IP + browser fingerprinting
3. Limited access: 10 tasks OR X chat messages
4. After limit: pay $10 to continue OR email login

**Path C: Telegram Login**
1. User clicks Telegram icon (Ⓣ)
2. → Triggers Telegram native login popup (existing @daddyfreud_bot auth)
3. → Logged in
4. **Note:** Telegram login currently doesn't persist memory on web → needs fix

### DB Requirements (New)
- `users` table: needs `email` column
- `verification_codes` table: email + code + expiry
- `guest_sessions` table: fingerprint + usage counter + paid flag

---

## 2. Logged-In Experience

### Layout

```
┌─────────────────────────────────────────┐
│  daddy wordmark          ⚙️ (system)   │  ← Top bar
├─────────────────────────────────────────┤
│                                         │
│                                         │
│            TASKS PAGE                   │  ← Main area (80% white)
│                                         │
│  ┌──────────────[input]─────────────┐   │
│  │  Write a task...     [Enter]     │   │  ← Task input
│  └──────────────────────────────────┘   │
│                                         │
│  ┌──────────────────────────────────┐   │
│  │  ☐ Task item 1          [⏵][?][✓]│  │  ← Task row
│  │  ☐ Task item 2          [⏵][?][✓]│  │
│  │    └─ ☐ Sub-task 2.1         [✓] │  │  ← Sub-tasks (broken down)
│  │    └─ ☐ Sub-task 2.2         [✓] │  │
│  │  ☐ Task item 3          [⏵][?][✓]│  │
│  └──────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

### Task Row Actions

| Button | Action |
|--------|--------|
| ⏵ (Start) | Begins timer for this task. Changes to ⏹ (Stop) |
| ? (Help) | Opens AI chat sidebar — AI knows task context |
| ✓ (Done) | Marks task complete. If subtasks exist, checks them all |

### Break Feature
- "Break" triggered via natural language or a UI mechanism
- User writes messy task → AI resolver processes it
- Breaks into max 3 actionable sub-steps
- Sub-items can be broken again (max 2 levels deep)
- Checking ALL sub-items = main item checked automatically
- Deep level example: "Clean kitchen" → "Wash dishes/Clr counter/Sweep" → "Wash dishes" → "Scrub/rinse/Dry"

### Help Feature
- Opens chat sidebar (desktop) or full-screen (mobile)
- AI receives full task context + user's history
- AI speaks to user's emotions (therapist mode, per soul.md)
- AI knows: we're on web, UI looks like this, user has these tasks

---

## 3. Task Resolver (AI Logic)

When user types a messy task, AI resolver decides:

```
User input: "I need to fix the leaking sink"
                      │
                      ▼
┌─────────────────────────────────────┐
│         RESOLVER                    │
│                                     │
│  Classifies:                        │
│  • Technical? → research / steps    │
│  • Software? → web search / code    │
│  • Recipe? → OpenAlex / articles    │
│  • Cleanliness? → instructions      │
│  • Emotional? → therapy approach    │
│  • General? → break down logically  │
│                                     │
│  Then:                              │
│  • Searches web if needed           │
│  • Reads relevant articles          │
│  • Generates actionable sub-steps   │
└─────────────────────────────────────┘
                      │
                      ▼
      Sub-steps displayed under main task
```

**Resolver sources (free/research):**
- Web search (for how-to, recipes, tutorials)
- OpenAlex (for academic/health questions)
- Soul.md (for emotional/therapy support)
- DaddyFreud AI brain (for general reasoning)

---

## 4. Timer System

- Each task has a ⏵ Start button
- Clicking starts a timer (visible count-up)
- While running: button changes to ⏹ Stop
- Stop timer → logs duration
- Timer persists across page refreshes (localStorage)
- Optional: Pomodoro-style (25min work, 5min break)

---

## 5. System Popup (Mini Menu)

Triggered by ⚙️ icon in top bar

```
┌─────────────────────────────┐
│         ⚙️ System           │
│                             │
│  Telegram: [Link/Unlink]    │
│                             │
│  Credits: $12.50            │
│  [Top Up →]                 │
│                             │
│  [Log Out]                  │
└─────────────────────────────┘
```

---

## 6. Data Model

```sql
-- Users
users (
  id UUID PK,
  telegram_id BIGINT UNIQUE,
  email VARCHAR UNIQUE,
  fingerprint VARCHAR UNIQUE,  -- guest user IP + browser hash
  created_at TIMESTAMP,
  last_login TIMESTAMP
)

-- Verification codes (email login)
verification_codes (
  id UUID PK,
  email VARCHAR,
  code VARCHAR(6),
  expires_at TIMESTAMP,
  used BOOLEAN DEFAULT FALSE
)

-- Guest limits
guest_sessions (
  id UUID PK,
  fingerprint VARCHAR,
  tasks_used INT DEFAULT 0,
  chat_messages_used INT DEFAULT 0,
  max_tasks INT DEFAULT 10,
  max_chat INT DEFAULT 30,
  paid BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP
)

-- Tasks
tasks (
  id UUID PK,
  user_id UUID FK,
  parent_task_id UUID NULL,  -- for sub-tasks
  title TEXT,
  status ENUM('pending','active','done'),
  depth INT DEFAULT 0,       -- 0=root, 1=sub, 2=sub-sub
  position INT,              -- ordering
  timer_elapsed INT DEFAULT 0,  -- seconds
  created_at TIMESTAMP,
  completed_at TIMESTAMP NULL
)

-- Task notes (AI context)
task_context (
  id UUID PK,
  task_id UUID FK,
  ai_classification VARCHAR,  -- 'technical','recipe','cleanliness','emotional','general'
  ai_breakdown TEXT,          -- AI's reasoning for the breakdown
  source_urls TEXT[],         -- research sources used
  created_at TIMESTAMP
)
```

---

## 7. Architecture & Integration Points

### Frontend
- **Framework:** Next.js (already in DaddyWeb repo)
- **UI:** Brutalist white, 80% whitespace, monospace fonts
- **Components to build:**
  - `OnboardingFlow` — email/code/guest/telegram state machine
  - `TaskList` — todo list with enter-to-save
  - `TaskRow` — single task with [⏵][?][✓]
  - `Timer` — count-up/down with start/stop
  - `ChatSidebar` — AI help panel (slide-in from right)
  - `SystemPopup` — mini profile/settings modal
  - `Wordmark` — daddy logo

### Backend
- **Auth:** Supabase + email verification + Telegram OAuth
- **Tasks CRUD:** Supabase Postgres
- **AI Resolver:** Edge Function (brain-judgment or new resolver function)
- **Research:** OpenAlex + web search (existing DaddyWeb infra)
- **Memory:** Needs fix for Telegram-web user consistency
- **Payments:** Stripe (existing) for $10 guest unlock

### Integration Points with Existing Stack
- **DaddyWeb (Cloudflare Worker):** /api/chat for help sidebar
- **daddyfreudbot (Supabase Edge Function):** brain-judgment for task resolution
- **Soul.md:** AI personality for emotional help mode
- **Supabase Storage:** session data, user preferences

### Design References to Steal
- **fluid-functionalism:** ThinkingIndicator + ThinkingSteps for AI reasoning display during task breakdown
- **sensory-ui:** Sound feedback on task completion, timer end, AI response
- **nowhere:** URL fragment encoding for sharable task lists (guest mode)
- **useProximityHover:** Task list hover preview for timer/help/check buttons

---

## Priority Order

1. ✅ Onboarding flow (email + guest + telegram)
2. ✅ Task list CRUD (add, check, delete)
3. ✅ Break feature (AI resolver → sub-tasks)
4. ✅ Help feature (AI chat sidebar)
5. ✅ Timer system
6. ✅ System popup (credits, telegram link, logout)
7. ⬜ Memory sync (Telegram-web AI consistency)
8. ⬜ Sound feedback (sensory-ui integration)
9. ⬜ Sharable URL encode (nowhere pattern for guest tasks)
