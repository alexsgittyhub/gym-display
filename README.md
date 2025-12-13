# 🏋️ GymOS v4.0

### [🔴 CLICK TO VIEW LIVE WORKOUT](https://alexsgittyhub.github.io/gym-display/)

**GymOS** is a "headless" workout display system powered by a custom OpenAI GPT. It serves as a dashboard for a home gym, managing programming, plate math, and—critically—**persistent performance tracking**.

> **The Ecosystem:**
>
> 1.  **The Coach (GPT):** Plans workouts based on recovery, time, and historical data.
> 2.  **The Memory (JSON):** A dedicated file stores PRs and benchmark scores indefinitely.
> 3.  **The Display (Web App):** A "dumb terminal" that renders the plan and handles the math.

-----

## ⚡ Key Features

### 🧠 Persistent Memory (New)

  * **Long-Term Storage:** Unlike standard ChatGPT chats (which eventually forget context), GymOS reads and writes to `performance_log.json`.
  * **Auto-Progression:** The AI reads your current 1RM and benchmark scores *before* generating a workout to ensure progressive overload (Micro-loading +2.5/5 lbs).
  * **Benchmark Mode:** Native support for Named Workouts (e.g., "Fran", "Cindy"). It differentiates between "For Time" (store result) and "Training" (store volume).

### 🖥️ The Display (iPad/TV)

  * **Cyberpunk UI:** High-contrast dark mode with Acid Green accents, designed for visibility in a dim garage.
  * **Custom Plate Math:**
      * Inventory-aware algorithm (Specific to my setup: 45, 25, 15, 10, 5, 2.5 lbs).
      * Clicking a load (e.g., "255 lbs") renders the exact plate configuration visually.
  * **Smart Inputs:**
      * **Lifts:** Generates individual input boxes for every prescribed set.
      * **AMRAPs:** Automatically condenses input grids for "As Many Rounds As Possible" workflows.
      * **Runs:** Tracks Time & Incline %.
  * **💡 Wake Lock:** Forces the device screen to stay awake during the session.

### 🔄 The Feedback Loop

1.  **Execute:** Do the workout.
2.  **Log:** Click "Log Result" on the display to copy a summarized text block.
3.  **Update:** Paste the log back to the GPT. It runs a Python script to update `performance_log.json` with new PRs, then commits the changes to GitHub.

-----

## ⚙️ Architecture

This system relies on a circular data flow:

1.  **Read Phase:** GPT calls `getPerformanceLogMetadata` to see current stats (e.g., "Deadlift 1RM: 405").
2.  **Generation:** GPT creates a plan using strict JSON constraints and pushes it to `workout.json` via the GitHub API.
3.  **Rendering:** `index.html` fetches the raw JSON and renders the DOM.
4.  **Write Phase:** User pastes results -\> GPT parses data -\> GPT updates `performance_log.json`.

-----

## 🛠️ JSON Schema & Strict Logic

The application enforces a strict schema to prevent display crashes. The GPT is instructed to act as a "Compiler" to validate these rules before publishing.

### Critical Rules

  * **AMRAPs:** `sets` must be `1` (Integer), never `null`.
  * **Merged Notes:** Math and Form cues are combined into a single string to avoid key collisions.
  * **String Hygiene:** Base64 encoding strips all newlines to prevent API 422 errors.

### The Schema

```json
{
  "date": "YYYY-MM-DD",
  "title": "string",
  "notes": "string",
  "warmup": {
    "description": "string",
    "items": ["string"]
  },
  "blocks": [
    {
      "id": "A1",
      "name": "string",
      "rest_seconds": 120,
      "instructions": "string",
      "apple_watch": "string (optional)",
      "exercises": [
        {
          "name": "string",
          "type": "lift | run | bodyweight | mobility",
          "sets": 1, 
          "reps": "string",
          "load": "string",
          "last_best": "string (fetched from memory)",
          "notes": "Load: 45, 25. Keep back straight."
        }
      ]
    }
  ]
}
```

## 📦 Inventory Logic

The Plate Math visualizer is hardcoded to specific equipment availability:

  * **Plates:** 45, 25, 15, 10, 5, 2.5 lbs.
  * **Logic:** Prioritizes heaviest plates first.
  * **Bar:** Standard 45 lbs.

-----

### Setup

1.  **GymOS GPT:** Configure with the custom instruction set and Action Schema.
2.  **GitHub Repo:** Requires `workout.json` (Display) and `performance_log.json` (Memory).
3.  **Authentication:** Uses a GitHub PAT (Personal Access Token) via the OpenAI Action interface.
