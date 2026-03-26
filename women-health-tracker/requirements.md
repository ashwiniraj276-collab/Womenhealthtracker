# Requirements Document

## Introduction

Women Health Tracker is a mobile application designed to help women aged 15–45 track menstrual cycles, symptoms, and overall health. The app provides cycle predictions, symptom logging, health reminders, and insights through a clean, beginner-friendly interface. User data is stored securely with optional biometric or passcode protection.

## Glossary

- **App**: The Women Health Tracker mobile application
- **User**: A woman aged 15–45 using the App to track her health
- **Cycle**: A menstrual cycle from the first day of one period to the first day of the next
- **Period**: The menstruation phase of the Cycle
- **Fertile_Window**: The days within a Cycle when conception is most likely
- **Ovulation**: The phase of the Cycle when an egg is released, typically mid-cycle
- **Luteal_Phase**: The phase of the Cycle between Ovulation and the next Period
- **Symptom**: A health indicator logged by the User (e.g., cramps, mood, fatigue, headache)
- **Reminder**: A push notification sent to the User at a configured time
- **Dashboard**: The home screen displaying the User's current cycle status and insights
- **Log**: The screen where the User records period dates and Symptoms
- **Cycle_Predictor**: The component that calculates predicted period and Fertile_Window dates
- **Data_Store**: The local encrypted database storing all User data
- **Auth_Guard**: The component enforcing passcode or biometric access control

---

## Requirements

### Requirement 1: Period Logging

**User Story:** As a User, I want to log my period start and end dates, so that the App can track my cycle history accurately.

#### Acceptance Criteria

1. WHEN the User submits a period start date, THE Log SHALL record the date in the Data_Store.
2. WHEN the User submits a period end date, THE Log SHALL record the date and associate it with the corresponding period start date in the Data_Store.
3. IF the User submits an end date that is earlier than the start date, THEN THE Log SHALL display a validation error and reject the entry.
4. THE Log SHALL allow the User to edit or delete a previously logged period entry.
5. WHEN a period entry is saved, THE Cycle_Predictor SHALL recalculate predictions based on the updated history.

---

### Requirement 2: Cycle Prediction

**User Story:** As a User, I want the App to predict my next period and fertile window, so that I can plan ahead.

#### Acceptance Criteria

1. WHEN the User has logged at least one complete Cycle, THE Cycle_Predictor SHALL calculate and display the predicted next period start date.
2. WHEN the User has logged at least three complete Cycles, THE Cycle_Predictor SHALL calculate the average cycle length and use it for predictions.
3. WHEN the User has logged at least one complete Cycle, THE Cycle_Predictor SHALL calculate and display the predicted Fertile_Window dates.
4. THE Cycle_Predictor SHALL update all predictions within 2 seconds of new period data being saved.
5. IF the User has fewer than one complete Cycle logged, THEN THE Dashboard SHALL display a message indicating that more data is needed for predictions.

---

### Requirement 3: Calendar View

**User Story:** As a User, I want to see my cycle history on a calendar, so that I can visualize past and upcoming cycle phases.

#### Acceptance Criteria

1. THE App SHALL display a monthly calendar view showing logged period days, predicted period days, and predicted Fertile_Window days using distinct color coding.
2. WHEN the User taps a date on the calendar, THE App SHALL display any symptoms logged for that date.
3. WHEN the User navigates to a future month, THE App SHALL display predicted cycle phase dates based on Cycle_Predictor output.
4. THE App SHALL allow the User to navigate between months using forward and back controls.

---

### Requirement 4: Symptom Logging

**User Story:** As a User, I want to log daily symptoms, so that I can monitor how I feel throughout my cycle.

#### Acceptance Criteria

1. WHEN the User opens the Log screen, THE Log SHALL present a list of default symptom options including cramps, mood, fatigue, and headache.
2. WHEN the User selects one or more symptoms and saves, THE Log SHALL store the symptoms with the current date in the Data_Store.
3. THE Log SHALL allow the User to add custom symptom labels beyond the default list.
4. WHEN the User has logged symptoms for at least 7 days, THE App SHALL display symptom trends grouped by cycle phase.
5. THE Log SHALL allow the User to update or remove symptoms logged for the current day.

---

### Requirement 5: Health Reminders

**User Story:** As a User, I want to receive reminders for my period, ovulation, and medications, so that I stay informed and on schedule.

#### Acceptance Criteria

