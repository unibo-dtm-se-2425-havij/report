---
title: Future work
has_children: false
nav_order: 13
---

# Known issues and future work

This section summarizes limitations observed in the current implementation and realistic next steps.

## What is missing or limited

- **Barcode capture**: barcode input is manual; there is no camera scanning or OCR flow.
- **Product discovery**: users must know the barcode; there is no name-based search or browsing.
- **Nutrition scope**: only kcal, protein, carbs, and fat are tracked; micronutrients, fiber, sugar, and sodium are not included.
- **Editing**: entries can only be deleted and re-added; there is no edit or batch update flow.
- **Data portability**: there is no CSV export/import or backup workflow.

## What does not work as it should (or needs improvement)

- **Open Food Facts resilience**: lookups have no caching, retries, or rate-limit handling, so network/API issues stop the flow.
- **Nutrient parsing gaps**: the API adapter only reads kcal/macros; if kcal is missing but energy is given in kJ, calories can show as 0.
- **UI refresh**: after removing an entry, the log view does not auto-refresh (user must switch tabs or reload).
- **Persistence efficiency**: saving a day deletes and re-inserts all entries, which is fragile and inefficient for larger logs.

## Potential future developments

- Add barcode scanning (webcam/mobile) and a product name search fallback.
- Cache product lookups locally and reuse recent items to support offline or slow networks.
- Expand nutrition tracking (micronutrients, fiber, sugar, sodium) and convert kJ→kcal where needed.
- Introduce goals and insights (daily targets, macro balance, weekly/monthly trends).
- Add edit-in-place, batch operations, and data export/import for long-term tracking.
- Improve account features (username change, password reset) and optional encryption of the local database.

