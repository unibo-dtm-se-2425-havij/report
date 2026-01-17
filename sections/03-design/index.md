---
title: Design
has_children: false
nav_order: 4
---

# Design

This chapter describes how the design choices map the requirements into a small, maintainable Streamlit app. The goal is a clear separation between UI, application logic, and infrastructure so the code stays easy to test and extend.

## Architecture

- Architectural style: layered architecture (presentation, application, domain, infrastructure). This fits a small Streamlit app with clear separation of UI, business rules, and external adapters, and keeps domain logic independent from framework and IO.
- Why not others: event-based or shared-dataspace patterns add complexity without benefit for a local, synchronous app; hexagonal/ports-adapters is partially adopted via Protocol ports but full inversion is unnecessary.
- Structure: layered within a single process.
  - Presentation: `havij/presentation/streamlit_app.py` (Streamlit UI, user interaction).
  - Application services: `havij/application/services/*` (use cases, validation and orchestration).
  - Domain: `havij/domain/model/*` and `havij/domain/rules.py` (core entities, value objects, invariants).
  - Infrastructure: `havij/infrastructure/*` (SQLite persistence, Open Food Facts API adapter, config).
- Responsibilities:
  - Streamlit UI: Collects input, renders tables/metrics, manages session state, and calls services.
  - Application services: Orchestrate use cases and validation (add/remove meal entries, lookup product, signup/login).
  - Domain model: Encapsulates business data and calculations (nutrient totals, scaling, validation rules).
  - Infrastructure: Persist and load data from SQLite, call the Open Food Facts API, map external data into domain objects.

```plantuml
@startuml
skinparam componentStyle rectangle
skinparam packageBackgroundColor #f5f5f5
skinparam packageBorderColor #bdbdbd

skinparam component {
  BackgroundColor #e3f2fd
  BorderColor #90caf9
}

package "Presentation Layer" #e8f5e9 {
  component "Streamlit UI\nhavij/presentation/streamlit_app.py" as UI
}

package "Application Layer" #fff8e1 {
  component "Application Services\nhavij/application/services/*" as App
}

package "Domain Layer" #f3e5f5 {
  component "Domain Models & Rules\nhavij/domain/model/*\nhavij/domain/rules.py" as Domain
}

package "Infrastructure Layer" #ffebee {
  component "Repositories\nSqliteDayLogRepository\nSqliteUserRepository" as Repo
  component "Open Food Facts Adapter\nOpenFoodFactsCatalog" as API
  database "SQLite DB\ndata/app.sqlite" as DB
  cloud "Open Food Facts API" as OFF
}

UI --> App
App --> Domain
App --> Repo
App --> API
Repo --> DB
API --> OFF
Repo --> Domain
@enduml
```

## Infrastructure (mostly applies to distributed systems)

- This system is not distributed; it runs as a single Streamlit process on one machine.
- Components (local):
  - UI + application server: Streamlit app in a single Python process.
  - Database: SQLite file on the same machine (`data/app.sqlite` by default).
- External dependency:
  - Open Food Facts API accessed over HTTPS (outbound only).
- Discovery/naming:
  - SQLite location via `DB_PATH` environment variable (`havij/infrastructure/config.py`).
  - Open Food Facts base URL is configured in `OpenFoodFactsClient`.
  - No service discovery, load balancing, or inter-service networking needed.


## Modelling

### Domain driven design (DDD) modelling

- Bounded contexts:
  - Meal logging: capture meal entries, compute daily totals.
  - Product lookup: fetch product data by barcode.
  - User accounts: signup and authentication.
- Domain concepts:
  - Entities: `MealEntry` (entry_id), `DayLog` (day aggregate), `UserProfile` (user_id), `Product` (barcode).
  - Value objects: `Nutrients` (kcal/protein/carbs/fat).
  - Aggregate: `DayLog` is the aggregate root containing `MealEntry` and enforces basic invariants (e.g., grams > 0).
- Repositories/services:
  - Repositories (ports): `DayLogRepository`, `UserRepository`.
  - Application services: `MealService`, `ProductService`, `UserService` orchestrate use cases.
  - Domain services: none; domain rules are simple validation helpers in `havij/domain/rules.py`.
  - External catalog: `ProductCatalog` port implemented by `OpenFoodFactsCatalog`.
