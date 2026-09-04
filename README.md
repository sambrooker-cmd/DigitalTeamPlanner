# Digital Team Planner

A shared planning tool for the Acquisition, Retention, and Website teams.
Navigation is organized by team:

- **My Work** — everything assigned to the signed-in viewer (promotion
  tasks, emails, paid media tests, CRO/UX tests), soonest due date
  first, overdue items in red.
- **Overview** — a unified, read-only timeline rolling up every
  promotion, email, paid media test, and CRO/UX test into one
  chronological view, each row tagged with a team-colored badge. Click a
  row to jump to it in its own section. Searchable by name, filterable
  by kind (Promotion/Email/Paid Media/CRO-UX) and by assignee — for a
  promotion, the assignee filter matches if any of its tasks are
  assigned to that person, since a promotion itself has no single
  assignee.
- **Promotions** — cross-team, since every promotion involves all three
  teams: a kanban board (briefing through launch to results reporting)
  and a timeline.
- **Acquisition** — Paid Media Tests: a status board and a timeline.
- **Retention** — Emails: a status board and a month calendar.
- **Website** — CRO/UX Tests: a status board and a timeline.
- **Activity** — a live change log across everything above.

Each timeline (Promotions, Paid Media, CRO/UX) uses the same visual
language: a light bar for the prep/build period, a solid bar for when
it's actually live, and separate launch/end markers — so "still being
prepped" vs. "actually running" is always visually distinct.

