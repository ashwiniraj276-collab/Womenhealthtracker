# Implementation Plan: Women Health Tracker

## Overview

Single-file implementation (`women-health-tracker.html`) using vanilla JS + CSS with no build tooling or dependencies. All features are implemented and complete.

## Tasks

- [x] 1. Set up app shell, routing, and onboarding
  - [x] 1.1 Create `women-health-tracker.html` with embedded `<style>` and `<script>`, PWA meta tags, and five `<section>` screens
    - Define HTML structure: `#auth-screen`, `#onboarding`, `#app`, `#bottom-nav`
    - Apply soft pink/lavender color palette consistently
    - _Requirements: 8.1, 8.3_
  - [x] 1.2 Implement `Router.navigate(tabId)` and bottom nav tab switching within 300 ms
    - Use CSS `display` toggling with no async work
    - _Requirements: 8.1, 8.2_
  - [x] 1.3 Implement onboarding flow (≤5 slides) shown only on first launch
    - Gate on `onboardingComplete` flag in storage; set flag on completion
    - _Requirements: 8.4, 8.5_
  - [x]* 1.4 Write property test for onboarding shown only once (Property 20)
    - **Property 20: Onboarding Shown Only Once**
    - **Validates: Requirements 8.5**
  - [x] 1.5 Checkpoint — Ensure shell renders, nav works, and onboarding flow completes
    - Ensure all tests pass, ask the user if questions arise.

- [x] 2. Implement DataStore with encryption
  - [x] 2.1 Implement `DataStore.init`, `save`, `load`, `remove`, `clearAll` using Web Crypto API (PBKDF2 → AES-GCM)
    - Prepend random 12-byte IV to each ciphertext blob
    - Store PBKDF2 salt under `wht_salt`
    - _Requirements: 7.1, 7.5, 7.6_
  - [x]* 2.2 Write property test for encryption round-trip (Property 17)
    - **Property 17: Encryption Round-Trip**
    - **Validates: Requirements 7.1**
  - [x]* 2.3 Write property test for data deletion completeness (Property 19)
    - **Property 19: Data Deletion Completeness**
    - **Validates: Requirements 7.6**

- [x] 3. Implement Auth_Guard (passcode lock)
  - [x] 3.1 Implement `AuthGuard.enable`, `disable`, `verify`, `isLocked`, `reset`
    - Hash passcode with SHA-256; enforce 5-failure lockout with 30-second countdown
    - _Requirements: 7.2, 7.3, 7.4_
  - [x]* 3.2 Write property test for passcode verification (Property 18)
    - **Property 18: Passcode Verification**
    - **Validates: Requirements 7.2**
  - [x] 3.3 Write unit test for auth lockout after 5 wrong attempts
    - Assert `isLocked() === true` after 5 consecutive failures
    - _Requirements: 7.4_
  - [x] 3.4 Checkpoint — Ensure auth gate blocks app until correct passcode entered
    - Ensure all tests pass, ask the user if questions arise.

