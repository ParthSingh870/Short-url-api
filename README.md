# URL Shortener API & Management Platform

A high-performance URL shortening and analytics web service built with Node.js, Express, and MongoDB. The system converts lengthy URLs into unique, compact identifiers, provides fast HTTP 302 redirections, records granular visit analytics with timestamps, and ensures strict data isolation across user sessions using cookie-based authentication.

---

## Features

- **URL Shortening**: Generates collision-resistant, 7-9 character short identifiers for any valid destination URL using `shortid`.
- **Public URL Redirection**: Fast HTTP 302 redirection from `GET /url/:shortId` to the registered destination URL without requiring visitor authentication.
- **Click Analytics & History**: Tracks total visits and appends Unix timestamp records to a visit history log on each click.
- **Session-Based Authentication**: Secure stateful authentication using UUID v4 session IDs stored in an in-memory session map and transmitted via HTTP-only cookies (`uid`).
- **User-Isolated Dashboard**: Server-rendered EJS interface allowing logged-in users to view only their own shortened links and real-time click statistics.
- **Defensive Error Handling**: Graceful 400 and 404 responses for missing fields, duplicate user registrations, and non-existent short links, preventing unhandled runtime exceptions.

---

## Tech Stack

- **Runtime**: Node.js (v16+)
- **Backend Framework**: Express.js (v4.x)
- **Database & ODM**: MongoDB with Mongoose (v6.x)
- **View Engine**: EJS (Embedded JavaScript)
- **Session & Identity**: UUID (v4), Cookie-Parser
- **ID Generation**: Shortid

---

## Project Structure

```text
.
├── controllers/
│   ├── url.js             # Short URL generation and analytics query handlers
│   └── user.js            # User registration and authentication handlers
├── middlewares/
│   └── auth.js            # Session validation and route protection middlewares
├── models/
│   ├── url.js             # Mongoose schema for URLs, timestamps, and user references
│   └── user.js            # Mongoose schema for user credentials (name, email, password)
├── routes/
│   ├── staticRouter.js    # Routes rendering SSR views (dashboard, login, signup)
│   ├── url.js             # REST endpoints for short URL creation and analytics
│   └── user.js            # Routes handling signup and login actions
├── service/
│   └── auth.js            # Stateful in-memory session storage (Map: sessionId -> user)
├── views/
│   ├── home.ejs           # SSR dashboard displaying URL generation form and table
│   ├── login.ejs          # User login page
│   └── signup.ejs         # User signup page
├── connect.js             # MongoDB connection establishment utility
├── index.js               # Application entry point, middleware pipeline, and server bootstrap
├── package.json           # Project manifest, scripts, and dependencies
├── .env.example           # Template for required environment variables
└── README.md              # Project documentation
```

---

## Getting Started

### Prerequisites
- Node.js (v16 or higher)
- MongoDB running locally on port `27017` or a remote MongoDB Atlas cluster URI

### Installation & Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/ParthSingh870/Short-url-api.git
   cd Short-url-api
   ```

2. **Install project dependencies:**
   ```bash
   npm install
   ```

3. **Configure Environment Variables:**
   Copy the example environment file and adjust values as needed:
   ```bash
   cp .env.example .env
   ```
   *(On Windows PowerShell: `Copy-Item .env.example .env`)*

4. **Run the Application:**
   ```bash
   # Development mode with nodemon auto-reload:
   npm start

   # Production mode:
   node index.js
   ```
   The application will boot on `http://localhost:8000`.

---

## API Documentation

### Endpoints Overview

| Method | Endpoint | Auth Required | Description |
|---|---|:---:|---|
| `POST` | `/user` | No | Registers a new user account |
| `POST` | `/user/login` | No | Authenticates user credentials and issues session cookie |
| `POST` | `/url` | Yes (`uid` cookie) | Generates a new short URL for the authenticated user |
| `GET` | `/url/:shortId` | No (Public) | Increments visit history and redirects (302) to original URL |
| `GET` | `/url/analytics/:shortId` | Yes (`uid` cookie) | Returns total clicks and visit timestamps for a short URL |
| `GET` | `/` | Yes (`uid` cookie) | Renders user dashboard with list of owned URLs |
| `GET` | `/login` | No | Renders login page view |
| `GET` | `/signup` | No | Renders signup page view |

