# Requirement-to-Code Mapping Document
## Hostel Management System - Technical Implementation Matrix

This matrix maps every functional, security, and architectural requirement to its corresponding implementation in Java source files, methods, and MySQL database tables.

---

| # | Requirement Specification | Java Class / Method | Database Table / Column | Test / Verification Method |
|---|---------------------------|---------------------|-------------------------|----------------------------|
| **1** | **Admin Login & Auth** | `AuthService.login()`<br>`PasswordUtil.verifyPassword()` | `admins`<br>(`password_hash`, `salt`) | `PasswordUtilTest.java`<br>`LoginFrame.java` |
| **2** | **Salted SHA-256 Hashing** | `PasswordUtil.hashPassword()`<br>`PasswordUtil.generateSalt()` | `admins.password_hash`<br>`admins.salt` | `PasswordUtilTest.java` |
| **3** | **First Admin Bootstrap** | `AuthService.bootstrapInitialAdminIfEmpty()` | `admins` (auto-insert if empty) | `Main.java` / `LoginFrame.java` |
| **4** | **Student Registration** | `StudentService.registerStudent()`<br>`StudentDaoImpl.create()` | `students` table | `StudentServiceTest.java`<br>`StudentPanel.java` |
| **5** | **Duplicate RegNo Check** | `StudentService.registerStudent()`<br>`studentDao.findByRegisterNumber()` | `students.register_number`<br>(`UNIQUE KEY`) | `StudentServiceTest.testRegisterDuplicateStudent()` |
| **6** | **Search Students** | `StudentService.searchStudents()`<br>`StudentDaoImpl.search()` | `students` (`register_number`, `full_name`, `department`) | `StudentPanel.java` search box |
| **7** | **Safe Student Deletion** | `StudentService.deleteStudent()`<br>`studentDao.hasActiveAllocations()` | `students.student_id`<br>(`ON DELETE RESTRICT`) | `StudentServiceTest.testPreventDeleteActiveStudent()` |
| **8** | **Room Management (CRUD)** | `RoomService.addRoom()`<br>`RoomDaoImpl.create()` | `rooms`<br>(`uq_block_room`) | `RoomPanel.java`<br>`RoomServiceTest.java` |
| **9** | **Live Bed Availability** | `Room.getAvailableBeds()`<br>`RoomDaoImpl.findAll()` | `rooms.capacity` minus count of active `allocations` | `RoomPanel.java`<br>`DashboardPanel.java` |
| **10** | **Prevent Capacity Reduction Below Occupants** | `RoomService.updateRoom()`<br>`roomDao.getActiveOccupantsCount()` | `rooms.capacity`<br>`allocations.status` | `RoomServiceTest.testPreventReducingCapacityBelowOccupants()` |
| **11** | **Safe Room Deletion** | `RoomService.deleteRoom()`<br>`roomDao.hasActiveAllocations()` | `rooms.room_id`<br>(`ON DELETE RESTRICT`) | `RoomServiceTest.testPreventDeleteOccupiedRoom()` |
| **12** | **Room Allocation** | `AllocationService.allocateRoom()`<br>`AllocationDaoImpl.create()` | `allocations` table | `AllocationPanel.java` |
| **13** | **Prevent Multiple Active Allocations** | `AllocationService.allocateRoom()`<br>`allocationDao.findActiveAllocationByStudentId()` | `allocations.status = 'ACTIVE'` | `AllocationPanel.java` |
| **14** | **Prevent Over-Capacity** | `AllocationService.allocateRoom()` | `RoomOverCapacityException`<br>`rooms.capacity` | `PersonOopTest.java`<br>`AllocationPanel.java` |
| **15** | **Atomic Room Transfer** | `AllocationService.transferRoom()`<br>(JDBC Transaction `commit`/`rollback`) | `allocations`<br>(Old `TRANSFERRED`, New `ACTIVE`) | `AllocationPanel.java` |
| **16** | **Vacate Room** | `AllocationService.vacateRoom()`<br>`AllocationDaoImpl.vacate()` | `allocations.status = 'VACATED'`<br>`allocations.vacate_date` | `AllocationPanel.java` |
| **17** | **Fee Invoice Issuance** | `PaymentService.createFeeInvoice()`<br>`PaymentDaoImpl.create()` | `payments` (`amount_due`, `payment_status`) | `PaymentPanel.java` |
| **18** | **Fee Installment Recording** | `PaymentService.recordPayment()`<br>`PaymentDaoImpl.recordPayment()` | `payments.amount_paid`<br>`payments.payment_status` | `PaymentPanel.java` |
| **19** | **Zero Bank Card Storage** | `Payment.paymentMethod` accepts Cash/UPI/DD/Transfer only | `payments.payment_method`<br>(No card number column) | Verified in `schema.sql` |
| **20** | **Complaint Ticketing** | `ComplaintService.registerComplaint()`<br>`ComplaintDaoImpl.create()` | `complaints` table | `ComplaintPanel.java` |
| **21** | **Complaint Status Updates** | `ComplaintService.updateComplaintStatus()`<br>`ComplaintDaoImpl.updateStatus()` | `complaints.status`<br>`complaints.resolution_details` | `ComplaintPanel.java` |
| **22** | **Executive Dashboard** | `DashboardService.getDashboardStats()`<br>`DashboardDao.getDashboardStats()` | SQL aggregate `COUNT(*)`, `SUM()` across tables | `DashboardPanel.java` |
| **23** | **OOP: Encapsulation** | `Student`, `Room`, `Payment` private fields & getters/setters | All Model classes | `PersonOopTest.testEncapsulation()` |
| **24** | **OOP: Inheritance** | `Person` -> `Student`, `Warden` | Model hierarchy | `PersonOopTest.testInheritance()` |
| **25** | **OOP: Polymorphism** | `displayDetails()`, `getRoleDescription()` overridden | `Person` subclasses | `PersonOopTest.testRuntimePolymorphism()` |
| **26** | **OOP: Abstraction** | `abstract class Person`<br>`StudentDao`, `RoomDao` interfaces | DAO interfaces | `PersonOopTest.java` / Code Inspection |
| **27** | **OOP: Exception Handling** | Custom hierarchy in `com.hostel.exception` | Try-with-resources | `PersonOopTest.java`<br>`UIHelper.showError()` |
| **28** | **SQL Injection Safety** | Parameterized `PreparedStatement` in all DAO implementations | All SQL queries | Code Inspection |
| **29** | **Local DB Credentials** | `DatabaseConfig.loadConfiguration()` via `db.properties` | Git-ignored config | `db.properties.example` |
