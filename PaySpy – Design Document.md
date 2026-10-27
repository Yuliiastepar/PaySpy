# PaySpy – Design Document

*Version 1.0 (MVP) · 25 October 2026 · Derived from SPEC.docx and ACCEPTANCE\_CRITERIA.docx*

## 0. How to read this document

One rule governs this document: **every element must trace back to the specification.** Each part carries a *Basis* reference to a SPEC item (S1.1 … S5.3, or a SPEC section such as §4 Constraints) or to an acceptance criterion (F1-01 … F9-10, AC §n). An element with no basis is either padding and was removed, or it reveals a missing sub-task and is listed in section 7.

The SPEC describes one person, one phone, local data only and no server. The usual backend parts of a design document (REST endpoints, login, MySQL) therefore have no basis here. Sections 2 and 3 state what replaces them and why. Adding them would be padding. *Basis: SPEC §2 (no Admin, no login), §4 Data.*

## 1. Data model

**Storage:** Room over SQLite in app-private storage. *Basis: SPEC §4 Technologies, Data.*

**Conventions**

- **Money is a `Long` in minor units** (cents, kopiykas). `Float` and `Double` are never used, and SQLite has no real DECIMAL type. *Basis: AC §2 ‘Počítanie súm’, F6-06.*
- **NBU rate is a `Long` multiplied by 10 000** (50.0000 is stored as 500000). *Basis: AC F3-09 (four decimals).*
- **Timestamps are epoch milliseconds (UTC).** Calendar days and months are computed in the device time zone. *Basis: S4.1 and the working assumption ‘calendar month’.*
- **Enums are stored as TEXT.** Supported currencies are exactly `EUR` and `UAH`. *Basis: SPEC §4 Currencies.*

### 1.1 Table `card`

*Basis: S3.1, S3.2.*

| Column | Type | Note |
| --- | --- | --- |
| id | INTEGER PK |  |
| name | TEXT | e.g. ‘Tatra’ |
| bank | TEXT | Tatra banka, monobank, Sense Bank (free text, not an enum) |
| currency | TEXT | EUR or UAH |
| initial\_balance\_minor | INTEGER | entered as of first launch (working assumption) |

There is no stored `balance` column (see 4.5).

### 1.2 Table `category`

*Basis: S2.3, S4.2, S5.3.*

| Column | Type | Note |
| --- | --- | --- |
| id | INTEGER PK |  |
| name | TEXT | editable |
| kind | TEXT | EXPENSE or INCOME (income examples in AC: ‘Štipendium’, ‘Brigáda’) |
| is\_default | INTEGER | 1 only for ‘Nezaradené’, which cannot be deleted (F1-01 requires a category on every record) |

Seed rows: Nezaradené, Potraviny, Doprava, Bývanie a účty (expense) and Štipendium, Brigáda (income). *Basis: AC F1-01, T6 test set.*

### 1.3 Table `merchant_rule`

*Basis: S2.3, S5.3.*

| Column | Type | Note |
| --- | --- | --- |
| id | INTEGER PK |  |
| pattern | TEXT | normalised key (see `merchant_key` below) |
| category\_id | INTEGER FK → category |  |

### 1.4 Table `transactions`

*Basis: S1.1–S1.3, S2.1–S2.5, AC §2 and §8.* The name is plural because `transaction` is a reserved word in SQL.

