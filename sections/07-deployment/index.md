---
title: Deployment
has_children: false
nav_order: 8
---

# Deployment

Havij Nutrition is a local Streamlit application that runs on the user’s machine with minimal setup. It persists data in a local SQLite file and requires no dedicated server infrastructure, aside from outbound access to Open Food Facts for barcode lookups. A hosted demo is available at [https://havij-nutrition.streamlit.app/](https://havij-nutrition.streamlit.app/), but the Streamlit Cloud runtime uses ephemeral storage so user data does not persist across restarts.

## User installation

- **Yes.** End users need a local Python runtime and the project dependencies.
    * **Prerequisites**: Python 3.10+ and Poetry (recommended for reproducible installs).
    * **Install dependencies (from source)**:
        + `poetry install`
    * **Alternative install (PyPI)**:
        + `pip install havij_nutrition` (useful for inspection or reuse, but the Streamlit UI is still started from the repo entrypoint below).
    * **Run the app**:
        + `poetry run streamlit run havij/presentation/streamlit_app.py`
    * **Configuration**:
        + `DB_PATH` (optional) sets the SQLite file location. Default is `data/app.sqlite`. The app creates the parent directory automatically, so only write permissions are needed.
        + The app makes outbound HTTPS requests to Open Food Facts (`https://world.openfoodfacts.net`); offline use limits barcode lookups but manual entry still works.

## Server-side installation

- **Not required.** The application is designed to run locally as a Streamlit app on the user's machine, with SQLite for persistence.
- **Optional hosting**: If you want a shared/remote instance, install the same Python/Poetry dependencies on the server and run Streamlit with the same command as above. Ensure the server can reach Open Food Facts and that `DB_PATH` points to persistent storage. The Streamlit Cloud demo is intentionally configured as a lightweight showcase and does not retain user data between restarts.

- **No additional server software required.** Storage uses a local SQLite file and there are no message brokers or separate database services.