---

### Request & Response Examples

#### 1. Generate Short URL
- **Endpoint**: `POST /url`
- **Headers**: `Content-Type: application/json`, `Cookie: uid=<session_id>`
- **Request Body**:
  ```json
  {
    "url": "https://antigravity.google.com"
  }
  ```
- **Response**: Renders `home.ejs` with the newly created short ID (`{ id: "xK9L2aB" }`).

#### 2. Get URL Analytics
- **Endpoint**: `GET /url/analytics/:shortId`
- **Headers**: `Cookie: uid=<session_id>`
- **Response** (`200 OK`):
  ```json
  {
    "totalClicks": 3,
    "analytics": [
      {
        "timestamp": 1730855420000,
        "_id": "672b15749320e104192b001a"
      },
      {
        "timestamp": 1730855425000,
        "_id": "672b15799320e104192b001b"
      },
      {
        "timestamp": 1730855430000,
        "_id": "672b157e9320e104192b001c"
      }
    ]
  }
  ```

#### 3. Redirection
- **Endpoint**: `GET /url/:shortId`
- **Response** (`302 Found`):
  - `Location: https://antigravity.google.com`
  - Atomically appends `{ timestamp: Date.now() }` to the URL document's `visitHistory`.

---

## Authentication & Authorization

- **Session Management**: Upon successful login (`POST /user/login`), the server generates a unique UUID v4 identifier, maps it to the user object in an in-memory `Map` (`sessionIdToUserMap`), and sets a cookie named `uid` in the HTTP response.
- **Route Protection Middleware**: The `restrictToLoggedinUserOnly` middleware inspects the incoming `req.cookies.uid`. If the session ID is missing or invalid, the client is redirected to `/login`.
- **Data Isolation**: Each shortened URL stores a reference to its creator (`createdBy: req.user._id`). The dashboard query explicitly filters records (`URL.find({ createdBy: req.user._id })`), ensuring users can only see their own shortened links and statistics.

---

## Key Design Decisions

1. **Separation of Concerns (MVC)**: The codebase separates data definitions (`models`), business logic (`controllers`), request filtering (`middlewares`), and presentation (`views`), ensuring maintainability and clear unit boundaries.
2. **Atomic Click Tracking with `$push`**: URL redirection updates click analytics using Mongoose `findOneAndUpdate` with the `$push` operator. This ensures atomic updates to `visitHistory` in a single database round-trip without race conditions.
3. **Public Redirection vs. Protected Management**: Redirection (`GET /url/:shortId`) is publicly accessible so anyone receiving a short link can reach the destination, while administrative actions (link creation and analytics inspection) remain strictly authenticated.
4. **Indexed Identifiers in MongoDB**: The `shortId` field has a unique index (`unique: true`), ensuring $O(1)$ indexed lookup times when routing incoming redirection requests and preventing duplicate short IDs.
5. **Stateful In-Memory Session Architecture**: Uses a lightweight Map-based session store for fast $O(1)$ lookups, decoupling authentication from database reads on every single request.
6. **Defensive Error Handling**: Controller and routing boundaries validate null values and duplicate key conflicts (`E11000`), returning explicit HTTP 400 and 404 responses instead of triggering unhandled promise rejections that crash Node.js processes.

---

## Future Improvements

- **Password Hashing**: Integrate `bcryptjs` to hash user passwords before storing them in MongoDB.
- **Distributed Session Store**: Replace the in-memory JavaScript `Map` with Redis to support multi-instance horizontal scaling.
- **Custom Alias & Expiry**: Allow users to specify custom vanity short URLs and link expiration TTLs using MongoDB TTL indexes.
- **Rate Limiting**: Implement `express-rate-limit` to prevent brute-force abuse on redirection and creation endpoints.

---

## Author

**Parth Madhav Singh**  
3rd Year B.Tech, Computer Science & Engineering  
Indian Institute of Information Technology (IIIT) Bhagalpur  

- **GitHub**: [@ParthSingh870](https://github.com/ParthSingh870)
- **LinkedIn**: [linkedin.com/in/parth-singh-008288330](https://linkedin.com/in/parth-singh-008288330)
