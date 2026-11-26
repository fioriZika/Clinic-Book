# System models — ClinicBook (Week 6)

**Author:** Dajana Bardhi  
**Scope:** doctor availability and booking — FR-02, FR-03, FR-04, FR-05, FR-09, plus the added FR-16 and FR-17 from Week 5. The remaining Option A use cases are drawn so the system boundary is complete for the team.

Built directly from [../USER_STORIES.md](../USER_STORIES.md) and [../REQUIREMENTS.md](../REQUIREMENTS.md).

## Diagrams

| Diagram | PlantUML source | Rendered |
| --- | --- | --- |
| Use case | [use-case.puml](use-case.puml) | [use-case.svg](use-case.svg) |
| Class | [class-diagram.puml](class-diagram.puml) | [class-diagram.svg](class-diagram.svg) |
| Sequence — see slots and book one | [sequence-book-slot.puml](sequence-book-slot.puml) | [sequence-book-slot.svg](sequence-book-slot.svg) |
| State — appointment lifecycle | [state-appointment.puml](state-appointment.puml) | [state-appointment.svg](state-appointment.svg) |

## Traceability

| Requirement | Use case | Class | Sequence | State |
| --- | --- | --- | --- | --- |
| FR-02 — free slots | UC-02 | `BookingService.listFreeSlots`, `TimeSlot`, `BlockedPeriod` | Step 1 | — |
| FR-03 — book a slot | UC-03 / UC-04 | `BookingService.book`, `BookingCommand`, `Appointment` | Step 2 | `[*] → Booked` |
| FR-04 — no double booking | UC-05 | `UNIQUE(doctorId, slot.start)` on `Appointment` | `findByDoctorAndStart` + unique violation on insert | note on `Booked` |
| FR-05 — max 2 future | UC-05 | `Patient.futureBookedCount` | `countFutureBooked` break | note on `Booked` |
| FR-09 — no-show rule | UC-05 | `Patient.noShowCount` | `countNoShows` break (patient only) | note on `NoShow` |
| FR-16 — reschedule | UC-06 | `BookingService.reschedule` | — | `Booked → Cancelled` + new `Booked` |
| FR-17 — no overlap | UC-05 | `findOverlappingForPatient` | overlap break | — |

## Modelling decisions

- Slots are generated at 08:00–15:40 in 20-minute steps, Monday to Friday; past, booked and blocked slots are excluded (FR-02, US-03).
- Only a `BOOKED` appointment occupies a slot and counts towards the two-appointment limit; `CANCELLED` releases it again (US-06).
- Booking rules are checked in the order given in `REQUIREMENTS.md`; the first failure wins and its message is the one shown to the user.
- The insert is protected by a partial unique constraint `UNIQUE(doctor_id, start_at) WHERE status = 'BOOKED'`, so when two people book the same slot only one succeeds.
- `Clock` is injected rather than reading the system time directly, so the 24-hour and 90-day rules can be tested repeatably.
- A receptionist may book for a patient with three recent no-shows; the same patient booking online is refused (US-16).
- `BookingService` and `AppointmentRepository` anticipate the Week 7 layering (routes → services → repositories).

## Rendering the sources

The SVGs are committed so they render on GitHub without any tooling. To regenerate them you need Java and Graphviz (`dot` on the PATH):

```bash
java -jar plantuml.jar -tsvg docs/models/*.puml
```

---

## Cancellation and doctor daily schedule models

**Contributor:** Fioralba Zika  
**Scope:** appointment cancellation and the doctor's daily schedule.

The availability and booking models in this folder are unchanged. The folder already uses `docs/models/class-diagram.puml` for that model, so the cancellation and daily-schedule class model uses `class-diagram-fioralba.puml`.

### Purpose and links

| Diagram | PlantUML source | Rendered SVG | Purpose |
| --- | --- | --- | --- |
| Domain class model | [class-diagram-fioralba.puml](class-diagram-fioralba.puml) | [class-diagram-fioralba.svg](class-diagram-fioralba.svg) | Models the domain concepts for booking, cancellation and the doctor's daily schedule. It separates all stored appointment history from the small subset of future active bookings. |
| Appointment state model | [appointment-state-diagram.puml](appointment-state-diagram.puml) | [appointment-state-diagram.svg](appointment-state-diagram.svg) | Models the confirmed appointment lifecycle and the different cancellation permissions for a patient and a receptionist. |

### How to view or render

- The committed SVG files can be opened directly in a browser or viewed on GitHub; no tooling is needed.
- Edit the `.puml` files in any text editor.
- With a local PlantUML install (Java plus Graphviz), regenerate both SVGs with:

```bash
java -jar plantuml.jar -tsvg docs/models/class-diagram-fioralba.puml docs/models/appointment-state-diagram.puml
```

- Without a local install, paste a `.puml` file into <https://www.plantuml.com/plantuml/> or use the `/svg/` service.

The committed SVGs were rendered with the PlantUML online server (1.2026.9beta4). The rendered output was checked for syntax errors, text labels and multiplicities. Local rendering with Java and Graphviz was not verified.

### Traceability

