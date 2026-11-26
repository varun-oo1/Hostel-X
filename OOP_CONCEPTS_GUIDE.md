# Core Object-Oriented Programming (OOP) Demonstration Guide
## Hostel Management System - Faculty Evaluation Reference

This document maps the **6 core Java OOP concepts** directly to specific classes, methods, and line implementations in the codebase. Use this guide during your viva examination.

---

### Concept 1: ENCAPSULATION
**Definition**: Encapsulation is the mechanism of wrapping data (variables) and code (methods) into a single unit, and hiding internal object state using `private` access modifiers with controlled access through getters and setters.

* **Where implemented in Code**:
  - [`com.hostel.model.Student`](file:///src/main/java/com/hostel/model/Student.java): Private fields (`registerNumber`, `department`, `yearOfStudy`, `phone`, `email`, `emergencyContact`, `status`).
  - [`com.hostel.model.Room`](file:///src/main/java/com/hostel/model/Room.java): Private fields (`capacity`, `occupiedBeds`, `roomType`), with calculated business getter `getAvailableBeds()`.
  - [`com.hostel.model.Payment`](file:///src/main/java/com/hostel/model/Payment.java): Private fields (`amountDue`, `amountPaid`), with calculated business getter `getBalanceAmount()`.
  - [`com.hostel.util.ValidationUtil`](file:///src/main/java/com/hostel/util/ValidationUtil.java): Enforces validation before private fields are modified.
* **Viva Explanation**:
  > *"Faculty Question: Why did you declare fields private instead of public?"*
  > **Student Answer**: "If fields were public, any UI event handler or external class could set invalid values—such as a negative fee payment or a room capacity of -5. By keeping fields private and using getters and setters alongside service validation, we guarantee data integrity and protect internal object state."

---

### Concept 2: INHERITANCE
**Definition**: Inheritance is a mechanism where a new class (subclass/child) derives properties and behavior from an existing class (superclass/parent), promoting code reuse and establishing an *is-a* relationship.

* **Where implemented in Code**:
  - Superclass: [`com.hostel.model.Person`](file:///src/main/java/com/hostel/model/Person.java)
    - Contains common person attributes: `id`, `fullName`, `gender`, `phone`, `email`.
  - Subclasses:
    - [`com.hostel.model.Student`](file:///src/main/java/com/hostel/model/Student.java) (`public class Student extends Person`)
    - [`com.hostel.model.Warden`](file:///src/main/java/com/hostel/model/Warden.java) (`public class Warden extends Person`)
* **Code Proof**:
  ```java
  // Student inherits id, fullName, email, phone from Person via super(...)
  public Student(int id, String registerNumber, String fullName, ...) {
      super(id, fullName, gender, phone, email);
      this.registerNumber = registerNumber;
      ...
  }
  ```
* **Viva Explanation**:
  > *"Both Student and Warden share common human properties like Name, Phone, and Email. Instead of duplicating those fields in multiple classes, we created an abstract Person superclass. Student and Warden extend Person, gaining all common fields and adding role-specific attributes."*

---

### Concept 3: POLYMORPHISM
**Definition**: Polymorphism means "many forms". Specifically, **Runtime Polymorphism (Dynamic Method Dispatch)** allows a method call to an overridden method to be resolved dynamically at runtime rather than at compile time, based on the actual object instance.

* **Where implemented in Code**:
  - Overridden methods:
    - `displayDetails()` declared in [`Person`](file:///src/main/java/com/hostel/model/Person.java) and overridden with `@Override` in both [`Student`](file:///src/main/java/com/hostel/model/Student.java) and [`Warden`](file:///src/main/java/com/hostel/model/Warden.java).
    - `getRoleDescription()` and `getIdentificationNumber()`.
  - Dynamic Dispatch execution:
    - [`com.hostel.service.PersonService.generatePersonBadge(Person person)`](file:///src/main/java/com/hostel/service/PersonService.java)
    - [`com.hostel.model.PersonOopTest.testRuntimePolymorphism()`](file:///src/test/java/com/hostel/model/PersonOopTest.java)
* **Code Proof**:
  ```java
  Person p = new Student(...); // Upcasting to parent reference
  System.out.println(p.displayDetails()); // Calls Student's displayDetails() at runtime!
  
  p = new Warden(...);
  System.out.println(p.displayDetails()); // Calls Warden's displayDetails() at runtime!
  ```
* **Viva Explanation**:
  > *"We demonstrated runtime polymorphism by calling p.displayDetails() through a reference variable of type Person. At runtime, the Java Virtual Machine (JVM) dynamically inspects whether p points to a Student or a Warden and invokes the correct subclass method."*

---

### Concept 4: ABSTRACTION
**Definition**: Abstraction is the process of hiding complex implementation details and exposing only the essential interface to the user/caller.

* **Where implemented in Code**:
  1. **Abstract Classes**: [`com.hostel.model.Person`](file:///src/main/java/com/hostel/model/Person.java)
     - `public abstract class Person` defines abstract methods:
       - `public abstract String getRoleDescription();`
       - `public abstract String getIdentificationNumber();`
     - Concrete subclasses are forced to implement these methods.
  2. **Interface Abstraction**: [`com.hostel.dao.StudentDao`](file:///src/main/java/com/hostel/dao/StudentDao.java), [`RoomDao`](file:///src/main/java/com/hostel/dao/RoomDao.java), [`AllocationDao`](file:///src/main/java/com/hostel/dao/AllocationDao.java)
     - The Service and UI layers only interact with DAO interface contracts (`StudentDao.create()`), without knowing anything about MySQL driver syntax or SQL queries.
* **Viva Explanation**:
  > *"Abstraction is implemented in two ways: (1) An abstract Person class which cannot be instantiated directly and forces subclasses to define their role descriptions, and (2) DAO interfaces that completely abstract JDBC SQL queries away from the business and UI layers."*

---

### Concept 5: PACKAGES
**Definition**: Packages are namespaces that organize related classes and interfaces into modular, maintainable folder structures, preventing naming conflicts and enabling access control.

* **Package Layout in Project**:
  - `com.hostel.model`: Entity beans and data transfer objects (`Person`, `Student`, `Room`, `Allocation`, `Payment`, `Complaint`).
  - `com.hostel.dao`: Data access interfaces and JDBC implementations.
  - `com.hostel.service`: Business logic, validation, and transaction handling.
  - `com.hostel.ui`: Java Swing user interface panels, dialogs, and frames.
  - `com.hostel.config`: Database connection and credential management.
  - `com.hostel.exception`: Custom academic exception hierarchy.
  - `com.hostel.util`: Cryptographic utilities, date formatters, and regex validators.
* **Viva Explanation**:
  > *"We divided the project into 7 modular packages following the industry-standard 3-tier layered architecture (Presentation -> Service -> Data Access). This ensures separation of concerns and high maintainability."*

---

### Concept 6: EXCEPTION HANDLING
**Definition**: Exception handling is a structured mechanism in Java to handle runtime errors gracefully without causing the application to crash.

* **Where implemented in Code**:
  - Custom Exception Hierarchy in [`com.hostel.exception`](file:///src/main/java/com/hostel/exception/):
    - `AppException` (Root checked exception)
      - `DatabaseException` (wraps SQLExceptions)
      - `EntityNotFoundException` (when a record is absent)
      - `ValidationException` (input rule violation)
        - `DuplicateRecordException` (duplicate register numbers or rooms)
        - `RoomOverCapacityException` (over-capacity bed allocation)
  - `try-with-resources`: Automatically closes JDBC `Connection`, `PreparedStatement`, and `ResultSet` to prevent connection leaks.
  - User-Friendly UI Dialogs: Trapped exceptions in `SwingWorker.done()` are converted to clear error dialogs via `UIHelper.showError()`.
* **Code Proof**:
  ```java
  try (Connection conn = DatabaseConfig.getConnection();
       PreparedStatement stmt = conn.prepareStatement(sql)) {
      // Auto-closed upon block completion
      ...
  } catch (SQLException e) {
      throw new DatabaseException("Failed to load records: " + e.getMessage(), e);
  }
  ```
* **Viva Explanation**:
  > *"We avoid silently swallowing exceptions and never print raw technical stack traces to end users. All JDBC statements use try-with-resources for deterministic resource reclamation, and custom exceptions like RoomOverCapacityException and DuplicateRecordException provide precise error descriptions."*
