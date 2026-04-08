# Database Design Documentation: OnCall Attendance System

## 1. Project Overview
The OnCall System is designed to manage employee attendance through Geofencing technology. It ensures high-level governance by tracking real-time locations and calculating actual working hours to prevent time-theft and ensure compliance with company policies.


## 2. Database Schema (ERD)
The system is built on a relational database model designed for high precision and scalability.


## 3. Enumerations (Data Constraints)
To ensure data integrity and prevent invalid entries, the following Enums are implemented:

* **`ShiftDuration`**: [Eight_Hrs, Twelve_Hrs, TwentyFour_Hrs] - Standardizes shift lengths.
* **`NotificationType`**: [Push, Email, SMS, SystemAlert] - Manages communication channels.
* **`EmployeeLevel`**: [Intern, FreshGrade, Junior, TeamLeader, Senior, SectionHead] - Defines company hierarchy.
* **`LeaveStatus`**: [New, Pending, Review, Approved, Canceled, Postponed] - Tracks the lifecycle of requests.



## 4. Entity Dictionary & Justifications

### A. Organization & Identity
| Entity | Description |
-----------------------------
| **UserProfile** | Authentication & Account info. | **One-to-One with Employee:** Enforces that one physical employee has exactly one system identity for security. |
| **Roles** | Authorization levels. | Implements RBAC (Role-Based Access Control) to separate Admin, HR, and Staff permissions. |
| **Department / SubDepartment** | Organizational hierarchy. | Granular nesting allows for precise WorkZone assignments per sub-unit. |

### B. Geofencing & Tracking (The Core)
| Entity | Description | Technical Justification |
 ---------
| **WorkZoneLocation** | Pre-defined safe zones. | **Decimal(18,10):** Selected over `Double` to guarantee sub-meter precision in coordinate mapping. |
| **EmployeeLocation** | Raw GPS pings from mobile. | **Timestamp:** Records the exact moment of location capture to prevent historical data manipulation. |
| **WorkingRealTime** | Processed daily work logs. | **is_compliant (bool):** An automated flag that marks the day as 'valid' or 'violated' based on geofencing rules. |

### C. Compliance & Operations
| Entity | Description | Technical Justification |
------------
| **Attendance** | Check-in/out records. | Serves as the primary source for payroll, validated by `WorkingRealTime` data. |
| **Warning** | Automated disciplinary logs. | Ensures immediate accountability if an employee leaves the zone without permission. |
| **ErrorLog** | System health monitoring. | Captures stack traces for debugging API failures and location sync issues. |



## 5. Technical Design FAQs

### Q1: Why use `Decimal` for Latitude and Longitude?
**Answer:** In a Geofencing system, precision is non-negotiable. Floating-point types like `Double` can suffer from rounding errors during coordinate calculations. `Decimal` provides exact numeric storage, ensuring that the distance between an employee and the center of the WorkZone is calculated with millimeter accuracy.

### Q2: Why is the `MobileNumber` stored as a `Varchar`?
**Answer:** Storing phone numbers as integers removes the leading zero (e.g., `010...` becomes `10...`). Using `Varchar` preserves the zero, allows for international prefixes (e.g., `+20`), and follows the industry standard for non-computational numeric data.

### Q3: How does the system detect "Abuser Employee" (Absconding)?
**Answer:** The system doesn't just rely on the `Attendance` check-in. It cross-references `EmployeeLocation` pings against the `WorkZoneLocation`. If the location falls outside the radius for an unauthorized period, the `is_compliant` flag in `WorkingRealTime` is set to `false`, and an automated `Warning` is generated.

### Q4: Why use Enums for `Status` and `Type` fields?
**Answer:** Enums at the application level (ASP.NET Core) prevent "Magic Strings" and typos. While they are stored as `Int` or `Varchar` in the database, they provide a strict contract for the frontend and backend developers to follow.