- Domain events:
  - None explicitly modeled; operations are synchronous with direct persistence.

```plantuml
@startuml
left to right direction
skinparam componentStyle rectangle
skinparam packageBorderColor #bdbdbd
skinparam packageBackgroundColor #f5f5f5

package "Meal Logging\n(Bounded Context)" as MealCtx #e8f5e9 {
  component "DayLog\nAggregate Root" as DayLog
  component "MealEntry\nEntity" as MealEntry
  component "Nutrients\nValue Object" as Nutrients
}

package "Product Lookup\n(Bounded Context)" as ProductCtx #fff8e1 {
  component "Product\nEntity" as Product
  component "Nutrients\nValue Object" as Nutrients2
  component "ProductCatalog\nPort" as Catalog
  component "OpenFoodFactsCatalog\nAdapter" as OFFAdapter
}

package "User Accounts\n(Bounded Context)" as UserCtx #f3e5f5 {
  component "UserProfile\nEntity" as UserProfile
}

DayLog --> MealEntry : contains
MealEntry --> Nutrients : has
Product --> Nutrients2 : per 100g
OFFAdapter ..|> Catalog

MealCtx ..> ProductCtx : lookup product data
MealCtx ..> UserCtx : associate entries to user
@enduml
```

### Object-oriented modelling

- Main data types:
  - `MealEntry`: `entry_id`, `timestamp`, `barcode`, `product_name`, `grams`, `nutrients`.
  - `DayLog`: `day`, `entries`, methods `add_entry`, `remove_entry`, `total_nutrients`.
  - `Nutrients`: `kcal`, `protein_g`, `carbs_g`, `fat_g`, methods `scale`, `zero`.
  - `Product`: `barcode`, `name`, `nutrients_per_100g`, method `nutrients_for_grams`.
  - `UserProfile`: `user_id`, `username`, `created_at`.
  - Services: `MealService`, `ProductService`, `UserService` orchestrate use cases.
- Relationships:
  - `DayLog` contains many `MealEntry` objects.
  - `MealEntry` embeds `Nutrients` for the consumed amount.
  - `Product` embeds `Nutrients` for per-100g values.
  - Services depend on repository/catalog interfaces (ports).
  - `MealService` creates `MealEntry` instances and persists them by loading/updating a `DayLog` via `DayLogRepository`.

```plantuml
@startuml
skinparam classAttributeIconSize 0
skinparam classBackgroundColor #f7f7f7
skinparam classBorderColor #bdbdbd
skinparam interfaceBackgroundColor #fef7e0
skinparam interfaceBorderColor #f6c453

class MealEntry {
  +entry_id: str
  +timestamp: datetime
  +barcode: str
  +product_name: str
  +grams: float
  +nutrients: Nutrients
}

class DayLog {
  +day: date
  +entries: List[MealEntry]
  +add_entry(entry: MealEntry): void
  +remove_entry(entry_id: str): bool
  +total_nutrients(): Nutrients
}

class Nutrients {
  +kcal: float
  +protein_g: float
  +carbs_g: float
  +fat_g: float
  +scale(factor: float): Nutrients
  {static} +zero(): Nutrients
}

class Product {
  +barcode: str
  +name: str
  +nutrients_per_100g: Nutrients
  +nutrients_for_grams(grams: float): Nutrients
}

class UserProfile {
  +user_id: str
  +username: str
  +created_at: datetime
}

class MealService {
  +add_entry(user_id: str, day: date, product_name: str, grams: float, nutrients_per_100g: Nutrients, when: Optional[datetime], barcode: str): MealEntry
  +remove_entry(user_id: str, day: date, entry_id: str): bool
  +get_day_log(user_id: str, day: date): DayLog
  +get_day_totals(user_id: str, day: date): Nutrients
  +get_last_days_totals(user_id: str, end_day: date, days: int = 7): List[Tuple[date, Nutrients]]
  +assign_unowned_entries(user_id: str): int
}

class ProductService {
  +lookup_product(barcode: str): Product
}

class UserService {
  +signup(username: str, password: str): UserProfile
  +authenticate(username: str, password: str): Optional[UserProfile]
  +get_profile(user_id: str): Optional[UserProfile]
  +count_users(): int
}

interface DayLogRepository {
  +load_day(day: date, user_id: str): DayLog
  +save_day(log: DayLog, user_id: str): void
  +assign_unowned_entries(user_id: str): int
}

interface ProductCatalog {
  +get_by_barcode(barcode: str): Product
}

interface UserRepository {
  +create_user(user_id: str, username: str, password_hash: str, salt: str, created_at: datetime): UserProfile
  +get_auth_by_username(username: str): Optional[UserAuthRecord]
  +get_profile(user_id: str): Optional[UserProfile]
  +count_users(): int
}

class UserAuthRecord {
  +user_id: str
  +username: str
  +password_hash: str
  +salt: str
  +created_at: datetime
}

class SqliteDayLogRepository
class SqliteUserRepository
class OpenFoodFactsCatalog
class OpenFoodFactsClient

MealService ..> DayLogRepository
ProductService ..> ProductCatalog
UserService ..> UserRepository
DayLogRepository ..> DayLog
ProductCatalog ..> Product
UserRepository ..> UserProfile
UserRepository ..> UserAuthRecord

SqliteDayLogRepository ..|> DayLogRepository
SqliteUserRepository ..|> UserRepository
OpenFoodFactsCatalog ..|> ProductCatalog
OpenFoodFactsCatalog --> OpenFoodFactsClient

DayLog "1" *-- "many" MealEntry
MealEntry --> Nutrients
Product --> Nutrients

@enduml
```