| Column | Type | Basis |
| --- | --- | --- |
| id | INTEGER PK |  |
| type | TEXT: EXPENSE / INCOME / TRANSFER / UNRECOGNIZED | S2.1, S1.2 |
| source | TEXT: NOTIFICATION / MANUAL | F1-01 (‘zdroj = notifikácia’), F9-09 |
| occurred\_at | INTEGER | S1.1 ‘čas’, S2.1 ‘dátum’; for notifications it equals the notification’s `postTime` |
| received\_at | INTEGER NULL | AC ‘Čas prijatia’; set only for NOTIFICATION rows; used by dedup (F9) |
| created\_at | INTEGER | AC §8 `createdAt` (measures the 5 s limit in F1-01) |
| card\_id | INTEGER NULL FK | NULL = card not determined yet (open question ‘which card’) |
| transfer\_card\_id | INTEGER NULL FK | S2.1 transfers; target card |
| category\_id | INTEGER NULL FK | S2.1, S2.3; NULL for TRANSFER and UNRECOGNIZED |
| merchant | TEXT NULL | S1.1 |
| merchant\_key | TEXT NULL | trimmed and lower-cased in Kotlin (`lowercase(Locale.ROOT)`); used for dedup and rule matching, because SQLite’s `LOWER()` handles ASCII only and would break Cyrillic merchants (F9-06) |
| note | TEXT NULL | S2.1 |
| amount\_minor | INTEGER NULL | original amount; NULL only for UNRECOGNIZED |
| currency | TEXT NULL | original currency |
| transfer\_amount\_minor | INTEGER NULL | amount received, in the target card’s currency (open question ‘transfer between currencies’) |
| amount\_eur\_minor | INTEGER NULL | amount in EUR (S2.5, S5.2); equals `amount_minor` when the original is EUR |
| amount\_uah\_minor | INTEGER NULL | amount in UAH; equals `amount_minor` when the original is UAH |
| rate\_x10000 | INTEGER NULL | NBU rate (UAH per 1 EUR) used for the conversion; never changed retroactively (SPEC §4 Currencies, F3-05) |
| rate\_date | TEXT NULL (ISO date) | date of the rate actually used; differs from the payment date on fallback (F3-01, F3-06) |
| raw\_text | TEXT NULL | notification text kept unchanged, character for character (F1-05, F1-07) |

**Why both `amount_eur_minor` and `amount_uah_minor`?** The base currency is switchable (S5.2) and the switch must work offline and must not alter stored amounts or rates (F3-08). The report and balance totals must use the stored value, not today’s rate (F6-05), and each transaction is rounded individually (F3-02). Storing both snapshots lets every sum be a plain SQL `SUM` over one column. The ‘base amount’ is whichever column matches the current base currency. If that column is NULL the row is shown as ‘bez kurzu’ (F3-07).

**Integrity rules** (enforced with `CHECK` constraints and in the repository):

- UNRECOGNIZED has `raw_text` set and amount, currency, category and card empty.
- TRANSFER has `card_id`, `transfer_card_id` and `transfer_amount_minor`, and no category.
- A transfer between cards of the same currency stores the same amount in both fields.
- Manual and notification records live in the same table, so list, filters and reports need no special cases.

**Indexes:** `occurred_at` (month filter S2.4, report S4.1), `card_id` (S3.2), `category_id` (S4.3), and `(source, currency, amount_minor, merchant_key, received_at)` (F9 lookup).

### 1.5 Table `exchange_rate`

*Basis: S2.5, AC F3-05 to F3-09.* A cache of NBU data.

| Column | Type | Note |
| --- | --- | --- |
| rate\_date | TEXT PK (ISO date) |  |
| uah\_per\_eur\_x10000 | INTEGER |  |

It is kept separately from `transactions` so that ‘latest known rate’ (F3-06) is a single query. Only EUR/UAH is stored because SPEC §4 allows only these two currencies.

### 1.6 Preferences (DataStore)

*Basis: S5.2.* One key: `base_currency` (EUR by default). The notification-access state is read live from the system and is not stored (S5.1).

## 2. API and interfaces

The app has no server, so there are **no REST endpoints and no network API of its own** (SPEC §4 Data: ‘žiadny server’). Its interfaces are the following three groups.

### 2.1 External API consumed: NBU exchange rates

*Basis: S2.5, SPEC §2 External systems, AC F3-09.*

```
GET https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=EUR&date=YYYYMMDD&json
```

The response is a JSON array. The app reads `rate` and `exchangedate`. An empty array, an error or a timeout means ‘no rate for that date’. This is the only outbound traffic (SPEC §4 Data, Permissions). The exact URL shape must be confirmed against the NBU documentation during implementation (AC §8 assumption).

The Retrofit interface has one method: `suspend fun rate(valcode: String, date: String): List<NbuRate>`.

### 2.2 Android system interfaces

