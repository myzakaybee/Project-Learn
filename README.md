# School Fee, Notes & Updates — Thornfield Academy Portal

A small school portal with two views: a **staff/admin dashboard** for managing
fee records, class notes and announcements, and a **parent/student portal**
for viewing them. Built as a static site (HTML/CSS/vanilla JS, no build step,
no framework) so it's easy to host anywhere or adapt.

This is a working prototype intended as a portfolio / demo piece. Read
"Security & production notes" below before using it for anything beyond that.

## Running it

No installation and no build step. Either:

- Open `index.html` directly in a browser, or
- Serve the folder with any static file server, e.g. `npx serve .` or
  Python's `python3 -m http.server`, then visit `index.html`.

All data is stored in the browser's `localStorage`, seeded automatically the
first time the app loads.

## Demo credentials

| Role | Username | Password | Notes |
|---|---|---|---|
| School staff (admin) | `admin` | `Admin@123` | Full access to fees, notes, announcements, students |
| Parent | `parent1` | `Parent@123` | Linked to two children (Grade 8 and Grade 5) |
| Parent | `parent2` | `Parent@123` | Linked to one child (Grade 10) |

Use the "Reset demo data" button on the admin Students page to wipe your
browser's local copy back to this starting state at any time.

## Project structure

```
index.html            Login / role selection
admin.html             Staff dashboard shell
portal.html             Parent/student portal shell
assets/css/style.css     All styling and design tokens
assets/js/utils.js        Shared helpers (escaping, formatting, validation)
assets/js/db.js            Data layer (localStorage-backed, seed data)
assets/js/auth.js           Demo authentication + session handling
assets/js/admin.js           Admin dashboard logic
assets/js/portal.js           Parent/student portal logic
```

`db.js` is the only file that touches storage directly. Everything else
calls its methods (`DB.getFees()`, `DB.addNote()`, etc.), so swapping
`localStorage` for real `fetch()` calls to a backend API later mostly means
rewriting `db.js`, not the rest of the app.

## Security & production notes

This was built as a **frontend-only prototype**, which puts a hard ceiling
on how secure it can ever be — there is no server to enforce anything. The
points below are deliberate, documented limitations rather than oversights,
and a list of what to do before this handles anyone's real fee or student
information.

**Authentication is a demo, not real security.** Usernames and passwords
live in plain text inside `db.js` and are checked entirely in the browser.
Anyone who opens the browser's dev tools can read every password, forge a
session, or skip the login page entirely. Before any real use, this needs to
be replaced with server-side authentication: passwords hashed with a slow
algorithm (bcrypt/argon2), sessions or tokens issued and verified by a
server, and every role check (admin vs. parent) re-enforced server-side —
never trusted from the client alone.

**Data lives unencrypted in the browser.** All students, fees and messages
sit in `localStorage` on whichever device opened the app, readable by
anyone with access to that browser or device, and not synced between
devices. A real deployment needs an actual backend and database, with data
scoped per user on the server (e.g. a parent's API token only ever returns
that parent's own children — not enforced by trusting a client-side filter).

**Login throttling is cosmetic.** There's a simple lockout after five failed
attempts to slow down casual brute-forcing, but it's implemented in
`sessionStorage` and bypassed the moment someone clears storage or opens
dev tools. Real rate-limiting has to happen on the server.

**Cross-site scripting (XSS).** Every piece of user-entered text (student
names, fee descriptions, note content, announcement bodies, etc.) is passed
through an `escapeHTML()` helper before being inserted into the page, both
in regular content and in form field values, rather than written with raw
`innerHTML`. No `eval`, dynamic `Function` construction, or
`document.write` is used anywhere. This is good practice to keep, but it's
worth re-auditing if the app grows — any new spot that builds HTML from
stored data needs the same treatment.

**Other things to add before a real launch:** HTTPS everywhere; CSRF
protection once there's a server and cookies/sessions; parameterised
queries (or an ORM) once there's a real database, to rule out SQL
injection; a Content-Security-Policy header; removing the on-screen demo
credentials box; not having admins set a parent's password directly
(send an invite/reset link instead so the school never knows it); and an
audit log for who changed a fee record or deleted a student, since this
touches financial data.

None of this is meant to discourage using the prototype to demo the idea —
it's meant to make the gap between "demo" and "production" explicit, since
that gap is easy to forget once something looks finished.