1. WHEN the Cycle_Predictor has a predicted period start date, THE App SHALL send a Reminder to the User at a configurable number of days before the predicted date.
2. WHEN the Cycle_Predictor has a predicted Ovulation date, THE App SHALL send a Reminder to the User at a configurable number of days before the predicted date.
3. THE App SHALL allow the User to configure Reminders for medication or supplement intake at a User-specified time of day.
4. THE App SHALL allow the User to enable or disable each Reminder type independently.
5. WHEN a Reminder is triggered, THE App SHALL deliver a push notification with a descriptive message identifying the Reminder type.
6. IF the User has not granted notification permissions, THEN THE App SHALL prompt the User to enable notifications and explain why they are needed.

---

### Requirement 6: Dashboard

**User Story:** As a User, I want a dashboard showing my current cycle status and insights, so that I can quickly understand where I am in my cycle.

#### Acceptance Criteria

1. THE Dashboard SHALL display the User's current cycle phase (Period, Fertile_Window, Ovulation, or Luteal_Phase).
2. THE Dashboard SHALL display the number of days until the next predicted period.
3. WHEN the User has logged at least three Cycles, THE Dashboard SHALL display the User's average cycle length in days.
4. THE Dashboard SHALL display at least one contextual health tip relevant to the User's current cycle phase.
5. THE Dashboard SHALL refresh its displayed data within 1 second of the User opening the App.

---

### Requirement 7: Data Privacy and Security

**User Story:** As a User, I want my health data to be private and secure, so that I can trust the App with sensitive information.

#### Acceptance Criteria

1. THE Data_Store SHALL encrypt all User health data at rest using AES-256 encryption.
2. WHERE the User enables the passcode lock, THE Auth_Guard SHALL require the User to enter the correct passcode before granting access to the App.
3. WHERE the User enables biometric lock, THE Auth_Guard SHALL require successful biometric authentication before granting access to the App.
4. IF authentication fails 5 consecutive times, THEN THE Auth_Guard SHALL lock the App for 30 seconds before allowing further attempts.
5. THE App SHALL not transmit User health data to any external server without explicit User consent.
6. THE App SHALL provide a data deletion option that permanently removes all User data from the Data_Store.

---

### Requirement 8: Navigation and Usability

**User Story:** As a User, I want simple and intuitive navigation, so that I can use the App without a learning curve.

#### Acceptance Criteria

1. THE App SHALL provide a bottom navigation bar with five tabs: Home, Calendar, Log, Insights, and Settings.
2. WHEN the User taps a bottom navigation tab, THE App SHALL navigate to the corresponding screen within 300ms.
3. THE App SHALL use a soft color palette (pink, lavender, and pastel tones) consistently across all screens.
4. THE App SHALL display an onboarding flow of no more than 5 screens for first-time Users covering core features.
5. WHEN the User completes onboarding, THE App SHALL not display the onboarding flow again on subsequent launches.

---

### Requirement 9: Insights Screen

**User Story:** As a User, I want to view trends and patterns in my health data, so that I can better understand my body over time.

#### Acceptance Criteria

1. WHEN the User has logged symptoms for at least 14 days, THE App SHALL display a symptom frequency chart grouped by cycle phase.
2. THE App SHALL display cycle length history as a chart showing the last 6 Cycles.
3. WHEN the User has logged at least 3 Cycles, THE App SHALL display the average period duration in days.
4. THE App SHALL update Insights data each time the User navigates to the Insights screen.

---

### Requirement 10: Pregnancy Mode (Optional)

**User Story:** As a User, I want to switch to a pregnancy mode, so that I can track pregnancy milestones instead of cycle data.

#### Acceptance Criteria

1. WHERE the User enables Pregnancy Mode, THE App SHALL replace cycle tracking views with a pregnancy week tracker.
2. WHERE the User enables Pregnancy Mode, THE App SHALL display weekly pregnancy milestones and tips.
3. WHERE the User disables Pregnancy Mode, THE App SHALL restore all standard cycle tracking views and data.

---

### Requirement 11: Health Report Export (Optional)

**User Story:** As a User, I want to export my health data, so that I can share it with a healthcare provider.

#### Acceptance Criteria

1. WHERE the User requests a health report export, THE App SHALL generate a PDF report containing cycle history and symptom logs for a User-selected date range.
2. WHEN the export is complete, THE App SHALL allow the User to share the PDF via the device's native share sheet.
3. IF the export fails, THEN THE App SHALL display an error message and retain all User data in the Data_Store.
