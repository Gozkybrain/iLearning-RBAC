# iLearning - Design Context

## Design Philosophy

iLearning serves two distinct user worlds within the same school. Students, parents, and teachers interact with the platform on the go -- checking grades, attendance, assignments from their phones. Principal, bursar, and secretary run the school from a desk -- managing data, processing payments, generating reports on larger screens. The design respects this split.

---

## Responsive Strategy

Single breakpoint at **768px**. Below is mobile, above is desktop.

All `/dashboard/*` routes have a sidebar. On mobile the sidebar is hidden behind a hamburger toggle; on desktop it is always visible.

---

## Color Palette

### Primary - Teal (#00D4AA)

The anchor color. Used for primary actions, active states, links, and key accents.

| Token               | Usage                                        |
| ---------------------| ----------------------------------------------|
| `--clr-primary-500` | Primary buttons, links, active nav           |
| `--clr-primary-400` | Hover state                                  |
| `--clr-primary-600` | Active/pressed state                         |
| `--clr-primary-100` | Light background tint for cards and sections |

### Secondary - Purple (#7C3AED)

Used for secondary accents, badges, and highlights that need to stand out from primary.

| Token | Usage |
|-------|-------|
| `--clr-secondary-500` | Badges, secondary CTAs, highlights |
| `--clr-secondary-100` | Light background tint |

### Accent - Amber (#F59E0B)

Used for warnings, notifications, ratings, and status indicators.

| Token              | Usage                                 |
| --------------------| ---------------------------------------|
| `--clr-accent-500` | Warning banners, alerts, star ratings |
| `--clr-accent-100` | Light background for warning sections |

### Neutrals

| Token | Usage |
|-------|-------|
| `--clr-neutral-900` | Primary text, headings |
| `--clr-neutral-700` | Secondary text, metadata |
| `--clr-neutral-500` | Disabled text, placeholders |
| `--clr-neutral-300` | Borders, dividers, stroke |
| `--clr-neutral-100` | Card backgrounds, elevated surfaces |
| `--clr-neutral-50` | Page background |

### Semantic

| Token | Usage |
|-------|-------|
| `--clr-success` | Paid, present, completed |
| `--clr-danger` | Overdue, absent, failed |
| `--clr-warning` | Partial, late, pending |
| `--clr-info` | Announcements, system messages |

---

## Grading Color System

Academic grades use a distinct color scale from green to red. These are separate from the brand palette to make performance instantly scannable.

| Grade | Score Range | Color | Hex |
|-------|-------------|-------|-----|
| A | 75-100 | Deep Green | `#059669` |
| B | 60-74 | Green | `#65A30D` |
| C | 50-59 | Amber | `#D97706` |
| D | 40-49 | Orange | `#EA580C` |
| F | 0-39 | Red | `#DC2626` |

Attendance and fee status follow their own semantic colors:

| Status | Color | Hex |
|--------|-------|-----|
| Present / Paid | Green | `#16A34A` |
| Late / Partial | Amber | `#D97706` |
| Absent / Pending | Orange | `#EA580C` |
| Excused / Overdue | Red | `#DC2626` |

---

## Typography

| Token | Value |
|-------|-------|
| Font family (headings) | System sans-serif (Inter, SF Pro) |
| Font family (body) | System sans-serif |
| Font family (tabular) | Monospace for grades, fees, numbers |
| Base size | 16px |

Scale: 12, 14, 16, 18, 20, 24, 30, 36, 48, 60

---

## Spacing

Base unit: 4px

Scale: 0, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80

Radius: uniform **7px** across the app. All four tokens (`--radius-sm`,
`--radius-md`, `--radius-btn`, `--radius-lg`) equal 7px, so every card, button,
input, tile, and drawer shares the same corner. True circles (`50%`) and pills
(`999px`) keep their shape.

---

## Dashboard Layout Patterns

### Mobile-First Dashboards (Student, Parent, Teacher, Pending User)

```
+---------------------------+
| Top Bar (logo, av)        |  ~56px
+---------------------------+
|                           |
|   Content Area            |
|   (single column card     |
|    stack)                 |
|                           |
|                           |
+---------------------------+
| Bottom Nav (role tabs)    |  ~64px
+---------------------------+
```

On desktop, bottom nav becomes a hamburger-revealed sidebar, content goes to a 2-3 column grid.

### PC-First Dashboards (Principal, Bursar, Secretary)

