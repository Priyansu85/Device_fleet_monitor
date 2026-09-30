# Device Fleet Monitor

A lightweight device fleet monitoring system that registers devices, collects health telemetry, detects online/offline status, and generates alerts for abnormal resource usage.

## 1. Project Overview

The Device Fleet Monitor provides centralized visibility into a fleet of computers/devices.

### Core workflow

```text
Device Agent
    |
    | Heartbeat + telemetry
    v
FastAPI Backend
    |
    +----> Device Management
    +----> Status Monitoring
    +----> Alert Engine
    |
    v
PostgreSQL
    |
    v
Dashboard / API Consumers
```

## 2. Main Features

- Device registration and management
- Create, read, update, and delete device records
- Device heartbeat
- CPU usage monitoring
- Memory usage monitoring
- Disk usage monitoring
- Battery monitoring when available
- Device online/offline detection
- Historical telemetry storage
- Automatic threshold-based alerts
- Alert deduplication
- Alert resolution
- Dashboard summary API
- FastAPI Swagger/OpenAPI documentation
- PostgreSQL persistence

## 3. Technology Stack

### Backend
- Python
- FastAPI
- SQLAlchemy
- Pydantic
- psycopg2-binary

### Database
- PostgreSQL

### Device Agent
- Python
- psutil
- requests

### Planned / Extendable Frontend
- React.js
- Tailwind CSS
- Recharts

## 4. Project Structure

```text
device-fleet-monitor/
│
├── backend/
│   ├── venv/
│   ├── main.py
│   ├── database.py
│   │
│   ├── models/
│   │   ├── device.py
│   │   ├── telemetry.py
│   │   └── alert.py
│   │
│   ├── schemas/
│   │   ├── device.py
│   │   └── telemetry.py
│   │
│   ├── routes/
│   │   ├── devices.py
│   │   ├── telemetry.py
│   │   ├── alerts.py
│   │   └── dashboard.py
│   │
│   └── services/
│       ├── monitor.py
│       └── alerts.py
│
└── device-agent/
    ├── venv/
    └── agent.py
```

## 5. Database Design

### devices

Stores registered device information.

```text
id
device_uid
name
hostname
ip_address
os
device_type
location
status
last_seen
created_at
```

### telemetry

Stores time-based device health measurements.

```text
id
device_id
cpu_usage
memory_usage
disk_usage
temperature
battery
network_latency
timestamp
```

### alerts

Stores detected monitoring incidents.

```text
id
device_id
alert_type
severity
message
value
threshold
status
created_at
resolved_at
```

Relationship:

```text
devices 1 ---- * telemetry
devices 1 ---- * alerts
```

## 6. Installation

### Prerequisites

Install:

- Python 3.x
- PostgreSQL
- VS Code (recommended)

### Create the database

Create a PostgreSQL database named:

```text
device_fleet_db
```

The default PostgreSQL port is:

```text
5432
```

### Backend environment

From the backend folder:

```cmd
python -m venv venv
venv\\Scripts\\activate
python -m pip install fastapi uvicorn sqlalchemy psycopg2-binary
```

### Device agent environment

From the device-agent folder:

```cmd
python -m venv venv
venv\\Scripts\\activate
python -m pip install psutil requests
```

## 7. Database Configuration

In `backend/database.py`, configure:

```python
DATABASE_URL = "postgresql+psycopg2://postgres:YOUR_PASSWORD@localhost:5432/device_fleet_db"
```

Replace `YOUR_PASSWORD` with the PostgreSQL password.

> For a real deployment, credentials should be stored in environment variables instead of source code.

## 8. Run the Backend

From `device-fleet-monitor\\backend`:

```cmd
venv\\Scripts\\activate
venv\\Scripts\\python.exe -m uvicorn main:app
```

API:

```text
http://127.0.0.1:8000
```

Swagger documentation:

```text
http://127.0.0.1:8000/docs
```

## 9. Run the Device Agent

The device must already be registered with the backend.

In `device-agent/agent.py`:

```python
DEVICE_UID = "LAPTOP-001"
```

Start the agent:

```cmd
cd device-agent
venv\\Scripts\\activate
python agent.py
```

The agent periodically collects metrics and sends them to:

```text
POST /telemetry/heartbeat
```

## 10. API Endpoints

### Device Management

```text
POST   /devices/
GET    /devices/
GET    /devices/{device_id}
PUT    /devices/{device_id}
DELETE /devices/{device_id}
```

### Telemetry

```text
POST /telemetry/heartbeat
GET  /telemetry/{device_id}
```

### Alerts

```text
GET /alerts/
GET /alerts/active
```

### Dashboard

```text
GET /dashboard/summary
```

## 11. Example: Register a Device

```http
POST /devices/
```

