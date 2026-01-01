---
title: Concept
has_children: false
nav_order: 2
---

# Concept

The project is a small, user-facing web application built with Streamlit. It runs locally and provides a GUI in the browser for looking up food products by barcode, logging meals, and tracking daily/weekly nutrition totals. It also consumes an external web API (Open Food Facts) to populate nutrition data, and persists user data in a local SQLite database.

## Use cases and users

**Primary user (meal logger):**
- Where: at home, at work, or in a grocery store/kitchen while preparing food.
- When: daily, often multiple times per day (breakfast/lunch/dinner/snacks).
- How: opens the Streamlit app in a browser on a laptop or tablet; types a barcode or manually enters product data.
- Needs: add meal entries, see daily totals, and review 7-day trends.

**Guest user (not signed in):**
- Can browse the UI but is prompted to log in or sign up before adding/viewing entries.

## Data and storage

The app stores user accounts (username + password hash), meal entries (timestamp, product name, grams, barcode), and nutrition totals derived from per-100g nutrients. Data is persisted in a local SQLite file (`DB_PATH`, default `data/app.sqlite`). Product information is fetched on demand from Open Food Facts and not stored unless the user logs the entry.

## Interaction model

Users interact through a simple, tab-based Streamlit interface:
- Log a meal (barcode lookup + manual confirmation/editing).
- View today’s log and totals, and remove entries.
- View a 7-day trend chart.
- View profile information.