```
+--------+------------------+
|        | Top Bar           |
| Side   +------------------+
| bar    |                  |
| (220px)|   Content Area    |
|        |   (multi-panel    |
|        |    data tables,   |
|        |    forms, charts) |
|        |                  |
+--------+------------------+
```

Sidebar collapses to icons-only at 1024px.

---

## Component Design Principles

1. **Data density scales with screen**: Tables on mobile become cards. Cards on desktop become tables.
2. **Actions are contextual**: Edit/delete appear inline on desktop, slide-out on mobile.
3. **Filters are visible on desktop, hidden on mobile**: Mobile uses a filter drawer triggered by a button.
4. **Empty states are actionable**: No students? "Add your first student" button. No grades? "Upload grades" prompt.
5. **Loading is skeleton-based**: Every data section has a skeleton placeholder matching its final shape.
6. **Forms are single-column on mobile, multi-column on desktop**: Never horizontal labels on mobile.

---

## Accessibility Minimums

- All interactive elements must be keyboard accessible
- Color is never the sole indicator of status (use icons + labels)
- Touch targets minimum 44x44px on mobile
- Focus indicators visible on all interactive elements
- Form labels always visible (no placeholder-as-label pattern)
- Error messages tied to inputs via aria-describedby

---

## Notes

- No external design library (MUI, Chakra, etc). Custom CSS Modules only.
- No Tailwind. Plain CSS with custom properties for the token system above.
- The public Staff page (`/staffs`, `components/public/StaffsList.jsx`) lists
  real people from the `school` collection (role in teacher/principal/bursar/
  secretary) instead of hardcoded cards. Each card's destination follows the
  viewer: signed-out users go to `/login?redirect=/staffs`; signed-in users go
  to `/dashboard/<role>/people/<uid>` using their primary role (unknown role
  falls back to `user`). Every role has a `/people/[id]` page: secretary,
  bursar and principal use their existing editable detail page, while student,
  teacher, parent and user render the shared read-only `PersonProfile`
  (`components/dashboard/PersonProfile.jsx`, a hero + personal/contact info +
  record-meta layout with a "Back to staff" link to `/staffs`).
- Icons are a custom inline-SVG set in `components/ui/Icon.jsx` (stroke-based,
  `currentColor`), not FontAwesome or react-icons. Add a glyph by adding a path to
  the `paths` map. No icon fonts.
- Theming is a two-mode scheme (light -> dark) applied as
  `data-theme` on `<html>` and toggled from the header. Dark overrides the
  neutral/scale tokens in `globals.css`; the primary dark surface is `#2d2d2d`
  (`--clr-neutral-100`). Brand colors (teal/purple/amber) and the grading scale
  stay saturated. The saved choice persists to `localStorage` and is applied
  pre-paint by an inline script in the root layout (no flash).
- Headers (public site header and dashboard top bar) are transparent at page top
  and become solid on scroll (`scrolled` class, ~8px threshold) with a smooth
  transition. In dark mode the scrolled header background is `#2d2d2d`.
- Dashboard top bar (`HeaderUser`) carries no page title. On mobile it shows the
  logo, theme toggle, user menu and hamburger; on desktop the logo is hidden
  (the sidebar already shows the brand) and the hamburger is hidden too.
- The mobile sidebar drawer slides in from the **right** (`translateX(100%)`),
  sized `min(70vw, 340px)`, with a scrim behind it.
- Sidebar / drawer nav highlights **exact-path matches only** — being on
  `/dashboard/student/grades` does not highlight the `/dashboard/student`
  "Overview" item. Use `pathname === href`, never prefix matching.
- Dashboard responsive mode is set per role in `app/dashboard/<role>/layout.jsx`:
  `mobile-first` (student, parent, teacher, user) uses bottom nav on mobile and a
  hamburger-revealed sidebar on desktop; `pc-first` (secretary, bursar, principal)
  shows the sidebar always, collapsing to icons at 1024px and to a hamburger drawer
  below 768px. Each role owns a same-named segment (`/dashboard/student`,
  `/dashboard/principal`, ...) and bare `/dashboard` is a client redirect that
  routes the signed-in user to their own role dashboard.
- Avatars (header, drawer, profile badge) always render the initial underneath a
  circular photo when available. The photo source is `profilePictureUrl ||
  photoURL`; a failed image hides itself and the initial shows instead.
