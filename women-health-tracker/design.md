# Design Document: Women Health Tracker

## Overview

Women Health Tracker is a single-file web application (`index.html`) that runs entirely in the browser with no backend. All data is persisted in `localStorage` using AES-256 encryption via the Web Crypto API. The app targets mobile browsers and is designed as a Progressive Web App (PWA) with a soft pink/lavender aesthetic.

**Key design decisions:**

- Single `index.html` with embedded `<style>` and `<script>` — zero build tooling, zero dependencies
- `localStorage` as the sole data store, encrypted with a user-derived key (PBKDF2 → AES-GCM)
- Bottom tab navigation with five tabs: Home, Calendar, Log, Insights, Settings
- Cycle prediction via average of last 3 complete cycles (fallback: 28-day default)
- Push notifications via the Web Notifications API with `setTimeout`/`setInterval` scheduling
- Passcode lock enforced by `Auth_Guard` before any app content is rendered
- PDF export via `window.print()` with a dedicated print stylesheet
- Onboarding flow (≤5 screens) shown only on first launch, gated by a `onboardingComplete` flag in storage

---

## Architecture

The app is structured as a single-page application (SPA) with a hand-rolled router that swaps visible `<section>` elements based on the active tab. There is no virtual DOM — all UI updates are direct DOM mutations.

```
index.html
├── <head>  — meta, PWA manifest link, embedded <style>
└── <body>
    ├── #auth-screen        — passcode / biometric gate
    ├── #onboarding         — first-launch wizard (5 slides)
    ├── #app                — main app shell (hidden until auth passes)
    │   ├── #screen-home    — Dashboard
    │   ├── #screen-calendar
    │   ├── #screen-log
    │   ├── #screen-insights
    │   └── #screen-settings
    └── #bottom-nav         — five tab buttons
    <script>  — all application logic
```

### Module Boundaries (logical, within the single script block)

```
┌─────────────────────────────────────────────────────┐
│                     App Shell                        │
│  Router · BottomNav · ThemeManager · OnboardingFlow  │
└───────────────┬─────────────────────────────────────┘
                │
   ┌────────────▼────────────┐
   │       Data Layer         │
   │  DataStore (encrypt/     │
   │  decrypt localStorage)   │
   └────────────┬────────────┘
                │
   ┌────────────▼────────────┐   ┌──────────────────┐
   │     Business Logic       │   │   Auth_Guard      │
   │  CyclePredictor          │   │  (passcode lock)  │
   │  SymptomManager          │   └──────────────────┘
   │  ReminderScheduler       │
   │  InsightsEngine          │
   │  ReportExporter          │
   └────────────┬────────────┘
                │
   ┌────────────▼────────────┐
   │       UI Screens         │
   │  Dashboard · Calendar    │
   │  Log · Insights          │
   │  Settings                │
   └─────────────────────────┘
```

---

## Components and Interfaces

### DataStore

Responsible for all reads and writes to `localStorage`. Every write encrypts the payload; every read decrypts it.

```js
DataStore.save(key, value)        // encrypt → localStorage.setItem
DataStore.load(key)               // localStorage.getItem → decrypt → parse
DataStore.remove(key)             // localStorage.removeItem
DataStore.clearAll()              // localStorage.clear() — used by data deletion
DataStore.init(passphrase)        // derive CryptoKey via PBKDF2, store in memory
```

Encryption scheme: PBKDF2 (SHA-256, 100 000 iterations) derives a 256-bit AES-GCM key from the user's passcode (or a device-generated random passphrase when no passcode is set). A random 12-byte IV is prepended to each ciphertext blob.

### Auth_Guard

```js
AuthGuard.enable(passcode)        // hash + store passcode, set authEnabled flag
AuthGuard.disable()               // remove passcode, clear authEnabled flag
AuthGuard.verify(passcode)        // returns true/false; increments failCount
AuthGuard.isLocked()              // returns true if failCount >= 5 within lockout window
AuthGuard.reset()                 // clears failCount after 30-second lockout
```

