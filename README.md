# Promotion Planner

A kanban and timeline board for planning marketing promotions across the
Acquisition, Retention, and Website teams — from briefing through launch to
results reporting.

## Live board

`index.html` is a single self-contained page (React, loaded from cdnjs, no
build step) backed by Firebase Firestore, which is what gives it real
shared, multi-user editing: everyone who opens the link sees the same board
and each other's changes live, no Claude account or shared organization
required.

### Hosting it

The page is plain static HTML, so any static host works. The simplest is
GitHub Pages on this repo:

1. **Settings → Pages** → Source: **Deploy from a branch** → pick this
   branch (or `main`, once merged) → folder `/ (root)` → **Save**.
2. GitHub publishes it at `https://<org>.github.io/Promotionsplanner/`.
   Share that link with your team — anyone with it can open and edit the
   board, regardless of what Claude (or Google) account they're on.

### Firestore setup

The page talks to a Firebase project (`promotions-planner`) via Firestore.
In the [Firebase console](https://console.firebase.google.com), under
**Firestore Database → Rules**, use:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /promotions/{promoId} {
      allow read, write: if true;
      match /tasks/{taskId} {
        allow read, write: if true;
      }
    }
    match /meta/{docId} {
      allow read, write: if true;
    }
  }
}
```

**Trade-off to know about:** these rules have no authentication check —
anyone who has the Firebase config (visible in `index.html`'s source, which
is normal for client-side Firebase apps) can read or write the board's data
directly, not just through the app's UI. That's fine for an internal team
tool passed around by link, but it isn't real access control. If you want
that later, add Firebase Authentication (e.g. Google sign-in restricted to
your company's email domain) and tighten the rules above to require
`request.auth != null`.

### Data model

- `promotions/{promoId}` — one document per promotion (`name`, `launchDate`,
  `description`).
- `promotions/{promoId}/tasks/{taskId}` — one document per task/result card
  (`team`, `type`, `title`, `assignee`, `due`, `status`, `notes`), so
  concurrent edits from different teams never overwrite each other.

The board seeds itself with sample promotions the first time the shared
store is empty (guarded by a Firestore transaction so two people opening it
at once don't double-seed).

## Updating the board's code

Edit `index.html` and push — GitHub Pages picks up the change automatically
on the next deploy from the configured branch.
