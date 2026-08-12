# Trakies Backend API

> A robust, modular RESTful backend system for managing trekking and tour operations, participant profiles, geofenced checkpoint tracking, transport and room allocations, tour cloning, and media management.

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [Running the Project](#running-the-project)
- [Database Setup](#database-setup)
- [Data Model / Schema Documentation](#data-model--schema-documentation)
- [API Documentation](#api-documentation)
  - [Authentication & User Management](#authentication--user-management)
  - [Tour Management](#tour-management)
  - [Geofencing & Checkpoints](#geofencing--checkpoints)
  - [Accommodation & Room Allocation](#accommodation--room-allocation)
  - [Transport & Seat Allocation](#transport--seat-allocation)
  - [Bookings & Members](#bookings--members)
  - [Media & AWS S3 Storage](#media--aws-s3-storage)
  - [Expenses & Notes](#expenses--notes)
  - [Notifications & FAQs](#notifications--faqs)
- [Authentication & Authorization](#authentication--authorization)
- [Request Validation](#request-validation)
- [Error Handling](#error-handling)
- [API Response Format](#api-response-format)
- [Pagination & Aggregation](#pagination--aggregation)
- [Testing & Quality Assurance](#testing--quality-assurance)
  - [Testing Roadmap](#testing-roadmap)
  - [Load Testing](#load-testing)
- [Docker Support](#docker-support)
- [Deployment](#deployment)
- [Security](#security)
  - [Security Guidelines](#security-guidelines)
- [Coding Standards](#coding-standards)
- [How to Contribute](#how-to-contribute)
- [Contribution Rules](#contribution-rules)
- [Good First Issues](#good-first-issues)
- [Issue Guidelines](#issue-guidelines)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Commit Guidelines](#commit-guidelines)
- [Learning Path for Students](#learning-path-for-students)
- [How to Understand an API](#how-to-understand-an-api)
- [Common Development Problems](#common-development-problems)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Roadmap](#roadmap)
- [License](#license)
- [Code of Conduct](#code-of-conduct)
- [Security Policy](#security-policy)
- [Recommended Open-Source Files](#recommended-open-source-files)
- [Project Metadata](#project-metadata)

---

## Overview

**Trakies** is an open-source backend web service designed specifically for adventure travel companies, trekking organizations, and tour operators. It powers operations such as creating tours, managing participant bookings, tracking participant progress live through geofenced checkpoints, allocating rooms and transport seats, recording tour expenses, and uploading tour media to cloud storage.

### What Problem It Solves

Managing adventure tours involves complex logistical workflows:
- Tracking live trek progress across checkpoints in remote areas.
- Managing tour participant rosters, emergency contacts, and identification details.
- Organizing inclusions/exclusions, baggage lists, and custom notes per tour.
- Allocating hotel rooms and transport bus seats to registered guests.
- Duplicating recurring tour templates efficiently without manual re-entry.

Trakies consolidates these workflows into a unified RESTful API platform.

### Who It Is For

- **Developers**: Real-world reference for building Express applications using modern ES modules, MongoDB aggregation pipelines, and AWS S3 presigned URL integration.
- **Students & Beginners**: An accessible codebase to study request flows, custom middleware, transaction-based cloning, and MongoDB schema design.
- **Maintainers & Contributors**: A foundation for building production-ready tour management systems.

---

## Key Features

- 🔐 **Dual Authentication & Role-Based Access**:
  - JWT token generation & verification for Dashboard Superadmins.
  - Role-based authorization (`superadmin`, `admin`, `coordinator`) checked via custom middleware (`x-user-email` header verification).
- ⛰️ **Comprehensive Tour Management**:
  - Full CRUD operations for tours with date validations, cost, total seats, status flags, and custom URLs.
  - Transactional **Tour Cloning** (`/api/clone-tour`) using MongoDB Mongoose ACID sessions to duplicate tours along with inclusions, exclusions, checkpoints, notes, transport, and accommodation.
- 📍 **Geofenced Checkpoint & Location Tracking**:
  - Define checkpoints with geographic coordinates (`latitude`, `longitude`).
  - Track user check-ins, identify the latest unchecked checkpoint, and reset or view checked users per checkpoint.
- 🛏️ **Accommodation & Room Allocation**:
  - Manage accommodations and rooms per tour.
  - Allocate specific rooms to tour participants.
- 🚌 **Transport & Seat Allocation**:
  - Manage transport vehicles and boarding points.
  - Allocate vehicle seating to participants.
- 🎒 **Baggage & Packing Lists**:
  - Custom checklists for check-in baggage and backpack requirements per tour.
- 💰 **Tour Financials & Expenses**:
  - Record and manage tour operational expenses.
- ☁️ **Media & Cloud Storage**:
  - Direct secure uploads to **AWS S3** via presigned URLs (`@aws-sdk/s3-request-presigner`).
  - Tour image gallery management.
- 📄 **Pagination Utility**:
  - Built-in pagination helper (`utils/pagination.js`) supporting standard `find()` queries and MongoDB `$aggregate` pipelines.

---

## Tech Stack

| Technology | Version | Purpose |
| ---------- | ------- | ------- |
| **Node.js** | v18+ | JavaScript backend runtime environment |
| **Express.js** | ^4.19.2 | Fast, unopinionated web framework for Node.js |
| **MongoDB** | Database | NoSQL document database |
| **Mongoose** | ^8.6.1 | Object Data Modeling (ODM) library for MongoDB |
| **JWT (jsonwebtoken)** | ^9.0.2 | JSON Web Token implementation for authentication |
| **bcryptjs** | ^2.4.3 | Password hashing library |
| **AWS SDK v3 (`@aws-sdk/client-s3`)** | ^3.651.1 | Cloud storage client for media asset uploads |
| **Nodemailer** | ^7.0.11 | Email sending utility |
| **Artillery** | ^2.0 | Load testing tool (`load-test.yml`) |
| **Docker** | Container | Containerization specification (`Dockerfile`) |

---

## Architecture Overview

Trakies follows a layered **MVC (Model-View-Controller)** pattern structured around Express routing and MongoDB/Mongoose data models using modern ES module syntax (`import`/`export`).

```text
  Client Request (HTTP/HTTPS)
             │
             ▼
     ┌──────────────┐
     │   app.js     │  <-- CORS, JSON Middleware, Route Mounting
     └──────┬───────┘
            │
            ▼
     ┌──────────────┐
     │  Middleware  │  <-- JWT Check (authCheck), Role Validation (checkAdminRole)
     └──────┬───────┘
            │
            ▼
     ┌──────────────┐
     │    Router    │  <-- Endpoint Route Handlers (24 Routers)
     └──────┬───────┘
            │
            ▼
     ┌──────────────┐
     │  Controller  │  <-- Business Logic, Input Extraction, Response Formatting
     └──────┬───────┘
            │
            ▼
     ┌──────────────┐
     │ Models/Utils │  <-- Mongoose Schemas (28 Models), Pagination, ApiError
     └──────┬───────┘
            │
            ▼
     ┌──────────────┐
     │  Database    │  <-- MongoDB Database ("Indore_hackathon")
     └──────────────┘
```

### Architectural Components

- **Server Entry Point (`index.js`)**: Connects to MongoDB via `db/db.connection.js` and boots up the Express server on `process.env.PORT || 9000`.
- **Application Setup (`app.js`)**: Configures CORS origins, JSON request parsing, and mounts 24 specific router modules under the `/api/*` path hierarchy.
- **Routing Layer (`router/`)**: Maps HTTP verbs and URI paths to their designated controller functions and applies authorization middleware.
- **Controller Layer (`controllers/`)**: Executes operational logic, handles request parameters/body payload, constructs MongoDB query/aggregation pipelines, and formats HTTP responses.
- **Model Layer (`models/`)**: Defines Mongoose schemas, field validations, defaults, and collection relationships.
- **Utilities (`utils/`)**: Provides shared utilities such as pipeline-compatible pagination (`utils/pagination.js`) and custom HTTP errors (`utils/ApiError.js`).

---

## Project Structure

```text
trakies_backend/
├── controllers/               # 26 Controller modules implementing core application logic
│   ├── Included&NotIncluded.js
│   ├── TrackLead.controller.js
│   ├── accommodation.js
│   ├── allocatedAccommodation.js
│   ├── allocatedTransport.js
│   ├── backPack&checkInBaggage.controller.js
│   ├── booking.controller.js
│   ├── bordingPoints.js
│   ├── checkPoint.controller.js
│   ├── checkedPoint.controller.js
│   ├── clone.controler.js
│   ├── expanse.controller.js
│   ├── faq.js
│   ├── guest.controller.js
│   ├── image.controller.js
│   ├── interested.controller.js
│   ├── member.controller.js
│   ├── notes.controller.js
│   ├── notification.controller.js
│   ├── post.controller.js
│   ├── room.controller.js
│   ├── s3bucket.js
│   ├── tour.controller.js
│   ├── transport.js
│   ├── user.controller.js
│   └── user.js
├── db/                        # Database connection bootstrap
│   └── db.connection.js
├── middleware/                # Authentication & Role-based authorization middleware
│   ├── authCheck.js
│   └── checkAdminRole.js
├── models/                    # 28 Mongoose data models
│   ├── Accommodation.js
│   ├── Admin.js               # Stores User roles (admin, coordinator)
│   ├── AllocatedAccommodation.js
│   ├── AllocatedTransport.js
│   ├── BackPack.js
│   ├── BoardingPoints.js
│   ├── Booking.js
│   ├── CheckInBaggage.js
│   ├── DashboardUsers.js      # Stores Superadmin credentials (email, password, role)
│   ├── Expanse.js
│   ├── FAQ.js
│   ├── Guest.js
│   ├── Image.js
│   ├── Included.js
│   ├── Interested.js
│   ├── Member.js
│   ├── NotIncluded.js
│   ├── Note.js
│   ├── Notification.js
│   ├── Post.js
│   ├── Room.js
│   ├── SeenNotification.js
│   ├── Tour.js
│   ├── TourLead.js
│   ├── Transport.js
│   ├── UserProfile.js
│   ├── checkPoints.js
│   └── checkedPoints.js
├── router/                    # 24 Express router files
│   ├── Included.js
│   ├── accommodation.router.js
│   ├── allocatedAccommodation.js
│   ├── allocatedTransport.js
│   ├── auth.js
│   ├── backPack.js
│   ├── boardingPoints.router.js
│   ├── booking.router.js
│   ├── checkInBaggage.js
│   ├── checkPoint.router.js
│   ├── checkedPoint.router.js
│   ├── expanse.router.js
│   ├── faq.router.js
│   ├── image.router.js
│   ├── interested.router.js
│   ├── member.router.js
│   ├── notIncluded.js
│   ├── notes.js
│   ├── notification.js
│   ├── post.router.js
│   ├── tour.js
│   ├── trackLead.router.js
│   ├── transport.router.js
│   └── user.router.js
├── utils/                     # Shared utilities
│   ├── ApiError.js
│   ├── pagination.js
│   └── validateInputs.js
├── .env                       # Environment variables configuration file
├── .gitignore                 # Git ignore rules
├── Dockerfile                 # Docker configuration (Node 18 base image)
├── app.js                     # Express app setup and middleware configuration
├── index.js                   # Application entry point
├── load-test.yml              # Artillery load test configuration
├── package.json               # NPM metadata, dependencies, and scripts
└── temp.js                    # React Native location tracking reference helper
```

---

## Prerequisites

Before running Trakies locally, make sure you have the following installed:

- **Node.js**: v18.0.0 or higher
- **npm**: v9.0.0 or higher
- **MongoDB**: Local instance running on `mongodb://localhost:27017` or a cloud MongoDB Atlas connection string.
- **AWS Account** *(Optional)*: AWS S3 bucket and IAM keys for testing image upload functionality.

---

## Installation

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd trakies_backend
   ```

2. **Install Dependencies**:
   ```bash
   npm install
   ```

3. **Configure Environment Variables**:
   Create a `.env` file in the root directory (see [Environment Variables](#environment-variables)).

4. **Start the Database**:
   Ensure your local MongoDB daemon is running or your MongoDB Atlas instance is accessible.

5. **Run the Development Server**:
   ```bash
   npm run dev
   ```
   The server will start listening on `http://localhost:9000`.

---

## Environment Variables

The application relies on environment variables loaded via `dotenv`. Create a `.env` file in the project root based on the following specifications:

| Variable | Required | Description | Example / Default Value |
| -------- | -------- | ----------- | ----------------------- |
| `PORT` | No | HTTP server port | `9000` |
| `MONGO_URL` | Yes | MongoDB connection URI | `mongodb+srv://<user>:<pass>@cluster.mongodb.net/` |
| `SECRETE_KEY` | Yes | JWT secret key for token signing & verification | `your_jwt_secret_key` |
| `ACCESS_KEY` | Yes | AWS S3 Access Key ID for presigned URL generation | `AKIAXXXXXXXXXXXXXXXX` |
| `SECRET_ACCESS_KEY` | Yes | AWS S3 Secret Access Key | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |

> [!WARNING]
> Note the exact key name `SECRETE_KEY` (with the extra 'E') as declared in the current codebase.

---

## Running the Project

### Available Scripts

The following scripts are defined in `package.json`:

| Command | Description |
| ------- | ----------- |
| `npm run dev` | Starts the server in development mode using `nodemon` (auto-reloads on file changes). |
| `npm start` | Runs the server in production mode using `node index.js`. |
| `npm test` | Placeholder test script (currently unconfigured). |

---

## Database Setup

The application connects to MongoDB using Mongoose in `db/db.connection.js`. By default, it targets the database named `Indore_hackathon`.

### Database Architecture

```text
 Tour (Core Template)
  ├── Images (Lookup by tourId / id)
  ├── CheckPoints (Lookup by tourId)
  │    └── CheckedPoints (User Check-ins)
  ├── Includeds & NotIncludeds (Tour Features)
  ├── CheckInBaggages & BackPacks (Packing Requirements)
  ├── Accommodations -> Rooms -> AllocatedAccommodations
  ├── Transports -> BoardingPoints -> AllocatedTransports
  ├── Bookings -> Members (Participants)
  └── Expanses (Financial Records)

 User Roles & Authentication
  ├── Admin (Stores user roles: "admin", "coordinator")
  ├── DashboardUsers / Admin (Stores Superadmin credentials)
  └── UserProfile (Stores personal identification & details)
```

---

## Data Model / Schema Documentation

Below are key Mongoose schemas used across the application:

### 1. Tour (`models/Tour.js`)

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `name` | String | Yes | - | Tour name |
| `email` | String | Yes | - | Organizer / Admin email |
| `difficulty` | String | Yes | - | Difficulty level (e.g., Easy, Moderate, Hard) |
| `location` | String | Yes | - | Location / Region of the tour |
| `description` | String | Yes | - | Full tour description |
| `total_seats` | Number | Yes | - | Maximum total capacity |
| `distance` | String | Yes | - | Distance covered (e.g., "45 km") |
| `tour_start` | Date | Yes | - | Start date of tour |
| `tour_end` | Date | Yes | - | End date of tour |
| `booking_close` | Date | Yes | - | Booking cutoff date |
| `status` | Boolean | No | `false` | Tour active status |
| `tour_cost` | String | Yes | - | Price per seat |
| `can_admin_reject` | Boolean | Yes | `false` | Flag allowing admin rejection |
| `enable_payment_getway`| Boolean | Yes | `false` | Payment gateway toggle |
| `tourType` | String | Yes | - | Category (e.g., Trekking, Camping) |
| `state` | String | Yes | - | State/Region location |
| `faqUrl` | String | No | - | Link to FAQ document |
| `consentFormUrl` | String | No | - | Link to consent form PDF |

### 2. UserProfile (`models/UserProfile.js`)

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `email` | String | Yes | User email address |
| `name` | String | Yes | Full name |
| `dob` | Date | No | Date of birth |
| `age` | String | No | Calculated / entered age |
| `gender` | String | No | Gender |
| `contact` | String | No | Primary phone number |
| `emergency_contact` | String | No | Emergency phone number |
| `id_number` | String | No | Government ID number |
| `id_type` | String | No | Government ID type (Aadhaar, Passport, etc.) |
| `address` | String | No | Residence address |
| `info` | String | No | Additional personal information |

### 3. CheckPoint (`models/checkPoints.js`)

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `name` | String | Yes | - | Checkpoint name |
| `tourId` | ObjectId | Yes | - | Reference to Tour |
| `description` | String | No | - | Details about the checkpoint |
| `type` | String | No | - | Checkpoint category |
| `activated` | Boolean | No | `false` | Active state |
| `longitude` | Number | No | - | Geographic longitude |
| `latitude` | Number | No | - | Geographic latitude |

---

## API Documentation

Below is a reference of the primary API endpoints exposed by Trakies.

### Authentication & User Management

Base Route: `/api/admin` and `/api/users`

| Method | Endpoint | Access | Header Requirements | Description |
| ------ | -------- | ------ | ------------------- | ----------- |
| `POST` | `/api/admin/login` | Public | None | Superadmin login; returns JWT token. |
| `POST` | `/api/admin/verify` | Public | None | Verifies a given JWT token. |
| `POST` | `/api/users/signup` | Superadmin | `x-user-email` | Registers a new user with a specified role (`admin`, `coordinator`). |
| `POST` | `/api/users/signin` | Public | None | User sign-in by email; returns role & aggregated profile details. |
| `GET` | `/api/users/get-all` | Superadmin | `x-user-email` | Retrieves all registered system users. |
| `DELETE` | `/api/users/delete?id=<ID>` | Superadmin | `x-user-email` | Deletes a user by ID. |
| `POST` | `/api/users/createProfile` | Public | None | Creates a detailed user profile. |
| `POST` | `/api/users/updateProfile` | Public | None | Updates a user profile by `id`. |
| `GET` | `/api/users/getProfile` | Public | `email` | Gets user profile details by `email` header. |

#### Example: Superadmin Login

```http
POST /api/admin/login
Content-Type: application/json

{
  "email": "superadmin@example.com",
  "password": "yourpassword"
}
```

Response:

```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### Tour Management

Base Route: `/api/tour` and `/api`

| Method | Endpoint | Access | Header Requirements | Description |
| ------ | -------- | ------ | ------------------- | ----------- |
| `POST` | `/api/tour/create-tour` | Admin/Coordinator | `x-user-email` | Creates a new tour record. |
| `GET` | `/api/tour/get-alltours?status=Active` | Public | None | Returns paginated list of active/inactive tours with nested images, inclusions, exclusions, baggage lists, and booked count. |
| `GET` | `/api/tour/get-tour?id=<ID>` | Public | None | Fetches single tour by ID with all aggregated metadata. |
| `POST` | `/api/tour/update-tour` | Public | None | Updates tour details by `id` in payload. |
| `DELETE` | `/api/tour/delete-tour?id=<ID>` | Admin/Coordinator | `x-user-email` | Deletes a tour by ID. |
| `GET` | `/api/clone-tour?tourId=<ID>` | Admin/Coordinator | `x-user-email` | Clones a tour and all related child sub-documents in a single transaction. |

---

### Geofencing & Checkpoints

Base Route: `/api` and `/api/checked`

| Method | Endpoint | Access | Description |
| ------ | -------- | ------ | ----------- |
| `POST` | `/api/create-point` | Public | Adds a checkpoint to a tour. |
| `POST` | `/api/update-point?id=<ID>` | Public | Updates checkpoint details. |
| `DELETE` | `/api/delete-point?id=<ID>` | Public | Deletes a checkpoint. |
| `GET` | `/api/get-points?tourId=<ID>` | Public | Gets paginated checkpoints for a tour with check-in counts. |
| `GET` | `/api/get-latestUnchecked?tourId=<ID>&email=<EMAIL>&type=<TYPE>` | Public | Resolves the latest unchecked point for a participant. |
| `POST` | `/api/checked/add` | Public | Marks a checkpoint as completed for a user email. |
| `GET` | `/api/checked/get?email=<EMAIL>&tourId=<ID>` | Public | Gets checked status of all points for a user on a tour. |
| `DELETE` | `/api/checked/reset?id=<CHECKPOINT_ID>` | Public | Resets all user check-ins for a specific checkpoint. |

---

### Media & AWS S3 Storage

Base Route: `/api` and `/api/image`

| Method | Endpoint | Access | Description |
| ------ | -------- | ------ | ----------- |
| `POST` | `/api/putObject` | Public | Generates AWS S3 presigned URL for direct file upload. |
| `POST` | `/api/image/create-image` | Public | Saves image URL metadata linked to a tour `id`. |
| `GET` | `/api/image/get-image?id=<TOUR_ID>` | Public | Fetches image gallery for a tour. |
| `DELETE` | `/api/image/delete-image?id=<ID>` | Public | Removes an image entry. |

#### Example S3 Presigned URL Request

```http
POST /api/putObject
Content-Type: application/json

{
  "fileName": "summit_photo.jpg",
  "contentType": "image/jpeg"
}
```

Response:

```json
"https://trekies-anshu.s3.eu-north-1.amazonaws.com/uploads/summit_photo.jpg?X-Amz-Algorithm=..."
```

---

## Authentication & Authorization

Trakies implements two complementary security mechanisms:

```text
 Client Request
       │
       ├─► Has Authorization Header? (Bearer <JWT>) ──► authCheck Middleware ──► req.user
       │
       └─► Requires Admin Access? ─────────────────────► checkAdminRole / checkSuperAdmin
                                                               │
                                                               ▼
                                                  Inspect Header: x-user-email
                                                               │
                                                               ▼
                                                  Query MongoDB User/Admin Role
```

1. **JWT Verification (`middleware/authCheck.js`)**:
   - Parses the `Authorization` header for `Bearer <token>`.
   - Verifies the signature using `process.env.SECRETE_KEY`.
   - Attaches decoded payload to `req.user`.

2. **Role Authorization (`middleware/checkAdminRole.js`)**:
   - `checkAdminRole`: Inspects the `x-user-email` header, verifies user exists in `User` model, and checks that role is `"admin"` or `"coordinator"`.
   - `checkSuperAdmin`: Inspects `x-user-email` header, verifies user in `Admin` (`DashboardUsers.js`), and checks that role is `"superadmin"`.

---

## Request Validation

Input validation is enforced within controller functions. 

- **Custom Error Validation**: The helper `utils/validateInputs.js` checks string presence.
- **Header Validations**: Admin endpoints require `x-user-email` headers.
- **Required Body Parameters**: Controllers explicitly verify mandatory parameters before executing database queries and raise `ApiError` (or 400 Bad Request) on missing values.

---

## Error Handling

Trakies uses a combination of custom error classes and standard HTTP status handling:

- `utils/ApiError.js`: A class extending JavaScript `Error` with `status` and `message` properties.
- **Status Codes Used**:
  - `200 OK` / `201 Created`: Successful operations.
  - `400 Bad Request`: Missing mandatory parameters or invalid format.
  - `401 Unauthorized`: Invalid credentials or missing JWT token.
  - `403 Forbidden`: Token validation failure or insufficient user role permissions.
  - `404 Not Found`: Requested document or entity does not exist.
  - `405 Method Not Allowed / Already Registered`: Duplicate registration errors.
  - `500 Internal Server Error`: Unhandled database runtime errors.

---

## API Response Format

Responses consistently return structured JSON.

### Success Response

```json
{
  "message": "CheckPoint created successfully",
  "data": {
    "_id": "64c87b01680e4cb9aa783f56",
    "name": "Base Camp 1",
    "tourId": "64c87b01680e4cb9aa783f55",
    "latitude": 30.3165,
    "longitude": 78.0322,
    "activated": true,
    "createdAt": "2026-08-12T10:00:00.000Z"
  }
}
```

### Error Response

```json
{
  "message": "Access denied: Admins only"
}
```

---

## Pagination & Aggregation

Pagination is handled via `utils/pagination.js`. It supports standard queries as well as complex aggregation pipelines.

```javascript
const { results, pagination } = await paginate(Tour, {
  pipeline: customAggregationPipeline,
  query: req.query,
  defaultLimit: 10,
});
```

### Pagination Output Format

```json
{
  "tours": [...],
  "pagination": {
    "currentPage": 1,
    "totalPages": 5,
    "totalItems": 48,
    "limit": 10
  }
}
```

---

## Testing & Quality Assurance

### Testing Roadmap

Automated unit and integration test suites are not currently implemented (`npm test` is a placeholder). Contributors are encouraged to help set up testing!

Recommended testing structure to contribute:
- **Framework**: Jest or Vitest
- **Supertest**: For Express API endpoint integration testing
- **mongodb-memory-server**: For isolated database unit testing

### Load Testing

The repository includes an **Artillery** load testing configuration file (`load-test.yml`).

To run a load test against a running instance:
```bash
npx artillery run load-test.yml
```

---

## Docker Support

The application includes a `Dockerfile` for containerized execution.

### Building and Running with Docker

1. **Build Docker Image**:
   ```bash
   docker build -t trakies-backend .
   ```

2. **Run Docker Container**:
   ```bash
   docker run -p 9000:9000 --env-file .env trakies-backend
   ```

---

## Deployment

Trakies can be deployed to any Node.js host (AWS ECS/Amplify, EC2, Render, Railway, DigitalOcean App Platform).

1. Ensure target Node.js version is **18+**.
2. Supply environment variables listed in the [.env](#environment-variables) section.
3. Configure MongoDB Atlas network access rules for your host IP.
4. Execute `npm start`.

---

## Security

### Security Practices Implemented

- **CORS Protection**: Whitelisted origin domains configured in `app.js`.
- **JWT Signature Verification**: Signed tokens using HMAC SHA-256 via `jsonwebtoken`.
- **Role Verification**: Middleware safeguards administrative endpoints.
- **Presigned S3 URLs**: Secure client-side uploads without exposing AWS secret credentials.

### Security Guidelines

> [!CAUTION]
> Never commit `.env` files, AWS keys, or production database credentials to Git repositories.

If you discover a security vulnerability, please notify maintainers privately rather than opening a public issue.

---

## Coding Standards

When writing code for Trakies, follow these conventions:

- **ES Modules**: Always use `import / export` syntax. Include file extensions in local imports (e.g., `import Tour from "../models/Tour.js";`).
- **Async/Await**: Prefer `async/await` over raw promise chains.
- **Naming Conventions**:
  - Model files: `PascalCase.js` (e.g., `Tour.js`, `UserProfile.js`).
  - Routers: `camelCase.router.js` or `camelCase.js`.
  - Controllers: `camelCase.controller.js` or `camelCase.js`.
- **Error Handling**: Wrap controller logic in `try/catch` blocks and pass actionable HTTP error messages.

---

## How to Contribute

We welcome contributions from developers, students, and open-source enthusiasts!

```text
Fork Repository
       │
       ▼
Clone Local Copy
       │
       ▼
Create Feature Branch (git checkout -b feature/amazing-feature)
       │
       ▼
Make Changes & Test Locally
       │
       ▼
Commit Changes (git commit -m "feat: add amazing feature")
       │
       ▼
Push to Branch (git push origin feature/amazing-feature)
       │
       ▼
Open Pull Request
```

### Steps to Contribute

1. Fork the project repository on GitHub.
2. Clone your fork locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/trakies_backend.git
   cd trakies_backend
   ```
3. Create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```
4. Commit your changes following commit standards.
5. Push to your branch and open a Pull Request against `main`.

---

## Contribution Rules

### Before Contributing

- Review the codebase architecture and inspect relevant controllers/models.
- Verify your changes locally by running `npm run dev`.
- Ensure no `.env` credentials or debug logs are committed.

### Pull Request Checklist

- [ ] I have tested my changes locally.
- [ ] Code follows existing project conventions (ES Modules, error handling).
- [ ] No secret keys or `.env` files are included.
- [ ] API routes and models are updated if required.

---

## Good First Issues

If you are a beginner looking to contribute, here are great starting areas:

1. **Add Unit/Integration Tests**: Set up Jest or Vitest and write initial tests for `utils/pagination.js` or user auth routes.
2. **Centralize Input Validation**: Enhance `utils/validateInputs.js` with express-validator or Zod schema validation.
3. **Standardize Response Payloads**: Refactor legacy endpoints to return uniform success/error JSON formats.
4. **Environment Config Validation**: Add validation logic on server boot to verify required `.env` variables exist.
5. **Code Cleanup**: Remove unused variables and standardize casing across router imports.

---

## Issue Guidelines

When opening a GitHub issue, please use the following formats:

### Bug Report

```text
Description: Clear description of the bug.
Steps to Reproduce:
1. Call endpoint POST /api/...
2. Send body {...}
Expected Behavior: What should happen.
Actual Behavior: What actually happens (include error logs).
Environment: Node version, OS, MongoDB version.
```

---

## Pull Request Guidelines

PR titles should follow standard prefixes:

- `feat:` New feature or endpoint implementation.
- `fix:` Bug fix.
- `docs:` Documentation improvements.
- `refactor:` Code refactoring without functionality changes.
- `test:` Adding or updating tests.
- `chore:` Dependency or build script updates.

Example: `feat: implement user registration validation middleware`

---

## Learning Path for Students

If you are using this repository to learn backend development, follow this step-by-step path:

```text
Step 1: Inspect index.js and app.js to see how Express starts and mounts routes.
   │
Step 2: Study db/db.connection.js to understand Mongoose connections.
   │
Step 3: Read models/Tour.js and models/UserProfile.js to understand NoSQL schemas.
   │
Step 4: Trace an API call end-to-end (e.g., POST /api/admin/login -> router/auth.js -> controllers/user.js).
   │
Step 5: Inspect middleware/checkAdminRole.js to see how request headers authorize users.
   │
Step 6: Check controllers/clone.controler.js to see MongoDB Mongoose ACID transactions in action.
   │
Step 7: Pick a "Good First Issue" and submit your first Pull Request!
```

---

## How to Understand an API

Here is how a request flows through the Trakies backend codebase:

```text
Incoming HTTP Request: POST /api/tour/create-tour
                        │
                        ▼
                 app.js mounts /api/tour -> router/tour.js
                        │
                        ▼
                 router/tour.js invokes checkAdminRole middleware
                        │
                        ▼
                 checkAdminRole inspects 'x-user-email' header & DB role
                        │
                        ▼
                 Executes createTour in controllers/tour.controller.js
                        │
                        ▼
                 Instantiates Mongoose Tour model (models/Tour.js)
                        │
                        ▼
                 Saves to MongoDB -> Returns 201 Created JSON response
```

---

## Common Development Problems

1. **MongoDB Connection Failed**:
   - Verify MongoDB is running locally or that your Atlas cluster IP access whitelist includes your current IP address.
   - Ensure `MONGO_URL` is set correctly in `.env`.
2. **CORS Error from Frontend Client**:
   - Update the `allowedOrigins` array in `app.js` to include your client app URL.
3. **AWS S3 Upload Authorization Errors**:
   - Verify `ACCESS_KEY` and `SECRET_ACCESS_KEY` credentials in `.env`.
4. **Access Denied on Admin Endpoints**:
   - Ensure the request includes the `x-user-email` header with an email registered as `admin` or `coordinator` in the database.

---

## Troubleshooting

### Error: `Data base connection failed!`
- **Cause**: Invalid connection string or unreachable database host.
- **Fix**: Check `MONGO_URL` in `.env` and verify network connectivity.

### Error: `Email is required in header for authentication`
- **Cause**: Endpoint uses `checkAdminRole` middleware, but `x-user-email` header was omitted.
- **Fix**: Add `x-user-email: admin@example.com` to your request headers.

---

## FAQ

#### Q: Which Node.js version should I use?
Node.js v18 or higher is recommended (matching the base image in `Dockerfile`).

#### Q: How does tour cloning work under the hood?
Tour cloning (`/api/clone-tour`) uses MongoDB session transactions (`mongoose.startSession()`). It copies a tour document and all associated child entities (checkpoints, notes, transport, accommodation, inclusions, exclusions) atomically. If any insertion fails, the entire transaction rolls back.

#### Q: Why is AWS S3 used in `controllers/s3bucket.js`?
To allow clients to upload images directly to AWS S3 using secure, short-lived presigned URLs without streaming large binary files through the Node.js server.

---

## Roadmap

- [ ] Add unit and integration testing suite (Jest / Supertest).
- [ ] Implement request validation library (Zod or Express-Validator).
- [ ] Add OpenAPI / Swagger UI documentation generation.
- [ ] Standardize error response middleware across all controllers.
- [ ] Set up GitHub Actions CI workflow for linting and tests.

---

## License

This project does not currently specify an explicit open-source license. Maintainers may consider adding an **MIT License** or **Apache 2.0 License**.

---

## Code of Conduct

Maintainers and contributors are expected to uphold an inclusive, safe, and respectful environment for everyone. (Creating a formal `CODE_OF_CONDUCT.md` file is recommended).

---

## Security Policy

For security concerns or to report vulnerabilities, please contact maintainers directly before opening public issues. (Creating a formal `SECURITY.md` file is recommended).

---

## Recommended Open-Source Files

Based on inspecting this repository, the following open-source community files are recommended:

| File | Status | Recommendation |
| ---- | ------ | -------------- |
| `README.md` | Existing (Updated) | Essential project overview & documentation. |
| `.env.example` | Missing | Add to provide a safe sample environment file. |
| `CONTRIBUTING.md` | Missing | Add to extract contribution guidelines into a separate guide. |
| `LICENSE` | Missing | Add an MIT or Apache 2.0 license file. |
| `CODE_OF_CONDUCT.md` | Missing | Add Contributor Covenant code of conduct. |
| `SECURITY.md` | Missing | Add security reporting guidelines. |

---

## Project Metadata

- **Repository Name**: `trakies_backend`
- **Package Name**: `trakies`
- **Runtime**: Node.js (ES Modules)
- **Primary Framework**: Express.js & Mongoose
- **Default Port**: 9000