| Interface | Basis | Purpose |
| --- | --- | --- |
| `NotificationListenerService` (`PaySpyListener`) | S1.1, S1.4, F1-03 | Receives Google Wallet notifications while the app is closed, including after a reboot with no manual launch |
| `Settings.ACTION_NOTIFICATION_LISTENER_SETTINGS` intent | S5.1 | Opens the system screen where the user grants access |
| `NotificationManagerCompat.getEnabledListenerPackages()` | S5.1 | Shows whether access is already granted |
| `WorkManager` job `RateSyncWorker` | F3-05, F3-07 | Downloads rates (periodic daily run, plus on demand when a record lacks a rate); requires a network connection |
| Debug-only: `adb shell cmd notification post …` | AC §8 | Simulates notifications; allowed only through the debug package allow-list (3.3) |

### 2.3 Internal interfaces (repository layer used by the ViewModels)

| Interface | Basis |
| --- | --- |
| `CaptureService.onNotification(pkg, title, text, postTime)` | S1.1–S1.4 |
| `TransactionRepository.add / update / delete / observe(filter: month?, categoryId?, cardId?)` | S2.1, S2.2, S2.4 |
| `CardRepository.add / update / observeWithBalances()` and `totalFreeMoney(base)` | S3.1–S3.3 |
| `ReportRepository.observeMonth(month, base): MonthReport` (income, expenses, balance, per-category sums, largest category, average daily expense) | S4.1–S4.4 |
| `CategoryRepository` and `RuleRepository` (CRUD) | S5.3 |
| `RateRepository.resolve(date, allowNetwork)` and `syncPending()` | S2.5, F3-06, F3-07 |
| `SettingsRepository.baseCurrency: Flow<Currency>` and `setBase()` | S5.2 |

All observers return `Flow`, so the report and balances refresh without an app restart (F6-07). Time (`Clock`) and the rate source (`RateSource`) are injected so tests can use fixed time and mock rates (AC §8).

Screens (Compose): Transactions (S2.4), Transaction editor (S2.1, S2.2, S1.2), Cards (S3), Report (S4), Settings (S5). *Basis: SPEC §3 grouping.*

## 3. Authorization and roles

### 3.1 Authentication

None. One person uses one phone and there is no sign-in. *Basis: SPEC §2, §4 Users.*

### 3.2 Roles

One implicit role, *User*, with full rights. The *Admin* role does not exist, so there are no admin endpoints and no 403 handling. *Basis: SPEC §2.*

### 3.3 Protection that does apply

With no accounts, protection comes from the Android platform and a few manifest settings:

- **Private storage.** The database is in the app sandbox. No `ContentProvider` and no exported Activity or Service are declared. The only exported component is `PaySpyListener`, protected by `BIND_NOTIFICATION_LISTENER_SERVICE`, which only the system can hold. *Basis: SPEC §4 Data.*
- **No cloud copy.** `android:allowBackup="false"` and data-extraction rules exclude the database, because otherwise Android would silently upload it to Google. *Basis: SPEC §4 Data (‘žiadny cloud’), §5 (backup is out of scope).*
- **Declared permissions: only `INTERNET` and notification-listener access.** *Basis: SPEC §4 Permissions.* The battery-optimisation advice is a text hint with a link to the system settings screen (no extra permission). *Basis: SPEC §4, S5.1.*
- **Notification source filter.** Only the Google Wallet package is processed (F1-04). The allow-list is a set of package names defined in `src/main` (Wallet) and extended in `src/debug` with `com.android.shell`. The release build contains no such extension. *Basis: SPEC §4 Payment source, AC F1-04 and §8.*
- **No PIN or biometrics.** Out of scope by decision. *Basis: SPEC §5.*

## 4. Key calculations

All critical logic is in pure Kotlin functions (package `domain`) with no Android dependencies, so each acceptance criterion can be checked by a unit test. Aggregation (`SUM`, filtering by month) is done by Room queries over the stored integer columns. Nothing derived is cached except the rate snapshots required by the SPEC.

### 4.1 Amounts and rounding

*Basis: AC §2, F3-02, F6-06.* Everything is an integer in minor units. Parsing text such as `12,40` goes through `BigDecimal` into cents and never through `Double`. Conversion uses integer arithmetic with half-up rounding:

