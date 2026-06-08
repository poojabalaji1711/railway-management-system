# Railway Management System — Setup Guide
## Java + JSP + Servlets + JDBC + MySQL + Apache Tomcat

---

## Prerequisites

| Tool | Version | Download |
|------|---------|----------|
| JDK  | 17+     | https://adoptium.net |
| Apache Tomcat | 10.x | https://tomcat.apache.org |
| MySQL | 8.0+ | https://dev.mysql.com/downloads |
| Eclipse IDE (EE) | 2024+ | https://www.eclipse.org |
| MySQL Connector/J | 8.x | https://dev.mysql.com/downloads/connector/j |

---

## Step 1 — Database Setup

```bash
# Start MySQL and login
mysql -u root -p

# Run the schema and sample data scripts
source /path/to/RailwayMS/sql/schema.sql;
source /path/to/RailwayMS/sql/data.sql;

# Verify
USE railway_db;
SHOW TABLES;
SELECT * FROM users;
SELECT * FROM trains;
```

---

## Step 2 — Configure Database Connection

Open `src/main/java/com/railway/util/DBConnection.java` and update:

```java
private static final String DB_URL      = "jdbc:mysql://localhost:3306/railway_db?useSSL=false&serverTimezone=Asia/Kolkata";
private static final String DB_USER     = "root";
private static final String DB_PASSWORD = "YOUR_MYSQL_PASSWORD";
```

---

## Step 3 — Add Required JARs to WEB-INF/lib

Copy these JAR files into `src/main/webapp/WEB-INF/lib/`:

1. `mysql-connector-j-8.x.jar`  — JDBC driver
2. `jstl-1.2.jar`               — JSP Standard Tag Library
3. (Optional) `jbcrypt-0.4.jar` — for password hashing

---

## Step 4 — Import into Eclipse

1. **File → Import → Existing Maven Projects** (or Dynamic Web Project)
2. Point to the `RailwayMS` folder
3. Right-click project → **Properties → Project Facets**
   - Check: Dynamic Web Module 6.0, Java 17
4. **Server**: Add Apache Tomcat 10 (Window → Preferences → Server → Runtime)

---

## Step 5 — Deploy to Tomcat

### Via Eclipse:
1. Right-click project → **Run As → Run on Server**
2. Select your Tomcat 10 instance
3. Click **Finish**

### Via WAR file:
```bash
# Build WAR (if using Maven)
mvn clean package

# Copy WAR to Tomcat webapps
cp target/RailwayMS.war /path/to/tomcat/webapps/

# Start Tomcat
cd /path/to/tomcat/bin
./startup.sh   # Linux/Mac
startup.bat    # Windows
```

---

## Step 6 — Access the Application

Open your browser:  
`http://localhost:8080/RailwayMS/`

| Page | URL |
|------|-----|
| Home | /RailwayMS/ |
| Login | /RailwayMS/login.jsp |
| Register | /RailwayMS/register.jsp |
| User Dashboard | /RailwayMS/user/dashboard.jsp |
| Admin Dashboard | /RailwayMS/admin/dashboard.jsp |
| Search Trains | /RailwayMS/searchTrain |
| Admin — Trains | /RailwayMS/admin/trains |

---

## Project Folder Structure

