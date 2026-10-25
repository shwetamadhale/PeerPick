# PeerPick

A group recommendation app — share movies, books, music, and more with the people whose taste you actually trust.

> **MVP in progress.** The core loop is: someone shares a recommendation → their group sees it → someone reacts to it.

---

## Table of Contents

- [What is PeerPick?](#what-is-peerpick)
- [MVP Scope](#mvp-scope)
  - [In scope](#in-scope)
  - [Out of scope (v2 backlog)](#out-of-scope-v2-backlog)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Data Models](#data-models)
- [API Endpoints](#api-endpoints)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Running Locally](#running-locally)
- [Build Roadmap](#build-roadmap)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

---

## What is PeerPick?

PeerPick is a lightweight social app for sharing recommendations within small groups. Instead of asking the whole internet what to watch, read, or listen to, you ask your group. Post a recommendation with a short review, a rating, and a few tags. Your group sees it in their feed, reacts to it, and everyone gets better picks.

**Core loop:** Post → See → React.

---

## MVP Scope

The MVP is deliberately small. It exists to prove the core loop works for a handful of real users. Everything else is a v2 ticket.

### In scope

- **Auth** — sign up, log in, log out, JWT session. Basic profile (name, avatar).
- **Groups** — create a group, join via invite code. (No roles or admin permissions; creator is implicit admin.)
- **Recommendations** — post to a group with title, media type, review text, 1–5 rating, and free-text tags.
- **Feed** — chronological feed of recommendations from groups you're in. Like/dislike (one vote per user).
- **Filtering** — filter the feed by group, media type, and tag (client-side is fine).
- **Search** — search recommendations by title/tag (simple MongoDB text/regex query).

### Out of scope (v2 backlog)

- Image/poster uploads (Cloudinary)
- Comments and threaded discussion
- Notifications (in-app or email)
- Sort options beyond "newest" (e.g. "top rated")
- Group roles, permissions, settings, leaving/removing members
- Password reset and email verification
- Mobile responsiveness polish, dark mode, animations
- Rich tag system (autocomplete, taxonomy)

These aren't "missing" — they're intentionally deferred until the core loop is validated.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Frontend | React (Vite) |
| Backend | Node.js + Express |
| Database | MongoDB (Atlas free tier) + Mongoose |
| Auth | Passport.js (JWT strategy) |
| Image storage | Cloudinary *(v2 — not in MVP)* |
| Frontend hosting | Vercel |
| Backend hosting | Render |

---

## Project Structure

```
peerpick/
  client/               React app (Vite)
    src/
      pages/            Login, Signup, Feed, GroupView, NewRecommendation
      components/       RecommendationCard, GroupList, FilterBar
      api/              axios client + endpoint calls
      context/          AuthContext
  server/               Express API
    models/             User.js, Group.js, Recommendation.js
    routes/             auth.js, groups.js, recommendations.js
    middleware/         auth.js (JWT verification)
    config/             db.js
    server.js
```

---

## Data Models

Mongoose schemas, kept flat for MVP scale.

### User

```js
{
  _id,
  username,
  email,
  passwordHash,
  avatarUrl,       // optional, placeholder default
  createdAt,
}
```

### Group

```js
{
  _id,
  name,
  createdBy,       // User ref
  members,         // [User refs]
  inviteCode,      // short random string
  createdAt,
}
```

### Recommendation

```js
{
  _id,
  group,           // Group ref
  author,          // User ref
  title,
  mediaType,       // enum: movie | book | music | other
  reviewText,
  rating,          // 1–5
  tags,            // [String]
  likes,           // [User refs]
  dislikes,        // [User refs]
  createdAt,
}
```

`likes` and `dislikes` are arrays of user refs rather than separate collections — simplest way to check "did this user already vote" at MVP scale.

---

## API Endpoints

The full backend surface for MVP — about 10 endpoints.

### Auth

```
POST   /api/auth/signup
POST   /api/auth/login
GET    /api/auth/me
```

### Groups

```
POST   /api/groups                 create group
POST   /api/groups/:id/join        join via invite code
GET    /api/groups                 list my groups
```

### Recommendations & Feed

```
POST   /api/groups/:id/recommendations   create recommendation
GET    /api/groups/:id/recommendations   list for one group
GET    /api/feed                         aggregated feed across my groups
GET    /api/feed?search=&type=&tag=      filtered/search feed
POST   /api/recommendations/:id/like
POST   /api/recommendations/:id/dislike
```

---

## Getting Started

### Prerequisites

- Node.js 18+ and npm
- A MongoDB Atlas account (free tier) — or a local MongoDB instance
- Git

### Installation

Clone the repo and install dependencies for both apps:

```bash
git clone https://github.com/your-username/peerpick.git
cd peerpick

# Install server dependencies
cd server
npm install

# Install client dependencies
cd ../client
npm install
```

### Environment Variables

Create a `.env` file in `server/`:

```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_long_random_secret
CLIENT_URL=http://localhost:5173
```

Create a `.env` file in `client/`:

```env
VITE_API_URL=http://localhost:5000
```

> Use a long, random `JWT_SECRET`. Never commit `.env` files — they're already in `.gitignore`.

### Running Locally

In one terminal, start the backend:

```bash
cd server
npm run dev
```

In another terminal, start the frontend:

```bash
cd client
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

---

## Build Roadmap

The MVP is built in three phases. Ship after Phase 3.

**Phase 1 — Skeleton**

1. Express server + MongoDB connection + folder structure
2. User model + signup/login with JWT
3. React scaffold + auth pages + protected routing
4. Deploy the skeleton early (Render + Vercel) to prove the pipeline

**Phase 2 — Core loop**

5. Group model + create/join group
6. Recommendation model + create/view form
7. Feed page pulling recommendations from your groups
8. Like/dislike buttons

**Phase 3 — Usability pass**

9. Tag + media-type + search filtering on the feed
10. Basic styling pass (usable, not pretty)
11. Empty states, loading states, error handling on forms

---

## Deployment

| Service | Platform | Notes |
|---|---|---|
| Frontend | Vercel | Zero-config React deploy; set `VITE_API_URL` to the Render backend URL |
| Backend | Render | Free tier for small Node apps; set all `server/.env` vars in the dashboard |
| Database | MongoDB Atlas | Free tier is plenty for MVP |

Deploy the skeleton in Phase 1. Don't leave deployment until the end — proving the pipeline early avoids painful surprises at ship time.

---

## Contributing

This is an MVP under active development. If you're a collaborator:

1. Branch from `main` (`feat/…`, `fix/…`)
2. Keep PRs scoped to one roadmap item
3. Don't pull v2 backlog items into MVP PRs — file them as issues instead

---

## License

MIT — see `LICENSE` for details.
