# URL Shortener API & Web Application

A full-stack URL shortener backend application built with **Node.js**, **Express.js**, **MongoDB (Mongoose)**, and **EJS templating engine**. It allows users to create short links, tracks link click analytics with timestamps, and isolates short URLs per authenticated user account.

---

## 🚀 Features

- **URL Shortening**: Convert long web links into compact, unique short URLs using `shortid`.
- **Fast Redirection**: Fast HTTP 302 redirection from short links to original destination websites.
- **Analytics & Click Tracking**: Detailed click counting and timestamped visit history for every short URL.
- **Session-Based Authentication**: Secure cookie session management (`uid`) for user signup and login.
- **User Dashboard & URL Isolation**: Server-rendered EJS interface where users only view and manage their own shortened links.
- **Robust Error Handling**: Resilient against duplicate emails, missing parameters, and non-existent short links (clean 400/404 handling without server crashes).

---

## 🛠️ Tech Stack

- **Runtime**: [Node.js](https://nodejs.org/)
- **Framework**: [Express.js](https://expressjs.com/)
- **Database**: [MongoDB](https://www.mongodb.com/) with [Mongoose ODM](https://mongoosejs.com/)
- **Templating**: [EJS (Embedded JavaScript templates)](https://ejs.co/)
- **Authentication**: In-memory Session Mapping with UUID v4 & Cookie-Parser
- **ID Generator**: [shortid](https://www.npmjs.com/package/shortid)

---

## 📁 Project Structure

```text
short-url-nodejs/
│
├── controllers/
│   ├── url.js             # Handles URL creation and analytics fetching
│   └── user.js            # Handles signup and login authentication
│
├── middlewares/
│   └── auth.js            # Protects routes and attaches authenticated user
│
├── models/
│   ├── url.js             # Mongoose schema for short URLs and visit history
│   └── user.js            # Mongoose schema for user accounts
│
├── routes/
│   ├── staticRouter.js    # Renders home dashboard, login, and signup views
│   ├── url.js             # API routes for generating short URLs & analytics
│   └── user.js            # Auth routes for user signup and login
│
├── service/
│   └── auth.js            # In-memory session store mapping sessionId to user
│
├── views/
│   ├── home.ejs           # Main dashboard with URL generator and click stats table
│   ├── login.ejs          # Login form view
│   └── signup.ejs         # Signup form view
│
├── connect.js             # MongoDB connection utility
├── index.js               # Application entry point and server configuration
├── package.json           # Project dependencies and npm scripts
└── README.md              # Project documentation
```

---

## ⚙️ Installation & Setup

### 1. Prerequisites
- [Node.js](https://nodejs.org/) (v16 or higher)
- [MongoDB](https://www.mongodb.com/try/download/community) installed and running locally on port 27017, or a MongoDB Atlas connection string.

### 2. Clone the Repository
```bash
git clone https://github.com/ParthSingh870/Short-url-api.git
cd Short-url-api
```

### 3. Install Dependencies
```bash
npm install
```

### 4. Configure Environment (Optional)
By default, the application connects to local MongoDB at `mongodb://localhost:27017/short-url`.
To specify a custom MongoDB URI:
```bash
# Windows PowerShell
$env:MONGODB="mongodb://localhost:27017/short-url"

# Linux / macOS
export MONGODB="mongodb://localhost:27017/short-url"
```

### 5. Start the Server
```bash
# Start with nodemon (development)
npm start

# Or start with node
node index.js
```
The server will start at: `http://localhost:8000`

---

## 📡 API Endpoints

### 1. Authentication (`/user`)
| Method | Endpoint | Description | Payload |
|---|---|---|---|
| `POST` | `/user` | Register a new user | `{ name, email, password }` |
| `POST` | `/user/login` | Authenticate user & set session cookie | `{ email, password }` |

### 2. URL Management (`/url`)
| Method | Endpoint | Description | Auth Required |
|---|---|---|:---:|
| `POST` | `/url` | Generate a new short URL | Yes (Cookie) |
| `GET` | `/url/:shortId` | Redirects to original long URL | No (Public) |
| `GET` | `/url/analytics/:shortId` | Returns click count and visit timestamps | Yes (Cookie) |

### 3. Web Views (`/`)
| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Dashboard showing user's generated URLs and click statistics |
| `GET` | `/login` | Render login form |
| `GET` | `/signup` | Render signup form |

---

## 📊 Analytics Example Response

Request to `GET /url/analytics/:shortId`:
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
    }
  ]
}
```

---

## 🛡️ License

This project is licensed under the ISC License.
