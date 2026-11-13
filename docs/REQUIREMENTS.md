Requirements: Doctor availability and booking (Dajana Bardhi)


ClinicBook is a web application for a small family clinic in Tirana with four doctors. Patients book and cancel appointments online, receptionists manage the diary, and doctors see their own schedule. This part covers seeing free slots and booking appointments. IDs match the team's full requirements list.


1. Functional requirements
1.1 From the brief
ID	Requirement	Priority
FR-02	The system shall show each doctor's free 20-minute slots between 08:00 and 16:00, Monday to Friday.	Must
FR-03	A patient or a receptionist shall be able to book a free slot for a registered patient.	Must
FR-04	The system shall not allow two appointments for the same doctor in the same slot.	Must
FR-05	A patient shall not hold more than 2 future appointments at the same time.	Must
FR-09	A patient with 3 no-shows in the last 90 days shall be able to book only through a receptionist.	Should


1.2 Added 
ID	Requirement	Priority	Rationale
FR-16	A patient shall be able to reschedule an appointment to another free slot in one step.	Could	Avoids losing the old slot if the new booking fails.
FR-17	The system shall not allow a patient to hold two booked appointments at overlapping times.	Should	Stops a patient holding slots they cannot attend.


2. Non-functional requirements
ID	Type	Requirement
NFR-01	Product · efficiency	A doctor's free slots for one week shall appear in under 2 seconds with 5,000 appointments stored.
NFR-02	Product · usability	After 10 minutes of training, a receptionist shall be able to book a phone appointment in under 60 seconds.


3. Structured specification – Book an appointment (FR-03, FR-04, FR-05)
Field	Content
Function	Book an appointment
Description	Creates an appointment for a registered patient with a doctor in one free 20-minute slot.
Inputs · source	Doctor, slot start time, patient and acting user, from the booking screen; the current time from the clock.
Outputs · destination	The new appointment (status booked) or an error with the reason; shown on screen and stored in the database.
Information needed	The patient record; the acting user's role; the doctor's existing appointments and blocked periods; the patient's future appointments and no-shows.
Action	Check the pre-conditions in order and stop at the first failure. If all hold, create the appointment with status booked in one database transaction, protected by a unique (doctor, slot start) constraint.
Pre-conditions	The slot is inside clinic hours, Monday to Friday, and in the future. The slot is not in a period blocked by the doctor. The doctor has no booked appointment at that time (FR-04). The patient has fewer than 2 future appointments (FR-05) and no overlapping appointment (FR-17). If a patient is booking online, they have fewer than 3 no-shows in the last 90 days (FR-09).
Post-conditions	One new booked appointment exists for that doctor and slot; the slot is no longer free; the patient has one more future appointment. If the request is refused, nothing changes.
Side effects	None.
Errors	Slot taken → "slot no longer available" · limit reached → "maximum of 2 future appointments" · no-show rule → "please call reception" · overlap → "you already have an appointment at this time" · blocked or invalid slot → "slot not available".


4. Scenario – A receptionist books a phone appointment
Part	Content
Starting situation	Mrs. Elira Hoxha phones the clinic. Receptionist Arta is logged in. Elira is a registered patient with one future appointment. Dr. Berisha has a free slot on Thursday at 10:20.
Normal flow	1. Arta finds Elira's record. 2. Arta chooses Dr. Berisha and Thursday; the system lists the free slots (FR-02). 3. Elira picks 10:20; Arta selects it and confirms. 4. The system checks the rules and creates the appointment (FR-03, FR-05). 5. Arta reads the confirmation (doctor, date, time) to Elira.
What can go wrong	The slot was taken a second ago → "slot no longer available", Arta offers another slot (FR-04). Elira already has two future appointments → refused, and she may cancel one first (FR-05). Elira is not registered → Arta registers her first. The doctor has blocked the day → the slot is not offered.
Other activities	A patient may be looking at the same slot online at the same moment.
Finishing state	One new booked appointment exists (Dr. Berisha, Thursday 10:20, Elira); the slot is no longer free; Elira holds 2 future appointments.


5. Assumptions 
The limit of 2 future appointments (FR-05) also applies when a receptionist books.
A "future appointment" is one with status booked whose start is later than now.