- [x] 4. Implement period logging and CyclePredictor
  - [x] 4.1 Implement period log UI (start/end date inputs) with validation and CRUD operations
    - Reject end date earlier than start date with inline error
    - Persist entries to `wht_cycles` via DataStore
    - _Requirements: 1.1, 1.2, 1.3, 1.4_
  - [x]* 4.2 Write property test for cycle entry round-trip (Property 1)
    - **Property 1: Cycle Entry Round-Trip**
    - **Validates: Requirements 1.1, 1.2**
  - [x]* 4.3 Write property test for end-date validation (Property 2)
    - **Property 2: End-Date Validation Rejects Invalid Ranges**
    - **Validates: Requirements 1.3**
  - [x]* 4.4 Write property test for cycle entry deletion (Property 3)
    - **Property 3: Cycle Entry Deletion**
    - **Validates: Requirements 1.4**
  - [x] 4.5 Implement `CyclePredictor.predict(cycles)` with average of last 3 cycles and fallback to 28-day default
    - Compute `nextPeriodDate`, `ovulationDate`, `fertileWindowStart/End`, `avgCycleLength`, `phase`
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_
  - [x]* 4.6 Write property test for predictions populated for non-empty history (Property 4)
    - **Property 4: Predictions Populated for Non-Empty Cycle History**
    - **Validates: Requirements 2.1, 2.3**
  - [x]* 4.7 Write property test for average cycle length calculation (Property 5)
    - **Property 5: Average Cycle Length Calculation**
    - **Validates: Requirements 2.2, 6.3**
  - [x]* 4.8 Write property test for predictions reflecting updated history (Property 6)
    - **Property 6: Predictions Reflect Updated History**
    - **Validates: Requirements 1.5**
  - [x] 4.9 Write unit test for empty cycle history returning `phase: 'unknown'`
    - _Requirements: 2.5_
  - [x] 4.10 Checkpoint — Ensure period logging saves data and predictor returns correct output
    - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implement Calendar screen
  - [x] 5.1 Render monthly calendar grid with color-coded period, predicted period, and fertile window days
    - Use distinct colors for each phase; support forward/back month navigation
    - _Requirements: 3.1, 3.4_
  - [x]* 5.2 Write property test for month navigation round-trip (Property 9)
    - **Property 9: Month Navigation Round-Trip**
    - **Validates: Requirements 3.4**
  - [x] 5.3 Implement date tap to display symptoms logged for that date
    - _Requirements: 3.2_
  - [x]* 5.4 Write property test for calendar date symptoms round-trip (Property 7)
    - **Property 7: Calendar Date Symptoms Round-Trip**
    - **Validates: Requirements 3.2**
  - [x] 5.5 Render predicted cycle phase dates for future months from CyclePredictor output
    - _Requirements: 3.3_
  - [x]* 5.6 Write property test for calendar future month predictions matching predictor (Property 8)
    - **Property 8: Calendar Future Month Predictions Match Predictor**
    - **Validates: Requirements 3.3**

- [x] 6. Implement symptom logging
  - [x] 6.1 Implement Log screen with default symptom options (cramps, mood, fatigue, headache) and save/update/remove for current day
    - Persist to `wht_symptoms` via DataStore
    - _Requirements: 4.1, 4.2, 4.5_
  - [x] 6.2 Write unit test asserting default symptom list contains exactly: cramps, mood, fatigue, headache
    - _Requirements: 4.1_
  - [x] 6.3 Implement custom symptom label input and persistence under `wht_custom_symptoms`
    - Prefix custom labels with `"custom:"`
    - _Requirements: 4.3_
  - [x]* 6.4 Write property test for symptom log round-trip (Property 10)
    - **Property 10: Symptom Log Round-Trip**
    - **Validates: Requirements 4.2**
  - [x]* 6.5 Write property test for custom symptom label round-trip (Property 11)
    - **Property 11: Custom Symptom Label Round-Trip**
    - **Validates: Requirements 4.3**
  - [x]* 6.6 Write property test for symptom update and delete (Property 12)
    - **Property 12: Symptom Update and Delete**
    - **Validates: Requirements 4.5**
  - [x] 6.7 Checkpoint — Ensure symptom logging, custom labels, and CRUD all work end-to-end
    - Ensure all tests pass, ask the user if questions arise.

- [x] 7. Implement Dashboard screen
  - [x] 7.1 Render current cycle phase, days until next period, average cycle length, and contextual health tip
    - Refresh within 1 second of app open; show "Log your first period to get started" when no cycles exist
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_
  - [x]* 7.2 Write property test for dashboard data accuracy (Property 15)
    - **Property 15: Dashboard Data Accuracy**
    - **Validates: Requirements 6.1, 6.2**
  - [x]* 7.3 Write property test for health tips non-empty per phase (Property 16)
    - **Property 16: Health Tips Non-Empty Per Phase**
    - **Validates: Requirements 6.4**

- [x] 8. Implement Insights screen
  - [x] 8.1 Implement `InsightsEngine.getCycleLengthHistory`, `getSymptomFrequency`, `getAvgPeriodDuration`
    - Refresh data each time user navigates to Insights screen
    - _Requirements: 9.1, 9.2, 9.3, 9.4_
  - [x] 8.2 Render cycle length history chart (last 6 cycles) and average period duration
    - _Requirements: 9.2, 9.3_
  - [x] 8.3 Render symptom frequency chart grouped by cycle phase (requires ≥14 logged days)
    - _Requirements: 9.1_
  - [x]* 8.4 Write property test for symptom frequency grouped by phase (Property 21)
    - **Property 21: Symptom Frequency Grouped by Phase**
    - **Validates: Requirements 9.1**
  - [x]* 8.5 Write property test for cycle length history capped at 6 (Property 22)
    - **Property 22: Cycle Length History Capped at 6**
    - **Validates: Requirements 9.2**
  - [x]* 8.6 Write property test for average period duration (Property 23)
    - **Property 23: Average Period Duration**
    - **Validates: Requirements 9.3**