Lockout: after 5 consecutive failures the UI shows a 30-second countdown; `AuthGuard.isLocked()` returns `true` during this window.

### CyclePredictor

```js
CyclePredictor.predict(cycles)
// cycles: Array<{ startDate: string, endDate: string }>
// returns: {
//   nextPeriodDate: string,
//   fertileWindowStart: string,
//   fertileWindowEnd: string,
//   ovulationDate: string,
//   avgCycleLength: number,
//   avgPeriodDuration: number,
//   phase: 'period' | 'fertile' | 'ovulation' | 'luteal' | 'unknown'
// }
```

Algorithm:
1. If `cycles.length >= 3`, compute average cycle length from the last 3 complete cycles.
2. If `cycles.length == 1 or 2`, use that single/pair average.
3. If `cycles.length == 0`, return `phase: 'unknown'` and no dates.
4. `nextPeriodDate` = last period start + avgCycleLength days.
5. `ovulationDate` = `nextPeriodDate` − 14 days.
6. `fertileWindowStart` = `ovulationDate` − 5 days; `fertileWindowEnd` = `ovulationDate` + 1 day.
7. Current `phase` is determined by comparing today against the most recent cycle's dates.

### SymptomManager

```js
SymptomManager.log(date, symptoms)   // symptoms: string[]
SymptomManager.get(date)             // returns string[]
SymptomManager.update(date, symptoms)
SymptomManager.remove(date)
SymptomManager.getTrends(cycles)     // returns symptom frequency grouped by phase
SymptomManager.getCustomLabels()     // returns user-defined symptom strings
SymptomManager.addCustomLabel(label)
```

### ReminderScheduler

```js
ReminderScheduler.schedule(type, date, daysBefore)
// type: 'period' | 'ovulation' | 'medication'
ReminderScheduler.cancel(type)
ReminderScheduler.requestPermission()   // wraps Notification.requestPermission()
ReminderScheduler.notify(title, body)   // new Notification(title, { body })
```

Scheduling is done by computing `triggerTime = targetDate - daysBefore` and using `setTimeout` (for same-session) plus storing the scheduled time in `DataStore` so the scheduler can re-arm on next app open.

### InsightsEngine

```js
InsightsEngine.getCycleLengthHistory(cycles)   // last 6 cycle lengths → number[]
InsightsEngine.getSymptomFrequency(logs, cycles) // symptom → { period, fertile, luteal } counts
InsightsEngine.getAvgPeriodDuration(cycles)    // number (days)
```

### ReportExporter

```js
ReportExporter.generate(startDate, endDate)
// Builds a hidden #print-view DOM section with cycle + symptom table,
// then calls window.print()
```

### Router

```js
Router.navigate(tabId)   // shows target screen, hides others, updates nav highlight
Router.current()         // returns active tabId
```

Navigation must complete within 300 ms (requirement 8.2) — achieved by CSS `display` toggling with no async work.

---

## Data Models

All models are stored as JSON, encrypted, in `localStorage`.

### CycleEntry

```ts
interface CycleEntry {
  id: string;           // UUID v4
  startDate: string;    // ISO 8601 date "YYYY-MM-DD"
  endDate: string | null; // null until user logs end date
}
```

### SymptomLog

```ts
interface SymptomLog {
  date: string;         // ISO 8601 date
  symptoms: string[];   // e.g. ["cramps", "fatigue", "custom:bloating"]
}
```

Custom symptoms are prefixed with `"custom:"` to distinguish them from defaults.

### ReminderConfig

```ts
interface ReminderConfig {
  periodReminderEnabled: boolean;
  periodReminderDaysBefore: number;   // default 2
  ovulationReminderEnabled: boolean;
  ovulationReminderDaysBefore: number; // default 1
  medicationReminderEnabled: boolean;
  medicationReminderTime: string;     // "HH:MM" 24h
}
```

### AppSettings