### In case of a distributed system

- Not a distributed system; app runs as a single process with an external API call.


## Interaction

- Interaction flow:
  - UI → services: Streamlit handlers call application services for login, barcode lookup, and meal logging.
  - Services → infrastructure: `ProductService` calls `ProductCatalog` (Open Food Facts adapter), while `MealService` and `UserService` call repositories for SQLite persistence.
- Pattern:
  - Simple synchronous request/response with in-process calls and an external HTTP request for product lookup.

```plantuml
@startuml
skinparam sequenceMessageAlign center
actor User #cfe8ff
participant "Streamlit UI" as UI #e8f5e9
participant "ProductService" as ProductSvc #fff8e1
participant "OpenFoodFactsCatalog" as Catalog #fffde7
participant "OpenFoodFactsClient" as Client #fffde7
participant "Open Food Facts API" as OFF #f3e5f5
participant "MealService" as MealSvc #fff8e1
participant "SqliteDayLogRepository" as MealRepo #ffebee
database "SQLite DB" as DB #e0f7fa

User -> UI: Enter barcode / add entry
UI -> ProductSvc: lookup_product(barcode)
ProductSvc -> Catalog: get_by_barcode(barcode)
Catalog -> Client: get_product_by_barcode(barcode)
Client -> OFF: GET /product/{barcode}
OFF --> Client: product data
Client --> Catalog: Product
Catalog --> ProductSvc: Product
ProductSvc --> UI: Product info

User -> UI: Save meal entry
UI -> MealSvc: add_entry(...)
MealSvc -> MealRepo: load_day(day, user_id)
MealRepo -> DB: SELECT day entries
DB --> MealRepo: rows
MealRepo --> MealSvc: DayLog
MealSvc -> MealRepo: save_day(log, user_id)
MealRepo -> DB: DELETE + INSERT entries
DB --> MealRepo: OK
MealRepo --> MealSvc: OK
MealSvc --> UI: MealEntry
@enduml
```

