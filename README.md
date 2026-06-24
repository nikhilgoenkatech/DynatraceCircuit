# 🕵️ Dynatrace Circuit — Setup Guide

A secret-agent themed, scenario-based game that teaches Dynatrace observability concepts through interactive missions. Players investigate incidents, analyze traces, and uncover the "culprit" — all powered by real Dynatrace data.

This guide walks you through everything in order: deploying the app, configuring it for your event, uploading scenarios, and sharing it with players.

---

## What You Need Before Starting

| Requirement | Details |
|---|---|
| Dynatrace Environment | SaaS, with admin access |
| IAM permissions | `app-engine:apps:install`, `app-engine:apps:run` |
| OAuth client scopes | `document:documents:read`, `document:documents:write`, `document:documents:admin` |
| Node.js | v24 or later |
| Distribution package | `adoptionday-v1.x.x.zip` (provided separately) |
| Scenario files | `*.json` files (provided separately) |

---

## Step 1 — Deploy the App

The distribution package contains a pre-built app. You do not need to compile anything.

**1.1** Unzip the distribution package:

```bash
   1. unzip dynatrace-circuit.zip
   2. cd dynatrace-circuit
   3. npm install
   4. npx dt-app deploy --skip-build --environment-url https://<tenant>.apps.dynatrace.com
```

The CLI will prompt you to authenticate via browser the first time. Once deployed, the app appears under **Apps** in your Dynatrace menu as **Dynatrace Circuit**.

---

## Step 2 — Open the Admin Panel

All configuration is done from inside the app. There is no settings icon — the Admin Panel is accessed directly via URL.

Navigate to:
```
https://YOUR_ENV_ID.apps.dynatrace.com/ui/apps/my.totalservicemeltdown/admin
```

> You must be logged in as a Dynatrace user with admin-level OAuth scopes (`document:documents:admin`) to access this page. Regular players cannot see it.

---

## Step 3 — Configure Event Mode

Event Mode controls user registration, score tracking, and what players see. Configure this **before** inviting players.

In the Admin Panel under **Event Mode Configuration**:

| Setting | What it does | Recommended for a live event |
|---|---|---|
| **Event Mode** | Enables user registration and full score tracking | ✅ Turn ON |
| **Enable Screensaver** | Shows a landing screensaver on load and after 90s of inactivity | ✅ Turn ON |
| **Ask Nickname Once** | OFF = players are prompted for a nickname after every scenario; ON = asked only once | ✅ Turn ON |
| **Enable Survey Link** | Shows a post-scenario survey link to players | Optional |
| **Hide Leaderboard Tab** | Hides the Leaderboard tab from the top navigation bar | ❌ Leave OFF — players should see the leaderboard |
| **Hide Analytics Tab** | Hides the Analytics tab from the top navigation bar | Leave OFF unless not needed |
| **Disable Debug Logs** | Suppresses debug output in the browser console | ✅ Turn ON for live demos |
| **Enable LinkedIn Sharing** | Lets players share their score on LinkedIn after completing a scenario | Optional |
| **Enable BizEvent Prefix** | Applies a custom prefix to event data — used to isolate data across tenants | ✅ Turn ON if sharing a Dynatrace environment across multiple events |

**About BizEvent Prefix:** When enabled, enter a short unique prefix (e.g. `myapp`). The event provider will become `myappTotalServiceMeltdown`. This prevents score and event data from mixing when multiple instances of the game run on the same Dynatrace environment.

Click **Save Configuration** when done.

---

## Step 4 — Configure User Auth Mode (Optional)

User Auth Mode ties player scores to their Dynatrace account email address, giving each player a persistent identity across sessions.

In the Admin Panel under **User Auth Mode Configuration**:

| Setting | What it does |
|---|---|
| **User Auth Mode** | ON = scores tied to Dynatrace account email; OFF = nickname-based identity only |
| **Allow Nickname Change** | Lets players update their display name after registering |
| **Show Nickname Warnings** | Displays a warning to the player when their nickname is changed |
| **Require Nickname** | Nickname is mandatory for leaderboard display |

> The three sub-settings are only active when **User Auth Mode** is turned ON.

Click **Save Configuration** when done.

---

## Step 5 — Upload Scenarios

Scenarios are the missions players work through. They are provided as `.json` files alongside this package.

> **File naming:** The filename does not matter — the app identifies each scenario by the `id` field inside the JSON content, not by the filename. You can name files anything you like (e.g. `scenario-1.json`, `operation-silent-outage.json`). What must be unique is the `id` field within each JSON file.

**5.1** In the Admin Panel, scroll to the **Upload New Scenario** section

**5.2** Click **Upload JSON Template** and select a scenario `.json` file

**5.3** Repeat for each scenario file — upload them one at a time

**5.4** Once all scenarios are uploaded, click **Make All Documents Public**

> ⚠️ **This step is critical.** By default, uploaded scenarios are private and only visible to admins. Players cannot see any scenarios until you click **Make All Documents Public**.

**5.5** Click **Verify Document Access** to confirm. You should see:
```
✅ X public, 0 private out of X total documents
```