```ts
interface AppSettings {
  onboardingComplete: boolean;
  passcodeEnabled: boolean;
  passcodeHash: string | null;        // SHA-256 hex of passcode
  pregnancyMode: boolean;
  pregnancyStartDate: string | null;
  colorTheme: 'default';              // extensible
  notificationPermission: 'granted' | 'denied' | 'default';
}
```

### localStorage Key Map

| Key | Value type |
|-----|-----------|
| `wht_cycles` | `CycleEntry[]` |
| `wht_symptoms` | `SymptomLog[]` |
| `wht_reminders` | `ReminderConfig` |
| `wht_settings` | `AppSettings` |
| `wht_custom_symptoms` | `string[]` |
| `wht_salt` | base64 PBKDF2 salt |

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Property 1: Cycle Entry Round-Trip

*For any* valid period start date and end date (where end >= start), saving a cycle entry and then loading it from the DataStore should return an entry with the same start date, end date, and a stable ID.

**Validates: Requirements 1.1, 1.2**

### Property 2: End-Date Validation Rejects Invalid Ranges

*For any* pair of dates where endDate < startDate, the validation function should reject the entry and the DataStore should remain unchanged.

**Validates: Requirements 1.3**

### Property 3: Cycle Entry Deletion

*For any* cycle entry that exists in the DataStore, deleting it by ID and then loading all cycles should return a list that does not contain that entry.

**Validates: Requirements 1.4**

### Property 4: Predictions Populated for Non-Empty Cycle History

*For any* non-empty array of complete cycle entries, CyclePredictor.predict should return non-null values for nextPeriodDate, fertileWindowStart, fertileWindowEnd, and ovulationDate.

**Validates: Requirements 2.1, 2.3**

### Property 5: Average Cycle Length Calculation

*For any* array of three or more complete cycles, the avgCycleLength returned by CyclePredictor.predict should equal the arithmetic mean of the last three cycle lengths (in days), rounded to the nearest integer.

**Validates: Requirements 2.2, 6.3**

### Property 6: Predictions Reflect Updated History

*For any* two different cycle histories H1 and H2 (where H2 is H1 plus one new entry), CyclePredictor.predict(H2) should return a different nextPeriodDate than CyclePredictor.predict(H1) whenever the new entry changes the average cycle length.

**Validates: Requirements 1.5**

### Property 7: Calendar Date Symptoms Round-Trip

*For any* date and any non-empty list of symptoms, logging those symptoms for that date and then querying symptoms for that same date should return the same list.

**Validates: Requirements 3.2**

### Property 8: Calendar Future Month Predictions Match Predictor

*For any* future calendar month and any cycle history, the set of dates marked as predicted period or fertile window in the calendar view should exactly match the date ranges computed by CyclePredictor.predict for that month.

**Validates: Requirements 3.3**

### Property 9: Month Navigation Round-Trip

*For any* starting calendar month, navigating forward N months and then backward N months should return to the original month.

**Validates: Requirements 3.4**

### Property 10: Symptom Log Round-Trip

*For any* non-empty set of symptom strings and a date, saving them via SymptomManager.log and then retrieving via SymptomManager.get for that date should return the same set of symptoms.

**Validates: Requirements 4.2**

### Property 11: Custom Symptom Label Round-Trip

*For any* non-empty custom label string, adding it via SymptomManager.addCustomLabel and then calling SymptomManager.getCustomLabels should return a list that includes that label.

**Validates: Requirements 4.3**

### Property 12: Symptom Update and Delete

*For any* symptom log entry for a given date, updating it with a new symptom list should result in SymptomManager.get returning the new list; removing it should result in SymptomManager.get returning null or an empty list.

**Validates: Requirements 4.5**

### Property 13: Reminder Scheduling Accuracy

*For any* predicted event date (period or ovulation) and any configured daysBefore value (>= 0), the scheduled trigger time computed by ReminderScheduler should equal the event date minus daysBefore days (at the start of that day).

**Validates: Requirements 5.1, 5.2**

### Property 14: Reminder Config Persistence

