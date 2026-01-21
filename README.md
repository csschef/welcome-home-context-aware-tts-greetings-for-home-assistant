# Welcome Home – Context-Aware TTS Greetings (Home Assistant)

This repository contains a **context-aware “Welcome Home” automation setup for Home Assistant**, designed to play spoken greetings when people arrive home.

The greetings adapt dynamically based on:
- Who arrives home
- Whether they are accompanied by children
- The current weekday
- Whether guest mode is enabled

---

## ⚠️ Important Context & Design Assumptions

This template is **intentionally opinionated** and based on the following real-world scenario:

- **2 adults**
- **2 children**
- Children are of **school and preschool age**
- Children **do not have their own device trackers**
- Arrival context is inferred indirectly using **timers and zones**

Because the children do not have trackers, the system **infers** whether they are with an adult based on:
- Adults being detected at *school* or *preschool*
- Adults later arriving home within a reasonable time window

This means the solution is **not 100% foolproof**, but in real use it works **~99% of the time** and avoids the complexity of trackers on children.

---

## 🧠 How It Works (High-Level)

1. **Timers Automation** - [`automation-welcome-home-timers.yaml`](automation-welcome-home-timers.yaml)

   - Starts timers when adults are detected at school or preschool
   - Starts arrival timers when adults enter the home zone

2. **Welcome Home Automation** - [`automation-welcome-home-tts.yaml`](automation-welcome-home-tts.yaml)
   - Triggered by a door sensor (or similar arrival signal)
   - Evaluates active timers to determine:
     - Who arrived
     - Whether children are likely with them
   - Calls the greeting script with calculated context

3. **Greeting Script** - [`script-welcome-home-messages.yaml`](script-welcome-home-messages.yaml)
   - Selects a spoken message based on:
     - Adults present
     - Children present
     - Weekday
     - Guest mode
   - Plays TTS via a media player

---

## ✅ Prerequisites

### 1️⃣ Zones (Required)

You **must** have the following zones configured in Home Assistant:

- `zone.home`
- `zone.school`
- `zone.preschool`

These zones are used to infer context and must match the zone names referenced in the automations.

> ⚠️ The *school* and *preschool* zones do **not** represent the children directly — they are used to infer that an adult is likely picking up a child.

---

### 2️⃣ Persons (Required)

You must have two `person` entities:

- `person.person_a`
- `person.person_b`

These should be linked to the appropriate device trackers for the two adults.

---

### 3️⃣ Timer Helpers (Required)

Create the following **Timer helpers** in Home Assistant.

#### Arrival timers

Used to detect who has just arrived home:

- `timer.arrival_timer_person_a`
- `timer.arrival_timer_person_b`

#### Child context timers

Used to infer whether children are with an adult:

- `timer.person_a_leaves_school`
- `timer.person_b_leaves_school`
- `timer.person_a_leaves_preschool`
- `timer.person_b_leaves_preschool`

> ⏱️ Timer duration is flexible, but **30–90 minutes** is usually a good starting point.

---

### 4️⃣ Media Player (Required)

A media player capable of TTS, for example:

- Google Cast
- Sonos
- Alexa Media Player
- Any Cloud TTS–compatible device

Referenced in the automations as:

```yaml
media_player.YOUR_MEDIA_PLAYER