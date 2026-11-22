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