- UAH → EUR: `eur = roundHalfUp(uah × 10000 ÷ rate_x10000)`
- EUR → UAH: `uah = roundHalfUp(eur × rate_x10000 ÷ 10000)`

Checks against the criteria: 33333 kop at 50.0000 → 667 cents (F3-02a); 10000 kop at 48.5000 → 206 cents (F3-02b); 1 kop at 50.0000 → 0 cents (F3-02c); 1240 cents → 62000 kop (F3-08).

### 4.2 Resolving the rate

*Basis: S2.5, SPEC §4 Currencies, F3-01, F3-03 to F3-07, F3-09.* The payment date is the device-local date of `occurred_at`, never today’s date (F3-04). `RateRepository.resolve(date, allowNetwork)` works as follows:

1. Look up the exact date in `exchange_rate`.
2. If it is missing and `allowNetwork` is true, request that date from NBU with a short timeout (2 s) and cache the result.
3. If it is still missing, use the latest cached rate **before** the date (weekends, holidays, no internet). The rate’s own date goes into `rate_date` (F3-06).
4. If the cache is empty, return ‘none’. The record is saved without the missing side, shown as ‘bez kurzu’, and filled by `syncPending()` once rates arrive, using the rate for the payment date (F3-07).

`allowNetwork` is true only when the base-currency amount cannot be computed without a rate. A EUR transaction with EUR as base makes no NBU request (F3-03), and its UAH snapshot is filled from the cache only. The conversion factor is stored together with the transaction and is not changed by later rate downloads (F3-05) or by editing the date (open question ‘recalculate on edit’). Editing the amount recalculates both amounts from the **stored** rate (F3-05).

### 4.3 Capture pipeline (notification → record)

*Basis: S1.1–S1.4, S2.3, AC F1, F9.* Steps run in order inside one serialised unit (a `Mutex` plus a Room `@Transaction`), so two notifications arriving together cannot both pass the dedup check:

1. **Filter** by package name; anything else is ignored with no record (F1-04).
2. **Parse** with `NotificationParser.parse(title, text): ParseResult`. The result is either `Parsed(amount, currency, merchant)` or `Unrecognized`. The parser never throws and never returns an amount for empty text, text without a number, a number without a currency, or an unknown currency such as ‘XYZ’ (F1-06). Money is read with exact decimal parsing (F1-02). The regular expressions are finalised against the real bank samples collected in stage E1 (SPEC open question; AC §4 note).
3. **Unrecognized** → store a record of type UNRECOGNIZED with unchanged `raw_text`. This is also the destination for refunds and declined payments until their format is known (S1.2, F1-05, SPEC open questions).
4. **Dedup** (F9, see 4.4).
5. **Category** (S2.3): look for the longest `merchant_rule.pattern` contained in `merchant_key`. If none matches, use ‘Nezaradené’ (F1-01).
6. **Card** (SPEC open question ‘which card’): choose the only card whose currency equals the payment currency. If there are zero or several, leave `card_id` NULL and mark the record in the list as ‘choose card’.
7. **Convert** (4.2), write with `createdAt = now`, `received_at = postTime`, `source = NOTIFICATION`, type EXPENSE.

The work runs in a coroutine owned by the service, and the database insert is the only blocking step before the record exists. This supports the limit of at most 5 seconds from receipt to storage (F1-01).

Steps 3, 5 and 6 also apply to an UNRECOGNIZED record the user later completes (F1-07). Completing the record changes its type to EXPENSE and keeps `raw_text`.

### 4.4 Duplicate check

*Basis: S1.3, AC §7 F9-01 to F9-10.* A new parsed notification is a duplicate if a stored record satisfies all of the following:

```
source = NOTIFICATION AND currency = :c AND amount_minor = :a
AND merchant_key = :key AND ABS(received_at − :t) <= 120000
```

- The 120 s limit is **inclusive** (F9-02).
- The comparison is against **stored** records only. In F9-07 (t, t+100, t+200) the middle notification is dropped, so the third is compared with t and saved.
- The check queries the database, so it survives a process kill (F9-08).
- Manual records have no `received_at` and are never matched, and a notification is never blocked by one (F9-09).
- A duplicate writes nothing, so balance and report are unaffected (F9-10).

### 4.5 Card balance and free money