Every promotion, task, email, and test can be **duplicated** from its edit
modal — it pre-fills a new, unsaved copy (dates and result fields cleared,
nothing else) so a repeat promotion or test doesn't mean re-typing the whole
form; nothing is written until you adjust it and hit Save. Each of those
also carries its own **comment thread**, for back-and-forth that would
otherwise overwrite the single free-text notes field. Promotions, Emails,
Paid Media, and CRO/UX each have an **Export CSV** button (on the
promotion's board, and by the "+ New…" button on the other three) for
pulling the current data into a deck or report.

## Live board

`index.html` is a single self-contained page (React, loaded from cdnjs, no
build step) backed by Firebase Firestore and Firebase Authentication, which
is what gives it real shared, multi-user editing restricted to your team:
everyone signs in with Google, and only people you've explicitly approved
can see or edit the board — no Claude account or shared organization
required.

### Hosting it

The page is plain static HTML, so any static host works. The simplest is
GitHub Pages on this repo:

1. **Settings → Pages** → Source: **Deploy from a branch** → pick this
   branch (or `main`, once merged) → folder `/ (root)` → **Save**.
2. GitHub publishes it at `https://<org>.github.io/Promotionsplanner/`.
   Share that link with your team.

### Firebase setup

The page talks to a Firebase project (`promotions-planner`).

**1. Enable Google sign-in.** In the
[Firebase console](https://console.firebase.google.com) → **Authentication**
→ **Sign-in method** → enable **Google**.

**1b. Authorize your GitHub Pages domain.** Still under **Authentication →
Settings → Authorized domains**, add `<org>.github.io` (Firebase only
allows sign-in from domains listed here — `localhost` and the project's own
`firebaseapp.com` are added by default, but your GitHub Pages domain isn't,
and sign-in will fail with an "unauthorized domain" error until you add it).

**2. Approve who can access the board.** Under **Firestore Database →
Data**, create a collection named `allowlist`. For each teammate you want to
let in, add a document whose **document ID is their exact Google account
email** (e.g. `sam@gmail.com`) — the document's fields don't matter, it can
be empty or hold a note like `{ note: "added 3 Sep" }`. To revoke someone,
delete their document. This is the only way to manage access; it isn't
editable from the app itself, on purpose.

**3. Set the Firestore rules.** Under **Firestore Database → Rules**, use:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAllowed() {
      return request.auth != null &&
        request.auth.token.email != null &&
        exists(/databases/$(database)/documents/allowlist/$(request.auth.token.email));
    }

    function isCommentAuthor() {
      return isAllowed() && request.auth.token.email == resource.data.byEmail;
    }

    match /promotions/{promoId} {
      allow read, write: if isAllowed();
      match /tasks/{taskId} {
        allow read, write: if isAllowed();
        match /comments/{commentId} {
          allow read, create: if isAllowed();
          allow update: if false;
          allow delete: if isCommentAuthor();
        }
      }
      match /comments/{commentId} {
        allow read, create: if isAllowed();
        allow update: if false;
        allow delete: if isCommentAuthor();
      }
    }

    match /emails/{emailId} {
      allow read, write: if isAllowed();
      match /comments/{commentId} {
        allow read, create: if isAllowed();
        allow update: if false;
        allow delete: if isCommentAuthor();
      }
    }

    match /paidTests/{testId} {
      allow read, write: if isAllowed();
      match /comments/{commentId} {
        allow read, create: if isAllowed();
        allow update: if false;
        allow delete: if isCommentAuthor();
      }
    }

    match /croTests/{testId} {
      allow read, write: if isAllowed();
      match /comments/{commentId} {
        allow read, create: if isAllowed();
        allow update: if false;
        allow delete: if isCommentAuthor();
      }
    }

    match /activity/{entryId} {
      allow read, write: if isAllowed();
    }

    match /meta/{docId} {
      allow read, write: if isAllowed();
    }

    match /allowlist/{email} {
      allow read: if request.auth != null && request.auth.token.email == email;
      allow write: if false;
    }

    match /users/{email} {
      allow read: if isAllowed();
      allow write: if request.auth != null && request.auth.token.email == email && isAllowed();
    }
  }
}
```

Anyone not signed in, or signed in but not on the allowlist, is bounced to a
sign-in or "ask an admin" screen before ever reaching the board — enforced
by these rules server-side, not just hidden in the UI.

### Data model

- `promotions/{promoId}` — one document per promotion (`name`, `launchDate`,
  `endDate` (optional), `description`, `archived` (boolean), plus
  `createdBy`/`createdAt`/`updatedBy`/`updatedAt`). When `endDate` isn't
  set, the timeline estimates the promotion runs 14 days from launch.
  Archived promotions are hidden from the timeline and, by default, from
  the board's promotion tabs (a "Show archived" toggle reveals them there
  so they can be unarchived or deleted).
- `promotions/{promoId}/tasks/{taskId}` — one document per task/result card
  (`team`, `type`, `title`, `assignee`, `due`, `status`, `notes`, plus the
  same attribution fields), so concurrent edits from different teams never
  overwrite each other.
- `emails/{emailId}` — one document per BAU/lifecycle email (`subject`,
  `sendDate`, `status` — Briefing/Testing/Approved/Scheduled/Sent/Reported —
  `assignee`, `relatedPromoId` (optional, links it to a promotion),
  `hubspotLink` (optional), `notes`, plus the same attribution fields).
  Shown under the **Retention** section as a status board (drag between
  columns) or a month calendar (click a day to add one, click a chip to
  edit it).
  This is where the email team's planning calendar and Trello-style
  briefing status live — HubSpot itself stays the tool that builds and
  sends the email; `hubspotLink` just points at it.
- `paidTests/{testId}` and `croTests/{testId}` — one document per
  experiment (Paid Media and CRO/UX respectively), each shown under its
  own team section as a status board or a timeline. Both are
  "hypothesis → variant → result" shaped but with different fields and
  stages, driven by the `TEST_TRACKERS` config in `index.html` rather
  than duplicated code — add a third tracker (e.g. QA/stability) by
  adding a new entry there, not by copying a whole module. Paid Media:
  `name`, `channel`, `hypothesis`, `variants`, `startDate`, `endDate`,
  `budget`, `assignee`, `outcome`, `result`, status Planned → Live →
  Analyzing → Concluded. CRO/UX: `name`, `area`, `hypothesis`,
  `variant`, `metric`, `startDate`, `endDate`, `assignee`, `outcome`,
  `result`, status Idea → Building → Live → Analyzing → Concluded. On
  the timeline, the "prep" bar runs from when the test was logged
  (`createdAt`) to `startDate`, and "live" runs `startDate` to `endDate`
  (or a 14-day estimate if `endDate` isn't set yet).
- `.../comments/{commentId}` — a comment thread under any commentable item
  (`promotions/{promoId}/comments`, `promotions/{promoId}/tasks/{taskId}/comments`,
  `emails/{emailId}/comments`, `paidTests/{testId}/comments`,
  `croTests/{testId}/comments`). Each comment is `text`, `by`, `byEmail`,
  `at` — anyone on the allowlist can read and post; only the author (matched
  on `byEmail`) can delete their own comment. Not shown in the Activity log,
  which tracks structural changes rather than conversation.
- `activity/{entryId}` — an append-only change log: one document per create,
  edit, move, or delete anywhere on the board (`action`, `by`, `at`,
  `promoName`, `taskTitle`, `detail`). Shown newest-first under the
  **Activity** tab (last 100 entries).
- `allowlist/{email}` — access control, managed only from the Firebase
  console (see above).
- `users/{email}` — the team directory that powers assignee dropdowns
  and My Work. Each person writes only their own document (`email`,
  `displayName`, `lastSeen`) the moment they sign in — nothing to
  pre-populate. Assignee fields store this email (not a free-typed
  name), so "assigned to me" can match reliably; a value that doesn't
  match anyone in the directory (an old free-text name, or a teammate
  who hasn't signed in yet) still shows up as a selectable option on
  that record rather than being silently dropped.
- `meta/seed` — a sentinel document used once, to guard against seeding
  sample data twice if two people open an empty board at the same moment.

The board seeds itself with sample promotions the first time the shared
store is empty (guarded by a Firestore transaction so two people opening it
at once don't double-seed).

## Updating the board's code

Edit `index.html` and push — GitHub Pages picks up the change automatically
on the next deploy from the configured branch.