*For any* reminder configuration (enabled flags and time values), saving it to the DataStore and reloading it should return an identical configuration object, with each reminder type's enabled state independent of the others.

**Validates: Requirements 5.3, 5.4**

### Property 15: Dashboard Data Accuracy

*For any* cycle history, the phase and daysUntilNextPeriod values displayed by the dashboard should exactly match the phase and the difference (in days) between today and nextPeriodDate as returned by CyclePredictor.predict.

**Validates: Requirements 6.1, 6.2**

### Property 16: Health Tips Non-Empty Per Phase

*For any* valid cycle phase value ('period', 'fertile', 'ovulation', 'luteal', 'unknown'), the tips lookup function should return an array of at least one non-empty string.

**Validates: Requirements 6.4**

### Property 17: Encryption Round-Trip

*For any* JavaScript object, encrypting it via DataStore.save and then decrypting it via DataStore.load should return an object deeply equal to the original; the raw localStorage value should not contain any plaintext field names or values from the original object.

**Validates: Requirements 7.1**

### Property 18: Passcode Verification

*For any* passcode string P, after calling AuthGuard.enable(P): AuthGuard.verify(P) should return true, and AuthGuard.verify(Q) should return false for any Q ≠ P.

**Validates: Requirements 7.2**

### Property 19: Data Deletion Completeness

*For any* DataStore state containing one or more entries, calling DataStore.clearAll() and then calling DataStore.load for any previously stored key should return null.

**Validates: Requirements 7.6**

### Property 20: Onboarding Shown Only Once

*For any* app session where onboardingComplete is true in AppSettings, the onboarding flow should not be rendered; setting onboardingComplete to true and reloading should preserve that flag.

**Validates: Requirements 8.5**

### Property 21: Symptom Frequency Grouped by Phase

*For any* symptom log containing entries across at least 14 distinct days and a corresponding cycle history, InsightsEngine.getSymptomFrequency should return a non-empty object with keys for at least one cycle phase, where each phase's symptom counts are non-negative integers.

**Validates: Requirements 9.1**

### Property 22: Cycle Length History Capped at 6

*For any* cycle history of length N, InsightsEngine.getCycleLengthHistory should return an array of length min(N, 6), containing the lengths of the most recent cycles in chronological order.

**Validates: Requirements 9.2**

### Property 23: Average Period Duration

*For any* array of three or more complete cycles, InsightsEngine.getAvgPeriodDuration should return a positive number equal to the arithmetic mean of the individual period durations (endDate − startDate in days).

**Validates: Requirements 9.3**

### Property 24: Pregnancy Mode Round-Trip

*For any* app state with pregnancy mode disabled, enabling pregnancy mode and then disabling it should restore all cycle tracking data and views to their original state, with no cycle entries lost.

**Validates: Requirements 10.3**

### Property 25: Pregnancy Milestones Non-Empty Per Week

*For any* pregnancy week number between 1 and 42 inclusive, the pregnancy milestones lookup should return at least one non-empty milestone string.

**Validates: Requirements 10.2**

### Property 26: Report Content Completeness

*For any* date range [start, end] and a DataStore containing cycle entries and symptom logs, the content generated by ReportExporter.generate(start, end) should include every cycle entry whose startDate falls within [start, end] and every symptom log entry whose date falls within [start, end].

**Validates: Requirements 11.1**

### Property 27: Data Retained After Failed Export

*For any* DataStore state, if ReportExporter.generate throws an error, all cycle entries and symptom logs present before the export attempt should still be retrievable from the DataStore after the error.

**Validates: Requirements 11.3**

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| End date before start date | Inline validation error shown in Log screen; entry not saved |
| localStorage quota exceeded | Toast error: "Storage full. Please export and clear old data." |
| Notification permission denied | Settings screen shows a banner with a link to browser settings |
| Auth lockout (5 failures) | 30-second countdown displayed; all input disabled during lockout |
| CyclePredictor called with 0 cycles | Returns `{ phase: 'unknown', ...nullDates }`; Dashboard shows "Log your first period to get started" |
| Decryption failure (corrupted data) | DataStore.load returns `null`; app treats key as missing and logs a console warning |
| Export (print) cancelled by user | No error shown; data unchanged |
| Invalid date input (non-date string) | Validation rejects with "Please enter a valid date" |