```json
{
  "device_uid": "LAPTOP-001",
  "name": "Developer Laptop",
  "hostname": "MY-PC",
  "ip_address": "192.168.1.10",
  "os": "Windows",
  "device_type": "Laptop",
  "location": "Lab-A"
}
```

Example response:

```json
{
  "message": "Device registered successfully",
  "device_id": 1,
  "device_uid": "LAPTOP-001"
}
```

## 12. Heartbeat and Monitoring

Example heartbeat payload:

```json
{
  "device_uid": "LAPTOP-001",
  "cpu_usage": 42.5,
  "memory_usage": 61.2,
  "disk_usage": 58.7,
  "temperature": null,
  "battery": 82.0,
  "network_latency": null
}
```

When a valid heartbeat is received:

1. The backend finds the registered device.
2. The device status changes to `ONLINE`.
3. `last_seen` is updated.
4. Telemetry is stored in PostgreSQL.
5. Alert rules are evaluated.

## 13. Offline Detection

Current prototype configuration:

```text
Heartbeat interval: 10 seconds
Offline threshold: 30 seconds
Monitoring check interval: 10 seconds
```

If no heartbeat is received for more than the threshold, the device is marked:

```text
OFFLINE
```

When heartbeats resume:

```text
ONLINE
```

## 14. Alert Rules

Current threshold-based rules:

```text
CPU usage       >= 90%  -> HIGH_CPU
Memory usage    >= 90%  -> HIGH_MEMORY
Disk usage      >= 90%  -> HIGH_DISK
Temperature     >= 80 C -> HIGH_TEMPERATURE
```

Alerts use:

```text
ACTIVE
RESOLVED
```

Active alerts are updated instead of creating a new alert for every heartbeat, reducing alert spam.

## 15. Dashboard Summary

The dashboard summary endpoint provides fleet-level KPIs.

Example:

```json
{
  "total_devices": 10,
  "online_devices": 8,
  "offline_devices": 2,
  "active_alerts": 3
}
```

## 16. Demo Flow

```text
1. Start PostgreSQL
2. Start FastAPI
3. Open /docs
4. Register LAPTOP-001
5. Start the device agent
6. Verify device becomes ONLINE
7. Verify telemetry is stored
8. Send high CPU telemetry and create an alert
9. Return CPU to normal and verify alert resolution
10. Stop the device agent
11. Wait for the offline threshold
12. Verify device becomes OFFLINE
13. Start the agent again
14. Verify device becomes ONLINE
```

## 17. Security Considerations

Recommended production improvements:

- JWT authentication for users
- Role-based access control
- Device-specific authentication/API keys
- HTTPS/TLS
- Password hashing
- Environment variables for secrets
- Rate limiting
- Input validation
- Audit logging

## 18. Design Rationale

### FastAPI
Used for fast REST API development, type validation, and automatic API documentation.

### PostgreSQL
Used for structured relational data between devices, telemetry, and alerts.

### SQLAlchemy
Provides a Python ORM and keeps database operations organized.

### Device Agent
Runs on monitored machines and collects local system metrics.

### Heartbeat
Provides periodic proof that a device is communicating with the backend.

### Separate telemetry table
A device generates many time-based measurements, so telemetry is stored separately from the device record.

### Threshold alerts
Simple, deterministic, easy to test, and suitable as a first monitoring layer. ML-based anomaly detection can be added later.

## 19. Current Limitations

This is a prototype. Possible extensions include:

- React dashboard
- WebSocket live updates
- MQTT for large fleets
- JWT authentication
- Device authentication
- Email/Slack/Teams notifications
- Remote commands
- Command queues
- Audit logs
- ML anomaly detection
- Docker/Docker Compose deployment
- Production observability

Hardware temperature may not be available through `psutil` on every system.

## 20. Future Architecture

```text
                 React Dashboard
                       |
                REST + WebSocket
                       |
                 FastAPI Backend
                /       |       \\
               /        |        \\
        Devices     Telemetry    Alerts
               \\        |        /
                \\       |       /
                  PostgreSQL
                       ^
                       |
              Device Agents / MQTT
                       |
                 Fleet of Devices
```

## 21. Project Status

### Implemented

- [x] FastAPI backend
- [x] PostgreSQL database
- [x] SQLAlchemy integration
- [x] Device CRUD
- [x] Telemetry model
- [x] Heartbeat API
- [x] Python device agent
- [x] Live CPU/RAM/disk collection
- [x] Online/offline monitoring
- [x] Alert model
- [x] Threshold alerts
- [x] Alert resolution
- [x] Telemetry history API
- [x] Dashboard summary API

### Planned

- [ ] React dashboard
- [ ] JWT authentication
- [ ] Device authentication
- [ ] WebSocket updates
- [ ] MQTT integration
- [ ] Remote commands
- [ ] Advanced anomaly detection
- [ ] Production deployment
