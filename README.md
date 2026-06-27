# SQL-Database-Implementation-Querying
# Lantern Library — Events Booking Database

![SQL](https://img.shields.io/badge/SQL-MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=flat-square)

A relational database built in pure SQL to manage event bookings for a public library. Covers schema design, data population, integrity constraints, and analytical querying.

---

## Schema

Eight normalised tables covering the full lifecycle of a library event.

![ERD Diagram](erd.png)

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

## Queries

#### Rooms booked per event
```sql
SELECT r.Room_Name, e.Event_Name, e.Event_Date
FROM Event e
INNER JOIN Room r ON e.RoomID = r.RoomID;
```

#### Revenue per event — confirmed bookings only
```sql
SELECT e.Event_Name,
       COUNT(eb.BookingID)  AS Confirmed_Bookings,
       SUM(e.Ticket_Cost)   AS Total_Earnings
FROM EventBooking eb
INNER JOIN Event e ON eb.EventID = e.EventID
WHERE eb.Booking_Status = 'Confirmed'
GROUP BY e.Event_Name
ORDER BY Total_Earnings DESC;
```

#### Most popular event types
```sql
SELECT et.Description, COUNT(eb.BookingID) AS Total_Bookings
FROM EventBooking eb
INNER JOIN Event     e  ON eb.EventID    = e.EventID
INNER JOIN EventType et ON e.EventTypeID = et.EventTypeID
GROUP BY et.Description
ORDER BY Total_Bookings DESC;
```

Full query set in [`/queries`](./queries/).

---

## How to run

```bash
# 1. Create the schema
mysql -u your_username -p < schema/schema.sql

# 2. Seed the data
mysql -u your_username -p < data/seed.sql
```

---

## Project structure

```
├── erd.png
├── schema/
│   └── schema.sql
├── data/
│   └── seed.sql
└── queries/
    ├── case1_rooms_booked.sql
    ├── case2_event_search.sql
    ├── case3_update_delete.sql
    └── case4_aggregations.sql
```

---

**[Your Name]** · [LinkedIn](#) · [Portfolio](#)
