# Hostel Management System (Java Swing + MySQL)

A complete, production-grade native desktop application designed for college Computer Science and Engineering (CSE) academic evaluation (OOPs & DBMS). Built with **Java 17**, **Java Swing**, **FlatLaf**, **JDBC**, **MySQL 8.x**, and **Maven**.

---

## 🌟 Key Features

1. **Executive Dashboard**: Real-time aggregated metrics directly derived from MySQL:
   - Total Students, Total Rooms, Occupied Beds, Available Beds, Pending Fee balances, Unresolved Complaints.
2. **Student Management**:
   - Full CRUD operations with search by Register Number, Name, or Department.
   - Comprehensive validation (phone format, email format, emergency contact, unique register numbers).
   - Foreign-key protection preventing unsafe deletions of students with active room allocations or payment history.
3. **Room & Capacity Management**:
   - Tracking block, room number, floor, room type, bed capacity, occupied beds, and available beds.
   - Intelligent capacity safeguard preventing administrators from reducing room capacity below the current occupant count.
4. **Room Allocation & Transfers**:
   - Prevents multiple active allocations for the same student.
   - Enforces room bed limits to prevent over-capacity.
   - Multi-table atomic room transfers using **JDBC Database Transactions** (`commit` / `rollback`).
5. **Fee & Payment Management**:
   - Issue fee invoices and record partial/full installment payments.
   - Tracks amounts due, amounts paid, payment method (Cash, UPI, Demand Draft, Bank Transfer), and dates.
   - Zero storage of sensitive credit/debit card numbers.
6. **Complaints Helpdesk**:
   - Student grievance ticketing (Plumbing, Electrical, Wi-Fi, Food, Housekeeping).
   - Lifecycle tracking: `OPEN` -> `IN_PROGRESS` -> `RESOLVED` with resolution remarks.

---

## 🏛️ Architecture & Package Structure

The codebase is organized into a clean 3-tier layered architecture:

```
c:/Users/.../java project/
├── pom.xml                               # Maven build descriptor with Java 17 & FlatLaf
├── db.properties.example                 # Template for local MySQL credentials
├── sql/
│   ├── schema.sql                        # Safe DDL script (CREATE IF NOT EXISTS)
│   └── sample_data.sql                   # Realistic sample records with SHA-256 admin
├── output-screenshots/                   # Directory to save application evaluation screenshots
└── src/
    ├── main/java/com/hostel/
    │   ├── Main.java                     # Application entry point with FlatLaf setup
    │   ├── config/
    │   │   └── DatabaseConfig.java       # Safe JDBC connection manager (db.properties / ENV)
    │   ├── exception/
    │   │   ├── AppException.java         # Base custom exception
    │   │   ├── DatabaseException.java    # JDBC / SQL wrapper exception
    │   │   ├── ValidationException.java  # Input / Business rule violation
    │   │   ├── DuplicateRecordException.java
    │   │   ├── EntityNotFoundException.java
    │   │   └── RoomOverCapacityException.java
    │   ├── model/
    │   │   ├── Person.java               # [ABSTRACTION & INHERITANCE] Abstract base class
    │   │   ├── Student.java              # [INHERITANCE & POLYMORPHISM] Extends Person
    │   │   ├── Warden.java               # [INHERITANCE & POLYMORPHISM] Extends Person
    │   │   ├── Admin.java                # Authentication model
    │   │   ├── Room.java                 # Room entity with dynamic bed availability
    │   │   ├── Allocation.java           # Room allocation record
    │   │   ├── Payment.java              # Fee invoice and payment model
    │   │   ├── Complaint.java            # Maintenance grievance model
    │   │   └── DashboardStats.java       # Executive summary DTO
    │   ├── dao/
    │   │   ├── AdminDao.java & AdminDaoImpl.java
    │   │   ├── StudentDao.java & StudentDaoImpl.java
    │   │   ├── RoomDao.java & RoomDaoImpl.java
    │   │   ├── AllocationDao.java & AllocationDaoImpl.java
    │   │   ├── PaymentDao.java & PaymentDaoImpl.java
    │   │   ├── ComplaintDao.java & ComplaintDaoImpl.java
    │   │   └── DashboardDao.java
    │   ├── service/
    │   │   ├── AuthService.java          # Authentication & SHA-256 password hashing
    │   │   ├── StudentService.java       # Student validation & referential integrity
    │   │   ├── RoomService.java          # Capacity limits & room rules
    │   │   ├── AllocationService.java    # Atomic JDBC transactions for room transfers
    │   │   ├── PaymentService.java       # Fee ledger & installment calculations
    │   │   ├── ComplaintService.java     # Ticketing & status updates
    │   │   ├── DashboardService.java     # Aggregate metrics
    │   │   └── PersonService.java        # Live OOP Polymorphism demonstration
    │   ├── ui/
    │   │   ├── UIHelper.java             # Color palette, table styling, and dialog helpers
    │   │   ├── LoginFrame.java           # Modern login window with connection test tool
    │   │   ├── MainFrame.java            # Sidebar navigation container
    │   │   ├── DashboardPanel.java       # Real-time metric cards and tables
    │   │   ├── StudentPanel.java         # Student CRUD and search
    │   │   ├── RoomPanel.java            # Room capacity management
    │   │   ├── AllocationPanel.java      # Room allocation and transfer dialogs
    │   │   ├── PaymentPanel.java         # Fee ledger and payments
    │   │   └── ComplaintPanel.java       # Maintenance grievance helpdesk
    │   └── util/
    │       ├── PasswordUtil.java         # Cryptographic salt + SHA-256 hashing
    │       ├── DateUtil.java             # Date formatting and parsing
    │       └── ValidationUtil.java       # Email, phone, and number validators
    └── test/java/com/hostel/
        ├── model/PersonOopTest.java      # Automated tests for OOP concepts
        ├── service/StudentServiceTest.java
        ├── service/RoomServiceTest.java
        └── util/PasswordUtilTest.java & ValidationUtilTest.java
```

