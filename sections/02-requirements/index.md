---
title: Requirements
has_children: false
nav_order: 3
---

# Requirements

## User stories

- **As a busy eater**, I want to enter a barcode to prefill nutrition values so I can log meals faster.
- **As a nutrition‑minded user**, I want to log what I ate (with grams) and see daily totals so I can track my intake.
- **As a returning user**, I want to sign up and log in so my logs are saved across sessions.
- **As someone reviewing habits**, I want to see the last 7 days of totals so I can spot trends.

## Requirements analysis

### Functional requirements

**FR1. Account creation and login**  
The system lets users create an account and authenticate with a username and password.  
Acceptance criteria:
- A new username can be registered with a password and becomes the active session.
- Duplicate usernames are rejected with a clear error message.
- Valid credentials log the user in; invalid credentials do not.

**FR2. Barcode lookup**  
The system lets users look up a product by barcode and prefill product name and nutrients per 100g.  
Acceptance criteria:
- A numeric barcode returns a product name and 100g nutrients when available.
- Non‑numeric or empty barcodes are rejected with a clear error message.
- If the product is not found or the API fails, the user sees a friendly error.

**FR3. Manual meal entry**  
The system allows users to enter a product name, quantity in grams, and nutrients per 100g (optionally using a barcode lookup), then add it to a chosen day.  
Acceptance criteria:
- When a user adds an entry with grams > 0 and a non‑empty product name, the entry is stored for the selected day.
- Nutrients are scaled from “per 100g” values to the logged grams.
- The entry includes timestamp, product name, grams, nutrients, and barcode if available.

**FR4. Daily log and totals**  
The system shows the user’s entries for a selected day and the total nutrients for that day.  
Acceptance criteria:
- The day view lists all entries with time, product, barcode, grams, and nutrients.
- A totals section shows summed kcal, protein, carbs, and fat.
- If there are no entries, the user sees a “No entries yet for this day.” message.

**FR5. Remove entries**  
The system lets users delete a selected meal entry from a day.  
Acceptance criteria:
- Selecting an entry and confirming removal deletes it from the day log.
- If the entry no longer exists or is invalid, the user is warned.

**FR6. Weekly trends**  
The system shows totals for the last 7 days ending on a selected date.  
Acceptance criteria:
- A 7‑row table shows totals for each day in the range.
- A line chart visualizes kcal, protein, carbs, and fat over time.

**FR7. Profile view**  
The system shows the logged‑in user’s profile basics.  
Acceptance criteria:
- The profile displays username and account creation timestamp.
- If the profile cannot be found, the user sees an error.

### Non-functional requirements

**NFR1. Data persistence**  
Meal logs and user accounts must persist across app restarts.  
Acceptance criteria:
- After restarting the app, previously saved entries and accounts remain available.
- **Note:** this requirement applies to local or self-hosted deployments with persistent storage; the Streamlit Cloud demo uses ephemeral storage and does not retain user data between restarts.

**NFR2. Password safety**  
Passwords must not be stored in plain text.  
Acceptance criteria:
- Stored credentials use a salted hash and are verified with constant‑time comparison.

**NFR3. Usability for logged‑out users**  
The UI should clearly indicate when login is required.  
Acceptance criteria:
- “Log Meal”, “Today”, “Last 7 Days”, and “Profile” views show a prompt when not logged in.

**NFR4. Resilience to API errors**  
External API failures should not crash the UI.  
Acceptance criteria:
- API errors are shown as user‑friendly messages while the app remains usable.

### Implementation constraints

**IR1. Python + Streamlit**  
The app is implemented in Python 3.10+ with Streamlit for the UI.  
Justification: Python and Streamlit enable rapid iteration and a small local‑app footprint.

**IR2. Open Food Facts API v2**  
Product lookup uses the Open Food Facts public API.  
Justification: A free, open dataset avoids maintaining a local product catalog.

**IR3. SQLite persistence**  
Local data is stored in a SQLite database configurable via `DB_PATH`.  
Justification: SQLite offers zero‑setup persistence for a single‑user desktop app.

### Glossary

- **Barcode**: Numeric product identifier (EAN/UPC) used for lookups.
- **Nutrients per 100g**: Nutrition values normalized to 100 grams, used for scaling.
- **Meal entry**: One logged item with timestamp, grams, and scaled nutrients.
- **Day log**: All meal entries for a single calendar day.
- **Totals**: Sum of kcal, protein, carbs, and fat for a day or range.