```plantuml
@startuml
skinparam sequenceMessageAlign center
actor User #cfe8ff
participant "Streamlit UI" as UI #e8f5e9
participant "UserService" as UserSvc #fff8e1
participant "SqliteUserRepository" as UserRepo #ffebee
participant "MealService" as MealSvc #fff8e1
participant "SqliteDayLogRepository" as MealRepo #ffebee
database "SQLite DB" as DB #e0f7fa

User -> UI: Sign up
UI -> UserSvc: count_users()
UserSvc -> UserRepo: count_users()
UserRepo -> DB: SELECT COUNT(*)
DB --> UserRepo: count
UserRepo --> UserSvc: count
UserSvc --> UI: count

UI -> UserSvc: signup(username, password)
UserSvc -> UserRepo: get_auth_by_username(username)
UserRepo -> DB: SELECT * FROM users WHERE username = ?
DB --> UserRepo: row/none
UserRepo --> UserSvc: record/none
UserSvc -> UserSvc: generate salt
UserSvc -> UserSvc: _hash_password(password, salt) [PBKDF2]
UserSvc -> UserRepo: create_user(...)
UserRepo -> DB: INSERT INTO users
DB --> UserRepo: OK
UserRepo --> UserSvc: UserProfile
UserSvc --> UI: UserProfile

alt first user
  UI -> MealSvc: assign_unowned_entries(user_id)
  MealSvc -> MealRepo: assign_unowned_entries(user_id)
  MealRepo -> DB: UPDATE meal_entries SET user_id = ?
  DB --> MealRepo: updated
  MealRepo --> MealSvc: count
  MealSvc --> UI: count
end

User -> UI: Login
UI -> UserSvc: authenticate(username, password)
UserSvc -> UserRepo: get_auth_by_username(username)
UserRepo -> DB: SELECT * FROM users WHERE username = ?
DB --> UserRepo: row/none
UserRepo --> UserSvc: record/none
UserSvc -> UserSvc: _hash_password(password, salt) [PBKDF2]
UserSvc -> UserSvc: hmac.compare_digest(expected, candidate)
UserSvc --> UI: UserProfile or None
@enduml
```

```plantuml
@startuml
skinparam sequenceMessageAlign center
actor User #cfe8ff
participant "Streamlit UI" as UI #e8f5e9
participant "MealService" as MealSvc #fff8e1
participant "SqliteDayLogRepository" as MealRepo #ffebee
database "SQLite DB" as DB #e0f7fa

User -> UI: View day log
UI -> MealSvc: get_day_log(user_id, day)
MealSvc -> MealRepo: load_day(day, user_id)
MealRepo -> DB: SELECT day entries
DB --> MealRepo: rows
MealRepo --> MealSvc: DayLog
MealSvc --> UI: DayLog

UI -> MealSvc: get_day_totals(user_id, day)
MealSvc -> MealRepo: load_day(day, user_id)
MealRepo -> DB: SELECT day entries
DB --> MealRepo: rows
MealRepo --> MealSvc: DayLog
MealSvc --> UI: Nutrients totals
@enduml
```

```plantuml
@startuml
skinparam sequenceMessageAlign center
actor User #cfe8ff
participant "Streamlit UI" as UI #e8f5e9
participant "MealService" as MealSvc #fff8e1
participant "SqliteDayLogRepository" as MealRepo #ffebee
database "SQLite DB" as DB #e0f7fa

User -> UI: View last 7 days
UI -> MealSvc: get_last_days_totals(user_id, end_day, days=7)
loop for each day
  MealSvc -> MealRepo: load_day(day, user_id)
  MealRepo -> DB: SELECT day entries
  DB --> MealRepo: rows
  MealRepo --> MealSvc: DayLog
  MealSvc --> UI: (date, totals)
end
@enduml
```

## Behaviour

- Component behavior:
  - Streamlit UI maintains session state (current user, lookup values) and renders logs/totals.
  - `MealService` validates grams, scales nutrients, creates `MealEntry`, updates `DayLog`, and computes daily/weekly totals.
  - `UserService` handles signup, hashes passwords with PBKDF2, and authenticates via constant-time comparison.
  - `ProductService` validates barcodes and fetches product/nutrient data.
  - Application services are stateless; repositories manage persistence through a shared SQLite connection.
  - The Open Food Facts client holds an HTTP session but no domain state.
- State updates:
  - Only services mutate state via repositories; UI remains stateless aside from session variables.
  - `MealService.assign_unowned_entries` links existing meal entries to the first created user.


## Data-related aspects (in case persistent storage is needed)

- Stored data:
  - Users (username, password hash, salt, created_at).
  - Meal entries (day, timestamp, barcode, product name, grams, nutrients).
  - Stored in SQLite (`data/app.sqlite` by default) for local persistence.
- Storage model:
  - Relational tables: `users`, `meal_entries`. SQLite is lightweight and sufficient for a single-user local app.
- Database access:
  - `SqliteUserRepository` queries by username or user_id, and inserts new users.
  - `SqliteDayLogRepository` loads entries by day/user (`SELECT ... WHERE day AND user_id`) , deletes and re-inserts for save.
  - Concurrency is minimal (single-process Streamlit), so no special locking needed.
- Shared data:
  - Only the SQLite file is shared across services within the same process; no cross-process sharing.
