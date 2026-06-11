# 📊 Project Database Design

This document describes the initial database design for the education platform supporting both individual and group lessons.

---

# 1. Main Entities

## Users

Represents all system users (students, instructors, admins).

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    role VARCHAR(20) NOT NULL CHECK (role IN ('student', 'instructor', 'admin')),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Instructors

Stores instructor-specific profile data.

```sql
CREATE TABLE instructors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID UNIQUE NOT NULL,
    bio TEXT,
    specialization VARCHAR(255),
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'inactive', 'pending')),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_instructor_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
);
```

---

## Availability Slots

Represents instructor availability schedule.

```sql
CREATE TABLE availability_slots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    instructor_id UUID NOT NULL,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'available' CHECK (status IN ('available', 'booked', 'unavailable')),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_slot_instructor
        FOREIGN KEY (instructor_id)
        REFERENCES instructors(id)
        ON DELETE CASCADE
);
```

---

## Lessons

Represents scheduled lessons (supports both individual and group lessons).

```sql
CREATE TABLE lessons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    instructor_id UUID NOT NULL,
    slot_id UUID UNIQUE,
    lesson_type VARCHAR(20) NOT NULL CHECK (lesson_type IN ('individual', 'group')),
    max_participants INT DEFAULT 1,
    status VARCHAR(20) DEFAULT 'scheduled' CHECK (status IN ('scheduled', 'completed', 'cancelled')),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_lesson_instructor
        FOREIGN KEY (instructor_id)
        REFERENCES instructors(id)
        ON DELETE CASCADE,
    CONSTRAINT fk_lesson_slot
        FOREIGN KEY (slot_id)
        REFERENCES availability_slots(id)
        ON DELETE SET NULL
);
```

---

## Bookings

Stores student participation in lessons.

```sql
CREATE TABLE bookings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    lesson_id UUID NOT NULL,
    student_id UUID NOT NULL,
    status VARCHAR(20) DEFAULT 'booked' CHECK (status IN ('booked', 'cancelled', 'rescheduled', 'completed')),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_booking_lesson
        FOREIGN KEY (lesson_id)
        REFERENCES lessons(id)
        ON DELETE CASCADE,
    CONSTRAINT fk_booking_student
        FOREIGN KEY (student_id)
        REFERENCES users(id)
        ON DELETE CASCADE
);
```

---

## Notifications

Stores system notifications for users.

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    type VARCHAR(50) NOT NULL,
    message TEXT NOT NULL,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT fk_notification_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
);
```

---

# 2. Relationships

- Users → Instructors (1 : 0..1)
- Instructors → Availability Slots (1 : M)
- Availability Slots → Lessons (1 : 1)
- Instructors → Lessons (1 : M)
- Lessons → Bookings (1 : M)
- Users (Students) → Bookings (1 : M)
- Users → Notifications (1 : M)

---

# 3. Business Assumptions

- System supports both individual and group lessons
- Group lessons allow multiple students per lesson
- Individual lessons are limited to one participant
- Each availability slot can be converted into a lesson
- Users are differentiated by role field
- Notifications are stored for history tracking

---

