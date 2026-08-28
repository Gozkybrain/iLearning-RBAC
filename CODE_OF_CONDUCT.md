# iLearning - Developer Documentation

The developer's guide to the iLearning codebase. It explains *how* the app is
built and *why* it is built this way, so anyone joining the codebase can work on
it without guessing. For the high-level "what the product does" story, read
[`README.md`](README.md).

> **Companion docs**
> - [`README.md`](README.md) - concise product overview + quick start.
> - [`CONTEXT.md`](CONTEXT.md) - the design system, tokens and per-dashboard
>   behaviour.

---

## Contents

1. [Architecture](#architecture)
2. [Stack](#stack)
3. [Roles & access control (RBAC)](#roles--access-control-rbac)
4. [Authentication & security](#authentication--security)
5. [Account approval flow](#account-approval-flow)
6. [Data model](#data-model)
7. [Design tokens & theming](#design-tokens--theming)

---

## Architecture

```
                     ┌──────────────────────────┐
                     │    Public Website Pages   │
                     │  (home, about, terms,     │
                     │   staffs, services)       │
                     └───────────┬──────────────┘
                                 │
                     ┌───────────▼──────────────┐
                     │   Next.js (App Router)    │
                     │   Auth guard (client)     │
                     │   + Firestore rules       │
                     └───────────┬──────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
┌───────▼────────┐   ┌───────────▼──────────┐  ┌─────────▼─────────┐
│  Public Pages   │   │   Dashboards          │  │  Firebase Auth     │
│  (SSR/SSG)      │   │   student | parent    │  │  Email/Password    │
│                 │   │   teacher | secretary │  │  Role on the user  │
│                 │   │   bursar | principal  │  │  doc school/{uid}  │
└─────────────────┘   └───────────┬──────────┘  └────────────────────┘
                                  │
                                  ▼
                 ┌────────────────────────────────┐
                 │  Firebase Firestore            │
                 │  school/ , school-news/ ,      │
                 │  school-fees/, school-payment/ │
                 └────────────────────────────────┘
```

Everything is a **Next.js App Router** app living at the repo root (no `src/`).
The public pages are server components; the dashboards and their data panels are
client components that talk to Firestore directly. There is **no backend API
layer yet** - the client reads Firestore with the published security rules as
the enforcement point.

---

## Stack

| Layer            | Technology                                                        |
| -----------------| ------------------------------------------------------------------|
| Framework        | Next.js (App Router)                                              |
| Language         | JavaScript                                                        |
| Styling          | CSS Modules (custom design tokens, no Tailwind / UI library)      |
| Auth             | Firebase Auth (Email/Password)                                    |
| RBAC             | Role on `school/{uid}.role` (client guard + Firestore rules)      |
| Database         | Firebase Firestore                                                |
| Admin SDK        | firebase-admin (server-only, env credentials)                     |
| Security         | CSP + security headers in `next.config.mjs`                       |
| Hosting          | Vercel                                                            |
| Icons            | Custom inline SVG set in `components/ui/Icon.jsx`                 |

---

## Roles & access control (RBAC)

### Roles

| Role          | Permissions                                                              | Scope          |
| --------------| -------------------------------------------------------------------------| ----------------|
| **user**      | Pending account after registration; sees Status + Profile only            | Self           |
| **student**   | View own profile, grades, attendance, notifications                       | Self           |
| **parent**    | Read-only access to linked children (grades, attendance, fees)            | Children only  |
| **teacher**   | Enter grades, take attendance, manage linked students, edit their class   | Assigned       |
| **secretary** | Manage people (students/teachers/parents/staff), announcements, analysis | School-wide    |
| **bursar**    | Manage fees, payments (salaries/expenses), people, announcements          | Financial      |
| **principal** | Super admin. People, announcements, finance, manage any user's roles      | System-wide    |

### Multiple roles

A user has a primary `role` plus any number of `extraRoles`
(e.g. `role: "teacher"`, `extraRoles: ["bursar"]`). The nav/dashboard switching
(`lib/roles.js` `getUserRoles` / `getOtherDashboards`) lets a multi-role user
move between their dashboards. Secretary and principal person pages read both
`role` and `extraRoles` to know which detail sections to show.

### Where the role lives

- Roles are stored on the Firestore doc `school/{uid}.role`, defaulting to
  `user` (pending). Unknown/empty roles are normalized to `user` so the guard
  never loops.
- `lib/roles.js` holds `ROLES`, `ROLE_META` (label + layout mode + path) and
  `getNavItems(role)` (ordered nav).
- The principal's "Manage roles" editor (inside the person edit form) sets the
  primary role select and toggles any number of extra-role checkboxes. Changes
  apply on next sign-in.

### Layout modes

Each dashboard is `mobile-first` (student, parent, teacher, user) or `pc-first`
(secretary, bursar, principal), set in `app/dashboard/<role>/layout.jsx`:

- **mobile-first** - bottom nav on mobile; hamburger-revealed sidebar on desktop.
- **pc-first** - sidebar always visible, collapsing to icons at 1024px and to a
  hamburger drawer below 768px.

---

## Authentication & security

**Implemented**
- **Auth**: Firebase Auth with Email/Password (web SDK in `lib/firebase.js`,
  provider in `context/AuthContext.jsx`). The provider `onSnapshot`s the user's
  `school/{uid}` doc so the role/profile stay live.
- **Route guard**: `components/auth/RequireAuth.jsx` wraps every dashboard route
  via `app/dashboard/layout.jsx`. It redirects signed-out users to
  `/login?redirect=<intended>`. Bare `/dashboard` is a client redirect to the
  signed-in user's role dashboard.
- **Firestore rules**: managed directly in the Firebase console (no rules file
  in the repo). They are the data-layer enforcement point.
- **Security headers**: CSP (Firebase domains + https images),
  `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`,
  `Referrer-Policy`, `Permissions-Policy`, `COOP` - set globally in
  `next.config.mjs`.
- **Open-redirect protection**: `/login?redirect=` only accepts same-origin
  internal paths.
- **Secrets**: the Admin service account is built from env vars
  (`FIREBASE_SERVICE_ACCOUNT`, or `FIREBASE_PRIVATE_KEY` + metadata fields in
  `lib/firebase-admin.js` / `scripts/_credentials.js`); nothing service-related
  is committed.

**Planned / roadmap**
- Server-side authorization (middleware / API routes) - the client route guard
  + Firestore rules are currently the enforcement points.
- Rate limiting on login, audit-log Cloud Functions, HTTP-only cookie sessions.
- Data encrypted at rest by Firebase; TLS in transit by default.

---

## Account approval flow

How an account moves from sign-up to fully active:

1. **Registration** - the user picks the dashboard they're applying for. The
   doc is created with `role: "user"` and `requestedRole: "<pick>"`, and they
   land on `/dashboard/user` (pending).
2. **Role approval** - an admin/principal sets `role` to the applied role; they
   land on that dashboard. `requestedRole` is not cleared yet, so the account is
   still technically pending (the profile header shows a "Not Active" tag).
3. **Profile completion (student)** - the student fills personal details except
   **teacher** and **admission number**, which are school-assigned.
4. **Final approval (class teacher)** - submitted details stay pending until
   the class teacher approves; on approval `requestedRole` is cleared and the
   "Not Active" tag disappears.

---

## Data model

Everything a person is lives on one Firestore doc: `school/{uid}`. The `role`
field decides which dashboard they get; everything else is optional detail.

Two conventions apply across the whole app:

- **`SITE_URL` is the single source of truth** for the domain. It lives in
  `lib/site.js` and is imported by the root layout (metadataBase), `sitemap.js`,
  `robots.js` and the 404 page. Change it once to update the whole app.
- **`className` is the canonical class field.** Writes always use `className`;
  reads fall back to `person.class` for old data. Never write `class` to
  Firestore.

### Profile docs

A **parent** (`role: "parent"`):

```
school/{parentUid} = {
  fullName: "Ngozi Ezeh",
  email: "ngozi.ezeh@ilearning.school",
  role: "parent",
  requestedRole: "parent",              // only while the account is not active
  gender: "Female", phone: "...", address: "...",
  createdAt: <Timestamp>, updatedOn: { date, uid, name },
  profilePictureUrl: "https://...",     // or photoURL from Auth
  students: {                           // her children map
    "<studentUid>": { id: "...", name: "Gozie Brain Izuka" },
    "<studentUid2>": { id: "...", name: "Kachi Eze", pending: true }
  }
}
```

A **student** carries their identity plus pointers back:

```
school/{studentUid} = {
  fullName: "Gozie Brain Izuka",
  email: "...", role: "student",
  gender: "Male", dob: "2009-04-17",
  admissionNumber: "IL/2026/0142",      // format IL/YYYY/XXXXXXXX
  className: "SS 1", session: "Third Term",
  phone: "...", address: "...",
  createdAt: <Timestamp>, updatedOn: { date, uid, name },
  parent: { id: "<parentUid>", name: "Ngozi Ezeh" },   // guardian / emergency
  teacher: { id: "<teacherUid>", name: "Mr. Godwin Okafor" },
  attendance: { ... },                  // date map, see Attendance
  grades: { ... }                       // term map, see Grades
}
```

A **teacher** is the same shape minus the academic maps, with its own `students`
map instead of a single pointer.

### Rules that hold across all docs

- Pointers are plain objects `{ id, name }`, kept denormalized so lists render
  without extra reads.
- `pending: true` on a link means self-service, awaiting teacher approval.
- `updatedOn` is a map `{ date, uid, name }` (editor + timestamp); `createdAt`
  never changes. When sorting by recency, sort by `createdAt`/`date`, not by the
  `updatedOn` map.
- Maps like `students` are replaced wholesale with `update()` when edited from a
  profile page so removals stick (never `set(..., merge)` for maps).

### Family links

Three ways a link comes into existence:

1. **Script** (`linkFamily.js`) - approved links written directly.
2. **Self-service** - a parent adds a child or a student picks a guardian from
   the profile pages. Both sides get `pending: true`.
3. **Teacher-initiated** - added from the teacher profile; both sides go live
   immediately. Take-overs are blocked if the student already has a teacher.

Approval happens on the teacher's **Students** tab
(`/dashboard/teacher/students`): pending rows show an Approve button that flips
`pending: false` on both sides. Removal clears both sides, but only when the
back-pointer still points at the remover.

The **source of truth** for whether a link is teacher or parent is the
student's doc (`teacher.id` vs `parent.id`), not the shared `students` map. Dual
role (teacher + parent) person pages split each linked student into Students vs
Children by reading the student's doc.

### School news (announcements)

Announcements live in the `school-news` collection, rendered by the shared
`NotificationsPanel` (reused by every dashboard). Each doc:

| Field         | Type      | Notes                                              |
| ------------- | --------- | --------------------------------------------------- |
| `title`       | string    | Headline shown in bold                              |
| `snippet`     | string    | Short one-liner in the list view                    |
| `description` | string    | Longer version, for detail views                    |
| `role`        | string    | Target role, or `all` (missing = `all`)             |
| `date`        | timestamp | Used to sort newest first                           |
| `createdBy`   | map       | `{ id, name }` of the author                        |

### School fees

Fee payments live in the `school-fees` collection, one doc per request:

```
school-fees/{docId} = {
  parentId, parentName, studentId, studentName,
  description, amount, accountNumber,        // reference from generateFeeAccountNumber
  status: "pending" | "approved",            // (reject replaced with delete)
  moderatedBy: { id, name },                 // set on approve
  createdAt: <Timestamp>
}
```

- Parents create docs from `/dashboard/parent/fees`; the status starts
  `pending`. Pending children (unapproved links) can't be paid for.
- Approval surfaces in two places: the Bursar Fees page
  (`/dashboard/bursar/fees`) and the shared `FeesTab` drawer. On either action
  the doc gets `status` + `moderatedBy`.
- Static transfer details come from `PAYMENT_ACCOUNT` in `lib/site.js`;
  `generateFeeAccountNumber()` mints the reference (prefix from `SITE_NAME`).

### School payments

Payments out of the school (salaries, expenses) live in `school-payment`:

```
school-payment/{docId} = {
  amount, accountName, bank, accountNumber,
  fullName: "<sender>", receiverName: "<receiver>",
  description, date, payerId, receiverId
}
```

The Bursar Payment page (`/dashboard/bursar/payment`) records them: the bursar
is the sender (`payerId` = current user), the chosen person is the receiver.
The shared `PaymentsTab` and the list both render rows as
"From {payer} · To {receiver}" with each party linked to the people directory.

### Attendance

Attendance is a **date map on the user's own doc**, not a collection:

```
school/{uid} -> attendance: { "2026-05-01": true, "2026-05-02": true }
```

| Aspect   | Rule                                                          |
| -------- | -------------------------------------------------------------- |
| Key      | ISO date (`YYYY-MM-DD`)                                       |
| Value    | `true` = attended that day                                    |
| Absent   | Any weekday missing between the first and last recorded dates |
| Weekend  | Never stored or displayed                                     |

Rendered by the shared `AttendancePanel` (summary + a Mon-Fri month calendar).
Teachers mark the register at `/dashboard/teacher/attendance`: one date per
session, present/absent toggles; saving writes `true` entries and deletes keys
for absentees (dot-path updates only).

### Grades

Grades keep a student's full academic history as a **term map** on the doc:

```
school/{uid} -> grades: {
  "ss1-first-term": {
    class: "SS 1", session: "First Term", position: 2,
    subjects: {
      Mathematics: { ca: 32, exams: 48, bonus: 0, total: 80, remark: "..." }
    }
  }
}
```

| Field | Notes                                                          |
| ----- | --------------------------------------------------------------- |
| key   | Slug like `ss1-first-term`; alphabetical order = chronological |
| `class`/`session` | Display labels for the term                           |
| `position` | Overall class rank for the whole term                     |
| `subjects` | Keyed by subject name                                      |
| `ca`/`exams`/`bonus` | Out of 40 / out of 60 / extra credit               |
| `total` | Stored exactly as entered (`ca + exams + bonus`)              |
| `remark` | Teacher remark for the subject                               |

Rendered by the shared `GradesPanel` (term rows open a drawer with per-subject
detail). Letter grades/colors come from `gradeFromScore()` in `utils/format.js`.
Teachers enter grades at `/dashboard/teacher/grades`; the term key comes from
their own class + session, subjects removed in the form are `deleteField()`-ed
so the merge can't resurrect them. Terms entered this way may have no position
until one is stored ("Position pending").

---

## Design tokens & theming

- Custom properties in `styles/globals.css`. Teal primary (#00D4AA), purple
  secondary (#7C3AED), amber accent (#F59E0B), and a grading scale
  (A -> F green to red). Uniform 7px radius everywhere.
- Two-mode theming (light -> dark) applied as `data-theme` on `<html>`, toggled
  from the header, persisted to `localStorage`, applied pre-paint (no flash).
- Single responsive breakpoint at **768px**. Data density scales with screen
  (tables become cards on mobile, etc.).

See [`CONTEXT.md`](CONTEXT.md) for the full design system.
