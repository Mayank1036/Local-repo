# Database ER Diagram - Web-MedConsult

## 📊 Entity-Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         WEB-MEDCONSULT DATABASE SCHEMA                      │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────┐
│      PATIENTS        │
├──────────────────────┤
│ PK: id (INTEGER)     │
│────────────────────  │
│ name (TEXT)          │
│ email (TEXT) UNIQUE  │
│ password (TEXT)      │
│ phone (TEXT)         │
│ age (INTEGER)        │
│ gender (TEXT)        │
│ address (TEXT)       │
│ created_at (TEXT)    │
└──────────────────────┘
          │
          │ 1
          │
          │ registers/books
          │
          │ *
          ▼
┌──────────────────────┐         ┌──────────────────────┐
│    APPOINTMENTS      │    *    │      SESSIONS        │
├──────────────────────┤ ◄────── ├──────────────────────┤
│ PK: id (INTEGER)     │ belongs │ PK: id (INTEGER)     │
│──────────────────────│   to    │──────────────────────│
│ FK: patient_id       │──┐      │ FK: doctor_id        │
│ FK: doctor_id        │  │      │ session_date (TEXT)  │
│ FK: session_id       │──┘      │ start_time (TEXT)    │
│ appointment_date     │         │ end_time (TEXT)      │
│ appointment_time     │         │ max_bookings (INT)   │
│ notes (TEXT)         │         │ created_at (TEXT)    │
│ status (TEXT)        │         └──────────────────────┘
│ created_at (TEXT)    │                   ▲
└──────────────────────┘                   │
          │                                │ 1
          │ *                              │
          │                                │ creates
          │                                │
          │                                │ *
          │ booked_with                    │
          │                                │
          │ *                              │
          ▼                                │
