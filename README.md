# Digital_Buisness_Directory
# LocalList: Digital Business Directory (MERN)

A full-stack directory where visitors can search and browse businesses, owners can list their own, and admins moderate everything.

**Stack:** MongoDB + Mongoose, Express.js, React 18 (Vite), Node.js. Auth is JWT with bcrypt-hashed passwords.

## Features

- **Search and filter:** partial-match search across name, description, city and address, plus category and city filters and pagination. Search is debounced.
- **Business records:** name, description, category, location (address, city, state, PIN), contact (phone, email) and website.
- **CRUD:** add, view, update and delete businesses; admins also manage categories and users.
- **Authentication:** register, log in, session restored on refresh.
- **Role-based access:** enforced in the API (middleware) and mirrored in the UI.
- **Moderation:** owner listings start as *pending*; an admin approves or rejects them. Editing an approved listing sends it back for review.

| Action | Visitor | User | Owner | Admin |
|---|:-:|:-:|:-:|:-:|
| Search, browse and view approved listings | yes | yes | yes | yes |
| Add listings | | | yes (pending) | yes (auto-approved) |
| Edit / delete listings | | | own only | all |
| Approve / reject listings | | | | yes |
| Manage categories and users | | | | yes |

## Getting started

Requires Node 18+ and a running MongoDB (local or Atlas).

```bash
# 1. API
cd server
npm install
cp .env.example .env        # set MONGO_URI and a long random JWT_SECRET
npm run seed                # optional: sample data and demo accounts (clears existing data)
npm run dev                 # http://localhost:5000

# 2. Client (second terminal)
cd client
npm install
npm run dev                 # http://localhost:5173  (proxies /api to port 5000)
```

Demo accounts created by `npm run seed`:

| Role | Email | Password |
|---|---|---|
| Admin | admin@example.com | admin123 |
| Owner | owner@example.com | owner123 |
| User | user@example.com | user123 |

Public sign-up only creates *user* or *owner* accounts. Promote someone to admin from the Admin → Users tab (or by editing the `role` field in MongoDB).

## Project structure

```
server/
  app.js, server.js, seed.js
  config/db.js
  models/        User, Category, Business
  middleware/    auth.js (protect, optionalAuth, authorize), error.js
  controllers/   auth, business, category, admin
  routes/        auth, businesses, categories, admin
client/src/
  api.js                      axios instance with token header
  context/AuthContext.jsx     login, register, logout, session restore
  components/                 Navbar, ProtectedRoute, BusinessEntry, StatusBadge
  pages/                      Home, BusinessDetail, Auth, BusinessForm, Dashboard, Admin
```

## REST API

| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| POST | `/api/auth/register` | public | Create account |
| POST | `/api/auth/login` | public | Log in, returns JWT |
| GET | `/api/auth/me` | logged in | Current user |
| GET | `/api/businesses?search=&category=&city=&page=&limit=` | public | Search approved listings |
| GET | `/api/businesses/cities` | public | Cities that have listings |
| GET | `/api/businesses/:id` | public / owner / admin | One listing (non-approved only for owner/admin) |
| GET | `/api/businesses/mine` | owner, admin | My listings, any status |
| GET | `/api/businesses/all?status=` | admin | Moderation queue |
| POST | `/api/businesses` | owner, admin | Create listing |
| PUT | `/api/businesses/:id` | owner (own), admin | Update listing |
| DELETE | `/api/businesses/:id` | owner (own), admin | Delete listing |
| PATCH | `/api/businesses/:id/status` | admin | Approve / reject |
| GET | `/api/categories` | public | List categories |
| POST / PUT / DELETE | `/api/categories[/:id]` | admin | Manage categories (delete blocked if in use) |
| GET | `/api/admin/stats` | admin | Counts for the dashboard |
| GET | `/api/admin/users` | admin | List users |
| PATCH | `/api/admin/users/:id/role` | admin | Change a role |
| DELETE | `/api/admin/users/:id` | admin | Delete user and their listings |

Errors are returned as `{ "message": "..." }` with 400 (validation), 401 (not logged in), 403 (wrong role), 404 or 409 (duplicate).

## Production notes

`cd client && npm run build` creates `client/dist`. Serve it from any static host and set `CLIENT_URL` on the API to that origin. If the API is on a different origin, change `baseURL` in `client/src/api.js` to the full API URL.

## Ideas for extension

Image upload for listings, ratings and reviews, a map view using coordinates, rate limiting and helmet on the API, and password reset by email.
