# Promotion Planner

A kanban and timeline board for planning marketing promotions across the
Acquisition, Retention, and Website teams — from briefing through launch to
results reporting.

## Live board

`promotion-planner.html` is published as a Claude Artifact, which is what
gives it real shared, multi-user editing: everyone who opens the link sees
the same board and each other's changes live, no server of your own to run.

- Open the artifact and use **Share** on the page to let your teammates in.
- The `db` capability it declares is organization-scoped: every reader and
  writer must be a signed-in member of the same Claude organization as the
  artifact's owner. It cannot be made public.
- If a viewer opens it somewhere that capability isn't granted (for example,
  the raw file outside claude.ai), the board falls back to saving changes to
  that browser's local storage only, and shows a "Local only" indicator
  instead of "Live – synced with your team".

## Updating the board's code

Edit `promotion-planner.html` (a single self-contained page: React, loaded
from cdnjs, with no build step) and republish it as an Artifact to push
changes live to the same link.

### Data model

- `promotions/{promoId}` — one document per promotion (`name`, `launchDate`,
  `description`).
- `promotions/{promoId}/tasks/{taskId}` — one document per task/result card
  (`team`, `type`, `title`, `assignee`, `due`, `status`, `notes`), so
  concurrent edits from different teams never overwrite each other.

The board seeds itself with sample promotions the first time the shared
store is empty (guarded by a lease so two people opening it at once don't
double-seed).
