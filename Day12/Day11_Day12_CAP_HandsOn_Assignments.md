# Day 11-12: Hands-On CAP Coding Assignments (4 Assignments)

## Duration: 1.5-2 Hours Each | Individual or Pair Programming

---

## Assignment 1: Hospital Management System

### Problem Statement

Build a CAP application for "MediCare Hospital" that manages doctors, patients, appointments, and departments.

### Entities to Design

| Entity | Key | Important Fields |
|--------|-----|-----------------|
| Departments | UUID | name, floor, head (doctor name), capacity, phone, isActive |
| Doctors | UUID | firstName, lastName, specialization, qualification, experience (years), fee, departmentId, phone, email, availableDays (e.g., "Mon,Tue,Wed"), isActive |
| Patients | UUID | firstName, lastName, dateOfBirth, gender, bloodGroup, phone, email, address, emergencyContact, registrationDate |
| Appointments | UUID | patientId, doctorId, appointmentDate, appointmentTime, status (Scheduled/Completed/Cancelled/NoShow), reason, notes, fee |
| MedicalRecords | UUID | patientId, doctorId, appointmentId, diagnosis, prescription, testRecommended, recordDate |

### Requirements

1. Use namespace: `hospital.management`
2. Define reusable types: `Name`, `Phone`, `Email`, `Amount`, `MedicalNote`
3. Create enums: `Gender (Male/Female/Other)`, `BloodGroup (A+/A-/B+/B-/O+/O-/AB+/AB-)`, `AppointmentStatus`
4. Use UUID keys for all entities
5. Add default values: appointment status = "Scheduled", isActive = true
6. Create CSV sample data: 4 departments, 6 doctors, 5 patients, 8 appointments
7. Expose all entities in a `HospitalService`
8. Deploy to SQLite and verify all tables

### Deliverables

- Working `cds watch` with all entities accessible
- `test.http` file with:
  - GET all doctors filtered by specialization
  - GET appointments for a specific date
  - POST a new patient
  - POST a new appointment
  - GET patients filtered by blood group
  - PATCH appointment status to "Completed"

---

## Assignment 2: University Course Registration System

### Problem Statement

Build a CAP application for "TechUniversity" that handles courses, students, professors, enrollments, and grades.

### Entities to Design

| Entity | Key | Important Fields |
|--------|-----|-----------------|
| Departments | code (String) | name, building, headProfessor, establishedYear |
| Professors | UUID | firstName, lastName, email, phone, departmentCode, designation (Assistant/Associate/Full/Distinguished), joinDate, salary, officeRoom |
| Courses | code (String, e.g. "CS101") | title, description, credits, maxStudents, currentEnrolled, semester (1-8), departmentCode, professorId, schedule (e.g. "MWF 10:00-11:00"), roomNumber, isActive |
| Students | UUID | rollNumber (String, unique), firstName, lastName, email, phone, dateOfBirth, admissionYear, currentSemester, cgpa, departmentCode, isActive |
| Enrollments | Composite (studentId + courseCode) | enrollmentDate, status (Enrolled/Dropped/Completed), grade (A/B/C/D/F/null), gradePoints, attendancePercent |
| Exams | UUID | courseCode, examType (Midterm/Final/Quiz/Assignment), date, maxMarks, weightagePercent |

### Requirements

1. Use namespace: `edu.university`
2. Reusable types: `Email`, `Phone`, `Percentage`, `GradePoint`, `Semester`
3. Enums: `Designation`, `EnrollmentStatus`, `Grade`, `ExamType`
4. Composite key for Enrollments (studentId + courseCode — a student enrolls in a course only once per semester)
5. Default values: enrollment status = "Enrolled", isActive = true, currentEnrolled = 0
6. CSV data: 3 departments, 5 professors, 6 courses, 8 students, 12 enrollments
7. Expose in service `UniversityService`
8. Deploy and verify

### Deliverables

- Working project with `cds watch`
- `test.http` file with:
  - GET courses filtered by department and semester
  - GET students with CGPA above 8.0
  - POST new student enrollment
  - GET all enrollments for a specific student
  - Filter courses where currentEnrolled < maxStudents (seats available)
  - PATCH enrollment to add grade

---

## Assignment 3: Fleet & Delivery Management System

### Problem Statement

Build a CAP application for "SpeedLogistics" — a delivery company managing vehicles, drivers, shipments, and routes.

### Entities to Design