| Requirement / story | Priority / status | Where it appears in these models |
| --- | --- | --- |
| FR-02, US-03 - free 20-minute slots | Must, confirmed | `TimeSlot` class and slot note in the class diagram. |
| FR-03, US-04, US-05 - book a free slot | Must, confirmed | Initial `book` transition; `Patient`, `Appointment` and `Receptionist` classes. |
| FR-04, US-06 - no double booking | Must, confirmed | `TimeSlot` note and `Booked` note: at most one `BOOKED` appointment per doctor and start time. |
| FR-05, US-07 - at most 2 future appointments | Must, confirmed | `Patient.futureBookedCount` and the `Patient` note. The limit is a constraint on the future `BOOKED` subset, not the lifetime association multiplicity. |
| FR-06, US-08 - patient cancels own appointment | Must, confirmed | `Appointment.cancel` guard and the `patient cancels` transition: owner, future start and at least 24 hours before start. |
| FR-06, US-09 - receptionist cancels | Must, confirmed | `Appointment.cancel` guard and the `receptionist cancels` transition: future `BOOKED` appointment, no 24-hour limit. |
| FR-07, US-10 - doctor's daily schedule | Must, confirmed | `Doctor.viewDailySchedule`, `DailyScheduleEntry` and the `Doctor` note: non-cancelled entries, chronological order, time, patient name and status. |
| FR-09, US-16 - repeat no-shows | Should, confirmed but lower priority | `Patient.noShowsLast90Days` and the `Patient` note. No `No-show` state is drawn. |
| FR-17, US-19 - no overlapping patient bookings | Should, confirmed but lower priority | `Patient.hasOverlappingBooked` and the `Patient` note. |
| FR-16 - reschedule | Could, confirmed but not modelled here | Not included in these two diagrams; see open questions. |
| US-01 / patient registration | Referenced but not defined in shared docs | Not modelled; see open questions. |

### Confirmed requirements, modelling choices and unresolved questions

**Confirmed in the shared documents**

- Free slots are 20 minutes, 08:00-16:00, Monday to Friday, and are not offered when past, booked or blocked (FR-02, US-03).
- Booking creates a `BOOKED` appointment for a registered patient and must not double-book a doctor or exceed two future appointments (FR-03, FR-04, FR-05).
- Cancellation uses an explicit actor and the 24-hour rule; a successful cancellation changes the status to `CANCELLED` and releases the slot (FR-06, US-08, US-09).
- A doctor sees only their own non-cancelled appointments for a chosen day, ordered by start time (FR-07, US-10).
- Blocked periods and the no-show/overlap rules are mentioned in the booking specification (FR-09, FR-17).

**Modelling choices in these diagrams**

- `User` is an abstract superclass for `Patient`, `Receptionist` and `Doctor`. It expresses the acting-user role without adding service or repository classes.
- `DailyScheduleEntry` is marked `derived`: it is a view over appointments, not a new stored business record.
- The status enum contains only `BOOKED` and `CANCELLED`, the two statuses explicitly used by the shared cancellation and booking text.
- `Appointment.cancel(actor, now)` carries the permission guard. The state diagram shows the same guard as two separate cancellation events.
- All appointments are shown with a lifetime multiplicity of `0..*`. The "at most 2" rule is written as a constraint on the future `BOOKED` subset.
- `TimeSlot` and `BlockedPeriod` are modelled because the booking specification explicitly needs them. No clock, repository or service infrastructure is drawn.

**Unresolved questions for the team**

1. Who records `Attended` and `No-show`, and when? The shared documents do not define this. FR-08 and FR-12 are referenced in `state-appointment.puml` but are not defined in `REQUIREMENTS.md`, so no automatic attendance or no-show transition is drawn here.
2. Is rescheduling (FR-16, Could) in scope? If yes, does it create a new appointment and cancel the old one, or update the same record? `state-appointment.puml` shows one interpretation; these diagrams do not.
3. Patient registration is referenced as US-01 but is not written in the shared documents or present on `develop`. How should `registered` be defined?
4. The proposed acceptance rules and assumptions in `REQUIREMENTS.md` sections 7 and 10 still need team agreement: cancelled appointments not counting towards the limit, keeping cancelled records, exactly 24 hours being allowed, and using Europe/Tirane time.
5. Where do no-show records come from for FR-09, and can a doctor blocked period have a reason or an actor who creates it? These details are not in the shared documents.
6. The last acceptance criterion of US-10 is incomplete in `USER_STORIES.md` ("Given there are"). This wording needs completion before implementation.

### Consistency between the two diagrams

Both diagrams use the same two statuses, `BOOKED` and `CANCELLED`. Both say that only a future `BOOKED` appointment occupies a slot and counts towards the two-appointment limit. Both make cancellation an explicit event with a guard; neither treats the passage of time as a cancellation event. The state diagram is therefore a behaviour view of the same `Appointment` concept shown in the class diagram.

### Short presentation script

**Class diagram:** "The class diagram models three user types: patient, receptionist and doctor. A patient and a doctor each have many appointment records over time, but only the booked ones with a future start are active. The limit of two applies to that active subset, not to all history. Every appointment occupies one 20-minute slot, and a slot can have at most one booked appointment. A receptionist may cancel as well as a patient. The doctor's daily list is a derived view that leaves out cancelled appointments."

**State diagram:** "A new appointment starts in Booked. There are two explicit ways to cancel it. The patient can cancel only their own appointment and only at least 24 hours before it starts. A receptionist can cancel any future booked appointment, including within 24 hours. Passing time does not cancel anything. Cancelled is the end of the modelled lifecycle, but the record is kept and the slot is free again. No automatic Attended or No-show transitions are modelled because the shared requirements do not say who records them or when."