- The public site menu (About, Services, Staffs) reappears in
  the dashboard sidebar/drawer below a divider; `Home` is not included once
  signed in. Logged-in users get a "Dashboard" link right after Home in the
  public header.
- Security: CSP + security headers are applied globally in `next.config.mjs`
  (img-src allows `https:` for profile pictures); Firestore access is enforced
  by security rules managed in the Firebase console; the Firebase Admin service account is built
  from env vars, never hardcoded.
- Profile relations: a student `school/{uid}` doc carries `teacher {id, name}`
  and `parent {id, name, pending}` (parent is the guardian/emergency contact);
  a teacher or parent doc carries `students { <studentUid>: {id, name} }`.
  Self-service links made from the parent/student profile pages write both
  sides with `pending: true` until a teacher approves; teacher-initiated adds
  go live immediately (skipped if the student already has another teacher).
  Secretary-initiated links (from the person detail page) go live immediately
  with no pending state. Removal clears both sides. Approval happens on the
  teacher's Students tab. Written with `scripts/linkFamily.js` (merge-only,
  idempotent) and documented in `scripts/Roles.md`.
- `className` is the canonical field name for class data on Firestore. Writes
  always use `className`. Reads use `person.className || person.class` as
  fallback for old data that may use `class`. Never write `class` to Firestore.
- Admission number format: `IL/YYYY/XXXXXXXX` where YYYY is the current year
  and XXXXXXXX is the first 8 alphanumeric characters of the user's uid,
  uppercased. Generated by `genAdmission(uid)` (local helper in each profile
  page). Admission number is always the first item in student detail grids.
  When not set, a small inline "Generate" ghost button appears instead of
  "Not set". Only secretary, teacher, and principal can generate admission
  numbers; parents and the student see read-only values.
- Dual-role support: a user may have `extraRoles` containing roles like
  `"teacher"` or `"parent"` in addition to their primary `role`. The
  secretary person detail page detects both `role` and `extraRoles` to
  determine which sections to show. If a person is both a teacher and a
  parent, a "Students / Children" sub-tab switcher appears within the Details
  tab. If only one applies, that section shows directly.
- Teacher class/session cascade: when a secretary edits a teacher's class or
  session from the person detail page, the change propagates to all linked
  students. The code iterates over `person.students` and writes the updated
  `className`/`session` to each student doc.
- All profile pages save `updatedOn` as a map: `{ date, uid, name }` where
  `date` is the Firestore timestamp, `uid` is the editor's uid, and `name` is
  their display name. Display renders "Updated on {date}" with a line break
  then "by {name}". The `fmtDate` helper unwraps the `.date` field for
  backward compatibility with old plain-date values.
- Parent "My Children" page (`/dashboard/parent/children`) lists the parent's
  `students` map as cards; approved children link to a detail shell at
  `/dashboard/parent/children/[id]` (guarded: uid must be in `students` and not
  pending), pending ones render inert with an amber pill and no tap target.
  The detail page has Profile / Grades / Attendance tabs reusing the shared
  panels; the Profile tab's Class teacher row links to the read-only teacher
  profile at `/dashboard/parent/teachers/[teacherUid]`.
- Parent overview (`/dashboard/parent`) shows a children strip, aggregated
  stats (attendance averaged across children with registers, pending-fee total
  from live `school-fees` counts) and a two-item announcements preview.
- Attendance is a per-user date map (`attendance: { "YYYY-MM-DD": true }`) on
  `school/{uid}`; missing weekdays in the recorded range are absences. Rendered
  by the shared `AttendancePanel` (summary + Mon-Fri month calendar), which
  accepts any student's map so parent and teacher views reuse it. Teachers mark
  the register at `/dashboard/teacher/attendance`: one date per session,
  their approved students with present/absent toggles; save writes `true`
  entries and deletes keys for absentees (dot-path updates only).