- [x] 9. Implement Health Reminders
  - [x] 9.1 Implement `ReminderScheduler.schedule`, `cancel`, `requestPermission`, `notify`
    - Use `setTimeout` for same-session scheduling; persist scheduled times in DataStore for re-arming on next open
    - Prompt for notification permission if not granted
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6_
  - [x] 9.2 Implement reminder settings UI (enable/disable per type, days-before, medication time)
    - Persist config to `wht_reminders` via DataStore
    - _Requirements: 5.3, 5.4_
  - [x]* 9.3 Write property test for reminder scheduling accuracy (Property 13)
    - **Property 13: Reminder Scheduling Accuracy**
    - **Validates: Requirements 5.1, 5.2**
  - [x]* 9.4 Write property test for reminder config persistence (Property 14)
    - **Property 14: Reminder Config Persistence**
    - **Validates: Requirements 5.3, 5.4**
  - [x] 9.5 Write unit test asserting `requestPermission()` is called before scheduling when permission not granted
    - _Requirements: 5.6_
  - [x] 9.6 Checkpoint — Ensure reminders schedule correctly and settings persist across sessions
    - Ensure all tests pass, ask the user if questions arise.

- [x] 10. Implement Settings screen
  - [x] 10.1 Implement passcode enable/disable UI wired to `AuthGuard`
    - _Requirements: 7.2_
  - [x] 10.2 Implement data deletion option calling `DataStore.clearAll()`
    - _Requirements: 7.6_
  - [x] 10.3 Implement Pregnancy Mode toggle that replaces cycle views with pregnancy week tracker
    - Display weekly milestones and tips; restore cycle views on disable
    - _Requirements: 10.1, 10.2, 10.3_
  - [x]* 10.4 Write property test for pregnancy mode round-trip (Property 24)
    - **Property 24: Pregnancy Mode Round-Trip**
    - **Validates: Requirements 10.3**
  - [x]* 10.5 Write property test for pregnancy milestones non-empty per week (Property 25)
    - **Property 25: Pregnancy Milestones Non-Empty Per Week**
    - **Validates: Requirements 10.2**
  - [x] 10.6 Write unit test for pregnancy mode: enabling replaces cycle views; disabling restores them
    - _Requirements: 10.1_

- [x] 11. Implement PDF export
  - [x] 11.1 Implement `ReportExporter.generate(startDate, endDate)` that builds a `#print-view` DOM section and calls `window.print()`
    - Include all cycle entries and symptom logs within the selected date range
    - Show error toast if export fails; retain all data on failure
    - _Requirements: 11.1, 11.2, 11.3_
  - [x]* 11.2 Write property test for report content completeness (Property 26)
    - **Property 26: Report Content Completeness**
    - **Validates: Requirements 11.1**
  - [x]* 11.3 Write property test for data retained after failed export (Property 27)
    - **Property 27: Data Retained After Failed Export**
    - **Validates: Requirements 11.3**

- [x] 12. Final integration and wiring
  - [x] 12.1 Wire all components together: DataStore → CyclePredictor → Dashboard, Calendar, Insights
    - Ensure predictor recalculates within 2 seconds of new period data saved
    - _Requirements: 1.5, 2.4_
  - [x] 12.2 Write unit test asserting bottom nav renders exactly 5 tabs: Home, Calendar, Log, Insights, Settings
    - _Requirements: 8.1_
  - [x] 12.3 Write unit test asserting onboarding has ≤5 slides
    - _Requirements: 8.4_
  - [x] 12.4 Final checkpoint — Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional property/unit tests and can be skipped for faster MVP
- All tasks are marked complete — implementation exists in `women-health-tracker.html`
- Each task references specific requirements for traceability
- Property tests use [fast-check](https://github.com/dubzzz/fast-check) with a minimum of 100 iterations each
- Each property test includes a comment tag: `// Feature: women-health-tracker, Property N: <property_text>`