```
RailwayMS/
├── src/
│   └── main/
│       ├── java/com/railway/
│       │   ├── model/
│       │   │   ├── User.java
│       │   │   ├── Train.java
│       │   │   ├── Ticket.java
│       │   │   ├── Passenger.java
│       │   │   ├── Station.java
│       │   │   └── Payment.java
│       │   ├── dao/
│       │   │   ├── UserDAO.java
│       │   │   ├── TrainDAO.java
│       │   │   ├── TicketDAO.java
│       │   │   ├── PassengerDAO.java
│       │   │   └── StationDAO.java
│       │   ├── servlet/
│       │   │   ├── LoginServlet.java
│       │   │   ├── RegisterServlet.java
│       │   │   ├── LogoutServlet.java
│       │   │   ├── TrainServlet.java
│       │   │   ├── SearchTrainServlet.java
│       │   │   ├── BookTicketServlet.java
│       │   │   ├── CancelTicketServlet.java
│       │   │   └── ReportServlet.java
│       │   └── util/
│       │       └── DBConnection.java
│       └── webapp/
│           ├── WEB-INF/
│           │   └── web.xml
│           ├── css/
│           │   └── style.css
│           ├── js/
│           │   └── main.js
│           ├── index.jsp
│           ├── login.jsp
│           ├── register.jsp
│           ├── error.jsp
│           ├── user/
│           │   ├── dashboard.jsp
│           │   ├── searchResults.jsp
│           │   ├── booking.jsp
│           │   ├── seatSelection.jsp
│           │   ├── payment.jsp
│           │   ├── bookingConfirmation.jsp
│           │   ├── myTickets.jsp
│           │   └── cancelTicket.jsp
│           └── admin/
│               ├── dashboard.jsp
│               ├── trains.jsp
│               ├── stations.jsp
│               ├── passengers.jsp
│               └── reports.jsp
└── sql/
    ├── schema.sql
    └── data.sql
```

---

## ER Diagram (Text Representation)

```
USERS ────────────────────── TICKETS ──────────────── TRAINS
 user_id (PK)                 ticket_id (PK)            train_id (PK)
 name                         pnr_number (UNIQUE)        train_number
 email (UNIQUE)               user_id (FK → users)       train_name
 password                     train_id (FK → trains)     source
 phone                        travel_date                destination
 role                         booking_date               departure_time
                              status                     arrival_time
                              fare                       total_seats
                               │                         available_seats
                               │                         fare
                               ▼
                        TICKET_PASSENGERS ─────────── PASSENGERS
                         id (PK)                        passenger_id (PK)
                         ticket_id (FK)                 name
                         passenger_id (FK)              age
                         seat_number                    gender
                         coach                          phone
                         ticket_class
                               │
                               ▼
                           PAYMENTS
                            payment_id (PK)
                            ticket_id (FK)
                            amount
                            payment_status
                            payment_method
                            transaction_id

ROUTES ─────────────────────────────────────────────────────
 route_id (PK)
 train_id (FK → trains)
 station_id (FK → stations)
 stop_order

STATIONS
 station_id (PK)
 station_name
 station_code (UNIQUE)
```

---

## Key SQL Queries

### Daily bookings report
```sql
SELECT DATE(booking_date) AS day,
       COUNT(*) AS total_bookings,
       SUM(fare) AS revenue
FROM tickets
WHERE status = 'Confirmed'
  AND booking_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
GROUP BY DATE(booking_date)
ORDER BY day DESC;
```

### Revenue by train
```sql
SELECT t.train_name, t.train_number,
       COUNT(tk.ticket_id) AS bookings,
       SUM(tk.fare)         AS total_revenue
FROM trains t
JOIN tickets tk ON t.train_id = tk.train_id
WHERE tk.status = 'Confirmed'
GROUP BY t.train_id
ORDER BY total_revenue DESC;
```

### Available trains between two stations
```sql
SELECT * FROM trains
WHERE LOWER(source)      = LOWER(?)
  AND LOWER(destination) = LOWER(?)
  AND available_seats    > 0;
```

---

## Common Issues & Fixes

| Problem | Fix |
|---------|-----|
| `ClassNotFoundException: com.mysql.cj.jdbc.Driver` | Add `mysql-connector-j.jar` to `WEB-INF/lib` |
| `HTTP 404` on servlets | Check `@WebServlet` annotation URL matches form action |
| JSTL `c:forEach` not working | Add `jstl-1.2.jar` to `WEB-INF/lib` |
| Tomcat 10 `javax.servlet` errors | Use `jakarta.servlet.*` imports (Tomcat 10 = Jakarta EE 9+) |
| Session not persisting | Ensure `req.getSession(true)` not `false` on login |

---

## Notes for College Submission

- Implement BCrypt password hashing using `jbcrypt-0.4.jar`
- Add input validation on all forms (both client JS and server-side)
- Add a `LogoutServlet` that calls `session.invalidate()`
- Use `PreparedStatement` everywhere (prevents SQL injection)
- Use database transactions for booking and cancellation (shown in TicketDAO)