All errors are surfaced via a shared `showToast(message, type)` utility that renders a dismissible banner at the top of the screen. No errors are silently swallowed.

---

## Testing Strategy

### Dual Testing Approach

Both unit tests and property-based tests are required. They are complementary:

- **Unit tests** cover specific examples, integration points, and edge cases
- **Property-based tests** verify universal correctness across randomized inputs

### Unit Tests (specific examples and edge cases)

- Onboarding: first launch shows onboarding; second launch skips it
- Default symptom list contains exactly: cramps, mood, fatigue, headache (Requirement 4.1)
- Bottom nav renders exactly 5 tabs: Home, Calendar, Log, Insights, Settings (Requirement 8.1)
- Onboarding has ≤ 5 slides (Requirement 8.4)
- Auth lockout: 5 wrong passcode attempts triggers `isLocked() === true` (Requirement 7.4)
- Empty cycle history: CyclePredictor returns `phase: 'unknown'` (Requirement 2.5)
- Notification permission not granted: `requestPermission()` is called before scheduling (Requirement 5.6)
- Pregnancy mode: enabling replaces cycle views; disabling restores them (Requirement 10.1)

### Property-Based Tests

**Library**: [fast-check](https://github.com/dubzzz/fast-check) (JavaScript, runs in browser or Node)

Each property test must run a **minimum of 100 iterations**.

Each test must include a comment tag in the format:
`// Feature: women-health-tracker, Property N: <property_text>`

| Property | Test description |
|----------|-----------------|
| P1 | Generate random valid date pairs; save + load cycle entry |
| P2 | Generate random date pairs where end < start; assert rejection |
| P3 | Generate random cycle list; delete one; assert absent |
| P4 | Generate random non-empty cycle arrays; assert all prediction fields non-null |
| P5 | Generate random arrays of ≥3 cycles; assert avgCycleLength = mean of last 3 |
| P6 | Generate H1; derive H2 by appending a cycle that changes the average; assert predictions differ |
| P7 | Generate random date + symptom list; log + retrieve; assert equality |
| P8 | Generate random future month + cycle history; assert calendar dates match predictor output |
| P9 | Generate random month + N (1–24); navigate forward N, back N; assert same month |
| P10 | Generate random symptom sets + dates; save + load; assert equality |
| P11 | Generate random label strings; add + list; assert inclusion |
| P12 | Generate random symptom entries; update then delete; assert correct state |
| P13 | Generate random event dates + daysBefore; assert trigger = date − daysBefore |
| P14 | Generate random ReminderConfig objects; save + load; assert deep equality |
| P15 | Generate random cycle histories; assert dashboard phase + daysUntil match predictor |
| P16 | For each phase enum value; assert tips array length ≥ 1 |
| P17 | Generate random JS objects; encrypt + decrypt; assert deep equality + raw value is not plaintext |
| P18 | Generate random passcode strings; enable + verify correct/wrong; assert true/false |
| P19 | Generate random DataStore state; clearAll; assert all keys return null |
| P20 | Set onboardingComplete = true; reload; assert onboarding not shown |
| P21 | Generate ≥14-day symptom logs + cycle history; assert grouped frequency non-empty |
| P22 | Generate cycle arrays of length 1–20; assert history length = min(N, 6) |
| P23 | Generate ≥3 complete cycles; assert avgPeriodDuration = mean of durations |
| P24 | Generate cycle data; enable pregnancy mode; disable; assert cycle data unchanged |
| P25 | For each week 1–42; assert milestones array length ≥ 1 |
| P26 | Generate date range + data; generate report; assert all in-range entries present |
| P27 | Generate DataStore state; simulate export failure; assert data unchanged |