- Teacher dashboard menu: Overview, Students, Grades, Attendance, Profile.
  Students (`/dashboard/teacher/students`) is the moderation surface - pending
  family links get an Approve button that flips both sides; approved students
  open `/dashboard/teacher/students/[id]` with the same three-tab shell as the
  parent child view. Its Profile tab is editable by the teacher (gender, dob,
  class, session) via an Edit details form; name/email/admission number stay
  read-only and a save refetches the doc so the header card refreshes. Edit
  mode also manages the guardian link with add (search-and-pick over
  role=parent docs), remove, and approve actions; Save diffs against the
  original link and mirrors every change onto both sides (`parent` map on the
  student, `students.{id}` entry on the parent, removals via deleteField()).
  Teacher-added guardians go live approved. The
  Parent/guardian row links to `/dashboard/teacher/parents/[id]` (read-only
  guardian profile, same shell as the parent-side teacher viewer; guarded so
  only parents of approved students open) and pending guardian links carry a
  Pending tag plus Approve button that clears `parent.pending` on the student
  and `students.{id}.pending` on the parent doc. The
  teacher profile manages its `students` map with
  search-and-pick like the parent profile, but teacher-added links are
  approved on write.- Teacher overview (`/dashboard/teacher`): greeting, a "Top students" strip
  showing only the two best performers for the current term (ranked by mean
  subject total from `grades[termKey]`, tile line shows their average and
  letter in grade color), stat cards (total students, average grade as the
  most common class letter, weakest subject as the lowest subject mean across
  graded students) and a two-item announcements preview. No scores yet renders
  an empty state instead of the strip.
- School fees live in the `school-fees` collection:
  `{ parentId, parentName, studentId, studentName, description, amount,
  accountNumber, status: "pending"|"approved", moderatedBy:{id,name}, createdAt }`.
  Parents create docs from `/dashboard/parent/fees`; transfer details come
  from `PAYMENT_ACCOUNT` in `lib/site.js` and `generateFeeAccountNumber()`
  mints the reference. Pending children are excluded from the payment drawer.
  In the books these are credits into the school account, so a person's
  profile shows them under the Fees tab (docs where `parentId` matches).
  Fee status is either "pending" or "approved". The reject action has been
  replaced with delete (removes the doc from Firestore entirely).
  Names in fee/payment list rows are plain text; names in detail drawers
  are links to the person's profile page.
- Payments to staff live in the `school-payment` collection:
  `{ amount, accountName, bank, accountNumber, fullName (payer name),
  receiverName, description, date, payerId, receiverId }`. In the books
  these are debits out of the school account (salaries and expenses). A
  person's profile shows them under the Payment tab (docs where `receiverId`
  matches), backed by the shared `PaymentsTab` component (`components/dashboard/`),
  newest first; rows show only description and "From {fullName}", and tapping
  opens a right-side drawer with the amount plus date/sender/account details
  (reuses the grades drawer shell; list styled like the FAQ accordion box).
- A person's profile (`/dashboard/bursar/people/[id]`) has Details, Fees and
  Payment tabs. Details shows gender, email, phone, class, session and joined
  date (not role or UID), plus linked Children (from `students`) and Parent /
  Guardian (from `parent`), each linking to that person's profile. Fees is the
  `FeesTab` (credits in, `school-fees` where `parentId` matches), Payment is
  the `PaymentsTab` (debits out, `school-payment` where `receiverId` matches).
  Both tabs query the person on either side of the record and label the
  opposite party: the Fees tab shows "For {student}" when they paid (parentId
  match) and "By {parent}" when paid for them (studentId match); the Payment
  tab shows "From {payer}" when paid to them (receiverId match) and "To
  {receiver}" when paid by them (payerId match). The opposite party always
  renders as a link to `/dashboard/bursar/people/[id]`.
- The bursar dashboard menu is Overview, Fees (parent school-fee records from
  `school-fees`), Payment (salaries/expenses the school pays out from
  `school-payment`), People (students, teachers and staff via tabs),
  Announcements and Profile. Its profile page mirrors the teacher shell minus
  class/session.
- Bursar Overview (`/dashboard/bursar`): greeting + live date, 3 stat card
  skeleton placeholders during load, then 3 live stat cards: Net worth
  (collected fees minus total payments, green if positive, red if negative,
  hint explains formula), Paid fees (approved-only sum in green, with pending
  total as hint underneath in small text), Total payments (all payments with
  record count). Below the cards: "Needs review" section showing up to 3
  pending fees as clickable FAQ-style rows with amber amount pills and chevrons,
  linking to the fees page; empty state when no pending fees. "Announcements"
  section using the shared `NotificationsPanel` with `role="bursar"` and
  `limit={2}`, linked to `/dashboard/bursar/announcements` via "See all" with
  chevron.

