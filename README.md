<div align="center">

# NETGRADER

### Network and Infrastructure Grader Automation Platform

*An enterprise-grade automated grading platform for network lab assignments, fully integrated with GNS3 virtualization and deployed on-premise at the School of Information Technology, KMITL.*

[![ElysiaJS](https://img.shields.io/badge/ElysiaJS-Bun_Runtime-F472B6?style=for-the-badge&logo=bun&logoColor=white)](https://elysiajs.com/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Worker-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Nuxt](https://img.shields.io/badge/Nuxt_4-Frontend-00DC82?style=for-the-badge&logo=nuxt.js&logoColor=white)](https://nuxt.com/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Database-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![GNS3](https://img.shields.io/badge/GNS3_3.0+-Integrated-2496ED?style=for-the-badge&logo=cisco&logoColor=white)](https://www.gns3.com/)

</div>

---

> **Note:** This project is deployed on an intranet for faculty use only. Repositories are private due to security concerns. This README serves as a comprehensive showcase of the system's architecture, features, and technical depth.

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Core Features](#core-features)
  - [GNS3 3.0+ Integration](#1-gns3-30-integration)
  - [Automated Grading Pipeline](#2-automated-grading-pipeline)
  - [YAML Custom Task Templates](#3-yaml-custom-task-templates)
  - [IP & Subnet Management](#4-ip--subnet-management)
  - [Large Subnet Mode (Exam Infrastructure)](#5-large-subnet-mode-exam-infrastructure)
  - [IPv6 Support](#6-ipv6-support)
  - [Late Penalty System](#7-late-penalty-system)
  - [Duplication Detection](#8-duplication-detection)
  - [AI Integration](#9-ai-integration)
  - [Real-Time Progress Tracking](#10-real-time-progress-tracking)
- [Authentication & Security](#authentication--security)
- [DevOps & CI/CD](#devops--cicd)
- [Team](#team)

---

## Overview

**NetGrader** is a full-stack platform designed to automate the grading of networking lab assignments at the university level. It addresses a core challenge in networking education: **manually assessing student configurations across virtual network devices is time-consuming, error-prone, and does not scale** to large classes or exam scenarios.

### What NetGrader Solves

| Challenge | NetGrader's Solution |
|---|---|
| Manual grading of CLI configurations | Automated SSH/Telnet-based inspection using Nornir + Netmiko + NAPALM |
| Inconsistent lab environments | GNS3-based virtualization with per-student isolated projects |
| Scaling to exam scenarios (100+ students) | Hash-based server sharding + automated subnet allocation |
| Rigid, hardcoded grading checks | YAML-driven custom task templates with an in-browser editor |
| Late submission tracking | Configurable penalty formulas with soft/hard deadlines |
| Lab copying detection | Multi-signal duplication detection (metadata, config hashes, textual similarity) |

### Key Metrics

- **47 slides** of presentation documentation covering architecture and features
- **3 microservice repositories** with clear separation of concerns
- **18 backend API modules** covering courses, labs, submissions, GNS3, AI, themes, and more
- **8 custom YAML task templates** for various network grading scenarios
- Supports **IPv4 and IPv6** network configurations
- Integrated with **GNS3 3.0+** multi-user environments with resource pool isolation
- Full **CI/CD pipeline** with Jenkins, Docker, health checks, and auto-rollback

---

## System Architecture

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                              On-Premise Infrastructure                        │
│                        (School of Information Technology)                      │
│                                                                               │
│  ┌─────────────┐       ┌──────────────┐       ┌─────────────────────────┐    │
│  │ Client Layer │       │  Web Browser  │       │       Data Layer        │    │
│  │              │       │              │       │                         │    │
│  │  Student    │ HTTPS │  ┌──────────┐│ REST  │  ┌─────────────────┐   │    │
│  │  Instructor ├──────►│  │ Nuxt 4   ├┼──────►│  │  ElysiaJS       │   │    │
│  │  TA         │       │  │ Web UI   ││◄─ SSE │  │  (Bun Runtime)  │   │    │
│  │              │       │  └──────────┘│       │  └──┬──┬──┬────────┘   │    │
│  └─────────────┘       └──────────────┘       │     │  │  │             │    │
│                                                │     │  │  │  ┌────────┐│    │
│                                                │     │  │  ├─►│MongoDB ││    │
│  ┌─────────────┐                               │     │  │  │  └────────┘│    │
│  │ LDAP Server  │◄───── Auth ──────────────────┤     │  │  │  ┌────────┐│    │
│  └─────────────┘                               │     │  │  ├─►│ MinIO  ││    │
│                                                │     │  │  │  └────────┘│    │
│                                                │     │  │  │  ┌────────┐│    │
│  ┌─────────────────────────────────────┐       │     │  │  └─►│RabbitMQ││    │
│  │         Grading Layer               │       │     │  │     └───┬────┘│    │
│  │  ┌────────────────────────────────┐ │       │     │  │         │     │    │
│  │  │  FastAPI Grading Worker        │ │  HTTP │     │  │    AMQP │     │    │
│  │  │  ┌──────────┐ ┌─────────────┐ │◄┼──Callback──┘  │         │     │    │
│  │  │  │ Nornir    │ │ Scoring     │ │ │       │        │    Consume    │    │
│  │  │  │ Framework │ │ Service     │ │◄┼────────────────┼─────────┘    │    │
│  │  │  └──────┬───┘ └─────────────┘ │ │       │        │              │    │
│  │  └─────────┼──────────────────────┘ │       └────────┼──────────────┘    │
│  │       SSH  │                        │                │                    │
│  │  ┌─────────▼──────────────────────┐ │                │                    │
│  │  │    Student Virtual Network Lab │ │                │                    │
│  │  │  (GNS3 3.0+ Multi-Server)     │ │                │                    │
│  │  └────────────────────────────────┘ │                │                    │
│  └─────────────────────────────────────┘                │                    │
└───────────────────────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Student submits** a lab through the Nuxt web UI (HTTPS)
2. **Elysia backend** validates the request, creates a submission record in MongoDB, and **publishes a grading job** to RabbitMQ (AMQP)
3. **FastAPI worker** consumes the job from the queue, establishes SSH connections to the student's virtual lab devices using **Nornir + Netmiko/NAPALM**
4. The worker executes grading tasks (ping tests, command checks, NAPALM getters, custom YAML tasks) and **calculates scores** via the Scoring Service
5. **Progress updates** and **final results** are sent back to the Elysia backend via HTTP callbacks
6. The Elysia backend streams **real-time progress** to the frontend using **Server-Sent Events (SSE)**

---

## Tech Stack

### Frontend — `netgrader-frontend`
| Technology | Purpose |
|---|---|
| **Nuxt 4** (Vue 3) | SSR/SPA framework with file-based routing |
| **Tailwind CSS v4** | Utility-first styling |
| **Reka UI** (Headless) | Accessible component primitives |
| **Shadcn-Nuxt** | Pre-styled component library |
| **TipTap** | Rich text editor for lab instructions |
| **CodeMirror** | In-browser YAML editor with syntax highlighting |
| **GSAP** + **Motion Vue** | Smooth animations and transitions |
| **TanStack Table** | Advanced data tables with sorting & filtering |
| **VeeValidate** + **Zod** | Schema-based form validation |
| **Day.js** | Date/time manipulation |
| **Vue Sonner** | Toast notifications |

### Backend API — `netgrader-backend-elysia`
| Technology | Purpose |
|---|---|
| **ElysiaJS** (Bun Runtime) | High-performance TypeScript REST API |
| **MongoDB** + **Mongoose** | Document database with schema validation |
| **MinIO** (S3-compatible) | Object storage for files & exports |
| **RabbitMQ** (AMQP) | Message queue for grading job distribution |
| **LDAP Authentication** | University SSO integration |
| **JWT** (Bearer tokens) | Stateless session management |
| **bcrypt** | Password hashing for course access |
| **Google Generative AI** | AI-powered assistance features |
| **Redis** | Caching layer (optional) |

### Grading Worker — `netgrader-backend-fastapi`
| Technology | Purpose |
|---|---|
| **FastAPI** + **Uvicorn** | Async Python worker service |
| **Nornir** | Network automation framework (task orchestration) |
| **Netmiko** | Multi-vendor SSH/Telnet to network devices |
| **NAPALM** | Vendor-neutral network API (getters: facts, interfaces, routing) |
| **TextFSM** + **NTC Templates** | Structured CLI output parsing |
| **Paramiko** | Low-level SSH connections |
| **PyYAML** | Custom task template parsing |
| **Pydantic** | Data validation for grading models |
| **aio-pika** | Async RabbitMQ consumer |
| **PySNMP** | SNMP-based device detection |

### Infrastructure & DevOps
| Technology | Purpose |
|---|---|
| **Docker** + **Docker Compose** | Containerized microservices |
| **Jenkins** | CI/CD pipeline with auto-rollback |
| **GNS3 3.0+** | Network virtualization (multi-server) |
| **Cisco IOL** | Lightweight router/switch emulation |
| **Ubuntu Docker** (systemd) | Virtual PCs with telnet console access |

---

## Repository Structure

### `netgrader-backend-elysia` — API Server

```
src/
├── config/              # Database, Redis, MinIO, RabbitMQ connections
├── modules/
│   ├── admin/           # Admin management endpoints
│   ├── ai/              # AI service integration (Google Generative AI)
│   ├── auth/            # LDAP authentication + JWT + role determination
│   ├── courses/         # Course CRUD with password-protected enrollment
│   ├── device-templates/# Network device template definitions
│   ├── enrollments/     # Student-course enrollment management
│   ├── gns3/            # GNS3 API communication (playground mode)
│   ├── gns3-student-lab/# GNS3 v3 multi-server student lab management
│   ├── labs/            # Lab configuration with VLAN, IPv6, subnet models
│   ├── parts/           # Lab parts/sections management
│   ├── playground/      # GNS3 sandbox testing environment
│   ├── profile/         # User profile management
│   ├── storage/         # MinIO file storage endpoints
│   ├── student-lab-sessions/ # IP allocation & session lifecycle
│   ├── submissions/     # Grading submission management + IP calculation
│   ├── task-templates/  # YAML custom task template CRUD
│   └── websocket/       # Real-time communication
├── plugins/             # Auth plugin (JWT verification middleware)
├── services/            # Shared services (lab session cleanup)
└── utils/               # Helpers, logger, rich content utilities
```

### `netgrader-backend-fastapi` — Grading Worker

```
app/
├── core/config.py          # Environment configuration
├── schemas/models.py       # Pydantic models for grading jobs
└── services/
    ├── connectivity/
    │   ├── connection_manager.py  # Nornir connection isolation
    │   ├── api_client.py          # HTTP callbacks to Elysia
    │   ├── minio_service.py       # Object storage for exports
    │   └── snmp_detection.py      # SNMP-based device discovery
    ├── grading/
    │   ├── nornir_grading_service.py  # Core: ping, SSH, command, NAPALM tasks
    │   ├── simple_grading_service.py  # Job orchestration & execution
    │   ├── scoring_service.py         # Test case evaluation & scoring
    │   ├── exception_handler.py       # Error classification
    │   └── ipv6_utils.py              # IPv6 comparison utilities
    ├── custom_tasks/          # Custom YAML task execution engine
    └── pipeline/
        └── queue_consumer.py  # RabbitMQ job consumer
custom_tasks/                  # YAML task template library
├── advanced_ping_test.yaml
├── console_terminal_test.yaml
├── curl_test.yaml
├── debug_example.yaml
├── dhcp_binding.yaml
├── linux_service_health.yaml
├── test_template.yaml
└── vlan_verification.yaml
```

### `netgrader-frontend` — Web Interface

```
app/
├── pages/
│   ├── index.vue           # Dashboard / landing page
│   ├── login.vue           # LDAP authentication page
│   ├── profile.vue         # User profile management
│   ├── courses/            # Course listing & detail pages
│   │   └── [c_id]/         # Dynamic course routes (labs, submissions)
│   ├── manage/             # Instructor management panel
│   └── submissions/        # Submission detail views
├── components/
│   ├── NavigationBar.vue   # Main navigation with role-based menus
│   ├── GradingProgress.vue # Real-time grading progress display
│   ├── NavbarTimer.vue     # Exam countdown timer
│   ├── editor/             # YAML & rich text editors
│   ├── wizard/             # Lab creation wizard (16 components)
│   ├── student/            # Student-facing lab views (11 components)
│   ├── course/             # Course management components
│   └── ui/                 # 430+ shared UI primitives (Shadcn)
├── composables/            # 37 Vue composables for shared logic
├── middleware/             # Route guards (auth, role-based)
├── plugins/                # App-level plugins
└── types/                  # TypeScript type definitions
```

---

## Core Features

### 1. GNS3 3.0+ Integration

NetGrader is **fully integrated with GNS3 version 3.0+**, leveraging its multi-user architecture for student isolation.

- **Automated Provisioning**: Lab environments are provisioned end-to-end — from creating GNS3 users, resource pools, and projects to establishing SSH connections for grading
- **Multi-Server Support**: Multiple GNS3 servers can be configured, with students distributed using **hash-based sharding** (XOR-based byte hashing across the SHA-256 digest of the student ID)
- **Role-Based Access Control**: Students can only see and interact with their own projects via GNS3 resource pools
- **Lazy Initialization**: Lab environments are initialized on-demand when a student begins their session, optimizing server resources
- **Device Mapping**: The system maps GNS3 node labels to console ports, enabling automated SSH/Telnet connections to specific virtual devices

```typescript
// Hash-based server sharding from gns3-student-lab/service.ts
calculateInitialServerIndex(userId: string): number {
    const hash = crypto.createHash('sha256').update(userId).digest();
    let xorByte = 0;
    for (const byte of hash) { xorByte ^= byte; }
    return xorByte % GNS3_SERVERS.length;
}
```

### 2. Automated Grading Pipeline

The grading pipeline is built on a **message queue architecture** for reliability and scalability:

1. **Job Publishing**: Elysia backend publishes grading jobs to RabbitMQ with device info, IP mappings, and task definitions
2. **Job Consumption**: FastAPI worker asynchronously consumes jobs from the queue
3. **Connection Management**: The `ConnectionManager` creates isolated or stateful Nornir instances per device
4. **Task Execution**: Supports multiple execution types:
   - **Ping Tasks** — ICMP reachability verification
   - **SSH Connectivity Tests** — Banner/prompt validation with timeout handling
   - **Command Tasks** — CLI command execution with Regex/TextFSM parsing
   - **NAPALM Tasks** — Vendor-neutral getters (facts, interfaces, BGP, routing tables)
   - **Custom YAML Tasks** — Instructor-defined task templates (see below)
5. **Scoring**: The `ScoringService` evaluates test cases with multiple comparison types:
   - `equals`, `not_equals`, `contains`, `not_contains`
   - `greater_than`, `less_than`, `regex_match`
   - `exists`, `count_equals`, `in_list`
   - IPv6-aware comparisons with link-local handling

### 3. YAML Custom Task Templates

A powerful feature allowing instructors to define **grading tasks declaratively** using YAML, eliminating the need for code changes:

```yaml
# Example: OSPF Neighbor Check
task_name: "ospf_neighbor_check"
description: "Verify OSPF neighbor relationships and neighbor count"
connection_type: "napalm"
author: "NetGrader Team"
version: "1.0.0"
points: 15

parameters:
  - name: "expected_neighbors"
    datatype: "integer"
    description: "Expected number of OSPF neighbors"
    required: false

commands:
  - name: "get_ospf_neighbors"
    action: "napalm_get"
    parameters:
      getter: "get_bgp_neighbors_detail"
    register: "ospf_data"

  - name: "parse_neighbor_count"
    action: "parse_output"
    parameters:
      input: "{{ospf_data}}"
      pattern: "Full/.*?(?=\\n|$)"
    register: "neighbor_matches"

validation:
  - field: "neighbor_matches.match_count"
    condition: "greater_than"
    value: 0
    description: "At least one OSPF neighbor in Full state"
```

**Built-in Template Library** includes: Advanced ping tests, console terminal checks, curl connectivity tests, DHCP binding verification, Linux service health checks, VLAN verification, testing, and debugging templates.

The frontend includes a **built-in YAML editor** (CodeMirror-based) with syntax highlighting, IntelliSense-style suggestions, and real-time validation.

### 4. IP & Subnet Management

NetGrader features a sophisticated IP address management system with multiple allocation modes:

| Mode | Description |
|---|---|
| **`student_id_based`** | Deterministic IP allocation based on student ID using Knuth's multiplicative hash |
| **`group_based`** | Instructor assigns IP ranges to groups |
| **`fixed_vlan`** | Static VLAN assignment |
| **`lecturer_group`** | Instructor-defined groupings |
| **`calculated_vlan`** | Algorithmic VLAN calculation |
| **`large_subnet`** | Exam mode: unique /24 subnets per student from a large pool (see below) |

Key capabilities:
- **Exempt IP Ranges**: Instructors can define IP ranges excluded from student allocation (e.g., infrastructure IPs)
- **Deterministic Assignment**: Same student always gets the same IPs across sessions
- **VLAN 2–4094 Support**: Full VLAN range with configurable per-VLAN subnet allocation
- **IP conflict prevention**: Built-in subnet validation and overlap detection

### 5. Large Subnet Mode (Exam Infrastructure)

Designed for **midterm/final exam scenarios** with 100+ concurrent students:

- **Automated Subnet Allocation**: Each student receives a unique `/24` subnet from a configurable private pool (`10.0.0.0/8`, `172.16.0.0/12`, or `192.168.0.0/16`)
- **Knuth's Multiplicative Hash**: Uses the Golden Ratio constant (`φ = 0x9E3779B9`) for uniform distribution of student IDs to subnet addresses
- **Collision-Free**: Guarantees no two students share the same subnet
- **Per-VLAN Subnets**: Multiple VLANs within each student's subnet, with configurable subnet sizes

```typescript
// Knuth's multiplicative hash for subnet allocation
const GOLDEN_RATIO = 0x9E3779B9;
const hash = ((studentIdNumeric * GOLDEN_RATIO) >>> 0);
const subnetIndex = hash % totalAvailableSubnets;
```

### 6. IPv6 Support

Comprehensive dual-stack networking support:

- **IPv6 Template Configuration**: Instructors define IPv6 prefix templates with automatic student-specific suffix generation
- **Management Network IPv6**: Each student gets a unique IPv6 address for out-of-band management
- **VLAN-aware IPv6**: Per-VLAN IPv6 subnet allocation with configurable Interface IDs
- **IPv6 Comparison Utilities**: Intelligent IPv6 comparison in grading (handles link-local, compressed forms, and prefix matching)

### 7. Late Penalty System

Configurable time-based penalty system for submissions:

- **Soft Deadline (Due Date)**: Submissions after this date receive a configurable penalty (default: 50%)
- **Hard Deadline (Available Until)**: Lab closes completely; no submissions accepted
- **Penalty Formula**: `Final Score = Raw Score × (1 - Penalty%)`
- **Instructor Overrides**: Manual override capability for individual submissions
- **Visual Indicators**: UI clearly shows late status, penalty applied, and adjusted scores

### 8. Duplication Detection

Multi-signal approach to identify lab copying:

| Signal | Description |
|---|---|
| **GNS3 Project Metadata** | Compares Node IDs — copied projects retain original IDs |
| **Configuration Hash Sums** | Hash-based fingerprinting of device configurations |
| **Textual Similarity** | Side-by-side config comparison with diff highlighting |

### 9. AI-Assisted Lab Creation (In Development)

Instructors can describe a lab scenario in natural language and have the system generate the full lab configuration automatically — skipping the multi-step wizard that requires extensive manual input (network topology, device definitions, IP schemes, task templates, etc.).

- Powered by **Google Generative AI** via the `/ai/generate` endpoint
- Aims to reduce lab creation time from minutes of form-filling to a single prompt
- Currently in active development by a Master's degree student building on top of the existing platform

### 10. Real-Time Progress Tracking

- **Server-Sent Events (SSE)** stream grading progress from backend to frontend
- **Granular Updates**: Shows current test being executed, tests completed, total tests, and percentage
- **Visual Progress Component**: `GradingProgress.vue` displays an animated, real-time progress view with per-test results

---

## Authentication & Security

| Feature | Implementation |
|---|---|
| **University SSO** | LDAP authentication against the institution's directory server |
| **Role Detection** | Automatic role assignment (Student/Instructor/TA) from LDAP attributes |
| **JWT Tokens** | Stateless authentication with configurable expiration |
| **Auth Middleware** | Global `authPlugin` protects all routes except `/health` |
| **Course Passwords** | Optional bcrypt-hashed passwords for private courses |
| **First-Login Provisioning** | Automatic GNS3 user + resource pool creation on first LDAP login |
| **Intranet-Only** | Deployed exclusively on the faculty intranet |

### User Roles

| Role | Capabilities |
|---|---|
| **Student** | View enrolled courses, start labs, submit assignments, view grades |
| **Instructor** | Create courses/labs, manage task templates, view all submissions, configure grading |
| **TA** | View courses, assist with grading oversight |
| **Admin** | System-wide management and user administration |

---

## DevOps & CI/CD

### Jenkins Pipeline

Each microservice has its own **Jenkinsfile** implementing a robust deployment pipeline:

```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐    ┌───────────────┐
│   Backup    │───►│  Pull Latest │───►│  Check .env  │───►│ Rebuild &     │
│ Current     │    │  Code (Git)  │    │  File Exists │    │ Deploy Docker │
│ Image       │    └──────────────┘    └─────────────┘    └───────┬───────┘
└─────────────┘                                                   │
                                                                  ▼
┌─────────────┐    ┌──────────────┐    ┌─────────────────────────────────┐
│  Cleanup    │◄───│   Verify     │◄───│  Health Check (6 retries, 5s)  │
│  Old        │    │  Deployment  │    │  Container UP + /health 200 OK │
│  Backups    │    └──────────────┘    └─────────────────────────────────┘
└─────────────┘

On Failure: Auto-rollback to previous backup image
```

**Pipeline Features:**
- **Image Backup**: Tags current Docker image before deployment
- **Automated Rollback**: On failure, automatically restores the previous backup image
- **Health Checks**: Retries health endpoint 6 times with 5s intervals
- **Backup Rotation**: Keeps the last 5 backup images, prunes older ones
- **Failure Diagnostics**: Dumps last 100 container log lines on failure

### Dockerized Services

All services run as **Docker containers** managed by Docker Compose:

| Service | Container | Runtime |
|---|---|---|
| API Server | `netgrader-backend-elysia` | Bun |
| Grading Worker | `netgrader-backend-fastapi` | Python (Uvicorn) |
| Frontend | `netgrader-frontend` | Node.js |
| Database | MongoDB | – |
| Object Storage | MinIO | – |
| Message Queue | RabbitMQ | – |

### Scheduled Tasks

The Elysia backend runs automated maintenance tasks:
- **Lab Session Cleanup** (hourly): Releases IP addresses from expired sessions
- **Housekeeping** (daily): Purges completed sessions older than 90 days

---

## Team

| Name | Student ID | Role |
|---|---|---|
| **Jessada Taengsuwan** | 65070041 | Full-Stack Developer |
| **Sahachindech Katedee** | 65070232 | Full-Stack Developer |

**Institution:** School of Information Technology, King Mongkut's Institute of Technology Ladkrabang (KMITL)

---

<div align="center">

*Built for networking education at IT KMITL*

</div>
