# SQL-Database-Implementation-Querying


![SQL](https://img.shields.io/badge/SQL-MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=flat-square)

A relational database built entirely in SQL to manage event bookings for a public library. Covers schema design, data population, integrity constraints, and analytical querying.

---


## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Database Schema](#database-schema)
- [Entity Relationships](#entity-relationships)
- [Getting Started](#getting-started)
- [SQL Features Demonstrated](#sql-features-demonstrated)
- [Sample Queries](#sample-queries)


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

