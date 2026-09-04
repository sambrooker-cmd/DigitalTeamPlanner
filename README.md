# Digital Team Planner

A shared planning tool for the Acquisition, Retention, and Website teams.
The top-level nav is a **left sidebar** — **My Work**, **Overview**,
**Stats**, **Boards**, **Activity**, plus a sixth item, **Admin**, visible
only to Admins — collapsible to icons-only (click the arrow at the
bottom of the sidebar) so it stays out of the way; the choice is
remembered per-browser, like the theme setting, and starts collapsed on
mobile. The four team channels live inside **Boards** behind their own
switcher rather than each getting a sidebar item of their own, and that
switcher shares a row with the section's own Board/Timeline (or
Board/Calendar) toggle — the two are independent questions ("which
channel," "which view"), not sequential steps, so they never stack as
separate bars. The result is at most one row of context controls above
any screen's actual content, on top of the sidebar:

Everyone who signs in has one of three roles: **Admin** (also manages who
has access and what role they hold), **Editor** (the default — can create,
edit, delete, comment, and attach files everywhere), or **Viewer**
(read-only: sees everything, including exports, but every create/edit/
delete control is hidden or disabled). See "Firebase setup" below for how
roles are set.

- **My Work** — everything assigned to the signed-in viewer (promotion
  tasks, emails, paid media tests, CRO/UX tests), soonest due date
  first, overdue items in red. Anything due today is pulled into its own
  "Due today" group at the top.
- **Overview** — a unified, read-only timeline rolling up every
  promotion, email, paid media test, and CRO/UX test into one
  chronological view, each row tagged with a team-colored badge. Click a
  row to jump to it in its own board. Searchable by name, filterable
  by kind (Promotion/Email/Paid Media/CRO-UX) and by assignee — for a
  promotion, the assignee filter matches if any of its tasks are
  assigned to that person, since a promotion itself has no single
  assignee. Any combination of search/kind/assignee can be named and
  saved ("+ Save current view") for one-click recall later, instead of
  resetting it each time — saved per-browser (like the theme choice
  below), not shared with the team.
- **Stats** — a handful of headline numbers read straight off the boards
  below: promotions live/in prep/archived, each test tracker's win rate
  (Win vs. Loss only — TBD/Inconclusive don't count either way) plus how
  many are live and logged in total, emails sent vs. planned this
  calendar month, and a team-health count of overdue tasks and emails
  (past their due/send date and not yet marked done). Nothing is stored
  separately for this — it's all computed live from existing data.
- **Boards** — a channel switcher (Promotions / Acquisition / Retention /
  Website) for the four team channels, each remembering its own
  board/timeline(/calendar) sub-view independently:
  - **Promotions** — cross-team, since every promotion involves all three
    teams: a kanban board (briefing through launch to results reporting)
    and a timeline. Which promotion you're viewing is picked from a
    dropdown next to its name (not a tab strip) — it isn't really
    navigation, just choosing a record, so it's styled as part of the
    board's own header rather than another nav bar; the dropdown itself
    still shows every promotion's name, dates, and progress at a glance,
    same as the old tab strip did.
  - **Acquisition** — Paid Media Tests: a status board and a timeline.
  - **Retention** — Emails: a status board and a month calendar.
  - **Website** — CRO/UX Tests: a status board and a timeline.
- **Activity** — a live change log across everything above, plus a
  **Recently Deleted** sub-view for restoring (or permanently deleting)
  anything soft-deleted — see below.
- **Admin** (Admins only) — the team's allowlist: add someone by email
  with a starting role, change anyone's role, or revoke their access —
  all without touching the Firebase console.

The header also has a **search box** that reaches everything at once —
promotions, individual tasks, emails, and both test trackers — regardless
of which tab you're on; picking a result jumps straight there. Next to it,
a **reminder bell** (🔔) shows a count of anything assigned to you that's
due today or overdue and still open — click it for the list, click an item
to jump straight to it. This is a passive, in-app reminder only: it's just
`My Work` filtered down, so it only reflects reality while you actually
have the app open — nothing is emailed or pushed to you outside the tab.
A **theme button** next to it cycles System → Light → Dark and remembers
the choice per-browser (dark mode otherwise just follows your OS setting
on its own).

Each timeline (Promotions, Paid Media, CRO/UX) uses the same visual
language: a light bar for the prep/build period, a solid bar for when
it's actually live, and separate launch/end markers — so "still being
prepped" vs. "actually running" is always visually distinct.

Every promotion, task, email, and test can be **duplicated** from its edit
modal — it pre-fills a new, unsaved copy (dates and result fields cleared,
nothing else) so a repeat promotion or test doesn't mean re-typing the whole
form; nothing is written until you adjust it and hit Save. Each of those
also carries its own **comment thread**, for back-and-forth that would
otherwise overwrite the single free-text notes field — type `@Name` (or
just `@FirstName`) to mention a teammate; it's highlighted in the thread and
the comment itself is visually called out for them, though nothing is
pushed to them outside the app yet. Each edit screen also has an
**Attachments** section — "+ Add file" to upload a document or image
(10MB cap), shown as a thumbnail for images or a filename link otherwise,
with size/uploader/date; only the person who uploaded a file can delete it.
Deleting a promotion, task, email, or
test shows an **Undo** on its toast for a few seconds — nothing changes in
Firestore until that window passes, so Undo is exact, not a re-creation.
After that window, the record is only *soft*-deleted (hidden everywhere,
but not actually removed) and lands in **Recently Deleted**, a sub-view of
the Activity tab, where it stays indefinitely — Restore brings it straight
back exactly as it was; Delete forever removes it for good and can't be
undone. Deleting a promotion this way doesn't touch its tasks, so restoring
it brings all of them straight back too. Each of those edit screens also has a collapsed
**History** section — that item's own slice of the Activity log, so you
don't have to scroll the full log to see what happened to just this one
(starts from when this shipped; older activity wasn't tagged with an item
id to filter by). Promotions, Emails,
Paid Media, and CRO/UX each have an **Export CSV** button (on the
promotion's board, and by the "+ New…" button on the other three) for
pulling the current data into a deck or report. Emails and Promotions also
have an **Export .ics** button next to it — Emails exports every email as
an all-day event on its send date; a promotion exports its own launch/end
dates — for importing into Outlook, Google Calendar, or Apple Calendar.

On the Emails **calendar** view, dragging an email's chip to a different
day reschedules it (updates `sendDate`) — same drag-and-drop as the status
board, just against days instead of columns.

For promotions that recur (Wave Season, Black Friday), any promotion's edit
screen has a **Save as template** button that snapshots its task structure —
which teams do what, with roughly what titles and who usually owns them, but
no dates or statuses, since those only make sense for one specific launch.
Starting a **new** promotion offers a "Start from a template" picker that
pre-fills the name/description and recreates that task list (assignees
carried over, due dates left blank for you to set) as soon as you save.
Templates are managed from a "Manage templates" link on the Promotions
board — rename or delete them there. They're a separate, lightweight
collection: not shown in the timeline, Stats, Overview, or search, and they
don't get comment threads or CSV export — a preset isn't a piece of tracked
work.

When something goes wrong (a save fails, a required field is missing), it
shows up as a small dismissible notification in the bottom corner rather
than a blocking browser alert — same information, just doesn't freeze the
tab while you read it.

## Live board

`index.html` is a single self-contained page (React, loaded from cdnjs, no
build step) backed by Firebase Firestore, Firebase Authentication, and
Firebase Cloud Storage (for attachments), which is what gives it real
shared, multi-user editing restricted to your team: everyone signs in with
Google, and only people you've explicitly approved can see or edit the
board — no Claude account or shared organization required.

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

**2. Approve who can access the board, and make yourself an Admin.** Under
**Firestore Database → Data**, create a collection named `allowlist`. For
each teammate you want to let in, add a document whose **document ID is
their exact Google account email** (e.g. `sam@gmail.com`) with a `role`
field set to `admin`, `editor`, or `viewer` — Admins can also manage this
list from inside the app (see the **Admin** tab, below), Editors can
create/edit/delete everything, Viewers can look but not touch anything.
A document with no `role` field defaults to `editor`. **Give at least one
person — probably yourself — `role: "admin"` here**, since that's the only
way to become the first Admin; everyone after that can be added/promoted/
revoked from the Admin tab instead of coming back to this console. To
revoke someone without the app, delete their document here.

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

    // A missing `role` field defaults to 'editor' — same default the client uses — so nobody's
    // access silently changes for allowlist docs created before roles existed.
    function myRole() {
      let doc = get(/databases/$(database)/documents/allowlist/$(request.auth.token.email));
      return 'role' in doc.data ? doc.data.role : 'editor';
    }

    function isEditor() {
      return isAllowed() && myRole() in ['editor', 'admin'];
    }

    function isAdmin() {
      return isAllowed() && myRole() == 'admin';
    }

    function isAuthor() {
      return isEditor() && request.auth.token.email == resource.data.byEmail;
    }

    match /promotions/{promoId} {
      allow read: if isAllowed();
      allow write: if isEditor();
      match /tasks/{taskId} {
        allow read: if isAllowed();
        allow write: if isEditor();
        match /comments/{commentId} {
          allow read: if isAllowed();
          allow create: if isEditor();
          allow update: if false;
          allow delete: if isAuthor();
        }
        match /attachments/{attachmentId} {
          allow read: if isAllowed();
          allow create: if isEditor();
          allow update: if false;
          allow delete: if isAuthor();
        }
      }
      match /comments/{commentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
      match /attachments/{attachmentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
    }

    match /emails/{emailId} {
      allow read: if isAllowed();
      allow write: if isEditor();
      match /comments/{commentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
      match /attachments/{attachmentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
    }

    match /paidTests/{testId} {
      allow read: if isAllowed();
      allow write: if isEditor();
      match /comments/{commentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
      match /attachments/{attachmentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
    }

    match /croTests/{testId} {
      allow read: if isAllowed();
      allow write: if isEditor();
      match /comments/{commentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
      match /attachments/{attachmentId} {
        allow read: if isAllowed();
        allow create: if isEditor();
        allow update: if false;
        allow delete: if isAuthor();
      }
    }

    match /promoTemplates/{templateId} {
      allow read: if isAllowed();
      allow write: if isEditor();
    }

    match /activity/{entryId} {
      allow read: if isAllowed();
      allow write: if isEditor();
    }

    match /meta/{docId} {
      allow read: if isAllowed();
      allow write: if isEditor();
    }

    // Everyone on the allowlist can read their own entry (that's how the app finds out its own
    // role); only Admins can list the whole collection or write to it — which is what makes the
    // in-app Admin tab work without ever touching the Firebase console again after bootstrapping.
    match /allowlist/{email} {
      allow get: if (request.auth != null && request.auth.token.email == email) || isAdmin();
      allow list: if isAdmin();
      allow write: if isAdmin();
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
by these rules server-side, not just hidden in the UI. Viewers get past that
screen (they're allowed in) but every write above requires `isEditor()`, so
a Viewer who somehow bypassed the UI (devtools, a stale tab) still can't
create, edit, or delete anything — the UI hiding those controls is a
convenience, not the actual security boundary.

**4. Enable Cloud Storage and set its rules.** Attachments need Firebase
Storage, which — unlike Firestore — isn't on by default. In the
[Firebase console](https://console.firebase.google.com) → **Storage** →
**Get started**, accept the default location/production-mode prompts (the
rules below replace whatever it scaffolds). Then under **Storage → Rules**,
use:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    function isAllowed() {
      return request.auth != null &&
        request.auth.token.email != null &&
        firestore.exists(/databases/(default)/documents/allowlist/$(request.auth.token.email));
    }

    function myRole() {
      let doc = firestore.get(/databases/(default)/documents/allowlist/$(request.auth.token.email));
      return 'role' in doc.data ? doc.data.role : 'editor';
    }

    function isEditor() {
      return isAllowed() && myRole() in ['editor', 'admin'];
    }

    match /attachments/{allPaths=**} {
      allow read: if isAllowed();
      allow write: if isEditor() && request.resource.size < 10 * 1024 * 1024;
      allow delete: if isEditor();
    }
  }
}
```

This mirrors the same allowlist gate (and now the same role check) as the
Firestore rules above — Storage rules can reach into Firestore with
`firestore.exists()`/`firestore.get()` to reuse them, so access stays
governed from the one `allowlist` collection — and caps uploads at 10MB
server-side, matching the app's own client-side check. Storage doesn't
support per-document ownership checks the way Firestore's
`resource.data.byEmail` does, so file deletion here is gated on role only
(any Editor/Admin could technically delete any attachment's Storage object
directly), while the app's own UI only ever offers the Delete button to the
uploader; the matching Firestore rule above (`allow delete: if isAuthor()`)
still stops anyone but the uploader from removing the attachment's metadata
doc, which is what the app actually reads.

### Data model

Every document type below that can be deleted from the board (promotions,
tasks, emails, both test trackers) also carries three optional soft-delete
fields — `deleted` (boolean), `deletedAt`, `deletedBy` — set when someone
deletes it and cleared entirely on Restore. A document without them has
simply never been deleted; nothing is backfilled or required at creation
time. See "Recently Deleted" above for the UI this powers.

- `promotions/{promoId}` — one document per promotion (`name`, `launchDate`,
  `endDate` (optional), `description`, `parentPromoId` (optional, links it
  to another promotion as its umbrella/central one), `archived` (boolean),
  plus `createdBy`/`createdAt`/`updatedBy`/`updatedAt`). When `endDate`
  isn't set, the timeline estimates the promotion runs 14 days from launch.
  Archived promotions are hidden from the timeline and, by default, from
  the board's promotion picker (a "Show archived" toggle reveals them
  there so they can be unarchived or deleted).

  For a big umbrella campaign (Wave Season, say) running several distinct
  sub-promotions underneath it, a sub-promotion can name the umbrella one
  as its `parentPromoId`. A "central" promotion — one that's *named as*
  someone else's `parentPromoId` — isn't a different kind of record: it's
  planned exactly the same way as any other promotion (same board, same
  task structure), just visually called out wherever promotions appear —
  the promotion picker, the Promotions timeline, and the Overview
  timeline — with a small "Central" badge and one shade bolder color
  (`PURPLE_DARK` where a regular promotion's bar is `PURPLE`, and so on),
  same accent family throughout rather than a new color. There's no
  separate list of sub-promotions or "part of" breadcrumb — the only
  place the relationship is set is the "Parent promotion" dropdown on a
  sub-promotion's own edit form; everywhere else, it's just a color.
- `promotions/{promoId}/tasks/{taskId}` — one document per task/result card
  (`team`, `type`, `title`, `assignee`, `due`, `status`, `notes`, plus the
  same attribution fields), so concurrent edits from different teams never
  overwrite each other.
- `emails/{emailId}` — one document per BAU/lifecycle email (`subject`,
  `previewText` (optional), `segment` (optional — the audience this send
  targets, e.g. "Past guests — Caribbean"), `cta` (optional — the primary
  link/action), `sendDate`, `status` —
  Briefing/Testing/Approved/Scheduled/Sent/Reported — `assignee`,
  `relatedPromoId` (optional, links it to a promotion), `briefDocLink`
  (optional — the brief/creative doc, kept separate from `hubspotLink`
  since briefing usually starts before HubSpot is even touched),
  `hubspotLink` (optional), `notes`, plus the same attribution fields).
  Shown under the **Retention** section as a status board (drag between
  columns) or a month calendar (click a day to add one, click a chip to
  edit it).
  This is where the email team's planning calendar and Trello-style
  briefing status live — HubSpot itself stays the tool that builds and
  sends the email; `hubspotLink` just points at it.

  For a send that repeats on a cadence (a weekly newsletter, a monthly
  loyalty email), **Repeat next week** / **Repeat next month** sit next to
  Duplicate on an existing email's edit screen — same pre-filled,
  nothing's-written-until-you-hit-Save pattern, but keeping the exact
  subject (it's the next occurrence of the same send, not a distinct new
  one) and advancing the send date automatically instead of leaving it
  blank.
- `paidTests/{testId}` and `croTests/{testId}` — one document per
  experiment (Paid Media and CRO/UX respectively), each shown under its
  own team section as a status board or a timeline. Both are
  "hypothesis → variant → result" shaped but with different fields and
  stages, driven by the `TEST_TRACKERS` config in `index.html` rather
  than duplicated code — add a third tracker (e.g. QA/stability) by
  adding a new entry there, not by copying a whole module. Paid Media:
  `name`, `channel`, `hypothesis`, `variants`, `startDate`, `endDate`,
  `budget`, `assignee`, `relatedPromoId` (optional, links it to a
  promotion), `outcome`, `result`, status Planned → Live → Analyzing →
  Concluded. CRO/UX: `name`, `area`, `hypothesis`, `variant`, `metric`,
  `startDate`, `endDate`, `assignee`, `relatedPromoId` (optional),
  `outcome`, `result`, status Idea → Building → Live → Analyzing →
  Concluded. On the timeline, the "prep" bar runs from when the test was
  logged (`createdAt`) to `startDate`, and "live" runs `startDate` to
  `endDate` (or a 14-day estimate if `endDate` isn't set yet). Most
  Acquisition/Website work is standalone, ongoing experimentation with no
  tie to any one promotion, so this link is optional, not required —
  a test either carries it or it doesn't, same as an email's own
  `relatedPromoId`.

  A promotion's own board shows a **Linked work** section listing any
  Paid Media Tests, CRO/UX Tests, and Emails that named it as their
  `relatedPromoId`, each with its own real status, clickable straight
  into its native edit modal — plus "+ Paid Media Test" / "+ CRO/UX
  Test" / "+ Email" buttons that open a new one pre-linked to that
  promotion. This is the literal same document shown in two places, not
  a duplicate kept in sync — editing it from the promotion's board or
  from its own team channel is the same write, so there's nothing that
  can drift out of sync between them.
- `promoTemplates/{templateId}` — a reusable promotion preset (`name`,
  `description`, `tasks` — an embedded array of `{team, type, title,
  assignee}`, no dates or status, plus the usual attribution fields).
  Not a promotion itself and not shown in the timeline, Stats, Overview,
  or search — see "Save as template" above.
- `.../comments/{commentId}` — a comment thread under any commentable item
  (`promotions/{promoId}/comments`, `promotions/{promoId}/tasks/{taskId}/comments`,
  `emails/{emailId}/comments`, `paidTests/{testId}/comments`,
  `croTests/{testId}/comments`). Each comment is `text`, `by`, `byEmail`,
  `mentions` (emails of anyone `@mentioned` in the text, matched against the
  team directory), `at` — anyone on the allowlist can read and post; only
  the author (matched on `byEmail`) can delete their own comment. Not shown
  in the Activity log, which tracks structural changes rather than
  conversation.
- `.../attachments/{attachmentId}` — file/image metadata under any of the
  same five item types as comments above. Each doc is `name`, `size`,
  `contentType`, `storagePath`, `url` (the Storage download URL), `by`,
  `byEmail`, `at`; the actual bytes live in Firebase Storage at
  `storagePath` (mirroring the Firestore path, under a top-level
  `attachments/` prefix), capped at 10MB. Anyone on the allowlist can read
  and upload; only the uploader (matched on `byEmail`) can delete their own
  file. Also not shown in the Activity log, for the same reason as comments.
- `activity/{entryId}` — an append-only change log: one document per create,
  edit, move, or delete anywhere on the board (`action`, `by`, `at`,
  `promoName`, `taskTitle`, `detail`, plus `promoId`/`taskId`/`emailId`/
  `testId` where relevant, so each item's own edit screen can show a
  filtered History of just itself). Shown newest-first under the
  **Activity** tab (last 100 entries).
- `allowlist/{email}` — access control: who can sign in (`role`: `admin` /
  `editor` / `viewer`, defaulting to `editor` if unset, plus `addedBy`/
  `addedAt` when added from the app). Bootstrapped from the Firebase
  console (see above); day-to-day, Admins manage it from the in-app
  **Admin** tab instead.
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
