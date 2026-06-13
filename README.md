# Database Design

This document describes the database structure for the Education Platform, supporting individual and group lessons, Magic Link authentication, and administrator approval workflows.

---

# Main Entities

## Users

Stores all platform users, including students, instructors, and administrators.

### Authentication

* Administrators authenticate using email and password.
* Students and instructors authenticate using Magic Links.
* New users are automatically created when signing in with a new email address.
* New accounts require administrator approval before receiving full access.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    first_name VARCHAR(100),
    last_name VARCHAR(100),

    email VARCHAR(255) UNIQUE NOT NULL,

    role VARCHAR(20) NOT NULL
        CHECK (role IN ('student', 'instructor', 'admin')),

    password_hash TEXT,

    approval_status VARCHAR(20) NOT NULL DEFAULT 'pending'
        CHECK (approval_status IN ('pending', 'approved', 'rejected')),

    is_email_verified BOOLEAN DEFAULT FALSE,

    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

---

## Magic Links

Stores temporary authentication tokens used for passwordless login.

```sql
CREATE TABLE magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    user_id UUID NOT NULL,

    token_hash TEXT NOT NULL,

    expires_at TIMESTAMP NOT NULL,

    used_at TIMESTAMP,

    created_at TIMESTAMP DEFAULT NOW(),

    CONSTRAINT fk_magic_link_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
);
```

---

## Instructors

Stores instructor-specific profile information.

```sql
CREATE TABLE instructors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    user_id UUID UNIQUE NOT NULL,

    bio TEXT,

    specialization VARCHAR(255),

    status VARCHAR(20) DEFAULT 'active'
        CHECK (status IN ('active', 'inactive')),

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

Represents instructor availability schedules.

```sql
CREATE TABLE availability_slots (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    instructor_id UUID NOT NULL,

    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,

    status VARCHAR(20) DEFAULT 'available'
        CHECK (status IN ('available', 'booked', 'unavailable')),

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

Represents scheduled lessons.

Supports both individual and group sessions.

```sql
CREATE TABLE lessons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    instructor_id UUID NOT NULL,

    slot_id UUID UNIQUE,

    title VARCHAR(255),

    description TEXT,

    lesson_type VARCHAR(20) NOT NULL
        CHECK (lesson_type IN ('individual', 'group')),

    max_participants INT NOT NULL DEFAULT 1,

    status VARCHAR(20) DEFAULT 'scheduled'
        CHECK (status IN ('scheduled', 'completed', 'cancelled')),

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

Stores lesson reservations made by students.

```sql
CREATE TABLE bookings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    lesson_id UUID NOT NULL,

    student_id UUID NOT NULL,

    status VARCHAR(20) DEFAULT 'booked'
        CHECK (
            status IN (
                'booked',
                'cancelled',
                'completed'
            )
        ),

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

Stores system notifications and user alerts.

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

# Relationships

| Entity                           | Relationship |
| -------------------------------- | ------------ |
| Users → Instructors              | 1 : 0..1     |
| Users → Magic Links              | 1 : M        |
| Instructors → Availability Slots | 1 : M        |
| Availability Slots → Lessons     | 1 : 1        |
| Instructors → Lessons            | 1 : M        |
| Lessons → Bookings               | 1 : M        |
| Users (Students) → Bookings      | 1 : M        |
| Users → Notifications            | 1 : M        |

---

# Business Rules

### Authentication

* Administrators log in using email and password.
* Students and instructors log in using Magic Links.
* Magic Link tokens are single-use and expire after a configured time period.

### User Registration

* If an email does not exist in the system, a new user record is automatically created.
* Newly created users receive `approval_status = 'pending'`.
* Users must verify their email through the Magic Link process.

### User Approval

* Administrators can approve or reject users from the admin panel.
* Approved users receive full platform access.
* Rejected users cannot book lessons.

### Booking Restrictions

* Users with `pending` status may have only one active booking.
* Users with `approved` status may book any available lessons.
* Individual lessons allow only one participant.
* Group lessons allow multiple participants based on `max_participants`.

### Dashboard

After authentication, users are redirected to their personal dashboard where they can view:

* Upcoming lessons
* Booking history
* Instructor information
* Calendar and schedule
* Notifications
* Account information

### Lesson Management

* Instructors manage their availability through availability slots.
* Each availability slot may be converted into a lesson.
* Lessons can be scheduled, completed, or cancelled.

---
