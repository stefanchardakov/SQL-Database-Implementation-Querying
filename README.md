# SQL-Database-Implementation-Querying
# Lantern Library — Events Booking Database

![SQL](https://img.shields.io/badge/SQL-MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=flat-square)

A relational database built in SQL to manage event bookings for a public library. Covers schema design, data population, integrity constraints, and analytical querying.

---

## Schema

Eight normalised tables covering the full lifecycle of a library event.

<img width="521" height="254" alt="ERD_executed" src="https://github.com/user-attachments/assets/854197aa-fea2-4259-899d-532a03a8d00e" />


| Table | Description |
|---|---|
| `Room` | Library rooms — capacity, hourly rate, facilities |
| `EventType` | Event categories (Workshop, Talk, Screening, etc.) |
| `Member` | Registered members with contact details and type |
| `Staff` | Staff members who coordinate events |
| `Event` | Core event records linking rooms, staff, and types |
| `EventBooking` | Member bookings — Confirmed, Cancelled, or Waitlisted |
| `RoomPayment` | Payments per event — Card, Cash, or Online |
| `Feedback` | Post-event ratings (1–5) and comments |

---
# 📚 Lantern Library — Event Management Database System

A relational database system designed to manage events, bookings, members, staff, rooms, payments, and feedback for **Lantern Library**. Built entirely in **SQL (MySQL)**, the project demonstrates database design principles including schema creation, referential integrity, data population, and analytical querying.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Database Schema](#database-schema)
- [Entity Relationships](#entity-relationships)
- [Getting Started](#getting-started)
- [SQL Features Demonstrated](#sql-features-demonstrated)
- [Sample Queries](#sample-queries)
- [Data Summary](#data-summary)
- [References](#references)

---

## Project Overview

The Lantern Library Event Management System enables a library to:

- Register and categorise library **members** (Student, Adult, Senior)
- Schedule **events** across multiple room types (talks, workshops, screenings, clubs, etc.)
- Manage **room bookings** and **staff assignments**
- Track **event bookings** with status tracking (Confirmed, Cancelled, Waitlisted)
- Record **payments** with multiple payment methods (Card, Cash, Online)
- Collect and store post-event **feedback** with ratings (1–5)

---

## Database Schema

Tables were created in dependency order — independent (parent) tables first, followed by dependent (child) tables.

### Independent Tables

| Table | Primary Key | Description |
|-------|------------|-------------|
| `Room` | `RoomID` | Library rooms with capacity, hourly rate, and facilities |
| `EventType` | `EventTypeID` | Categories of events (Talk, Workshop, Screening, etc.) |
| `Member` | `MemberID` (AUTO_INCREMENT from 100) | Library members with contact details and membership type |
| `Staff` | `StaffID` | Staff members with name and email |

### Dependent Tables

| Table | Primary Key | Foreign Keys | Description |
|-------|------------|-------------|-------------|
| `Event` | `EventID` | `EventTypeID`, `RoomID`, `StaffID`, `MemberID` | Core event records with date, duration, and ticket cost |
| `EventBooking` | `BookingID` | `MemberID`, `EventID` | Booking records with date and status |
| `RoomPayment` | `PaymentID` (AUTO_INCREMENT from 200) | `EventID`, `MemberID` | Payment records with method and status |
| `Feedback` | `FeedbackID` (AUTO_INCREMENT from 500) | `EventID` | Post-event ratings and comments |

### Key Schema Decisions

- `Member.MemberID` starts at **100** to distinguish members from other numeric IDs
- `RoomPayment.PaymentID` starts at **200** to distinguish payment records
- `Feedback.FeedbackID` starts at **500** to distinguish feedback records
- `Event.Event_Duration` uses a `CHECK` constraint (`> 0`) to prevent invalid values
- `Feedback.Feedback_Rating` uses a `CHECK` constraint (`> 0 AND < 6`) to enforce 1–5 ratings
- `Feedback.Feedback_Date` defaults to `CURRENT_DATE` for automatic timestamping
- `DECIMAL(6,2)` used for monetary fields (`Hourly_Rate`, `Ticket_Cost`, `Amount_Paid`) to ensure accurate currency representation

---

## Entity Relationships

```
EventType ──┐
Room        ├──► Event ──► EventBooking ──► Member
Staff       │         └──► RoomPayment  ──► Member
Member ─────┘         └──► Feedback
```

All foreign key relationships enforce referential integrity. When deleting a parent record (e.g., an Event), all child records across `EventBooking`, `Feedback`, and `RoomPayment` must be removed first.

---

## Getting Started

### Prerequisites

- MySQL 8.0+ (or compatible SQL engine)
- A SQL client (MySQL Workbench, DBeaver, or CLI)

### Setup Instructions

1. **Clone this repository**
   ```bash
   git clone https://github.com/your-username/lantern-library-db.git
   cd lantern-library-db
   ```

2. **Create the database**
   ```sql
   CREATE DATABASE LanternLibrary;
   USE LanternLibrary;
   ```

3. **Run the schema file** to create all tables
   ```bash
   mysql -u root -p LanternLibrary < schema.sql
   ```

4. **Populate with sample data**
   ```bash
   mysql -u root -p LanternLibrary < data.sql
   ```

5. **Run sample queries**
   ```bash
   mysql -u root -p LanternLibrary < queries.sql
   ```

---

## SQL Features Demonstrated

| Feature | Description |
|---------|-------------|
| `PRIMARY KEY` | Unique identifiers across all tables |
| `FOREIGN KEY` | Referential integrity between related tables |
| `AUTO_INCREMENT` | Automatic ID generation with custom starting values |
| `CHECK` constraints | Data validation on rating and duration fields |
| `DEFAULT` | Automatic date capture on feedback submission |
| `NOT NULL` | Mandatory field enforcement on foreign key columns |
| `INNER JOIN` | Multi-table data retrieval across 2–3 joined tables |
| `WHERE` | Filtered queries (e.g., tickets under £5) |
| `GROUP BY` | Aggregation of bookings and revenue per event/room |
| `ORDER BY` | Sorting results (alphabetical, ascending, descending) |
| `COUNT` / `SUM` | Aggregate functions for bookings and earnings |
| `UPDATE` | Modifying existing records (event rename and repricing) |
| `DELETE` | Cascading deletion respecting foreign key order |

---

## Sample Queries

### Rooms booked for each event
```sql
SELECT r.Room_Name, e.Event_Name, e.Event_Date
FROM Event e
INNER JOIN Room r ON e.RoomID = r.RoomID;
```

### All events sorted by event type
```sql
SELECT e.Event_Name, e.Event_Date, et.Description
FROM Event e
INNER JOIN EventType et ON e.EventTypeID = et.EventTypeID
ORDER BY et.Description;
```

### Events with ticket cost under £5
```sql
SELECT Event_Name, Ticket_Cost
FROM Event
WHERE Ticket_Cost < 5;
```

### Number of bookings per event type (most to least popular)
```sql
SELECT et.Description, COUNT(eb.BookingID) AS Total_Bookings
FROM EventBooking eb
INNER JOIN Event e ON eb.EventID = e.EventID
INNER JOIN EventType et ON e.EventTypeID = et.EventTypeID
GROUP BY et.Description
ORDER BY Total_Bookings DESC;
```

### Revenue from confirmed bookings per event
```sql
SELECT e.Event_Name, COUNT(eb.BookingID) AS Confirmed_Bookings,
       SUM(e.Ticket_Cost) AS Total_Earnings
FROM EventBooking eb
INNER JOIN Event e ON eb.EventID = e.EventID
WHERE eb.Booking_Status = 'Confirmed'
GROUP BY e.Event_Name
ORDER BY Total_Earnings DESC;
```

### Total potential earnings per room
```sql
SELECT r.Room_Name, SUM(e.Ticket_Cost) AS Total_Earnings
FROM Event e
INNER JOIN Room r ON e.RoomID = r.RoomID
INNER JOIN EventBooking eb ON e.EventID = eb.EventID
GROUP BY r.Room_Name
ORDER BY Total_Earnings DESC;
```

---

## Data Summary

| Entity | Count |
|--------|-------|
| Event Types | 11 |
| Members | 11 (IDs 100–110) |
| Staff | 10 |
| Rooms | 10 |
| Events | 20 |
| Bookings | 20 |
| Payments | 20 (IDs 200–219) |
| Feedback entries | 20 (IDs 500–519) |

**Booking statuses:** Confirmed · Cancelled · Waitlisted  
**Payment methods:** Card · Cash · Online  
**Payment statuses:** Paid · Pending  
**Member types:** Student · Adult · Senior

---

## References

- Connolly, T. and Begg, C. (2015). *Database Systems: A Practical Approach to Design, Implementation, and Management*, 6th edition. Harlow: Pearson. Available at: [www.pearsonglobaleditions.com/connolly](http://www.pearsonglobaleditions.com/connolly)

- Bush, J. (2020). *Learn SQL Database Programming: Query and Manipulate Databases from Popular Relational Database Servers Using SQL*. Packt Publishing. Available at: [https://research.ebsco.com/](https://research.ebsco.com/)

---

*Developed as part of a database design and implementation project.*
