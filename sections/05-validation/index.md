---
title: Validation
has_children: false
nav_order: 6
---

# Validation

## Testing approach

- The test suite focuses on small, isolated checks per layer (domain, application services, infrastructure adapters) to keep feedback fast and failures localized.
- Tests are organized in `tests/domain`, `tests/application`, and `tests/infrastructure`, mirroring the project architecture.
- `unittest` is used because it is in the Python standard library, integrates cleanly with Poetry tasks, and avoids extra runtime dependencies.
- Test doubles (`unittest.mock`) are used to isolate service logic from persistence and HTTP calls. No formal TDD workflow was enforced; tests were added alongside implementation to validate core behaviors.

## Testing (automated)


### Unit testing

- **Domain model unit tests**
    + `tests/domain/test_nutrients.py` validates nutrient addition and scaling of kcal/protein/carbs/fat values (supports FR3 and FR4 by ensuring per-100g scaling and total calculations are correct).
    + `tests/domain/test_meal.py` validates day log totals after adding multiple entries (supports FR4 totals).
- **Application service unit tests**
    + `tests/application/test_product_service.py` covers successful barcode lookup delegation and rejection of non-numeric barcodes (supports FR2).
    + `tests/application/test_meal_service.py` covers add-entry scaling and persistence, invalid grams rejection, remove-entry behavior for present/missing IDs, day log retrieval, daily totals, ordered last-days totals, and invalid day range handling (supports FR3, FR4, FR5, FR6).
    + `tests/application/test_user_service.py` covers signup/authentication happy path, empty credential validation, duplicate username rejection, wrong-password handling, profile retrieval, and user counting (supports FR1, FR7, and NFR2).
- **Test doubles**: `Mock` and `patch` are used in service tests to isolate repositories and UUID generation.
- **Status/coverage**: Latest local run (`poetry run poe test`) reports all discovered tests passing (0.004s). Coverage (`poetry run poe coverage-report`) reports 91% total coverage (223 statements, 20 missed). The CI workflow also publishes an HTML coverage report as an artifact.

### Integration testing

- **SQLite repository round-trip**
    + `tests/infrastructure/test_sqlite_repo.py` creates a temporary SQLite DB, initializes schema, saves a day log, reloads it, and checks entry fields and totals (supports NFR1 and IR3 by validating persistence and mapping).
- **Open Food Facts adapter parsing**
    + `tests/infrastructure/test_openfoodfacts_client.py` stubs the HTTP response, parses product name and nutrient values, and verifies scaling from 100g to smaller servings (supports FR2 and NFR4).
- **Status/coverage**: Integration tests run as part of the same `unittest` suite; coverage is included in the overall report.
- **Test doubles**: HTTP calls are stubbed with `patch` and a `FakeResp` response object to provide deterministic API payloads.

### System testing

- No automated end-to-end/system tests are implemented yet. Given the Streamlit UI, system tests would likely be browser-driven and aligned with the acceptance criteria in the requirements.
- Containers are not used for testing; the CI runs in GitHub Actions with Poetry-managed environments.

## Acceptance tests (manual)

- **Manual acceptance test plan (repeatable)**
    + **FR1**: Sign up with a new username, verify login succeeds; attempt duplicate username and confirm error; attempt invalid password and confirm login fails.
    + **FR2**: Enter a valid numeric barcode with known product; confirm name + per-100g nutrients prefill; test non-numeric barcode and confirm validation error; disconnect network or use an invalid barcode and confirm friendly error.
    + **FR3/FR4**: Add a manual entry with grams > 0; confirm the entry appears in the day log and totals are updated; confirm “No entries yet” when the day is empty.
    + **FR5**: Remove an existing entry and confirm it disappears; attempt removal of a missing entry and confirm warning.
    + **FR6**: Select a date range and confirm 7-day totals table and trend chart render.
    + **FR7**: Open the Profile view and confirm username + creation timestamp display.
    + **NFR1**: Restart the app and confirm previous entries remain.
    + **NFR3**: Log out and confirm protected views prompt for login.
- **Status**: Manual acceptance tests were executed once; all steps passed (100% success rate).
