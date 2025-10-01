# Functional Requirements – USU Student App Subsystem

### FR-1: Student Registration
- The app must provide a registration page for students to join their university-specific union.
- Validate registrations automatically through the university IT system.
- Assign a unique USU membership ID on successful registration.
- Store registration data securely on the backend.

### FR-2: Society Membership
- The app must allow studetns to search for societies from their union.
- Enable students to subscribe to societies to receive updates.
- Enable students to join, quit or unsubscribe societies.
- Display society information (name, theme, leadership).

### FR-3: Communication with Societies
- The app must allow students to send messages to societies they are members of.
- Display messages sent from societies like announcements and updates.
- Support group broadcasts to all society members.

### FR-4: Creating a new Societies
- The app must allow students to submit applications to create new societies.
- Include fields for society name, theme, proposed leader, commitee members, and contact info.
- Submit applications to Union Officers subsystem for approval.

### FR-5: Events
- The app must allow students to search for university-specific and nationwide events.
- Display event details (name, theme, organisers, date/time, venue, and participation constraints).
- Enable students to subscribe to events, register and purchase tickets.
- Show real-time updates for event changes (venue, time, status).

### FR-6: Notifications
- The app must push notifications to students based on subscriptions and preferences.
- Notify students about new events, society updates, or changes in event details.

---

## Dependencies on Other Subsystems

| Dependency | Purpose |
|------------|---------|
| **Authentication & Membership Service (Backend)** | Validate student registrations, manage login, assign USU membership IDs. |
| **Society Management (Union Officers Web System?)** | Approve new societies and update society membership info. |
| **Event Management (Society Leaders?)** | Provide event creation, updates and promotion for student participation. |
| **Notification Service (Backend)** | Deliver real-time push notifications to students based on subscriptions. |