┌──────────────────────┐                  │
│       DOCTORS        │──────────────────┘
├──────────────────────┤
│ PK: id (INTEGER)     │
│──────────────────────│
│ name (TEXT)          │
│ specialty (TEXT)     │
│ qualification (TEXT) │
│ experience (INTEGER) │
│ email (TEXT) UNIQUE  │
│ phone (TEXT)         │
│ rating (REAL)        │
│ available (BOOLEAN)  │
│ created_at (TEXT)    │
└──────────────────────┘
```

## 🗂️ Database Tables

### 1. **DOCTORS** 👨‍⚕️
Stores information about medical professionals in the system.

| Column         | Type    | Constraints | Description                        |
|----------------|---------|-------------|------------------------------------|
| id             | INTEGER | PRIMARY KEY | Unique doctor identifier           |
| name           | TEXT    | NOT NULL    | Doctor's full name                 |
| specialty      | TEXT    | NOT NULL    | Medical specialty                  |
| qualification  | TEXT    |             | Educational qualifications         |
| experience     | INTEGER |             | Years of experience                |
| email          | TEXT    | UNIQUE      | Contact email                      |
| phone          | TEXT    |             | Contact phone number               |
| rating         | REAL    | DEFAULT 4.5 | Doctor rating (1-5 scale)         |
| available      | BOOLEAN | DEFAULT 1   | Availability status (1=Yes, 0=No) |
| created_at     | TEXT    |             | Record creation timestamp          |

**Sample Values:**
- Specialties: Cardiology, Pediatrics, Dermatology, Orthopedics, Neurology
- Rating: 4.5 - 5.0
- Experience: 8-20 years

---

### 2. **PATIENTS** 🧑‍🤝‍🧑
Stores registered patient information and authentication details.

| Column       | Type    | Constraints | Description                    |
|--------------|---------|-------------|--------------------------------|
| id           | INTEGER | PRIMARY KEY | Unique patient identifier      |
| name         | TEXT    | NOT NULL    | Patient's full name            |
| email        | TEXT    | UNIQUE, NOT NULL | Login email (username) |
| password     | TEXT    | NOT NULL    | Hashed password (secure)       |
| phone        | TEXT    |             | Contact phone number           |
| age          | INTEGER |             | Patient age                    |
| gender       | TEXT    |             | Gender (Male/Female/Other)     |
| address      | TEXT    |             | Residential address            |
| created_at   | TEXT    |             | Registration timestamp         |

**Security Features:**
- Password: Hashed using Werkzeug's `generate_password_hash()`
- Email: Unique constraint prevents duplicate registrations
- No plain-text password storage

---

### 3. **SESSIONS** 📅
Represents doctor availability time slots for appointments.

| Column        | Type    | Constraints | Description                        |
|---------------|---------|-------------|------------------------------------|
| id            | INTEGER | PRIMARY KEY | Unique session identifier          |
| doctor_id     | INTEGER | FOREIGN KEY | References doctors(id)             |
| session_date  | TEXT    | NOT NULL    | Date of session (YYYY-MM-DD)      |
| start_time    | TEXT    | NOT NULL    | Session start time (HH:MM)        |
| end_time      | TEXT    | NOT NULL    | Session end time (HH:MM)          |
| max_bookings  | INTEGER | DEFAULT 5   | Maximum appointments allowed       |
| created_at    | TEXT    |             | Record creation timestamp          |

**Typical Values:**
- Duration: 1-3 hours per session
- Time Slots: Morning (09:00-12:00), Afternoon (14:00-17:00), Evening (18:00-21:00)
- Max Bookings: Usually 3-10 patients per session

**Business Rules:**
- Each session belongs to exactly one doctor
- Multiple patients can book the same session (up to max_bookings)
- Sessions can be filtered by date, doctor, and availability

---

### 4. **APPOINTMENTS** 🗓️
Records patient bookings for specific doctor sessions.

| Column            | Type    | Constraints | Description                        |
|-------------------|---------|-------------|------------------------------------|
| id                | INTEGER | PRIMARY KEY | Unique appointment identifier      |
| patient_id        | INTEGER | FOREIGN KEY | References patients(id)            |
| doctor_id         | INTEGER | FOREIGN KEY | References doctors(id)             |
| session_id        | INTEGER | FOREIGN KEY | References sessions(id)            |
| appointment_date  | TEXT    | NOT NULL    | Appointment date                   |
| appointment_time  | TEXT    | NOT NULL    | Appointment time                   |
| notes             | TEXT    |             | Patient symptoms/reason for visit  |
| status            | TEXT    | DEFAULT 'scheduled' | Appointment status        |
| created_at        | TEXT    |             | Booking timestamp                  |

**Status Values:**
- `scheduled` - Appointment confirmed (default)
- `completed` - Patient visited doctor
- `cancelled` - Appointment cancelled

**Business Rules:**
- Patient can only book one appointment per session (duplicate prevention)
- Patient can book multiple different sessions
- Appointments can be cancelled by patients
- Session must have available slots (bookings < max_bookings)

---

## 🔗 Relationships

### 1. **DOCTORS ↔ SESSIONS** (One-to-Many)
- **Relationship Type:** 1:N
- **Description:** One doctor can create multiple availability sessions
- **Foreign Key:** `sessions.doctor_id` → `doctors.id`
- **Cascade:** Deleting a doctor deletes all their sessions
- **Example:** Dr. Smith has 10 sessions scheduled for the next 2 weeks

### 2. **PATIENTS ↔ APPOINTMENTS** (One-to-Many)
- **Relationship Type:** 1:N
- **Description:** One patient can book multiple appointments
- **Foreign Key:** `appointments.patient_id` → `patients.id`
- **Cascade:** Deleting a patient removes their appointment history
- **Example:** John Doe has 3 upcoming appointments with different doctors

### 3. **DOCTORS ↔ APPOINTMENTS** (One-to-Many)
- **Relationship Type:** 1:N
- **Description:** One doctor can have multiple appointments
- **Foreign Key:** `appointments.doctor_id` → `doctors.id`
- **Cascade:** Deleting a doctor cancels all their appointments
- **Example:** Dr. Smith has 15 appointments scheduled this month

### 4. **SESSIONS ↔ APPOINTMENTS** (One-to-Many)
- **Relationship Type:** 1:N
- **Description:** One session can have multiple appointments (up to max_bookings)
- **Foreign Key:** `appointments.session_id` → `sessions.id`
- **Cascade:** Deleting a session cancels all associated appointments
- **Example:** Morning session on Nov 10 has 5 appointments booked

---

## 🎯 Cardinality Summary

```
DOCTORS (1) ──────────────── (N) SESSIONS
   │
   │ (1)
   │
   └──────────────────────── (N) APPOINTMENTS
                                      │
                                      │ (N)
                                      │
                                     (1)
                                   PATIENTS

SESSIONS (1) ──────────────── (N) APPOINTMENTS
```

**Cardinality Notation:**
- **1** = One (mandatory)
- **N** = Many (zero or more)
- **0..1** = Optional relationship
- **1..N** = At least one

---

## 📈 Sample Data Statistics

| Table        | Sample Records | Purpose                          |
|--------------|----------------|----------------------------------|
| doctors      | 5              | Multiple specialties available   |
| patients     | 0+ (dynamic)   | Created during registration      |
| sessions     | 42             | 7-14 days of availability        |
| appointments | 0+ (dynamic)   | Created when patients book       |

---

## 🔍 Common Queries

### 1. **Get all appointments for a patient**
```sql
SELECT a.*, d.name as doctor_name, d.specialty, s.session_date, s.start_time
FROM appointments a
JOIN doctors d ON a.doctor_id = d.id
JOIN sessions s ON a.session_id = s.id
WHERE a.patient_id = ?
ORDER BY a.appointment_date DESC
```

### 2. **Get available sessions for a doctor**
```sql
SELECT s.*, 
       (SELECT COUNT(*) FROM appointments WHERE session_id = s.id) as bookings
