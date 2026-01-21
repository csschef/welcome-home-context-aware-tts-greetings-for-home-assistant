# Welcome Home - Context-Aware TTS Greetings For Home Assistant

This repository contains a **context-aware “Welcome Home” automation setup for Home Assistant**, designed to play spoken greetings when people arrive home.

The greetings adapt dynamically based on:
- Who arrives home
- Whether they are accompanied by children
- The current weekday
- Whether guest mode is enabled

---

## ⚠️ Important Context & Design Assumptions

This is **intentionally opinionated** and based on the following real-world scenario:

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

## 🧠 How It Works

1. **Timers Automation** - [`automation-welcome-home-timers.yaml`](automation-welcome-home-timers.yaml)

   - Starts timers when adults are detected at school or preschool for longer than a set time and on certain dates and within certain hours
   - Starts arrival timers when adults enter the home zone

2. **Welcome Home Automation** - [`automation-welcome-home-tts.yaml`](automation-welcome-home-tts.yaml)
   - Triggered by a door sensor (or similar arrival signal)
   - Checks if silent_mode is on. (optional) 
   - Evaluates active timers to determine:
     - Who arrived
     - Whether children are likely with them
   - Calls the greeting script with calculated context

3. **Greeting Script** - [`script-welcome-home-messages.yaml`](script-welcome-home-messages.yaml)
   - Randomly selects a predefined spoken message based on:
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

> ⚠️ The *school* and *preschool* zones do **not** represent the children directly - they are used to infer that an adult is likely picking up a child.

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

> ⏱️ Timer duration is flexible, but **30-90 minutes** is usually a good starting point.

---

### 4️⃣ Media Player (Required)

A media player capable of TTS, for example:

- Google Cast
- Sonos
- Alexa Media Player
- Any Cloud TTS-compatible device

Referenced in the automations as:

```yaml
media_player.YOUR_MEDIA_PLAYER
```

--- 

### 5️⃣ Physical Arrival Trigger (Required)

The welcome message automation is intentionally **not triggered by person or zone changes**.

Instead, it relies on a **physical confirmation** that someone has actually entered the home — most commonly a **door sensor**.

This design choice:

- Prevents duplicate announcements  
- Avoids GPS drift false positives  
- Ensures the message plays **only when someone truly arrives**

#### Example: Door Sensor Trigger

```yaml
triggers:
  - trigger: state
    entity_id:
      - binary_sensor.YOUR_DOOR_SENSOR
    to: "on"
```

#### Alternative Arrival Signals

You may replace the door sensor with any reliable arrival signal, such as:

A hallway motion sensor

A smart lock state change

Any binary sensor that reliably indicates entry

>⚠️ Important
>This trigger is mandatory.
>Without it, the welcome automation will never execute, even if all timers are running correctly.
>

---

## 🛠️ Make It Your Own

This setup is meant to be a **starting point**, not a locked-down solution.

You are strongly encouraged to adapt it to your own household, devices, and preferences.

### What You Can Customize

You can freely adjust **all parts** of this solution, including but not limited to:

- **People**
  - Change the number of adults and children
  - Rename `person_a`, `person_b`, `child_a`, `child_b` to match your setup
  - Expand the logic to handle more people if needed

- **Zones**
  - Replace `school` and `preschool` with your own zone names
  - Add additional zones (work, gym, grandparents, etc.)
  - Adjust zone logic if children have their own device trackers

- **Device Trackers**
  - Use phones, smartwatches, cars, or custom trackers
  - Combine multiple trackers per person if desired
  - Replace zone-based detection with other presence logic

- **Arrival Triggers**
  - Swap the door sensor for:
    - Motion sensors
    - Smart locks
    - NFC tags
    - Any reliable binary sensor
  - Add additional safeguards or confirmation steps

- **Timers**
  - Adjust timer durations to better fit your daily routines
  - Remove timers you don’t need
  - Add new timers for other activities or locations

- **Messages**
  - Rewrite all welcome messages,
  - Change language, tone, humor, or formality
  - Add more variations per weekday
  - Replace TTS with notifications, media playback, or scripts

- **Media Output**
  - Change which media player is used
  - Add room-specific announcements
  - Play sounds, music, or other audio instead of speech

### Philosophy

This solution favors:

- **Deterministic behavior**
- **Low false positives**
- **Real-world confirmation of arrival**

It is intentionally modular so you can:

- Simplify it if your setup is smaller
- Expand it if your home is more complex

If something doesn’t match your household - **change it**.  
Nothing here is sacred 😉

---

Feel free to fork, modify, break apart, or rebuild this automation until it fits *your* home perfectly.
