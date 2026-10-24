# PeerPick — MVP Scope & Roadmap

## 1. Chosen Stack (locked in)

| Layer | Choice | Why |
|---|---|---|
| Frontend | React (Vite, not CRA) | Fast dev server, minimal config |
| Backend | Node.js + Express | Matches frontend language, huge ecosystem |
| Database | MongoDB (Atlas free tier) | Flexible schema, good fit for nested reviews/tags |
| Auth | Passport.js (JWT strategy) | No third-party lock-in, full control, free |
| Image storage | Cloudinary | Free tier, easy upload widget, auto image optimization |
| Frontend hosting | Vercel | Zero-config React deploys |
| Backend hosting | Render (instead of Heroku, which dropped free tier) | Free tier still exists for small Node apps |

This is the "no surprises" version of your original stack list — same technologies, just the specific tools that still have workable free tiers in 2026.

## 2. MVP Philosophy: Cut Ruthlessly

The full feature list is a v1.0 product. An MVP proves the *core loop* works: **someone shares a recommendation → their group sees it → someone reacts to it.** Everything else is scaffolding around that loop.

### In scope for MVP
- **Auth**: sign up, log in, log out, JWT session. Basic profile (name, avatar).
- **Groups**: create a group, invite by email/username, join a group. (No group roles/admin permissions yet — creator = implicit admin.)
- **Recommendations**: post a recommendation to a group — title, media type (movie/book/music/other), short review text, rating (1–5), tags (free text, comma-separated is fine for MVP).
- **Feed**: chronological feed of recommendations from groups you're in. Like/dislike (simple counter, one vote per user).
- **Filtering**: filter feed by group, media type, and tag (client-side filtering on fetched feed is fine for MVP).
- **Basic search**: search recommendations by title/tag (simple MongoDB text/regex query — no Elasticsearch).

### Explicitly OUT of MVP (v2 backlog)
- Image/poster uploads (Cloudinary) — start with a plain text field or a media-type icon instead.
- Comments/threaded discussion on recommendations.
- Notifications (in-app or email) for new posts/invites.
- Sort options beyond "newest" (e.g., "top rated") — add once you have real data.
- Group roles/permissions, group settings, leaving/removing members.
- Password reset flow, email verification.
- Mobile responsiveness polish, dark mode, animations.
- Rich tag system (autocomplete, tag taxonomy) — plain strings are enough at MVP stage.

Cutting these isn't "doing it wrong" — it's what makes an MVP shippable in weeks instead of months. Each cut item is a clean v2 ticket once the core loop is validated.

## 3. Data Models (MongoDB / Mongoose)

```
User
  _id
  username
  email
  passwordHash
  avatarUrl (optional, default placeholder)
  createdAt

Group
  _id
  name
  createdBy (User ref)
  members [User refs]
  inviteCode (short random string, simplest MVP invite mechanism)
  createdAt

Recommendation
  _id
  group (Group ref)
  author (User ref)
  title
  mediaType (enum: movie | book | music | other)
  reviewText
  rating (1-5)
  tags [String]
  likes [User refs]
  dislikes [User refs]
  createdAt
```

Keeping `likes`/`dislikes` as arrays of user refs (rather than a separate collection) is simplest at MVP scale and lets you easily check "did this user already vote."

## 4. API Endpoints (MVP surface)

```
POST   /api/auth/signup
POST   /api/auth/login
GET    /api/auth/me

POST   /api/groups                 create group
POST   /api/groups/:id/join        join via invite code
GET    /api/groups                 list my groups

POST   /api/groups/:id/recommendations   create recommendation
GET    /api/groups/:id/recommendations   list for one group
GET    /api/feed                         aggregated feed across my groups
GET    /api/feed?search=&type=&tag=      filtered/search feed

POST   /api/recommendations/:id/like
POST   /api/recommendations/:id/dislike
```

~10 endpoints total. That's the whole backend surface for MVP.

## 5. Suggested Build Order (roughly 3 phases)

**Phase 1 — Skeleton (get something running end-to-end)**
1. Express server + MongoDB connection + folder structure
2. User model + signup/login with JWT
3. React app scaffold + auth pages + protected routing
4. Deploy skeleton immediately (Render + Vercel) so the pipeline is proven early, not left until the end

**Phase 2 — Core loop**
5. Group model + create/join group
6. Recommendation model + create/view recommendation form
7. Feed page pulling recommendations from a user's groups
8. Like/dislike buttons

**Phase 3 — Usability pass**
9. Tag + media-type + search filtering on the feed
10. Basic styling pass (doesn't need to be pretty, needs to be usable)
11. Empty states, loading states, error handling on forms

Ship after Phase 3. That's a real MVP a few friends could actually use.

## 6. Suggested Repo Structure

```
peerpick/
  client/               React app (Vite)
    src/
      pages/             Login, Signup, Feed, GroupView, NewRecommendation
      components/        RecommendationCard, GroupList, FilterBar
      api/                axios client + endpoint calls
      context/           AuthContext
  server/               Express API
    models/             User.js, Group.js, Recommendation.js
    routes/             auth.js, groups.js, recommendations.js
    middleware/         auth.js (JWT verification)
    config/             db.js
    server.js
```

## 7. Next Step

Once you confirm this scope, I can scaffold the actual repo — both `client/` and `server/` folders, with auth wired up and a working feed page — so you have running code to build on rather than an empty repo.