- Bursar Fees (`/dashboard/bursar/fees`): 3 stat card boxes in a CSS grid
  (`.statGrid`): "Confirmed funds" (approved-only sum, green, with "Grand
  total: ₦X" hint, takes full first row on mobile via `.fullRow` class, 3 cols
  on desktop), "Pending fees" (pending sum, amber, with transaction count
  hint), "Moderated by me" (sum of fees moderated by current user, teal, with
  transaction count hint). Boxes act as tabs filtering the list below; default
  tab is "confirmed". Person filter via `SearchSelect` component in a
  `.personFilter` wrapper (70% width mobile, 50% desktop, right-aligned).
  Clickable rows open a right-side detail drawer (scrim + Escape + close +
  approve/delete buttons for pending fees). The `currentDetail` re-lookup
  keeps the drawer in sync after moderation writes. NO inline approve/delete
  buttons on rows. Fee statuses: "pending" (amber pill), "approved" (green
  pill). Empty states adapt to the active tab.

- Bursar Payment (`/dashboard/bursar/payment`): 2 stat card boxes in a CSS
  grid (`.statGrid2`): "Total payments" (all school payments), "Sent by me"
  (payments where payerId matches current user, active by default). Boxes
  filter the list. Person filter via `SearchSelect` with `.personFilter`
  wrapper. "Add payment" button opens an add-payment drawer with receiver
  picker (SearchSelect over all users), amount, description, date, bank,
  account name, account number fields. Clickable rows open a right-side detail
  drawer showing amount (green pill), Status, Date, Paid by (DetailLink), For
  (DetailLink), Reference/accountNumber. Rows show description + "to
  {receiver}" with amount pill and chevron. Escape closes either drawer.
  Empty states adapt to active tab.

- Bursar Announcements (`/dashboard/bursar/announcements`): full list view
  using `NotificationsPanel` with `role="bursar"`. Individual announcement
  detail at `/dashboard/bursar/announcements/[id]` fetches from `school-news`
  by doc ID, shows title, date, full description, with "Back to announcements"
  link.

- Bursar Profile (`/dashboard/bursar/profile`): hero card (avatar, name,
  email, role chips, dashboard switches, updated-on), tabs: "Account Info"
  (personal info + contact info with edit form, linked Students/Children for
  extra roles with links to `/dashboard/bursar/people/[id]`), "Fees"
  (AllFeesPanel showing fees where `moderatedBy.id === uid`, clickable rows
  with detail drawer) and "Payments" (PaymentsTab showing all payments, no
  `receivedOnly`, `basePath="/dashboard/bursar/people"`).

- Secretary dashboard menu is Overview, People, Announcements, Analysis,
  Profile. Its profile page mirrors the bursar shell (hero card with avatar,
  role chips, dashboard switches, Details + Fees + Payment tabs).

- Secretary Overview (`/dashboard/secretary`): greeting + live date, 3 stat
  card skeleton placeholders during load, then 3 live stat cards (Students,
  Staff, Parents with live counts from Firestore). Announcements preview via
  `NotificationsPanel` with `role="secretary"`, `limit={2}`,
  `basePath="/dashboard/secretary/announcements"`, "See all" link with chevron.

- Secretary People (`/dashboard/secretary/people`): tabs for Students,
  Teachers, Parents, Staff with badge counts, search by name/email, class
  select dropdown (grouped: Early Years, Primary, Junior Secondary, Senior
  Secondary) for students tab only, sort select (name, joined date, updated
  date), alphabetical list, links to person detail at
  `/dashboard/secretary/people/[id]`. Filters reset on tab change.