If any scenarios still show as private, click **Make All Documents Public** again and re-verify.

---

## Step 6 — Generating Custom Scenarios with Claude

You can use Claude (claude.ai) to generate new scenario JSON files that follow the correct schema. This is the fastest way to create scenarios tailored to a specific customer's technology stack, industry, or Dynatrace use case.

### How to do it

**6.1** First, download the official scenario template from the Admin Panel:
Admin Panel → **Download JSON** under *Example Scenario Template*

**6.2** Open [claude.ai](https://claude.ai) and start a new conversation

**6.3** Upload the downloaded template JSON file to Claude, then paste the prompt below — customising the parts in `[ ]` for your audience:

---

**Copy-paste prompt:**

```
I am building a scenario for a Dynatrace observability game called Dynatrace Circuit.
The game is secret-agent themed — players act as agents investigating an incident.

Here is the official scenario JSON template that defines the exact schema I need to follow:
[paste or attach the downloaded template JSON here]

Please generate a complete, valid scenario JSON file using this schema with the following details:

- Customer industry: [e.g. financial services / retail / healthcare / telco]
- Dynatrace features to highlight: [e.g. distributed tracing, Davis AI problem detection, log analytics, dashboards]
- Difficulty level: [easy / medium / hard]
- Number of quiz questions: [e.g. 5]
- Secret agent mission title: [e.g. "Operation Silent Outage" / "The Phantom Latency" / make one up]
- Any specific scenario context: [e.g. a payment service is throwing 500 errors, a deployment caused a spike]

Make sure:
- All required fields from the template schema are present
- Quiz questions are multiple-choice with one correct answer
- Clues and hints reference real Dynatrace concepts (traces, spans, problem cards, Davis AI, etc.)
- The tone is dramatic and spy-thriller flavoured
- The output is a single valid JSON object with no extra explanation, ready to save as a .json file
```

---

**6.4** Review the generated JSON — check that all required fields are present and the questions make sense for your audience

**6.5** Save it as a `.json` file (e.g. `operation-silent-outage.json`)

**6.6** Upload it via the Admin Panel → **Upload JSON Template** as described in Step 5

### Tips for better scenarios

- Give Claude a real Dynatrace problem or trace as context — paste in a sample problem description or metric names to make the questions more realistic
- Ask Claude to generate multiple scenarios in one session and vary the difficulty across them
- If the JSON fails validation on upload, paste the error message back into Claude and ask it to fix the specific field
- For a consistent theme, tell Claude the codename series upfront (e.g. "all missions should be named Operation [adjective] [noun]")

---

## Step 7 — Verify Scenarios Are Ready

In the Admin Panel, scroll to the **Search & Filter** section and confirm all uploaded scenarios appear in the list.

You can filter by:
- **Status:** Published / Unpublished / Active / Inactive
- **Category:** All / Custom / Built-In

All scenarios should show as **Published** and **Active** before opening the game to players.

If something looks wrong:
- Scenario missing → click **Rebuild Index**
- Duplicates appearing → click **Clean Duplicates**
- Scenario stuck or broken → click **Force Delete Stuck Scenario**, then re-upload

---

## Step 8 — The Leaderboard and Podium

### Leaderboard

The Leaderboard is accessible to all players via the top navigation bar, and also directly at:

```
https://YOUR_ENV_ID.apps.dynatrace.com/ui/apps/my.totalservicemeltdown/leaderboard
```

**What players see:**
- A ranked list of all players sorted by total score
- Each player's nickname, score per scenario, and overall rank
- Live updates as scores come in — players can watch their rank change in real time

> Make sure **Hide Leaderboard Tab** is OFF in Event Mode Configuration, otherwise this tab won't appear for players.

> **Tip for events:** Keep the leaderboard open on a shared screen or projector throughout the session — it drives engagement and friendly competition between scenarios.

### Podium

The Podium view is designed for end-of-event display. It highlights the top 3 players and is accessible directly at:

```
https://YOUR_ENV_ID.apps.dynatrace.com/ui/apps/my.totalservicemeltdown/leaderboard?view=podium
```

- Shows a full-screen, celebration-style view of the top 3 ranked players
- Designed to be put on a projector or large monitor during the event wrap-up
- Can also be reached from within the Leaderboard tab using the podium toggle

> **Tip:** Open the podium URL on a separate browser tab and switch to it fullscreen when announcing the winners at the end of the session.

### Analytics

The Analytics tab is accessible at:

```
https://YOUR_ENV_ID.apps.dynatrace.com/ui/apps/my.totalservicemeltdown/analytics
```

- Shows aggregate game data — scenario completion rates, average scores, and time taken per scenario
- Useful for the host to monitor progress during the event and see which scenarios players found hardest

---

## Step 9 — Share with Players

Once the app is deployed and scenarios are live, share the app URL with your audience:

```
https://YOUR_ENV_ID.apps.dynatrace.com/ui/apps/my.totalservicemeltdown
```

Players need:
- A Dynatrace user account in your environment
- The `document:documents:read` OAuth scope assigned to their account

> **Tip:** Create a dedicated user group in Dynatrace with this scope pre-assigned so players can join without any per-person manual steps.

---

## Admin Panel — Quick Reference

Access the Admin Panel at: `https://YOUR_ENV_ID.apps.dynatrace.com/ui/apps/my.totalservicemeltdown/admin`

| Button | When to use |
|---|---|
| **Upload JSON Template** | Add a new scenario for the first time |
| **Force Upload (Overwrite)** | Replace an existing scenario with an updated version |
| **Force Delete Stuck Scenario** | Remove a scenario that failed to upload cleanly |
| **Rebuild Index** | Scenario uploaded but not appearing in the list |
| **Clean Duplicates** | Same scenario showing more than once |
| **Make All Documents Public** | After any upload — makes scenarios visible to all players |
| **Verify Document Access** | Confirm all scenarios are public and accessible |
| **Show All Document Details** | Inspect individual document metadata and privacy status |
| **Download JSON** | Download the scenario template to create your own scenarios |

---

## Game Data — BizEvents & DQL

Every player action in Dynatrace Circuit — scenario completions, quiz answers, scores, and session starts — is stored as a **BizEvent** in your Dynatrace environment's Grail data store. This means you can query, analyse, and dashboard all game activity using DQL directly in Dynatrace Notebooks or Dashboards.

### Event provider

All events are written under the provider:

```
TotalServiceMeltdown
```

If you enabled the **BizEvent Prefix** in Event Mode Configuration (e.g. prefix `myapp`), the provider becomes:

```
myappTotalServiceMeltdown
```

Use this provider value in all DQL queries below — substitute `TotalServiceMeltdown` with your prefixed version if applicable.

---

### Useful DQL queries

**All game events — raw view**
```
fetch bizevents
| filter event.provider == "TotalServiceMeltdown"
| sort timestamp desc
```

**All score submissions (one row per player per scenario)**
```
fetch bizevents
| filter event.provider == "TotalServiceMeltdown"
| filter event.type == "scenario.completed"
| fields timestamp, player.nickname, scenario.id, scenario.title, score, timeTaken
| sort timestamp desc
```

**Leaderboard — total score per player**
```
fetch bizevents
| filter event.provider == "TotalServiceMeltdown"
| filter event.type == "scenario.completed"
| summarize totalScore = sum(score), scenariosCompleted = count(), by: player.nickname
| sort totalScore desc
```

**Scenario popularity — how many players attempted each scenario**
```
fetch bizevents
| filter event.provider == "TotalServiceMeltdown"
| filter event.type == "scenario.completed"
| summarize attempts = count(), avgScore = avg(score), by: scenario.title
| sort attempts desc
```

**Average score per scenario — identify which scenarios were hardest**
```
fetch bizevents
| filter event.provider == "TotalServiceMeltdown"
| filter event.type == "scenario.completed"
| summarize avgScore = avg(score), minScore = min(score), maxScore = max(score), by: scenario.title
| sort avgScore asc
```

**Player activity over time — when were people most engaged**
```
fetch bizevents
| filter event.provider == "TotalServiceMeltdown"
| filter event.type == "scenario.completed"
| summarize completions = count(), by: bin(timestamp, 5m)
| sort timestamp asc
```

**Individual player journey — all events for one player**
```
fetch bizevents
| filter event.provider == "TotalServiceMeltdown"
| filter player.nickname == "YourPlayerNickname"
| fields timestamp, event.type, scenario.title, score
| sort timestamp asc
```

---

### Where to run these queries

Open **Dynatrace Notebooks** from the Apps menu and paste any query above into a new DQL section. You can pin results to a **Dashboard** to display live scores on a shared screen during your event.

> **Tip:** The leaderboard and analytics tabs in the app are built on top of these same BizEvents — so if you want a custom view or want to combine game data with real observability metrics, Notebooks is the place to do it.

---

## Troubleshooting

**Can't access the Admin Panel**
→ Navigate directly to the `/admin` URL above — there is no settings icon in the app UI

**Players can only see some scenarios, not all**
→ Admin Panel → **Make All Documents Public** → confirm with **Verify Document Access**

**A scenario was uploaded but doesn't appear**
→ Admin Panel → **Rebuild Index**

**Scores from different events are mixing together**
→ Enable **BizEvent Prefix** in Event Mode Configuration with a unique prefix per event

**Players are asked for a nickname after every scenario**
→ Turn ON **Ask Nickname Once** in Event Mode Configuration → **Save Configuration**

**Player scores are not persisting between sessions**
→ Confirm **Event Mode** is turned ON and that **Save Configuration** was clicked after enabling it

**Leaderboard is not visible to players**
→ Confirm **Hide Leaderboard Tab** is OFF in Event Mode Configuration → **Save Configuration**

**App not loading after deploy**
→ Confirm `environmentUrl` in `app.config.json` has no trailing slash and matches your environment exactly

**Deploy fails with HTTP 403**
→ Your Dynatrace user needs `app-engine:apps:install` and `app-engine:apps:run` IAM policies assigned