*Basis: S3.2, S3.3, F9-10, AC §1 (single source of truth).* Balances are **computed on demand**, never stored:

```
balance(card) = initial_balance
  + Σ INCOME on card − Σ EXPENSE on card
  − Σ TRANSFER where card is source + Σ transfer_amount_minor where card is target
```

Income and expense amounts are taken from `amount_eur_minor` or `amount_uah_minor`, whichever matches the **card’s** currency. Rows where that column is NULL are excluded and the balance is marked ‘incl. records without rate’. This applies to a payment whose currency differs from the card’s, an approximation of the bank’s real conversion that the SPEC does not describe (section 7, item 2).

`totalFreeMoney(base)` sums card balances. A balance in a currency other than the base is converted with the latest cached rate (section 7, item 1).

### 4.6 Monthly report

*Basis: S4.1–S4.4, F6-01 to F6-10.* A month is a calendar month in the device time zone: `[first day 00:00, first day of next month 00:00)`. For the selected month and the current base currency (all sums are integer `SUM`s over the matching base column):

- Income = Σ of INCOME records. Expenses = Σ of EXPENSE records. TRANSFER and UNRECOGNIZED are excluded (F6-01).
- Balance = Income − Expenses. The sign decides the colour: positive green with ‘+’, negative red with ‘−’, zero neutral (S4.1, F6-01 to F6-03). The ViewModel exposes a three-value `BalanceSign`, and the colour is chosen in the UI.
- Per-category expenses give the pie chart and the list with amount and percentage (S4.2). Percentage is `roundHalfUp(category × 100 ÷ total)`: 120 ÷ 186 gives 65 % (F6-10). Tapping a category opens the transaction list with that month and category filter (S4.3, S2.4).
- Largest category is the category with the highest expense sum (S4.4, F6-10).
- Average daily expense = Expenses ÷ number of days, rounded half-up to cents. The day count is the full month length for a past month and *day of month of today* (inclusive) for the current month, taken from the injected `Clock` (F6-09). An empty month gives 0,00 € and never NaN (F6-04).

## 5. Project structure

*Basis: SPEC §4 Technologies (one module, MVVM, no unnecessary libraries), Development (one developer, small steps), Distribution (APK).*

One Git repository, one Gradle module (`:app`). The template’s separate `frontend/` and `backend/` do not apply because there is no backend.

```
payspy/
├─ app/
│  ├─ schemas/                     # exported Room schemas, committed
│  └─ src/
│     ├─ main/java/.../payspy/
│     │  ├─ data/      db/ (entities, DAOs), rates/ (Retrofit, RateSyncWorker), prefs/
│     │  ├─ domain/    pure Kotlin: Money, Converter, RateResolver, Parser, Dedup, ReportCalc
│     │  ├─ capture/   PaySpyListener, CaptureService
│     │  ├─ ui/        screens/ + ViewModels (transactions, editor, cards, report, settings)
│     │  └─ AppContainer.kt          # manual dependency wiring
│     ├─ debug/        AndroidManifest + package allow-list with com.android.shell
│     ├─ test/         JVM unit tests + resources/notifications/{tatra,monobank,sense}.json
│     └─ androidTest/  Room, listener/reboot, migration tests
├─ docs/               SPEC.docx, ACCEPTANCE_CRITERIA.docx, DESIGN
└─ README.md           how to build, run, install
```

**Rules**

- **Layers.** `domain` has no Android imports. ViewModels talk only to repositories. UI talks only to ViewModels. This is what makes the AC unit tests (F1-02, F1-06, F3-02, F6-06, F9) possible without a phone.
- **Test data.** At least 20 anonymised real notifications per bank (60 total) with expected amount, currency and merchant, committed with no card numbers (F1-02, SPEC §4 Privacy). Real samples are never pasted into the AI assistant before anonymisation.
- **Migrations.** `exportSchema = true`, and the schema JSON is committed. Every table change means a new version plus an explicit `Migration` plus a migration test. **`fallbackToDestructiveMigration` is forbidden**: the database is the only copy of the data (no backup, SPEC §5), and a destructive migration would wipe it silently.
- **Local run.** Open in Android Studio, run `./gradlew installDebug` on a physical phone (notification access and Google Wallet cannot be tested properly on an emulator). Simulated notifications use `adb shell cmd notification post`. *Basis: AC §8 ‘Testovací telefón’.*
- **Release and ‘deployment’.** `./gradlew assembleRelease`, signed with a local keystore kept outside the repo, then `adb install -r app-release.apk`. The same key must always be used, otherwise Android refuses the update and the user would have to uninstall and lose the data. *Basis: SPEC §4 Distribution.*
- **Build types.** Debug includes the shell allow-list and simulation support. Release does not (AC §8).
- **Work in small steps.** Stages follow the SPEC’s development constraint, with E1 collecting real notification samples before the parser is finalised (AC §4).