- Secretary Person Detail (`/dashboard/secretary/people/[id]`): role-aware
  profile view with full edit capability. Hero card with avatar, name, email,
  role chips (primary + extra roles). All non-student roles get Details, Fees,
  and Payments tabs. Students get Profile, Grades, Attendance, Fees, Payments
  tabs.
  - **Students (Profile tab):** Detail grid with admission number first
    (Generate button when unset), full name, email, DOB, gender, phone, class,
    session, date joined, updated. Guardian and Class Teacher rows link to
    their profiles. Edit form allows gender, DOB, phone, class, session.  - **Teachers (Details tab):** Detail grid with full name, email, phone,
    gender, DOB, class, session, date joined, updated. Edit form allows all
    fields; class/session fields appear for teachers (including via
    `extraRoles`). When class or session changes, the update cascades to all
    linked students. Linked students section with search-and-pick assign and
    remove. Assign writes both sides immediately (no pending).
  - **Parents (Details tab):** Same as teacher minus class/session fields.
    Linked children section with search-and-pick assign and remove. Assign
    writes both sides with `pending: false`.
  - **Dual-role (teacher + parent):** A "Students / Children" sub-tab switcher
    appears within the Details tab to switch between the two linked sections.
    The sections are not duplicates of the same `students` map: on load the app
    reads each linked student's doc and splits by relationship, showing under
    "Students" only those whose `teacher.id` is this person and under
    "Children" only those whose `parent.id` is this person (a student who is
    both appears under both).
  - **Other staff (secretary, bursar, principal):** Same detail grid and edit
    form (no class/session fields). If they have extra roles like teacher or
    parent, the corresponding linked sections appear.
  - **Fees tab:** `FeesTab` component showing school-fee records for this
    person. **Payments tab:** `PaymentsTab` component showing salary/expense
    records. Both work for all roles.

- Secretary Announcements (`/dashboard/secretary/announcements`): full list
  via `NotificationsPanel` with `role="secretary"`. Individual announcement
  detail at `/dashboard/secretary/announcements/[id]` fetches from `school-news`,
  shows title, date, description, with "Back to announcements" link.

- Secretary Analysis (`/dashboard/secretary/analysis`): 3 stat cards in grid
  (`.statGrid`): Collected fees (green, grand total hint), Total payments
  (neutral, record count), Net worth (green/red). Fees/Payments tabs with
  person filter and month filter via `SearchSelect`. Clickable rows open a
  right-side detail drawer with linked names (links in drawer only, plain text
  in rows). All links point to `/dashboard/secretary/people/`.

- Secretary Profile (`/dashboard/secretary/profile`): hero card (avatar,
  name, email, role chips, dashboard switches, updated-on), tabs: "Details"
  (personal info + contact info with edit form, linked Students/Children for
  extra roles with links to `/dashboard/secretary/people/[id]`), "Fees"
  (AllFeesPanel showing fees where `parentId === uid`, clickable rows with
  detail drawer) and "Payment" (PaymentsTab showing all payments, no
  `receivedOnly`, `basePath="/dashboard/secretary/people"`).

- Principal dashboard menu is Overview, People, Announcements, Finance, Profile.

- Principal Overview (`/dashboard/principal`): greeting + live date, a
  "School at a glance" headcount strip (Students, Staff, Parents via live
  counts) with skeletons during load, a "Financial summary" section (Collected
  fees green with pending hint, Total payments with record count, Net worth
  green/red) linked to `/dashboard/principal/finance`, a "Needs review"
  section showing up to 3 pending fees as amber FAQ-style rows with an empty
  state ("All clear"), and "Announcements" via `NotificationsPanel` with `all`
  (role badges shown) and `basePath="/dashboard/principal/announcements"`.

- Principal People (`/dashboard/principal/people`): shared people directory
  with tabs for Students, Teachers, Parents, Staff. Badge counts in tab
  buttons. Single Firestore query for all roles, client-side grouping. Text
  search (name/email), class select dropdown (grouped classes) for students
  tab only, sort (name, joined date, updated date). Filters reset on tab
  change. Links to `/dashboard/principal/people/[id]`.

- Principal Person Detail (`/dashboard/principal/people/[id]`): full edit
  capability (acts as super admin). Students get 5-tab layout (Profile, Grades,
  Attendance, Fees, Payments). Profile tab shows detail grid with admission
  number (Generate when unset), full name, email, DOB, gender, phone, class,
  session, date joined. Guardian (Approve pending / Remove / Change via
  search-and-pick over parents) and Class Teacher (Remove / Change via
  search-and-pick over teachers)   links go live immediately, no pending. Edit
  form for gender, DOB, phone, class, session. Non-students get 3-tab layout
  (Details, Fees, Payments). Details tab shows full info plus edit form,
  class/session cascade for teachers, linked Students (for teachers) / Children
  (for parents) with search-and-pick add/remove, dual-role sub-tab switcher.
  All links point to `/dashboard/principal/people/`. Fees = credits into the
  school (`FeesTab`, `school-fees` where `parentId` matches); Payments =
  debits out of the school (`PaymentsTab`, `school-payment` where `receiverId`
  matches). Available for ALL people (students, staff, parents). A "Manage
  roles" section lives inside the edit form (all people, students and
  non-students): the principal sets the primary role (select) and toggles any
  number of extra roles (checkboxes, one per role; the primary role is excluded
  and disabled) supporting multiple extra roles per person. Saving writes the
  details plus `role`, `extraRoles`, and `updatedOn`; role/nav changes apply
  on next sign-in.