---

## 🚀 Setup & Execution Guide

### Step 1: Database Initialization in MySQL Workbench
1. Open **MySQL Workbench**.
2. Connect to your local MySQL instance (port `3306`).
3. Open and execute [`sql/schema.sql`](file:///sql/schema.sql) (`File -> Open SQL Script -> Execute`).
   - This safely creates the `hostel_management` database and the 6 relational tables with InnoDB constraints.
4. Open and execute [`sql/sample_data.sql`](file:///sql/sample_data.sql) (`File -> Open SQL Script -> Execute`).
   - This seeds sample rooms, students, allocations, payments, complaints, and the default admin account:
     - **Username**: `admin`
     - **Password**: `admin123`

### Step 2: Configure Local Credentials
1. In the project root directory, copy `db.properties.example` and name it `db.properties`.
2. Edit `db.properties`:
   ```properties
   db.url=jdbc:mysql://localhost:3306/hostel_management?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
   db.username=root
   db.password=YOUR_ACTUAL_MYSQL_PASSWORD
   ```
   *(Note: `db.properties` is already listed in `.gitignore` so your password will never be committed).*

### Step 3: Build and Test
Run in your terminal / PowerShell:
```powershell
mvn clean test
```
All 13 automated tests will execute and confirm business logic, OOP inheritance, and hashing.

### Step 4: Run the Desktop Application
Run either via Maven:
```powershell
mvn compile exec:java -Dexec.mainClass="com.hostel.Main"
```
Or build the standalone executable JAR:
```powershell
mvn clean package -DskipTests
java -jar target/hostel-management-system-1.0.0.jar
```

---

## 🔒 Security & Best Practices

- **Zero Plaintext Passwords**: Passwords stored as SHA-256 hashes salted with cryptographically secure 16-byte random salts.
- **SQL Injection Prevention**: 100% of database interactions utilize parameterized `PreparedStatement`.
- **Database Safety**: All scripts use `CREATE TABLE IF NOT EXISTS` and `INSERT IGNORE`; tables and databases are never automatically dropped.
- **Data Integrity**: Enforced via InnoDB foreign keys with `ON DELETE RESTRICT` and transactional rollbacks on transfer failures.
