# SQL-Database-Implementation-Querying

A relational database built entirely in SQL to manage event bookings for a public library. The project covers schema design from an entity-relationship diagram through to data population, integrity enforcement, and analytical querying against real business requirements.

---

## Tech Stack

![SQL](https://img.shields.io/badge/SQL-MySQL-blue?style=flat-square&logo=mysql)
![Database](https://img.shields.io/badge/Type-Relational%20Database-lightgrey?style=flat-square)


---

## What This Project Demonstrates

- Translating a physical ERD into a working relational schema
- Writing DDL with appropriate data types, constraints, and auto-increment strategies
- Populating tables in dependency order to satisfy foreign key constraints
- Identifying and correcting data type errors in a given specification
- Writing multi-table `JOIN` queries across 3+ tables
- Aggregation, grouping, filtering, and ordering for business reporting
- Safe `UPDATE` and `DELETE` operations that respect referential integrity

---

## Database Schema

Eight tables modelling the full lifecycle of a library event — from categorisation and room assignment through to member bookings, payments, and post-event feedback.

```
Room ──────────────────────────┐
EventType ─────────────────┐  │
Member  ───────────────┐   │  │
Staff  ────────────┐   │   │  └──> Event ──> EventBooking
                   └───┴───┴──────> Event ──> RoomPayment
                                    Event ──> Feedback
```

| Table | Description |
|---|---|
| `Room` | Library rooms with capacity, hourly rate, and facilities |
| `EventType` | Event categories — Workshop, Talk, Screening, Exhibition, etc. |
| `Member` | Registered members with contact details and membership type |
| `Staff` | Library staff coordinating events |
| `Event` | Core event records — links rooms, staff, event types, and members |
| `EventBooking` | Member bookings per event; status: Confirmed / Cancelled / Waitlisted |
| `RoomPayment` | Payment records per event; method: Card / Cash / Online |
| `Feedback` | Post-event ratings (1–5) and written comments from members |

---

## Schema Design Notes

**Creation order** — independent tables (`Room`, `EventType`, `Member`, `Staff`) were created before dependent tables (`Event`, `EventBooking`, `RoomPayment`, `Feedback`) to satisfy foreign key constraints at build time.

**Data type corrections** — the original specification used `DECIMAL(3,2)` for all monetary fields, which caps values at £9.99. All monetary columns (`Ticket_Cost`, `Amount_Paid`, `Hourly_Rate`) were adjusted to `DECIMAL(6,2)`. `Member_Lname` was widened from `VARCHAR(5)` to `VARCHAR(50)` to store real surnames.

**Auto-increment offsets** — `MemberID` starts at 100 and `PaymentID` at 200, making records immediately identifiable by type when reading raw query results.

**Database-level constraints:**
- `Event_Duration CHECK > 0` — prevents zero or negative duration events
- `Feedback_Rating CHECK BETWEEN 1 AND 5` — enforces valid star ratings at the database level, not the application layer

---

## Creating the Schema

```sql
CREATE TABLE Room (
    RoomID      INT PRIMARY KEY,
    Room_Name   VARCHAR(50),
    Room_Capacity INT,
    Hourly_Rate DECIMAL(6,2),
    Facilities  VARCHAR(200)
);

CREATE TABLE EventType (
    EventTypeID INT PRIMARY KEY,
    Description VARCHAR(100)
);

CREATE TABLE Member (
    MemberID     INT PRIMARY KEY AUTO_INCREMENT,
    Member_Fname VARCHAR(50),
    Member_Lname VARCHAR(50),
    Member_Email VARCHAR(50),
    Member_Phone VARCHAR(30),
    Member_Type  VARCHAR(30)
);

CREATE TABLE Staff (
    StaffID      INT PRIMARY KEY,
    Staff_Name   VARCHAR(100),
    Staff_Email  VARCHAR(50)
);

CREATE TABLE Event (
    EventID        INT PRIMARY KEY,
    Event_Name     VARCHAR(100),
    EventTypeID    INT NOT NULL,
    Event_Date     DATE,
    Event_Duration INT CHECK (Event_Duration > 0),
    RoomID         INT NOT NULL,
    StaffID        INT NOT NULL,
    MemberID       INT NOT NULL,
    Ticket_Cost    DECIMAL(6,2),
    FOREIGN KEY (EventTypeID) REFERENCES EventType(EventTypeID),
    FOREIGN KEY (RoomID)      REFERENCES Room(RoomID),
    FOREIGN KEY (StaffID)     REFERENCES Staff(StaffID),
    FOREIGN KEY (MemberID)    REFERENCES Member(MemberID)
);

CREATE TABLE EventBooking (
    BookingID      INT PRIMARY KEY,
    MemberID       INT NOT NULL,
    EventID        INT NOT NULL,
    Booking_Date   DATE,
    Booking_Status VARCHAR(50),
    FOREIGN KEY (MemberID) REFERENCES Member(MemberID),
    FOREIGN KEY (EventID)  REFERENCES Event(EventID)
);

CREATE TABLE RoomPayment (
    PaymentID      INT PRIMARY KEY AUTO_INCREMENT,
    EventID        INT NOT NULL,
    MemberID       INT NOT NULL,
    Payment_Date   DATE,
    Amount_Paid    DECIMAL(6,2),
    Payment_Method VARCHAR(30),
    Payment_Status VARCHAR(30),
    FOREIGN KEY (EventID)  REFERENCES Event(EventID),
    FOREIGN KEY (MemberID) REFERENCES Member(MemberID)
);

CREATE TABLE Feedback (
    FeedbackID       INT PRIMARY KEY AUTO_INCREMENT,
    EventID          INT NOT NULL,
    Feedback_Date    DATE DEFAULT (CURRENT_DATE),
    Feedback_Rating  INT CHECK (Feedback_Rating BETWEEN 1 AND 5),
    Feedback_Comments TEXT,
    FOREIGN KEY (EventID) REFERENCES Event(EventID)
);
```

---

## Queries

### Rooms booked for each event

```sql
SELECT r.Room_Name, e.Event_Name, e.Event_Date
FROM Event e
INNER JOIN Room r ON e.RoomID = r.RoomID;
```

`INNER JOIN` is appropriate because `RoomID` in `Event` is `NOT NULL` — every event is guaranteed to have a room, so no rows will be silently dropped.

---

### All events sorted by type

```sql
SELECT e.Event_Name, e.Event_Date, et.Description
FROM Event e
INNER JOIN EventType et ON e.EventTypeID = et.EventTypeID
ORDER BY et.Description;
```

---

### Events with a ticket price under £5

```sql
SELECT Event_Name, Ticket_Cost
FROM Event
WHERE Ticket_Cost < 5
ORDER BY Ticket_Cost ASC;
```

---

### Update an event name and price

```sql
UPDATE Event
SET Event_Name  = 'Community Reading Picnic',
    Ticket_Cost = 20.00
WHERE Event_Name = 'Family Reading Picnic';
```

---

### Delete an event (with referential integrity)

Child records must be removed from dependent tables before the parent row can be deleted from `Event`.

```sql
DELETE FROM EventBooking WHERE EventID = 3;
DELETE FROM Feedback     WHERE EventID = 3;
DELETE FROM RoomPayment  WHERE EventID = 3;
DELETE FROM Event        WHERE EventID = 3;
```

---

### Most popular event types by number of bookings

```sql
SELECT et.Description, COUNT(eb.BookingID) AS Total_Bookings
FROM EventBooking eb
INNER JOIN Event     e  ON eb.EventID    = e.EventID
INNER JOIN EventType et ON e.EventTypeID = et.EventTypeID
GROUP BY et.Description
ORDER BY Total_Bookings DESC;
```

Three-table join chain: `EventBooking → Event → EventType`. `GROUP BY` collapses all bookings of the same type into a single counted row.

---

### Revenue per event (confirmed bookings only)

```sql
SELECT e.Event_Name,
       COUNT(eb.BookingID) AS Confirmed_Bookings,
       SUM(e.Ticket_Cost)  AS Total_Earnings
FROM EventBooking eb
INNER JOIN Event e ON eb.EventID = e.EventID
WHERE eb.Booking_Status = 'Confirmed'
GROUP BY e.Event_Name
ORDER BY Total_Earnings DESC;
```

The `WHERE` clause filters before aggregation, ensuring cancelled and waitlisted bookings are excluded from revenue figures.

---

### Potential earnings per room

```sql
SELECT r.Room_Name, SUM(e.Ticket_Cost) AS Potential_Earnings
FROM Event e
INNER JOIN Room         r  ON e.RoomID  = r.RoomID
INNER JOIN EventBooking eb ON e.EventID = eb.EventID
GROUP BY r.Room_Name
ORDER BY Potential_Earnings DESC;
```

---

## Dataset

The database is seeded with 70+ records across all tables.

| Table | Records |
|---|---|
| EventType | 13 |
| Event | 20 |
| Member | 6 |
| Staff | 6 |
| Room | 6 |
| EventBooking | 6 |
| RoomPayment | 6 |
| Feedback | 6 |

Booking statuses cover all three states (Confirmed, Cancelled, Waitlisted). Payment methods include Card, Cash, and Online. Member types include Student, Adult, and Senior.

---

## Related Projects

> **Project 1 — Normalisation & Logical/Physical Design**
> Covers the full database design process from raw unnormalised data through 1NF, 2NF, and 3NF, producing both a logical and physical ERD from scratch. *(Link coming soon)*

---

## Author

**[Your Name]**  
[LinkedIn](#) · [Portfolio](#) · [Email](#)
