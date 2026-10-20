# Helpdesk Management System

Java 17 + Spring Boot 3 + Spring Data JPA (Hibernate) + MySQL.
REST backend for raising, assigning, tracking and resolving support tickets.

## Modules (as in the presentation)
| Module | Package / class | What it does |
|---|---|---|
| User / Customer | `AuthController`, `UserService` | Register, login (HTTP Basic), view own tickets |
| Ticket Management | `TicketController`, `TicketService` | Create, view, comment, track status |
| Support Staff | `StaffController` | View assigned tickets, reply, set IN_PROGRESS / RESOLVED |
| Admin | `AdminController` | Manage users, assign tickets, monitor all tickets, reports |
| Notification | `NotificationService`, `NotificationController` | In-app notifications (+ optional email) on every event |

Workflow: Customer raises ticket (OPEN) -> admin notified -> admin assigns staff (IN_PROGRESS) -> staff and
customer notified -> staff and customer exchange comments -> staff marks RESOLVED -> customer notified ->
admin views the report.

## Requirements
JDK 17+, Maven 3.8+, MySQL 8 running on localhost:3306.

## Run
1. Edit `src/main/resources/application.properties` (or set `DB_USERNAME`, `DB_PASSWORD`, `ADMIN_PASSWORD`).
   The database `helpdesk_db` and all tables are created automatically.
2. Start the app:
   ```
   mvn spring-boot:run
   ```
3. API runs at http://localhost:8080. A default admin is created on first start:
   `admin@helpdesk.com` / `Admin@123` (change it in `application.properties`).

## Roles and URLs
| Role | Endpoints |
|---|---|
| Public | `POST /api/auth/register` |
| Any logged-in user | `GET /api/auth/me`, `GET /api/tickets/{id}`, `GET/POST /api/tickets/{id}/comments`, `GET /api/notifications`, `GET /api/notifications/unread-count`, `PUT /api/notifications/{id}/read` |
| CUSTOMER | `POST /api/tickets`, `GET /api/tickets/my` |
| STAFF | `GET /api/staff/tickets`, `PUT /api/staff/tickets/{id}/status` |
| ADMIN | `GET/POST /api/admin/users`, `PUT /api/admin/users/{id}/active?value=true|false`, `GET /api/admin/tickets[?status=OPEN]`, `PUT /api/admin/tickets/{id}/assign`, `PUT /api/admin/tickets/{id}/status`, `GET /api/admin/reports` |

Authentication is HTTP Basic (email + password). Passwords are stored BCrypt-hashed.

## Try the whole workflow with curl
```bash
# 1. Customer registers and raises a ticket
curl -X POST localhost:8080/api/auth/register -H "Content-Type: application/json" \
  -d '{"name":"Asha","email":"asha@mail.com","password":"asha123"}'

curl -u asha@mail.com:asha123 -X POST localhost:8080/api/tickets -H "Content-Type: application/json" \
  -d '{"title":"Cannot login","description":"Login page shows error 500","priority":"HIGH"}'

# 2. Admin creates a support staff account
curl -u admin@helpdesk.com:Admin@123 -X POST localhost:8080/api/admin/users -H "Content-Type: application/json" \
  -d '{"name":"Ravi","email":"ravi@helpdesk.com","password":"ravi123","role":"STAFF"}'

# 3. Admin views tickets and assigns ticket 1 to staff (use the staff id returned above)
curl -u admin@helpdesk.com:Admin@123 localhost:8080/api/admin/tickets
curl -u admin@helpdesk.com:Admin@123 -X PUT localhost:8080/api/admin/tickets/1/assign \
  -H "Content-Type: application/json" -d '{"staffId":2}'

# 4. Staff replies and resolves
curl -u ravi@helpdesk.com:ravi123 -X POST localhost:8080/api/tickets/1/comments \
  -H "Content-Type: application/json" -d '{"message":"Looking into it"}'
curl -u ravi@helpdesk.com:ravi123 -X PUT localhost:8080/api/staff/tickets/1/status \
  -H "Content-Type: application/json" -d '{"status":"RESOLVED"}'

# 5. Customer checks notifications; admin checks the report
curl -u asha@mail.com:asha123 localhost:8080/api/notifications
curl -u admin@helpdesk.com:Admin@123 localhost:8080/api/admin/reports
```
(On Windows CMD, use double quotes with escaped inner quotes, or use Postman.)

## Email notifications (optional)
Set `helpdesk.mail.enabled=true` and fill the `spring.mail.*` lines in `application.properties`.
With Gmail, use an App Password. In-app notifications work without any email setup.

## Project structure
```
src/main/java/com/helpdesk
  HelpdeskApplication.java
  config/      SecurityConfig, DataSeeder
  controller/  Auth, Ticket, Staff, Admin, Notification controllers
  service/     UserService, TicketService, NotificationService
  repository/  Spring Data JPA repositories
  model/       User, Ticket, Comment, Notification, Role, Status, Priority
  dto/         Dtos (request/response records)
  exception/   GlobalExceptionHandler
```

## Possible extensions (from the Future Scope slide)
JWT/OAuth2 login, SLA tracking with auto-escalation, WebSocket/SMS alerts, a web/mobile front end,
Docker + cloud deployment, charts and PDF/Excel report export.
