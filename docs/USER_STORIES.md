Stories
Doctor availability and booking (Dajana)
US-03 – See free slots of a doctor · FR-02 · Must As a patient, I want to see a doctor's free 20-minute slots for a chosen day, so that I can pick a time that suits me.
Slots run 08:00–16:00 (first slot 08:00, last slot 15:40), Monday to Friday only.
Booked, blocked and past slots are not shown as free.
Given a Saturday or Sunday, then no slots are shown.
US-04 – Book a free slot · FR-03 · Must As a patient, I want to book a free slot with a doctor, so that I do not need to phone the clinic.
Given a free slot, when I book, then an appointment with status booked is created for me and I see a confirmation (doctor, date, time).
Given the slot was taken a moment earlier, when I book, then I get "slot no longer available" and nothing is created.
US-05 – Book a slot for a patient by phone · FR-03 · Must As a receptionist, I want to book a slot on behalf of a registered patient, so that phone callers are handled in under 60 seconds (NFR-02).
Given a registered patient and a free slot, when I book, then the appointment is created for that patient.
Given the patient is not registered, then I am offered to register first (US-01).
US-06 – Never get a double-booked slot · FR-04 · Must As a patient, I want to be sure that no one else holds my slot, so that I am not turned away at the clinic.
Given doctor D has an appointment at 10:00 on a day, when anyone tries to book D at 10:00 that day, then it is refused.
Given two bookings for the same slot arrive at the same time, then exactly one succeeds.
Given a booked slot is cancelled, then it becomes free again.
US-07 – Limit of 2 future appointments · FR-05 · Must As the clinic manager, I want each patient limited to 2 future appointments, so that slots are shared fairly.
Given a patient with 2 future booked appointments, when they (or reception) try to book a third, then it is refused with an explanation.
Given one of the two is cancelled, then a new booking is allowed.
US-16 – Restrict repeat no-shows · FR-09 · Should As the clinic manager, I want patients with 3 no-shows in the last 90 days to book only through reception, so that wasted slots are reduced.
Given 3 no-shows within the last 90 days, when the patient books online, then it is refused with "please call reception".
Given the same patient, when a receptionist books, then it is allowed.
Given the 3rd no-show is 91 days old, then online booking works again.
US-19 – No overlapping appointments for one patient · FR-17 · Should As a patient, I do not want to be booked with two doctors at the same time, so that I do not hold slots I cannot use.
Given I have an appointment 10:00–10:20, when I try to book another doctor at 10:00, then it is refused.