FROM sessions s
WHERE s.doctor_id = ?
HAVING bookings < s.max_bookings
ORDER BY s.session_date, s.start_time
```

### 3. **Get doctors by specialty**
```sql
SELECT * FROM doctors
WHERE specialty = ? AND available = 1
ORDER BY rating DESC
```

### 4. **Check session availability**
```sql
SELECT s.*, 
       (s.max_bookings - COUNT(a.id)) as available_slots
FROM sessions s
LEFT JOIN appointments a ON s.id = a.session_id
WHERE s.id = ?
GROUP BY s.id
```

---

## 🛡️ Data Integrity Constraints

### Primary Keys
- Every table has an auto-incrementing `id` as PRIMARY KEY
- Ensures unique identification of each record

### Foreign Keys
- `sessions.doctor_id` → `doctors.id`
- `appointments.patient_id` → `patients.id`
- `appointments.doctor_id` → `doctors.id`
- `appointments.session_id` → `sessions.id`

### Unique Constraints
- `doctors.email` - No duplicate doctor emails
- `patients.email` - No duplicate patient emails (used for login)

### Default Values
- `doctors.rating` = 4.5
- `doctors.available` = 1 (TRUE)
- `sessions.max_bookings` = 5
- `appointments.status` = 'scheduled'

### NOT NULL Constraints
- Critical fields like name, email, passwords cannot be NULL
- Ensures data completeness

---

## 🔐 Security Considerations

1. **Password Storage**
   - Passwords are hashed using Werkzeug's `pbkdf2:sha256`
   - Never stored in plain text
   - Salt automatically generated

2. **Email Privacy**
   - Patient emails used only for authentication
   - Not shared with doctors or other patients

3. **Session Management**
   - Flask session stores only patient_id and patient_name
   - No sensitive data in browser cookies

4. **Appointment Privacy**
   - Patients can only view/cancel their own appointments
   - Doctors can see appointments for their sessions only
   - Admin has full access for management

---

## 📊 Database Size Estimates

| Table        | Avg Row Size | 100 Users  | 1000 Users | 10000 Users |
|--------------|--------------|------------|------------|-------------|
| doctors      | ~300 bytes   | 1.5 KB     | 15 KB      | 150 KB      |
| patients     | ~250 bytes   | 25 KB      | 250 KB     | 2.5 MB      |
| sessions     | ~150 bytes   | 6.3 KB     | 63 KB      | 630 KB      |
| appointments | ~200 bytes   | 40 KB      | 400 KB     | 4 MB        |
| **TOTAL**    | -            | **~73 KB** | **~728 KB**| **~7.3 MB** |

*Note: Estimates assume 2 appointments/patient, 42 sessions/5 doctors*

---

## 🚀 Database Initialization

The database is automatically created when you run `app.py` with sample data:
- ✅ 5 doctors (different specialties)
- ✅ 42 sessions (spread across 7-14 days)
- ✅ 0 patients (created during registration)
- ✅ 0 appointments (created when patients book)

**File Location:** `appointments.db` (SQLite database)

---

## 📝 Schema Version

- **Version:** 1.0
- **Created:** November 2025
- **Database Engine:** SQLite 3
- **Character Encoding:** UTF-8
- **Date Format:** ISO 8601 (YYYY-MM-DD)
- **Time Format:** 24-hour (HH:MM)

---

## 🔄 Future Enhancements

Potential schema improvements:
1. Add `reviews` table for patient feedback
2. Add `prescriptions` table linked to appointments
3. Add `medical_history` table for patient records
4. Add `notifications` table for appointment reminders
5. Add `payments` table for billing
6. Add `specializations` table (normalize specialty field)
7. Add indexing on frequently queried columns (email, dates)

---

## 📞 Database Management

### Backup
```bash
# Backup database
copy appointments.db appointments_backup.db
```

### Reset Database
```bash
# Delete and recreate
del appointments.db
python app.py
```

### View Database
```bash
# Using SQLite CLI
sqlite3 appointments.db
.tables
.schema
```

---

**Created for:** Web-MedConsult Doctor Appointment System  
**Database File:** `appointments.db`  
**ORM:** Raw SQLite3 (no ORM framework)  
**Migration Tool:** None (schema in app.py)