## 6. Technical choices and reasons

Choices fixed by the SPEC are marked **\[SPEC\]**. The others are decisions made here.

| Decision | Why | Alternatives rejected |
| --- | --- | --- |
| **Native Android, Kotlin, Jetpack Compose \[SPEC\]** | Only Android lets an app read other apps’ notifications (SPEC §5 ‘iOS’). Compose avoids XML layouts and fits a single developer. | iOS (cannot read notifications); Flutter or React Native (extra layer for a feature that needs a native service anyway) |
| **NotificationListenerService \[SPEC\]** | The system binds it even when the app is closed or has never been opened since boot (S1.4, F1-03). | A foreground service (needs a permanent notification and the system may still stop it); reading SMS (sensitive permission, a template per bank, SPEC §5); bank APIs (SPEC §5) |
| **Room over SQLite \[SPEC\]** | Local relational data with SQL aggregates for balances and the report. Compile-time checked queries and schema export for migrations. | **MySQL or any server database** (SPEC forbids a server, and the app would then need a hosting account); raw `SQLiteOpenHelper` (more boilerplate, no schema checks); a key-value store (no aggregations) |
| **Money as `Long` minor units, rates ×10 000** | The AC require exact integer sums (F6-06) and half-up rounding that can be checked by hand (F3-02). | `Double` (fails F6-06); `BigDecimal` columns (stored as text, which breaks SQL `SUM`) |
| **Both EUR and UAH snapshots stored per transaction** | See 1.4: offline base switch (F3-08), historical rate (F6-05), consistent per-row rounding. | Recomputing from the cache at display time (changes the past when the cache is refreshed, and slow for sums); storing only a base amount (the base switch would then need network and rewrite data) |
| **DataStore \[SPEC\]** for the base-currency setting | A single small preference; asynchronous and safe. | SharedPreferences (older API); a settings table (needless for one key) |
| **WorkManager \[SPEC\]** for rate download | Survives process death, runs only with network, retries. | AlarmManager and a custom service (retries and constraints written by hand) |
| **Retrofit \[SPEC\] with kotlinx.serialization** | One simple GET. kotlinx.serialization needs no reflection and is Kotlin-native. | Hand-written `HttpURLConnection` (JSON and error handling by hand); Moshi or Gson (an extra library with no advantage here) |
| **MVVM with manual wiring (`AppContainer`) \[SPEC: MVVM, no unnecessary libraries\]** | Five screens and a handful of repositories do not justify a DI framework. | Hilt (annotation processing and learning cost); Koin (extra dependency) |
| **Single Gradle module \[SPEC\]** | One developer, small app, faster builds. | Multi-module (overhead without benefit) |
| **Hand-written regex parser with real-sample tests** | The format must be fully predictable (F1-02 requires 100 % on 60 samples). One parser for Google Wallet only (SPEC §5). | An LLM or ML parser (not offline, not deterministic, and notification text would leave the phone) |
| **Pie chart: Compose `Canvas` (to be verified)** | SPEC S4.2 asks for a pie chart, and SPEC §4 lists Vico. As far as I know, Vico draws cartesian charts (columns, lines), not pie charts. If confirmed, a pie is about 40 lines of `Canvas` code and adds no library. | Adding a second chart library (contradicts ‘bez zbytočných knižníc’); a column chart instead of a pie (changes S4.2) |

Vico is still available for any bar or line chart added later, but the MVP needs none.

## 7. Gaps found while tracing (missing sub-tasks) and open decisions

