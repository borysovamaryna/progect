# Database Design (MVP)

## Overview

This database schema supports the MVP version of the Education Platform.

The platform allows:

* Student authentication via Magic Links
* Administrator authentication via email and password
* Student approval workflow
* Subject management
* Teacher availability management
* Lesson scheduling
* Lesson booking
* Email notifications and reminders

For the MVP, teachers are not managed as platform users. Teacher information is stored directly within availability slots and lessons.

---

# Main Entities

## Users

Stores platform users.

### Roles

* `student`
* `admin`

### Responsibilities

Students:

* Authenticate using Magic Links
* Book lessons
* View lesson history

Administrators:

* Authenticate using email and password
* Manage students
* Manage subjects
* Manage lesson availability
* Approve or reject users

---

## Magic Links

Stores temporary authentication tokens used for passwordless student login.

### Rules

* Single-use tokens
* Expiration date required
* Linked to a user account

---

## Subjects

Stores available school subjects.

### Examples

* English
* Mathematics
* Physics
* Chemistry

Subjects are assigned to availability slots and lessons.

---

## Availability Slots

Represents available lesson slots created by administrators.

Each slot contains:

* Subject
* Teacher name
* Teacher email (optional)
* Start time
* End time
* Availability status

### Statuses

* available
* booked
* unavailable

---

## Lessons

Represents scheduled lessons.

A lesson is created from an availability slot.

### Lesson Types

* individual
* group

### Statuses

* scheduled
* completed
* cancelled

---

## Bookings

Stores lesson reservations made by students.

### Statuses

* booked
* cancelled
* completed

Supports:

* Individual lessons
* Group lessons
* Booking history
* Cancellation tracking

---

## Notifications

Stores system notifications and email events.

### Examples

* Magic Link email
* Booking confirmation
* Booking cancellation
* Lesson reminder

---

# Relationships

| Entity                        | Relationship |
| ----------------------------- | ------------ |
| Users → Magic Links           | 1 : M        |
| Subjects → Availability Slots | 1 : M        |
| Availability Slots → Lessons  | 1 : 1        |
| Subjects → Lessons            | 1 : M        |
| Lessons → Bookings            | 1 : M        |
| Users → Bookings              | 1 : M        |
| Users → Notifications         | 1 : M        |

---

# Business Rules

## Authentication

### Administrator

Uses:

* Email
* Password

### Student

Uses:

* Magic Link

---

## Registration

If a user enters an email that does not exist:

1. Create a new student account
2. Set approval status to `pending`
3. Send Magic Link
4. Wait for administrator approval

---

## User Approval

Administrators can:

* Approve users
* Reject users
* View pending accounts

Only approved users receive full platform access.

---

## Availability Management

Administrators can:

* Create availability slots
* Edit availability slots
* Delete availability slots
* Assign teachers
* Assign subjects

---

## Lesson Booking

Students can:

* View available slots
* Book lessons
* Cancel bookings

System must:

* Prevent double booking
* Respect participant limits
* Hide unavailable slots

---

## Cancellation Policy

Cancellation is allowed up to 12 hours before the lesson.

Additional restrictions may be applied according to business requirements.

---

## Notifications

The platform should support:

* Magic Link emails
* Booking confirmations
* Booking cancellations
* Lesson reminders

---

# Future Enhancements

The MVP intentionally stores teacher information as text fields.

Future versions may introduce:

* Teachers table
* Teacher accounts
* Teacher authentication
* Teacher profiles
* Teacher-specific notifications
* Teacher availability management

The current design should allow migration to a dedicated Teachers entity without major architectural changes.
