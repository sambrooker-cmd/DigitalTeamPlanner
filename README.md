# Promotion Planner

A kanban and timeline board for planning marketing promotions across the
Acquisition, Retention, and Website teams — from briefing through launch to
results reporting.

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

    match /promotions/{promoId} {
      allow read, write: if isAllowed();
      match /tasks/{taskId} {
        allow read, write: if isAllowed();
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
  }
}
```

Anyone not signed in, or signed in but not on the allowlist, is bounced to a
sign-in or "ask an admin" screen before ever reaching the board — enforced
by these rules server-side, not just hidden in the UI.

### Data model

- `promotions/{promoId}` — one document per promotion (`name`, `launchDate`,
  `endDate` (optional), `description`, plus
  `createdBy`/`createdAt`/`updatedBy`/`updatedAt`). When `endDate` isn't
  set, the timeline estimates the promotion runs 14 days from launch.
- `promotions/{promoId}/tasks/{taskId}` — one document per task/result card
  (`team`, `type`, `title`, `assignee`, `due`, `status`, `notes`, plus the
  same attribution fields), so concurrent edits from different teams never
  overwrite each other.
- `activity/{entryId}` — an append-only change log: one document per create,
  edit, move, or delete anywhere on the board (`action`, `by`, `at`,
  `promoName`, `taskTitle`, `detail`). Shown newest-first under the
  **Activity** tab (last 100 entries).
- `allowlist/{email}` — access control, managed only from the Firebase
  console (see above).
- `meta/seed` — a sentinel document used once, to guard against seeding
  sample data twice if two people open an empty board at the same moment.

The board seeds itself with sample promotions the first time the shared
store is empty (guarded by a Firestore transaction so two people opening it
at once don't double-seed).

## Updating the board's code

Edit `index.html` and push — GitHub Pages picks up the change automatically
on the next deploy from the configured branch.