- Principal Announcements (`/dashboard/principal/announcements`): list view
  with "New announcement" button opening the `CreateAnnouncement` drawer.
  Uses `NotificationsPanel` with `all` prop and
  `basePath="/dashboard/principal/announcements"`. `refreshKey` state
  increments on creation to refetch. Individual announcement detail at
  `/dashboard/principal/announcements/[id]` shows icon box, title, createdBy
  name, formatted date, and description split into paragraphs.

- Principal Profile (`/dashboard/principal/profile`): hero card with avatar,
  name, email, role chips, dashboard switches. Tabs: "Details" (personal info
  + contact info with edit form) and "Payment" (PaymentsTab with `receivedOnly`
  and `basePath="/dashboard/principal/staff"`).

- Principal Finance (`/dashboard/principal/finance`): 3 stat cards in grid
  (`.statGrid`): Collected fees (green, grand total hint), Total payments
  (neutral, record count), Net worth (green/red). Fees/Payments tabs with
  person filter, month filter, and status filter (Pending/Approved) via
  `SearchSelect`. Clickable rows open a right-side detail drawer. Pending
  fees show Approve + Delete buttons. Approved fees and all payments show an
  Edit button that opens inline edit mode (amount, description, bank/account
  fields for payments). Principal acts as the moderator - approve writes
  `moderatedBy` with principal's uid. All drawer links point to
  `/dashboard/principal/people/`. Skeleton loading, empty states, Escape
  closes drawer.

- Shared components used by bursar:
  - `SearchSelect` (`components/dashboard/SearchSelect.jsx`): reusable
    searchable combobox with keyboard nav, internal open/search/activeIdx state,
    `e.nativeEvent.stopPropagation()` on Escape to avoid closing parent
    drawers. Props: `options, valueId, fallback, ariaLabel, searchPlaceholder,
    idPrefix, onPick`. Searches by email (students) or email (parents).
  - `StatCard` (`components/dashboard/StatCard.jsx`): link-based stat card
    with `value, color, label, hint, href` props. Renders `statStyles.hint`
    when hint is present.
  - `NotificationsPanel`: shared by all roles, `role` + optional `limit` +
    optional `basePath` + optional `all` props. When `basePath` is provided,
    announcement links use it instead of the current pathname. When `all` is
    true, shows announcements regardless of role filter.
  - `PaymentsTab` (`components/dashboard/PaymentsTab.jsx`): shared payments
    list with detail drawer. Props: `uid`, optional `basePath` (link prefix,
    defaults to `/dashboard/bursar/people`). Shows "..." instead of raw uid
    while async name resolution is in progress. Links in rows are plain text;
    links in drawer use `basePath` to route to the correct people directory.
  - `FeesTab` (`components/dashboard/FeesTab.jsx`): shared fees list.
    Props: `uid`. Queries `school-fees` where `parentId` matches. Shows
    status pills, "Total confirmed" sum at top. Names in rows are plain text;
    names in drawer are links to `/dashboard/bursar/people/[id]`. Approve
    and Delete buttons for pending fees in drawer.
  - `GradesPanel` (`components/dashboard/GradesPanel.jsx`): shared grades
    display. Props: `grades` (full grades map from a student doc). Renders
    term rows that open a right-side drawer with per-subject detail.
  - `AttendancePanel` (`components/dashboard/AttendancePanel.jsx`): shared
    attendance display. Props: `attendance` (date map from a student doc).
    Summary stats + Mon-Fri month calendar grid.
  - `CreateAnnouncement` (`components/dashboard/CreateAnnouncement.jsx`):
    shared drawer for creating announcements. Props: `open`, `onClose`,
    `onCreated`. Writes to `school-news` with `{ title, snippet, description,
    role, date, createdBy: { id, name } }`. Target audience select (all,
    student, teacher, parent, bursar, secretary, principal).
  - Detail drawer reuses `Grades.module.css` shell (`.scrim`, `.drawer`,
    `.drawerHead`, `.drawerTitle`, `.closeBtn`). Row list uses
    `Faq.module.css` (`.list`, `.item`, `.trigger`). Detail grid uses
    `user-dashboard.module.css` (`.detailGrid`, `.detailItem`, `.drawerBody`).
    Add-form styles from `Fees.module.css` (`.addForm`, `.field`, `.label`,
    `.input`, `.selectWrap`, `.selectBtn`, `.selectMenu`, `.option`).

