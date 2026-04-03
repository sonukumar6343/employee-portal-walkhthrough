# ENTNT Employee Management Portal — Full Codebase Walkthrough

## Table of Contents
- [1. High-Level Overview](#1-high-level-overview)
- [2. Solution Structure](#2-solution-structure)
- [3. Backend — .NET 8 Web API](#3-backend--net-8-web-api)
  - [3.1 Entry Point & Middleware Pipeline](#31-entry-point--middleware-pipeline)
  - [3.2 Authentication & Authorization](#32-authentication--authorization)
  - [3.3 Database Layer (Entity Framework Core)](#33-database-layer-entity-framework-core)
  - [3.4 Domain Models (55 Entities)](#34-domain-models-55-entities)
  - [3.5 Repository Pattern & Interfaces](#35-repository-pattern--interfaces)
  - [3.6 Controllers (32 API Controllers)](#36-controllers-32-api-controllers)
  - [3.7 DTOs & Mappers](#37-dtos--mappers)
  - [3.8 Background Services (17 Hosted Services)](#38-background-services-17-hosted-services)
  - [3.9 Helpers & Utilities](#39-helpers--utilities)
  - [3.10 Caching Layer](#310-caching-layer)
- [4. Frontend — React + TypeScript + Vite](#4-frontend--react--typescript--vite)
  - [4.1 Tech Stack & Dependencies](#41-tech-stack--dependencies)
  - [4.2 Application Bootstrap](#42-application-bootstrap)
  - [4.3 Authentication (MSAL)](#43-authentication-msal)
  - [4.4 Routing & Route Guards](#44-routing--route-guards)
  - [4.5 State Management (Context API)](#45-state-management-context-api)
  - [4.6 API Integration Layer (WebCalls)](#46-api-integration-layer-webcalls)
  - [4.7 Component Architecture](#47-component-architecture)
  - [4.8 Feature Modules](#48-feature-modules)
  - [4.9 Type System](#49-type-system)
  - [4.10 Permission System](#410-permission-system)
- [5. Console Applications](#5-console-applications)
  - [5.1 Invoice Generator](#51-invoice-generator)
  - [5.2 Weekly Report](#52-weekly-report)
- [6. CI/CD Pipeline (Azure DevOps)](#6-cicd-pipeline-azure-devops)
- [7. Data Flow Diagrams](#7-data-flow-diagrams)
- [8. Key Design Patterns](#8-key-design-patterns)
- [9. External Integrations](#9-external-integrations)
- [10. File Inventory](#10-file-inventory)

---

## 1. High-Level Overview

The **ENTNT Employee Management Portal** is a full-stack enterprise web application designed to manage the complete employee lifecycle at ENTNT. It handles:

- **Employee directory & profiles** (synced from Azure AD / Microsoft Entra ID)
- **Organizational hierarchy** — Departments → Teams → Employees with role-based assignments
- **Payroll & invoicing** — salary management, invoice generation, signing, and payment tracking
- **Time & attendance** — clock in/out, breaks, timesheets, work logs
- **Leave management** — sick/planned leave requests, approvals, balance tracking
- **Expense management** — trip budgets, expense claims, reimbursements
- **Missions & performance** — goal setting, performance scores, daily metrics
- **Notifications** — push notifications (VAPID/WebPush), daily/weekly/monthly/quarterly reports
- **Announcements & polls** — company-wide announcements with audience targeting
- **Policies** — policy management with version history
- **FAQ system** — categorized Q&A with public/private visibility
- **Articles & knowledge base** — employee-authored articles with comments, likes, skills tagging
- **Universal search** — cross-entity search with scoring and ranking
- **Contract & document management** — Aadhar, passport, employment contracts stored in Azure Blob Storage
- **Scheduling** — work schedule management with approval workflows

```mermaid
graph TB
    subgraph "Frontend (React + Vite)"
        UI["Employee Portal UI<br/>React 18 + TypeScript"]
        MSAL["MSAL.js<br/>Azure AD Auth"]
    end

    subgraph "Backend (.NET 8 API)"
        API["ASP.NET Core Web API<br/>32 Controllers"]
        SVC["17 Background Services<br/>(Hosted Services)"]
        REPO["37 Repositories<br/>(Repository Pattern)"]
    end

    subgraph "Data & External"
        DB["SQL Server<br/>(55+ Tables)"]
        GRAPH["Microsoft Graph API<br/>(User Sync, Photos)"]
        BLOB["Azure Blob Storage<br/>(Documents)"]
        KV["Azure Key Vault<br/>(Secrets)"]
    end

    subgraph "Scheduled Jobs"
        INV["Invoice Generator<br/>(Console App)"]
        RPT["Weekly Report<br/>(Console App)"]
    end

    UI --> MSAL
    UI -->|"REST API (Bearer Token)"| API
    API --> REPO --> DB
    API --> GRAPH
    API --> BLOB
    SVC --> DB
    SVC --> GRAPH
    INV -->|"HTTP + Bearer"| API
    RPT -->|"HTTP"| API
    INV --> KV
```

---

## 2. Solution Structure

The `.sln` file defines **3 .NET projects** plus a **standalone React frontend**:

```
ENTNT.EmployeeManagement/
├── ENTNT.EmployeeManagement.sln          # Visual Studio solution
├── azure-pipelines.yml                   # CI/CD pipeline definition
├── README.md                             # Project documentation
├── docs/                                 # Documentation
│   ├── DbTables.md                       # Database data dictionary
│   ├── EmployeePortal.md                 # Portal documentation
│   └── screenshots/                      # UI screenshots
│
├── entnt.employeemanagement.api/         # Backend API (Solution Folder)
│   └── entnt.employeemanagement.api/     # .NET 8 Web API Project
│       ├── Program.cs                    # Entry point & DI configuration
│       ├── appsettings.json              # Configuration
│       ├── Authorization/                # Auth handlers, policies, requirements
│       ├── Caching/                      # In-memory caching services
│       ├── Constants/                    # Role IDs and constants
│       ├── Controllers/                  # 32 API controllers
│       ├── Converters/                   # JSON datetime converters
│       ├── DbContexts/                   # EF Core DbContext (468 lines)
│       ├── DTOs/                         # 58+ Data Transfer Objects
│       ├── Factories/                    # Design-time DbContext factory
│       ├── Files/                        # Static file assets
│       ├── Helpers/                      # 20 utility/helper classes
│       ├── Interfaces/                   # 37 repository interfaces
│       ├── Mappers/                      # 9 entity-to-DTO mappers
│       ├── Migrations/                   # EF Core migrations
│       ├── Models/                       # 55 domain entity models
│       ├── Profiles/                     # AutoMapper profiles
│       ├── Repository/                   # 35 repository implementations
│       ├── Services/                     # 17 background hosted services
│       └── Types/                        # 8 enum/type definitions
│
├── entnt.employeemanagement.invoicegenerator/  # Scheduled invoice console app
│   └── Program.cs
│
├── entnt.employeemanagement.weeklyreport/      # Scheduled report console app
│   └── Program.cs
│
└── entnt.employeeportal.ui/             # React Frontend
    ├── package.json                     # Dependencies & scripts
    ├── vite.config.ts                   # Vite build configuration
    ├── tailwind.config.js               # TailwindCSS configuration
    ├── tsconfig.json                    # TypeScript configuration
    ├── index.html                       # HTML entry point
    └── src/
        ├── index.tsx                    # React bootstrap + MSAL provider
        ├── App.tsx                      # Root component, routing, auth flow
        ├── authProvider.tsx             # MSAL configuration & scopes
        ├── RouteGuard.tsx               # Role-based route protection
        ├── Model.tsx                    # 440+ lines of TypeScript interfaces
        ├── WebCalls.tsx                 # 3877 lines — all API call functions
        ├── Helpers.tsx                  # Utility functions
        ├── context/                     # React Context providers
        ├── hooks/                       # 12 custom hooks
        ├── components/                  # UI components (32+ files)
        ├── features/                    # Feature modules (Management, Announcement)
        ├── pages/                       # Page-level components
        ├── types/                       # 13 TypeScript type definition files
        ├── permissions/                 # Role-based permission definitions
        └── UI/                          # Reusable UI primitives
```

---

## 3. Backend — .NET 8 Web API

### 3.1 Entry Point & Middleware Pipeline

[Program.cs](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Program.cs) uses the minimal hosting model (top-level statements) and configures:

**Service Registration (DI Container):**
1. **Database**: `EmployeeDbContext` registered with both `AddDbContext` and `AddDbContextFactory` for SQL Server
2. **Authentication**: JWT Bearer via `Microsoft.Identity.Web` (Azure AD)
3. **Authorization**: Custom policy-based authorization with `MinimumAccessLevelPriorityHandler`
4. **Repositories**: 35+ scoped repository registrations (interface → implementation)
5. **Helpers**: `AuthHelper`, `ScheduleHelper`, `InvoiceGenerator`, `GraphApiCalls`
6. **Caching**: `IMemoryCache` + `OrgRolesCacheService`
7. **Mapping**: AutoMapper for entity ↔ DTO transformations
8. **Search**: `IEntitySearchHandler` registrations for multi-entity search
9. **External**: `GraphServiceClient` (singleton), `HttpClient`
10. **API Docs**: Swagger/OpenAPI with Bearer token authentication

**Middleware Pipeline:**
```
HttpsRedirection → CORS → Authentication → Authorization → MapControllers
```

**CORS Policy**: Allows `localhost:3000` (dev) and `myportal.entnt.in` (production).

### 3.2 Authentication & Authorization

#### Azure AD JWT Authentication
- Tokens are validated against Azure AD tenant `246d1166-294d-4f52-83df-388a351a6359`
- On `OnTokenValidated`, the system:
  1. Extracts the user's OID (Object ID) from the token
  2. Looks up the employee in the local database by `EmployeeId`
  3. Loads their `SystemRole` and injects custom claims:
     - `employee_id` — the Azure AD OID
     - `sys_priority` — numeric priority level (1–10)
     - `sys_role` — role code string (`VIEWER`, `ADMIN`)
  4. Rejects tokens for unregistered users

#### Authorization Policies
Defined in [AuthorizationPolicies.cs](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Authorization/Policies/AuthorizationPolicies.cs):

| Policy | System Priority | Org Role Priority | Purpose |
|--------|----------------|-------------------|---------|
| `ViewTeam` | ≥ 1 | ≥ 1 | Basic team viewing |
| `ManageTeam` | ≥ 5 | ≥ 6 | Team management |
| `ManageDepartment` | ≥ 7 | ≥ 9 | Department management |
| `SystemAdministration` | ≥ 9 | ≥ 11 | Full admin access |

The `MinimumAccessLevelPriorityHandler` evaluates whether the user's `sys_priority` claim meets the required threshold OR their organizational role priority (from the OrgRoles cache) does.

#### Dual Role System

| Role Type | Stored In | Purpose | Examples |
|-----------|-----------|---------|----------|
| **SystemRole** | `SystemRoles` table | Platform-level access | Viewer (1), Admin (10) |
| **OrgRole** | `OrgRoles` table | Organizational hierarchy | Employee (1), Task Lead (3), Team Lead (6), Acting Manager (8), Manager (10) |

### 3.3 Database Layer (Entity Framework Core)

[EmployeeDbContexts.cs](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/DbContexts/EmployeeDbContexts.cs) — 468 lines defining:

- **Connection**: SQL Server via `CareersContext` connection string
- **55 DbSet properties** exposing all entity tables
- **Seed data**: Pre-populates `SystemRoles` (Viewer, Admin) and `OrgRoles` (Employee, Manager, Acting Manager, Team Lead, Task Lead)
- **Fluent API configuration**: Extensive relationship mapping including:
  - Self-referencing `Employee → Manager` hierarchy
  - Many-to-many through history tables (Department ↔ Employee, Team ↔ Employee, Task ↔ Employee)
  - Composite keys, unique indexes, check constraints
  - Cascade vs. Restrict delete behaviors

### 3.4 Domain Models (55 Entities)

Organized into logical domains:

#### Core HR
| Model | Purpose |
|-------|---------|
| `Employee` | Central entity — Azure AD OID as PK, links to Manager, SystemRole, departments, teams, skills |
| `Department` | Organizational department with manager and created-by tracking |
| `DepartmentEmployeeHistory` | Temporal assignment of employees to departments with OrgRole |
| `Team` | Team within a department, has manager |
| `TeamEmployeeHistory` | Temporal assignment of employees to teams with OrgRole |
| `OrgTask` | Task within a team |
| `TaskEmployeeHistory` | Temporal assignment of employees to tasks |
| `JobRole` | Job title/role definition |
| `EmployeeJobRole` | Assignment of job roles to employees with effective dates |
| `SystemRole` | Platform access roles (Viewer, Admin) with priority 1–10 |
| `OrgRole` | Organizational hierarchy roles with priority 1–10 |

#### Payroll & Finance
| Model | Purpose |
|-------|---------|
| `Salary` | Base salary per employee |
| `BankDetail` | Employee bank account details (IFSC, SWIFT, PAN, GST) |
| `Invoice` | Monthly invoices with tracked documents |
| `InvoiceStatus` | Invoice status progression (Issued → Signed → Acknowledged → Paid) |
| `InvoiceStatusHistory` | Audit trail of status changes |
| `InvoiceConcern` | Concerns/disputes raised on invoices |
| `InvoiceConcernComment` | Comments on invoice concerns |
| `ConcernStatusHistory` | Status progression of concerns |
| `Expense` | Employee expense claims |
| `DefferedPayment` | Deferred payment tracking with leave context |
| `TripsBudget` | Budget allocation per employee for trips |

#### Time & Attendance
| Model | Purpose |
|-------|---------|
| `CheckInEvent` | Clock in/out events with actual timestamps |
| `BreakEvent` | Break start/end events within a shift |
| `TimeCardState` | Current clock state (clocked out/in/on break) |
| `Schedule` | Work schedule definitions (fixed/flexible) with approval |
| `Leave` | Leave requests (sick/planned) with approval flow |
| `WorkLog` | Hours logged against projects |

#### Communication & Content
| Model | Purpose |
|-------|---------|
| `Announcement` | Company announcements with type classification |
| `AnnouncementAudience` | Targeted audience (many-to-many with Employee) |
| `PollQuestion` / `PollOption` / `PollResponse` | In-announcement polls |
| `Notification` / `EmployeeNotification` | Push notification records |
| `NotificationSubscription` | WebPush subscription endpoints |
| `NotificationInterval` | Notification timing configuration |
| `Article` | Employee-authored knowledge base articles |
| `Comment` | Article comments |
| `ArticleSkill` / `ArticleLike` | Article metadata |
| `Policy` / `PolicyHistory` | Company policies with version control |
| `FAQCategory` / `FAQItem` | Categorized FAQ system |

#### Other
| Model | Purpose |
|-------|---------|
| `Mission` | Employee missions/goals with deadlines and rewards |
| `Contract` | Employment contract documents |
| `Client` / `Project` | Client and project tracking for work logs |
| `Watcher` | Manager watching employee activities |
| `ReportLog` | Audit log of generated reports |
| `Skill` / `EmployeeSkill` | Skill taxonomy and employee skill assignments |
| `EmployeePerformanceScore` | Daily performance metrics |
| `GraphSyncState` | Azure AD delta sync state tracking |
| `ProbationDetail` | Probation period details |
| `SystemProperties` | Key-value system configuration |

### 3.5 Repository Pattern & Interfaces

The API follows a strict **Repository Pattern** with 37 interface definitions in the [Interfaces/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Interfaces) directory and 35+ implementations in [Repository/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Repository).

All repositories are registered as **Scoped** services (one instance per HTTP request).

**Key repositories by complexity (file size):**

| Repository | Size | Key Operations |
|-----------|------|----------------|
| `NotificationRepository` | 51KB | Push notifications, subscriptions, distribution, read tracking |
| `AnnouncementRepository` | 44KB | CRUD, audience management, polls, analytics |
| `EmployeeRepository` | 37KB | Employee CRUD, Azure AD sync, department/team/job role assignment |
| `InvoiceRepository` | 31KB | Invoice lifecycle, signing, concerns, payment tracking |
| `LeaveRepository` | 30KB | Leave requests, approval, balance calculation, admin overrides |
| `CheckInRepository` | 24KB | Clock in/out, breaks, timesheet aggregation |
| `HierarchyRepository` | 16KB | Organizational tree building |

**Search System**: Uses a `Strategy Pattern` with `IEntitySearchHandler` implementations:
- `EmployeeSearchHandler`, `InvoiceSearchHandler`, `LeaveSearchHandler`
- `DepartmentSearchHandler`, `TeamSearchHandler`, `JobRoleSearchHandler`, `TimesheetSearchHandler`

Each handler implements entity-specific search logic, results are scored by `SearchScorer` and aggregated by `SearchRepository`.

### 3.6 Controllers (32 API Controllers)

All controllers follow the pattern: `[ApiController]` + `[Route("api/[controller]")]` + constructor DI.

| Controller | Endpoints | Domain |
|-----------|-----------|--------|
| `EmployeeController` | CRUD, department/team/job role assignment, bulk operations, Azure AD sync | Core HR |
| `DepartmentController` | Department CRUD with employee listing | Organization |
| `TeamController` | Team CRUD with member management | Organization |
| `JobRoleController` | Job role CRUD | Organization |
| `OrgRoleController` | Organization role management | Authorization |
| `SystemRoleController` | System role management | Authorization |
| `HierarchyController` | Org chart data retrieval | Organization |
| `InvoiceController` | Invoice lifecycle (issue, sign, acknowledge, pay, concerns) | Finance |
| `PayrollController` | Salary, bank details, probation management | Finance |
| `ExpenseController` | Expense CRUD, status updates, document storage | Finance |
| `TripsBudgetController` | Trip budget management | Finance |
| `CheckInController` | Clock in/out, break management | Time & Attendance |
| `TimeCardStateController` | Current timer state management | Time & Attendance |
| `ScheduleController` | Work schedule CRUD with approval workflow | Time & Attendance |
| `LeaveController` | Leave request CRUD, approval/rejection, admin apply | Leave Management |
| `MissionController` | Mission CRUD, completion tracking | Performance |
| `NotificationController` | Push notification management, subscription, stats | Notifications |
| `AnnouncementController` | Announcement CRUD, audience, polls, analytics | Communication |
| `PolicyController` | Policy CRUD with version history | Governance |
| `FAQCategoryController` | FAQ category management | Knowledge Base |
| `FAQController` | FAQ item CRUD, answering, pinning | Knowledge Base |
| `ArticleController` | Article CRUD, comments, likes, publishing | Knowledge Base |
| `SkillController` | Skill taxonomy, employee skill management | Skills |
| `WatcherController` | Watcher assignment and timesheet viewing | Monitoring |
| `ContractController` | Contract document upload/download (Azure Blob) | Documents |
| `AadharController` | Aadhar document management | Documents |
| `PassportController` | Passport document management | Documents |
| `SearchController` | Universal cross-entity search | Search |
| `ReportController` | Report generation trigger | Reporting |
| `ProjectsController` | Client and project management | Projects |
| `ConcernController` | Invoice concern management | Finance |
| `SystemPropertiesController` | System configuration key-value store | System |

### 3.7 DTOs & Mappers

**58+ DTOs** in [DTOs/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/DTOs) provide:
- Request DTOs (e.g., `CreateAnnouncementDto`, `AdminApplyLeaveDto`)
- Response DTOs (e.g., `EmployeeCurrentDetailsResponseDto`, `AnnouncementResponseDto`)
- Query objects (e.g., `EmployeeQueryObject`, `ExpenseQueryObject`)

**9 Manual mappers** in [Mappers/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Mappers) using extension methods (e.g., `employee.ToCurrentResponseDto()`).

**AutoMapper** is also configured via [InvoiceProfile.cs](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Profiles/InvoiceProfile.cs) for invoice-related mappings.

### 3.8 Background Services (17 Hosted Services)

> [!NOTE]
> All 17 hosted services are currently **commented out** in `Program.cs` (lines 167–181), likely to reduce resource usage during development. They are designed to run as `BackgroundService` instances.

| Service | Schedule | Purpose |
|---------|----------|---------|
| `NotificationService` | Continuous | Push notification delivery engine |
| `DailyNotificationService` | Daily | Daily notification aggregation & delivery |
| `SalaryReminderService` | Monthly | Salary payment reminders |
| `MonthlyPayrollService` | Monthly | Payroll calculation & processing (26KB) |
| `MonthlyInvoiceIssuerService` | Monthly | Auto-generate employee invoices |
| `WeeklyReportService` | Weekly | Weekly performance/activity reports (30KB) |
| `MonthlyReportService` | Monthly | Monthly reports (30KB) |
| `QuarterlyReportService` | Quarterly | Quarterly analytics reports (32KB) |
| `InvoiceConcernAutoCloseService` | Periodic | Auto-close stale invoice concerns |
| `PaidInvoiceStatusService` | Periodic | Auto-detect paid invoices |
| `InvoiceAcknowledgmentReminderService` | Periodic | Remind employees to acknowledge invoices |
| `InvoiceSigningReminderService` | Periodic | Remind employees to sign invoices |
| `ClockoutService` | Daily | Auto clock-out at end of day |
| `DailyPerformanceScoreService` | Daily | Calculate daily performance metrics |
| `AnnouncementNotificationService` | Event-driven | Send notifications for new announcements |
| `AverageResponseTimeNotificationService` | Periodic | Track notification response metrics |
| `EmployeeSyncService` | Periodic | Delta sync with Azure AD via Microsoft Graph |

### 3.9 Helpers & Utilities

20 helper classes in [Helpers/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Helpers):

| Helper | Size | Purpose |
|--------|------|---------|
| `InvoiceGenerator` | 25KB | PDF invoice generation using PdfSharpCore & DocX |
| `GraphApiCalls` | 16KB | Microsoft Graph API wrapper (user sync, photos, directory) |
| `LeaveReport` | 14KB | Leave balance calculations and reporting |
| `SendConcernNotificationEmail` | 12KB | Email notifications for invoice concerns |
| `EmailReportBuilder` | 8KB | HTML email template builder for reports |
| `AuthHelper` | 4KB | Authentication/authorization utility methods |
| `NotificationDistributionHelper` | 4KB | WebPush notification delivery |
| `SearchHelper` / `SearchScorer` | 3KB each | Search query parsing, result scoring |
| `CurrencyConverter` | 2KB | Currency conversion utilities |
| `EmailHelper` | 3KB | SMTP email sending |
| `TimeZoneConverter` | 1.5KB | UTC ↔ IST timezone conversion |
| `ScheduleHelper` | 1KB | Schedule validation logic |
| `QueryObject` | 2KB | Pagination & filtering base class |

### 3.10 Caching Layer

Located in [Caching/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.api/entnt.employeemanagement.api/Caching):
- `OrgRolesCacheService` — Caches organizational role lookups using `IMemoryCache` to avoid repeated DB queries during authorization checks.

---

## 4. Frontend — React + TypeScript + Vite

### 4.1 Tech Stack & Dependencies

| Category | Technology | Version |
|----------|-----------|---------|
| **Framework** | React | 18.2 |
| **Language** | TypeScript | 4.9.5 |
| **Build Tool** | Vite | 7.2.4 |
| **CSS** | TailwindCSS | 3.4.18 |
| **UI Library** | Ant Design (antd) | 5.29.1 |
| **Authentication** | @azure/msal-browser + msal-react | 3.7 / 2.0.9 |
| **Routing** | react-router-dom | 6.21.3 |
| **Icons** | @heroicons/react, lucide-react | latest |
| **UI Components** | @headlessui/react | 1.7.18 |
| **Rich Text** | @tiptap/react + starter-kit | 3.20 |
| **PDF Viewer** | react-pdf | 10.2 |
| **Charts** | react-chartjs-2 | 5.3.1 |
| **Calendar** | react-big-calendar | 1.19.4 |
| **Date Picker** | react-datepicker | 6.1 |

### 4.2 Application Bootstrap

```mermaid
graph TD
    A["index.tsx"] -->|"Creates MSAL instance"| B["MsalProvider"]
    B --> C["App.tsx"]
    C -->|"handleLogin()"| D{"Authenticated?"}
    D -->|No| E["Redirect to Azure AD login"]
    D -->|Yes| F["Check ID Token Roles"]
    F --> G["ThemeProvider"]
    G --> H["ConfigProvider (Ant Design)"]
    H --> I["NotificationProvider"]
    I --> J["AuthProvider"]
    J --> K["Navbar + Routes"]
```

[index.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/index.tsx):
1. Creates a `PublicClientApplication` with MSAL config
2. Wraps `<App />` in `<MsalProvider>`

[App.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/App.tsx) (448 lines):
1. Handles the MSAL redirect flow
2. Checks for `Admin`/`Manager` app roles
3. Sets up the provider hierarchy
4. Defines all application routes
5. Listens for service worker messages (notification sounds)
6. Observes theme changes via `MutationObserver`

### 4.3 Authentication (MSAL)

[authProvider.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/authProvider.tsx) configures:

| Config | Value |
|--------|-------|
| Client ID | `df2beffc-d76e-44b6-81d2-616e4b01c3ad` |
| Authority | `https://login.microsoftonline.com/246d1166-...` |
| Redirect URI | `http://localhost:3000/` (dev) |
| Cache | `sessionStorage` |
| Login Scopes | `User.Read`, `email`, `profile` |
| API Scopes | `api://1d9915c6-.../Employee.Read` |

**App Roles** (from Azure AD App Registration):
- `Application.Employee.Admin` — Full administrative access
- `Application.Employee.Manager` — Manager-level access

### 4.4 Routing & Route Guards

[RouteGuard.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/RouteGuard.tsx) implements role-based route protection:
1. Checks if user has required app roles (`Admin`/`Manager`)
2. For `Manager` role without `Admin`, validates the target user is a direct report via Microsoft Graph API
3. Allows self-access (user viewing their own profile)
4. Shows `<Unauthorized />` component for denied access

**Route Map:**

| Path | Component | Access |
|------|-----------|--------|
| `/` | `EmployeeDetails` | Authenticated |
| `/:userId` | `EmployeeDetails` (other user) | Admin/Manager |
| `/search` | `SearchResultsPage` | Admin/Manager |
| `/expenses` | `ExpensesDashboard` | Admin/Manager |
| `/payments` | `PaymentDashboard` | Admin/Manager |
| `/missions` | `MissionsDashboard` | Admin/Manager |
| `/manage-employees/*` | `ManagementEmployeeDashboard` | Admin/Manager |
| `/employee/:id/departments` | `EmployeeDepartment` | Admin/Manager |
| `/employee/:id/teams` | `EmployeeTeam` | Admin/Manager |
| `/employee/:id/jobRole` | `EmployeeJobRole` | Admin/Manager |
| `/management` | `Notifications` | Authenticated |
| `/notifications` | `Notifications` | Admin/Manager |
| `/notifications/:id` | `NotificationsOfEmployee` | Admin/Manager |
| `/schedules` | `ScheduleDashboard` | Authenticated |
| `/leaves` | `LeavesDashboard` | Admin/Manager |
| `/announcements` | `AnnouncementsDashboard` | Admin/Manager |
| `/operations` | `OperationsDashboard` | Admin/Manager |
| `/policies` | `PolicyPage` | Authenticated |
| `/faq` | `UserFAQPage` | Authenticated |
| `/articles` | `ArticlesFeed` | Authenticated |
| `/articles/:id` | `ArticleDetail` | Authenticated |
| `/articles/create` | `CreateArticle` | Authenticated |
| `/articles/my` | `MyArticles` | Authenticated |
| `/articles/edit/:id` | `EditArticle` | Authenticated |

> [!IMPORTANT]
> Several management features (`ManagementEmployeeDashboard`, `EmployeeDepartment`, `EmployeeTeam`, `EmployeeJobRole`) use **lazy loading** via `React.lazy()` for code splitting.

### 4.5 State Management (Context API)

Three React Context providers manage global state:

#### AuthContext
[AuthContext.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/context/AuthContext.tsx) exposes:
- `isAuthorizedUser` — is viewing own profile
- `isAuthorizedManager` — is the manager of the viewed employee
- `isAuthorizedAdmin` — has admin role
- `isManager` — has manager role (regardless of current view)

Logic: Checks Azure AD roles from ID token claims and validates direct report relationship via Graph API.

#### ThemeContext
[ThemeContext.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/context/ThemeContext.tsx):
- Manages `light`/`dark` theme toggle
- Persists preference to `localStorage`
- Adds/removes `dark` class on `<html>` element (TailwindCSS dark mode)

#### NotificationContext
[NotificationContext.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/context/NotificationContext.tsx):
- Provides `success()`, `error()`, `info()` toast notification methods
- Auto-dismisses after 5 seconds
- Uses `@headlessui/react` `Transition` for animations

### 4.6 API Integration Layer (WebCalls)

[WebCalls.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/WebCalls.tsx) — **3877 lines, 101KB** — is the centralized API client containing:

- **Microsoft Graph API calls**: Profile photos, user details, managers, direct reports, directory, timecards
- **Backend API calls**: CRUD operations for every domain entity
- Two API base URLs:
  - `APIURL` = `https://localhost:7161/api` (dev)
  - Production: `https://entntemployeeapi-d0fccdfzdrdfcdhf.centralindia-01.azurewebsites.net/api`

All functions accept an auth `token` parameter and use `fetch()` with `Bearer` authorization headers.

### 4.7 Component Architecture

#### Core Components (32+ files in `components/`)

| Component | Size | Purpose |
|-----------|------|---------|
| `EmployeeDetails` | 65KB | Main employee profile page — personal info, bank details, salary, invoices, expenses, timeline |
| `PaymentDashboard` | 79KB | Invoice and payment management dashboard |
| `LeavesDashboard` | 76KB | Leave management with calendar, filters, bulk actions |
| `OperationsDashboard` | 52KB | Operational metrics and management |
| `Notifications` | 52KB | Notification management and settings |
| `MissionsDashboard` | 39KB | Mission/goal management interface |
| `NotificationsOfEmployee` | 37KB | Per-employee notification history |
| `ScheduleDashboard` | 35KB | Work schedule management with calendar view |
| `PolicyPage` | 28KB | Policy listing, creation, version history |
| `Leaves` | 28KB | Individual leave management |
| `ClockInOutModal` | 25KB | Time clock with break tracking |
| `Missions` | 23KB | Mission cards and management |
| `Navbar` | 20KB | Navigation bar with search, user menu, theme toggle |
| `AnnouncementModal` | 20KB | Announcement creation with polls, audience targeting |
| `TripsBudget` | 20KB | Trip budget management |
| `Profile` | 42KB | Detailed employee profile view |
| `EmployeeSkills` | 19KB | Skill management and display |
| `Summary` | 18KB | Employee summary dashboard |
| `Timesheets` | 17KB | Timesheet viewing and management |
| `PolicyModal` / `PolicyHistoryViewer` | 13KB / 9KB | Policy editing and history |
| `Watcher` / `WatcherTimesheet` | 14KB / 6KB | Watcher feature for managers |
| `WorkSchedule` / `WorkEntryForm` | 13KB / 10KB | Schedule display and work logging |
| `Announcements` / `AnnouncementList` | 15KB / 10KB | Announcement browsing |

#### Sub-Component Directories

| Directory | Contents |
|-----------|----------|
| `Employee/` | `Directory.tsx`, `DirectoryFilter.tsx`, `Aadhar.tsx`, `Contract.tsx`, `Passport.tsx`, `AcceptNotificationModal.tsx` |
| `Payment/` | `PaymentDashboard.tsx` (79KB — complete payment management) |
| `Search/` | `SearchBar.tsx`, `SearchShared.tsx`, `searchConstants.ts`, `rows/` |
| `article/` | `ArticleCard.tsx`, `ArticleCommentsSection.tsx` |
| `Shared/` | Reusable components (Spinner, Unauthorized, AuthUserIdWrapper, etc.) |
| `styles/` | Component-specific CSS |
| `ui/` | Base UI primitives |

### 4.8 Feature Modules

Two feature modules follow a **domain-driven** folder structure:

#### Management Feature (`features/Management/`)
```
Management/
├── components/     # Management-specific UI components
├── hooks/          # Custom hooks for management operations
├── pages/          # Dashboard, EmployeeDepartment, EmployeeTeam, EmployeeJobRole
├── services/       # API service wrappers
├── types/          # TypeScript types
└── utils/          # Utility functions
```

#### Announcement Feature (`features/announcement/`)
```
announcement/
├── components/     # Announcement-specific UI components
├── hooks/          # Custom hooks for announcement operations
├── pages/          # AnnouncementsDashboard
├── services/       # API service wrappers
├── types/          # TypeScript types
└── utils/          # Utility functions
```

### 4.9 Type System

**13 TypeScript type files** in [types/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/types):

| File | Key Types |
|------|-----------|
| `EmployeesTypes.ts` | Employee interfaces, query objects |
| `DepartmentTypes.ts` | Department, assignment DTOs |
| `TeamTypes.ts` | Team, member history |
| `DesignationTypes.ts` | Job role types |
| `AnnouncementTypes.ts` | Announcement, poll, audience types |
| `SystemRoleTypes.ts` | System role definitions |
| `OrgType.ts` | Organization role types |
| `SkillTypes.ts` | Skill master and employee skill types |
| `AdminDashboardTypes.ts` | Dashboard statistics types |
| `articleTypes.ts` | Article CRUD types |
| `commentTypes.ts` | Comment types |
| `CompanyDetails.ts` | Company information types |
| `EmployeeJobRoleTypes.ts` | Job role assignment types |

**Core model file**: [Model.tsx](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/Model.tsx) — 445 lines defining all shared interfaces and enums (invoices, expenses, leaves, schedules, FAQ, search, etc.)

### 4.10 Permission System

[permissions/](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeeportal.ui/src/permissions):
- `permissions.ts` — Defines role-based permissions matrix:
  - **Admin**: Full CRUD on users, reports, settings
  - **Editor**: View users + edit reports
  - **Viewer**: View dashboard + reports only
- `Permission.tsx` — Permission component wrapper
- `hooks/` — Permission checking hooks

### 4.11 Custom Hooks (12 hooks)

| Hook | Purpose |
|------|---------|
| `UseApiToken` | Acquires API bearer token via MSAL |
| `UseGraphApiToken` | Acquires Microsoft Graph API token |
| `UseCurrentAccount` | Gets the current MSAL account |
| `UseClickOutside` | Detects clicks outside a ref element |
| `UseReadTime` | Calculates article read time |
| `useConfirmModal` | Reusable confirmation dialog |
| `useDebouncedValue` | Debounces a value for search inputs |
| `useFormatDate` | Date formatting utility |
| `useInitials` | Extracts name initials for avatars |
| `useNotification` | Access `NotificationContext` |
| `useSearchSummary` | Search summary statistics |
| `handleCustomRanges` | Custom date range picker logic |

---

## 5. Console Applications

### 5.1 Invoice Generator
[Program.cs](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.invoicegenerator/Program.cs)

- **Purpose**: Automated monthly invoice issuance (scheduled: `0 0 0 20 * *` — 20th of each month)
- **Flow**:
  1. Authenticates as a confidential client against Azure AD
  2. Retrieves client secret from Azure Key Vault (`entnt-employeeportal.vault.azure.net`)
  3. Calls `POST /api/Invoice/Issue` on the production API
- **Authentication**: Uses `MSAL ConfidentialClientApplication` with client credentials flow

### 5.2 Weekly Report
[Program.cs](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/entnt.employeemanagement.weeklyreport/Program.cs)

- **Purpose**: Triggers weekly report generation
- **Flow**: Calls `GET /api/Report/send` on the production API
- **Note**: Currently simplified to a direct HTTP call without authentication (auth code is commented out)

---

## 6. CI/CD Pipeline (Azure DevOps)

[azure-pipelines.yml](file:///d:/Entnt2/Employee-Managment/ENTNT.EmployeeManagement/azure-pipelines.yml) defines a two-job pipeline triggered on `main` branch:

### Job 1: Employee API (Build & Deploy)
```mermaid
graph LR
    A["Install .NET 8 SDK"] --> B["Install EF Core Tools"]
    B --> C["Download Secrets<br/>(Azure Key Vault)"]
    C --> D["Generate Migration"]
    D --> E{"Changes<br/>Detected?"}
    E -->|Yes| F["Push Migration<br/>to main"]
    F --> G["Update Database"]
    E -->|No| H["Skip Migration"]
    G --> I["dotnet publish"]
    H --> I
    I --> J["Deploy to Azure<br/>Web App"]
```

Key behaviors:
- Auto-generates EF Core migrations from DbContext changes
- Detects empty migrations (no schema changes) and removes them
- Pushes migration files back to `main` with `[ci skip]` to prevent infinite loops
- Applies migrations to production database using Key Vault connection string
- Deploys to `entntemployeeapi` Azure Web App

### Job 2: Employee Portal (Build & Deploy)
```mermaid
graph LR
    A["Install Node.js 20"] --> B["npm install"]
    B --> C["npm run build<br/>(tsc + vite build)"]
    C --> D["Deploy to Azure<br/>Web App"]
```

Deploys to `entntemployeeportal` Azure Web App.

---

## 7. Data Flow Diagrams

### Authentication Flow
```mermaid
sequenceDiagram
    participant User
    participant UI as React App
    participant MSAL as MSAL.js
    participant AAD as Azure AD
    participant API as .NET API
    participant DB as SQL Server

    User->>UI: Navigate to portal
    UI->>MSAL: Check authentication
    MSAL->>AAD: Redirect to login
    AAD-->>User: Login page
    User->>AAD: Enter credentials
    AAD-->>MSAL: ID Token + Access Token
    MSAL-->>UI: Authenticated
    UI->>API: API call (Bearer token)
    API->>API: Validate JWT (OnTokenValidated)
    API->>DB: Lookup Employee by OID
    DB-->>API: Employee + SystemRole
    API->>API: Inject custom claims
    API-->>UI: Response
```

### Invoice Lifecycle
```mermaid
stateDiagram-v2
    [*] --> Issued: MonthlyInvoiceIssuerService<br/>(auto-generated on 20th)
    Issued --> Signed: Employee signs
    Signed --> Acknowledged: Admin acknowledges
    Acknowledged --> Paid: Payment confirmed

    Issued --> ConcernRaised: Employee raises concern
    ConcernRaised --> ConcernResolved: Admin resolves
    ConcernResolved --> Signed: Employee re-signs

    note right of Issued: InvoiceGenerator creates PDF
    note right of Paid: PaidInvoiceStatusService auto-detects
```

### Employee Clock In/Out Flow
```mermaid
sequenceDiagram
    participant Emp as Employee
    participant UI as ClockInOutModal
    participant API as CheckInController
    participant DB as Database

    Emp->>UI: Click "Clock In"
    UI->>API: POST /api/CheckIn/clockin
    API->>DB: Create CheckInEvent
    API->>DB: Update TimeCardState → 1 (Clocked In)
    API-->>UI: Success

    Emp->>UI: Click "Start Break"
    UI->>API: POST /api/CheckIn/break/start
    API->>DB: Create BreakEvent (start)
    API->>DB: Update TimeCardState → 2 (On Break)

    Emp->>UI: Click "End Break"
    UI->>API: POST /api/CheckIn/break/end
    API->>DB: Update BreakEvent (end)
    API->>DB: Update TimeCardState → 1 (Clocked In)

    Emp->>UI: Click "Clock Out" + Log Work
    UI->>API: POST /api/CheckIn/clockout
    API->>DB: Create ClockOut event
    API->>DB: Save WorkLogs
    API->>DB: Update TimeCardState → 0 (Clocked Out)
```

---

## 8. Key Design Patterns

| Pattern | Usage |
|---------|-------|
| **Repository Pattern** | All data access encapsulated behind interfaces (`IEmployeeRepository → EmployeeRepository`) |
| **Dependency Injection** | Entire DI container configured in `Program.cs` with Scoped/Transient/Singleton lifetimes |
| **Strategy Pattern** | `IEntitySearchHandler` implementations for polymorphic entity search |
| **DTO Pattern** | Clean separation between API contracts and domain models |
| **Extension Methods** | Mappers implemented as extension methods (`employee.ToResponseDto()`) |
| **Feature Module Pattern** | Frontend `features/` directory isolates Management and Announcement domains |
| **Provider Pattern** | React Context providers for Auth, Theme, Notifications |
| **Route Guard Pattern** | `RouteGuard` component wraps protected routes with role checks |
| **Lazy Loading** | `React.lazy()` for management pages to reduce initial bundle size |
| **Hosted Services** | `BackgroundService` implementations for scheduled tasks |
| **CQRS-lite** | Separate DTOs for commands (create/update) vs. queries (response) |

---

## 9. External Integrations

| Service | Purpose | Configuration |
|---------|---------|---------------|
| **Azure Active Directory** | User authentication, app roles | `AzureAd` section in appsettings |
| **Microsoft Graph API** | User profiles, photos, org hierarchy, directory sync | Graph SDK v5.78 + delegated/app permissions |
| **Azure Blob Storage** | Contract, Aadhar, Passport document storage | `StorageAccount` connection string |
| **Azure Key Vault** | Secrets management for console apps | `entnt-employeeportal.vault.azure.net` |
| **Azure App Service** | Hosting for API and UI | Deployed via CI/CD pipeline |
| **WebPush (VAPID)** | Browser push notifications | VAPID keys in appsettings |
| **Open Exchange Rates** | Currency conversion data | Public API call |

---

## 10. File Inventory

### Backend Statistics
| Category | Count |
|----------|-------|
| Controllers | 32 |
| Domain Models | 55 |
| Repository Interfaces | 37 |
| Repository Implementations | 35+ |
| DTOs | 58+ |
| Mappers | 9 |
| Background Services | 17 |
| Helpers | 20 |
| Type/Enum Definitions | 8 |
| DbContext Tables (DbSets) | 55 |

### Frontend Statistics
| Category | Count |
|----------|-------|
| Components | 32+ files |
| Pages | 10+ |
| Custom Hooks | 12 |
| Type Definition Files | 13 |
| Context Providers | 3 |
| Feature Modules | 2 |
| API Functions (WebCalls) | 100+ |
| Lines of Code (WebCalls.tsx) | 3,877 |
| Lines of Code (Model.tsx) | 445 |

### NuGet Packages (Backend)
| Package | Purpose |
|---------|---------|
| `Microsoft.EntityFrameworkCore.SqlServer` 8.0.2 | SQL Server ORM |
| `Microsoft.Identity.Web` 2.17.0 | Azure AD authentication |
| `Microsoft.Graph` 5.78.0 | Microsoft Graph SDK |
| `AutoMapper` 12.0.1 | Object mapping |
| `Swashbuckle.AspNetCore` 6.5.0 | Swagger/OpenAPI |
| `Azure.Storage.Blobs` 12.19.1 | Blob storage |
| `PdfSharpCore` 1.3.63 | PDF generation |
| `DocX` 3.0.0 | Word document generation |
| `FreeSpire.Doc` 12.2.0 | Document processing |
| `WebPush` 1.0.12 | VAPID push notifications |

---

> [!TIP]
> **For new developers**: Start by understanding the `Employee` model (the central entity), the `Program.cs` DI configuration, and the `WebCalls.tsx` API layer. These three files form the backbone of the entire system.

> [!WARNING]
> The `appsettings.json` contains hardcoded secrets (Azure AD client secret, storage account key, VAPID keys). These should be migrated to Azure Key Vault or environment variables for production security.