These came up when checking the design against the SPEC. Each has a working default so development can start, and each needs a decision from the product owner.

1. **Total free money in mixed currencies (S3.3).** The SPEC does not say which rate converts a card balance in the non-base currency. *Default:* the latest cached NBU rate (a balance is ‘now’).
2. **A transaction whose currency differs from its card’s currency (S2.1, S3.2).** The balance formula assumes the same currency. *Default:* convert with the stored NBU rate (see 4.5). The bank’s real rate will differ slightly.
3. **Two cards in UAH (monobank and Sense Bank) and one in EUR.** A UAH payment cannot be matched to a card by currency alone. *Default:* leave the card empty and show ‘choose card’ (4.3, step 6). Does Google Wallet’s notification carry any card hint? To be checked on real samples.
4. **Deleting a card.** S3.1 and the role table allow adding and editing cards but never deleting. *Default:* no delete in MVP, because a deletion rule for its transactions would be a new feature.
5. **Inconsistency in the acceptance criteria.** F3-03 requires a stored rate of 1,0000 for a EUR transaction with EUR as base and no NBU request. F3-08 expects the same kind of record to convert to UAH at 50,0000 offline. *Default:* the stored rate is the NBU rate when known (otherwise empty), and the base amount equals the original (see 4.2). F3-03 should be reworded to ‘no NBU request, base amount equals the original amount’.
6. **Duplicate unrecognised notifications.** S1.3 defines duplicates by amount, currency and merchant, which an unparsed notification does not have. *Default:* not de-duplicated.
7. **Interface language** (SPEC open question): all text goes in string resources so the decision does not affect the code, but it must be settled before the screens stage.
8. **Minimum Android version.** Not in the SPEC. The phone’s Android version decides `minSdk`; none is chosen here.
9. **Google Wallet notification formats**, including refunds, declined payments and the card hint, are unknown until the E1 samples exist (SPEC open questions). The parser design is complete only after them.

## Appendix A. Traceability matrix

| SPEC item | Data model | Interface | Calculation / rule |
| --- | --- | --- | --- |
| S1.1 Capture payment | `transactions` (type, source, merchant, amount, currency, occurred\_at) | `PaySpyListener`, `CaptureService` | 4.3 |
| S1.2 Unrecognised record | `raw_text`, type UNRECOGNIZED | `CaptureService` | 4.3 step 3 |
| S1.3 No duplicates | `received_at`, `merchant_key`, dedup index | `CaptureService` | 4.4 |
| S1.4 Capture when closed | – | NotificationListenerService | 3.3, 6 |
| S2.1 Manual add | `transactions`, `transfer_*` | `TransactionRepository.add` | 1.4 integrity rules |
| S2.2 Edit and delete | `transactions` | `update`, `delete` | 4.2 (edit amount) |
| S2.3 Auto category | `merchant_rule`, `category` | `RuleRepository` | 4.3 step 5 |
| S2.4 List and filters | indexes | `observe(filter)` | – |
| S2.5 Original and base amount | `amount_*`, `rate_*`, `exchange_rate` | NBU API, `RateSyncWorker` | 4.1, 4.2 |
| S3.1 Add card | `card` | `CardRepository.add` | – |
| S3.2 Card balance | – (computed) | `observeWithBalances` | 4.5 |
| S3.3 Total free money | – (computed) | `totalFreeMoney` | 4.5 |
| S4.1–S4.4 Report | – (computed) | `ReportRepository` | 4.6 |
| S5.1 Notification access | – (system state) | listener settings intent | 2.2 |
| S5.2 Base currency | DataStore | `SettingsRepository` | 4.2 |
| S5.3 Categories and rules | `category`, `merchant_rule` | `CategoryRepository`, `RuleRepository` | 4.3 |
| SPEC §4 Constraints | – | – | 3.3, 5, 6 |
| AC §8 testability | `created_at`, `raw_text`, `rate_date` | injected `Clock`, `RateSource`; debug allow-list | 5 |

Elements deliberately **not** designed, because the SPEC puts them out of scope (§5): sign-in, PIN, SMS or bank-app parsing, backup and export, limits and alerts, period comparison, iOS, Google Play publication.