- Fees + Payment CSS grid system (in `Fees.module.css`):
  - `.statGrid`: 2 cols mobile, 3 cols desktop (768px breakpoint). Used by
    fees page (3 boxes). `.fullRow` child spans both columns on mobile, resets
    to `auto` on desktop.
  - `.statGrid2`: always 2 cols. Used by payment page (2 boxes).
  - `.personFilter`: 70% width mobile, 50% desktop, `margin-left: auto`,
    children forced to 100% width.
  - Value text in stat cards uses `clamp(20px, 5vw, 30px)` with ellipsis
    overflow to prevent blowout on small screens.
- Person profile CSS (in `user-dashboard.module.css`):
  - `.childRow`: flex row for linked person items (badge + meta + remove).
    Used by secretary/teacher profiles for linked students/children.
  - `.childBadge`: 44px circular avatar with initial letter.
  - `.childMeta`: column flex for name + subtitle.
  - `.childName`: name text (16px, bold). Doubles as clickable button in
    `.searchDropdown`.
  - `.childLine`: subtitle text (14px, neutral-500).
  - `.removeBtn`: 32px circular border button, danger-red on hover.
  - `.searchDropdown`: bordered scrollable box (`max-height: 260px`) for
    inline search results in person profile pages.
  - `.formActions`: flex row with gap for form button groups (Cancel/Save).
  - `.editBtn`: primary teal button with pencil icon for "Edit details"
    (compact: 36px height, 13px font).
  - `.saveBtn`: primary teal button for "Save changes".
  - `.ghostBtn`: transparent border button, danger-red on hover. Used for
    "Cancel" and "Generate" admission number button.
  - `.roleChips` / `.roleChip`: role label chips in hero cards (primary-100
    background, primary-600 text, with `.alt` variant in purple).
- People directory pattern: shared across bursar, secretary, and principal.
  Uses `Children.module.css` tabs (`.tabs`, `.tab`, `.tabActive`) with counts
  in parentheses. Search input by name/email, class select dropdown (grouped:
  Early Years, Primary, Junior Secondary, Senior Secondary) for students tab
  only, sort select (name, joined date, updated date). Filters reset on tab
  change. Row items use FAQ-list style with name, subtitle, chevron.
- Grades are a full-history map on `school/{uid}`
  (`grades[termKey] = { class, session, position, subjects: { name:
  { ca, exams, bonus, total, remark } } }`). Rendered by the shared
  `GradesPanel`: term rows that open a right-side drawer with per-subject
  detail. Grade letters/colors come from `utils/format.js`. Teachers enter
  grades at `/dashboard/teacher/grades`: the key comes from the teacher's own
  class + session (`"SS 1"` + `"First Term"` → `ss1-first-term`); approved
   students are compact rows (name, gender, class, saved average) that open a
   right-side drawer with per-subject CA/exam editing, and saving computes
   totals (`ca + exams + bonus`, preserving bonus/remark) via a dot-path write;
   subjects removed in the form are deleteField()-ed so the merge cannot
   resurrect them. A searchable subject select ("All subjects" plus every
   subject filed this term, type-to-filter with arrow/enter keys) swaps the
   roster rows to that subject's scores: CA/Exam in the meta line and
   a total+letter pill in grade color instead of the term average. Terms
   entered this way may have no position; the panel
   shows "Position pending" until one is stored.
- Writing style: never use em dashes (—) in user-facing copy. Use periods,
  commas, or plain hyphens instead.
- The app's own domain is a single source of truth: `SITE_URL` in `lib/site.js`,
  imported by the root layout, sitemap, robots, and the 404 page. Change it once
  to update the whole app. The static `public/robots.txt` is not used; robots is
  served from `app/robots.js`.

---

## Developer Rules

The user is in charge. Follow these without exception:

1. Never run `npm run dev`, `npm run build`, or any git command unless the user
   explicitly asks.
2. Wait for explicit approval before starting/stoping any long-running process,
   killing processes, or freeing ports.
3. Only touch the user's environment (processes, ports, terminals, installs)
   when asked.