| Entity | Key | Important Fields |
|--------|-----|-----------------|
| Vehicles | UUID | registrationNumber, type (Bike/Van/Truck/Trailer), make, model, year, capacity (kg), fuelType (Petrol/Diesel/Electric/CNG), status (Available/OnTrip/Maintenance/Retired), lastServiceDate, mileage, insuranceExpiry |
| Drivers | UUID | name, licenseNumber, licenseExpiry, phone, email, experience (years), rating (1-5), status (Available/OnTrip/OnLeave), vehicleId, joinDate |
| Customers | UUID | name, company, phone, email, address, city, pincode, gstNumber, tier (Regular/Premium/Enterprise) |
| Shipments | UUID | trackingNumber (String, e.g. "SL-2026-00001"), customerId, driverId, vehicleId, pickupAddress, deliveryAddress, pickupCity, deliveryCity, weight, status (Booked/PickedUp/InTransit/OutForDelivery/Delivered/Failed), bookedAt, pickedUpAt, deliveredAt, estimatedDelivery, actualDistance, charges, paymentStatus (Pending/Paid/COD) |
| Routes | UUID | fromCity, toCity, distance (km), estimatedHours, tollCharges, isActive |
| ServiceRecords | UUID | vehicleId, serviceDate, serviceType (Regular/Repair/Accident), cost, description, nextServiceDate |

### Requirements

1. Namespace: `logistics.fleet`
2. Reusable types: `Phone`, `Email`, `Amount`, `Distance`, `City`, `Address`, `Rating`
3. Enums: `VehicleType`, `FuelType`, `VehicleStatus`, `DriverStatus`, `ShipmentStatus`, `PaymentStatus`, `CustomerTier`
4. Generate tracking numbers in a pattern (hint: use default or handle in service later)
5. Default values: shipment status = "Booked", payment = "Pending", vehicle status = "Available"
6. CSV data: 5 vehicles, 4 drivers, 4 customers, 8 shipments, 5 routes
7. Expose in `LogisticsService`
8. Deploy and verify

### Deliverables

- Working project with `cds watch`
- `test.http` with:
  - GET available vehicles (status = Available)
  - GET shipments filtered by status "InTransit"
  - GET all shipments for a specific customer
  - POST a new shipment booking
  - PATCH shipment status to "Delivered" and set deliveredAt
  - GET routes between two cities
  - GET drivers sorted by rating descending

---

## Assignment 4: Event Management Platform

### Problem Statement

Build a CAP application for "EventHub" — a platform that manages events, venues, speakers, registrations, and tickets.

### Entities to Design

| Entity | Key | Important Fields |
|--------|-----|-----------------|
| Venues | UUID | name, address, city, capacity, type (Auditorium/ConferenceHall/Outdoor/Virtual), amenities (String — comma-separated), hourlyRate, contactPerson, phone, isActive |
| Events | UUID | title, description, eventType (Conference/Workshop/Seminar/Webinar/Meetup), venueId, startDate, endDate, startTime, endTime, maxAttendees, registeredCount, ticketPrice, status (Draft/Published/Ongoing/Completed/Cancelled), organizerName, organizerEmail, tags (String — comma-separated) |
| Speakers | UUID | name, email, phone, bio, company, designation, expertise (String), photoUrl, rating, totalTalks, isActive |
| EventSpeakers | Composite (eventId + speakerId) | topic, sessionTime, sessionDuration (minutes), roomNumber |
| Registrations | UUID | eventId, attendeeName, attendeeEmail, attendeePhone, company, ticketType (General/VIP/Student), registrationDate, status (Confirmed/Cancelled/Waitlisted/Attended), amountPaid, paymentId |
| Feedback | UUID | eventId, attendeeEmail, overallRating (1-5), contentRating (1-5), venueRating (1-5), speakerRating (1-5), comment, submittedAt |

### Requirements

1. Namespace: `platform.events`
2. Reusable types: `Email`, `Phone`, `Amount`, `Rating`, `Name`, `URL`
3. Enums: `EventType`, `EventStatus`, `TicketType`, `RegistrationStatus`, `VenueType`
4. Composite key for EventSpeakers (one speaker per event assignment)
5. Default values: event status = "Draft", registration status = "Confirmed", registeredCount = 0
6. CSV data: 3 venues, 4 events, 5 speakers, 6 event-speaker mappings, 10 registrations
7. Expose in `EventService`
8. Deploy and verify

### Deliverables

- Working project with `cds watch`
- `test.http` with:
  - GET published events sorted by startDate
  - GET events filtered by type "Workshop"
  - GET all registrations for a specific event
  - POST a new registration
  - GET speakers sorted by rating desc
  - GET events with available seats (registeredCount < maxAttendees)
  - PATCH event status from "Draft" to "Published"
  - POST feedback for an event

---

## Grading Rubric (Same for All 4 Assignments)

| Criteria | Marks |
|----------|-------|
| All entities defined correctly with appropriate types | 3 |
| Reusable types and enums used | 2 |
| Namespace, naming conventions, defaults applied | 1 |
| CSV data loads without errors | 1 |
| Service exposes all entities | 1 |
| `cds watch` runs clean (no errors) | 1 |
| `test.http` with all required queries working | 1 |
| **Total** | **10** |

---

## Submission Checklist

For each assignment:
- [ ] `cds watch` starts without errors
- [ ] All entities visible at `http://localhost:4004`
- [ ] CSV data is loaded (verify with GET requests)
- [ ] `test.http` included with all required queries
- [ ] OData queries work ($filter, $orderby, $select, $top)
- [ ] `cds deploy --to sqlite` succeeds
- [ ] `.gitignore` includes node_modules and *.sqlite
