# iLearning - All-in-One School Platform (Nigeria)

A complete digital school platform for Nigerian private schools. It runs the
school's **public website** (home, about, services, staff, terms) and the
**DIOS** - a Digital Institution Operating System with logins and a dedicated
dashboard for every role in the school.

> DEMO: [i-learning-os.vercel.app](https://i-learning-os.vercel.app)

---

## What it does

- **Public website** - a marketing site that tells the school's story and
  converts visitors into enrolments.
- **Student dashboard** - grades, attendance, notifications, profile.
- **Parent dashboard** - follow every child's grades, attendance and fees.
- **Teacher dashboard** - take registers, enter grades, manage students.
- **Secretary / Bursar / Principal dashboards** - people, announcements,
  finance, fees, payments and full-school management.

Any visitor can browse the public pages. Signing in unlocks the dashboard that
matches their account role (student, parent, teacher, secretary, bursar or
principal).

---

## Tech stack

| Layer       | Technology                                    |
| ------------ | --------------------------------------------- |
| Framework   | Next.js (App Router)                          |
| Language    | JavaScript                                    |
| Styling     | CSS Modules (no Tailwind, no UI library)      |
| Auth        | Firebase Auth (Email/Password)                |
| Database    | Firebase Firestore                            |
| Security    | CSP + security headers in `next.config.mjs`   |
| Hosting     | Vercel                                        |

---

## Quick start

```bash
# 1. Install dependencies
npm install

# 2. Create your env file from the template
cp env.example .env.local
# then fill in your Firebase config + service account

# 3. Start the dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Env vars

Copy `env.example` to `.env.local` and fill in:

- Firebase web SDK config (used by the client): `NEXT_PUBLIC_FIREBASE_*`
- Firebase Admin service account (server-only): `FIREBASE_SERVICE_ACCOUNT`,
  or `FIREBASE_PRIVATE_KEY` + the metadata fields it needs.

Nothing service-related is committed to the repo.

---

## Useful scripts

All scripts live in `scripts/`, use the Admin SDK and run against `.env.local`.
See `scripts/Roles.md` for the role-assignment workflow.

**Assign a role to an existing user** (each forces the user to log in again):

```bash
node --env-file=.env.local scripts/makeUser.js <uid>
node --env-file=.env.local scripts/makeStudent.js <uid>
node --env-file=.env.local scripts/makeParent.js <uid>
node --env-file=.env.local scripts/makeTeacher.js <uid>
node --env-file=.env.local scripts/makeSecretary.js <uid>
node --env-file=.env.local scripts/makeBursar.js <uid>
node --env-file=.env.local scripts/makePrincipal.js <uid>
node --env-file=.env.local scripts/addExtraRoles.js <uid>  # add secondary roles
```

**Create / attach accounts and seed data**:

```bash
node --env-file=.env.local scripts/createAccounts.js     # batch-create accounts/roles
node --env-file=.env.local scripts/seedNews.js           # sample announcements
node --env-file=.env.local scripts/seedGrades.js         # full grade history
node --env-file=.env.local scripts/seedAttendance.js     # attendance map
node --env-file=.env.local scripts/seedFees.js           # school-fee records
node --env-file=.env.local scripts/seedMoreFees.js       # richer fee/payment set
node --env-file=.env.local scripts/seedPayments.js       # salary history
node --env-file=.env.local scripts/linkFamily.js         # link a student's teacher + parent
```

Other scripts are one-off fixtures (e.g. `seedKachi.js`, `seedPrincipalParent.js`,
`soloPrincipal.js`, `addTunde.js`) - read each file's header comment before
running.

---

## Routes

| Path                             | Page                                        | Access |
| ---------------------------------| ---------------------------------------------| -------|
| `/`                              | Home                                        | Public |
| `/about`                         | About                                       | Public |
| `/terms`                         | Terms & Conditions                          | Public |
| `/staffs` / `/staffs/[id]`       | Staff directory / detail                    | Public |
| `/services` / `/services/[id]`   | Programs / program detail                   | Public |
| `/login` / `/register`           | Sign in / create account                    | Public |
| `/dashboard`                     | Redirect to your role dashboard             | Signed-in |
| `/dashboard/user`                | Pending + `profile`                         | Pending user |
| `/dashboard/student`             | Grades, attendance, notifications, profile  | Student |
| `/dashboard/parent`              | Children, fees, notifications, profile      | Parent |
| `/dashboard/teacher`             | Students, grades, attendance, profile       | Teacher |
| `/dashboard/secretary`           | People, announcements, analysis, profile    | Secretary |
| `/dashboard/bursar`              | Fees, payment, people, announcements, profile | Bursar |
| `/dashboard/principal`           | People, announcements, finance, profile     | Principal |

Every role also has a person profile page at
`/dashboard/<role>/people/<id>` (editable for secretary/bursar/principal,
read-only for everyone else).

---

## Project structure

Next.js App Router convention, at the repo root (no `src/`). Dashboard routes
are **flat** - one folder per role, no route groups - so each URL is simply
`/dashboard/<role>`.

```
.
├── app/
│   ├── layout.jsx               # root layout: theme script, auth provider
│   ├── page.jsx                 # / (home)
│   ├── about/  terms/  login/  register/
│   ├── staffs/  services/       # + [id] detail pages
│   ├── not-found.jsx  sitemap.js  robots.js
│   └── dashboard/
│       ├── layout.jsx           # RequireAuth guard (auth)
│       ├── page.jsx             # /dashboard -> role redirect
│       ├── user/ student/ parent/ teacher/
│       ├── secretary/ bursar/ principal/
│       └── <role>/people/[id]   # person detail (editable or read-only)
├── components/
│   ├── public/                  # SiteHeader, SiteFooter, Hero, StaffsList...
│   ├── navigation/              # DashboardLayout, HeaderUser, Sidebar, nav...
│   ├── dashboard/               # shared panels (GradesPanel, AttendancePanel,
│   │                            #   NotificationsPanel, PersonProfile, FeesTab,
│   │                            #   PaymentsTab, SearchSelect, StatCard)
│   ├── auth/                    # RequireAuth, GuestGuard
│   └── ui/                      # Card, Skeleton, EmptyState, Icon, Loader, Faq
├── context/
│   └── AuthContext.jsx          # auth provider + role normalization
├── lib/
│   ├── firebase.js              # Firebase web SDK (client)
│   ├── firebase-admin.js        # Firebase Admin SDK (server-only)
│   ├── roles.js                 # ROLES, ROLE_META, getNavItems
│   └── site.js                  # SITE_URL (single source of truth for domain)
├── scripts/                     # Admin SDK role / seed scripts
├── styles/
│   ├── globals.css              # design tokens + theme
│   ├── dashboard/               # shell/nav module styles
│   └── pages/                   # page module styles
├── utils/                       # format.js (currency, grade letters, dates)
├── env.example
└── next.config.mjs              # security headers + CSP
```

---

## API routes (planned)

> **Status: planned, not yet implemented.** No `/api/*` routes exist yet. RBAC is
> enforced client-side (route guard) plus Firestore security rules. The table
> below is the target once a server-side authorization layer is added.

| Method | Route                        | Auth                              | Description                         |
| -------| -----------------------------| -----------------------------------| -------------------------------------|
| POST   | `/api/auth/login`            | None                              | Login (Firebase ID token exchange)  |
| POST   | `/api/auth/logout`           | Any                               | Logout                              |
| GET    | `/api/auth/session`          | Any                               | Get current session                 |
| GET    | `/api/students`              | Principal, Teacher                | List students                       |
| GET    | `/api/students/[id]`         | Student, Parent, Teacher, Principal | Student details                  |
| GET    | `/api/grades` / POST         | Student/Parent/Teacher/Principal / Teacher | Grades                |
| GET    | `/api/attendance` / POST     | Same as grades / Teacher          | Attendance                          |
| GET    | `/api/fees`                  | Bursar, Principal                 | Fee records                         |
| POST   | `/api/fees/payment`          | Bursar                            | Record payment                      |
| GET    | `/api/timetable`             | Student, Teacher                  | Timetable                           |
| POST   | `/api/announcements`         | Secretary, Principal              | Create announcement                 |
| GET    | `/api/audit-log`             | Principal                         | View audit log                      |

---

## Documentation

Two deeper documents live alongside this README:

- **[`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)** - the developer's guide: how the
  app is built, the architecture, RBAC model, data shapes and how the pieces
  fit together.
- **[`CONTEXT.md`](CONTEXT.md)** - the design system and detailed behaviour of
  every dashboard and component.

If you want to jump straight into how the code works, start at
[`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).
