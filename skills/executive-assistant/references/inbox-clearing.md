# Inbox clearing notes

Use this file as a quick reminder of the default inbox posture.

## Core rules

- use `m365 outlook message list` over explicit `--startTime` / `--endTime` windows (and filter locally) — Outlook does not have Gmail's `is:unread` / `newer_than:` query idiom, and conversation-only views hide individual unread messages
- sweep the `sentitems` folder so unanswered outbound threads do not disappear
- for any reply to an existing thread, POST to the Graph `/me/messages/{id}/reply` endpoint via `m365 request` instead of a fresh `m365 outlook mail send` with `Re:` in the subject, so the `conversationId` is preserved
- use `/me/messages/{id}/replyAll` (not `/reply`) when the thread recipients should stay copied
- do not leave actionable email in a reviewed batch without one of these outcomes:
  - handled
  - acknowledged with next step
  - deliberately escalated
  - explicitly deferred for a stated reason
- when a thread creates a future dependency, add a follow-up task instead of relying on memory alone

## Good auto-handle cases

- scheduling and rescheduling
- invite creation / update / cancellation when authority is clear
- short factual or operational replies
- confirmations of receipt
- obvious admin / vendor notices
- newsletters and noise

## Ask first when the thread is

- legal or policy-sensitive
- investor / board / fundraising related
- pricing-sensitive
- press or reputation-sensitive
- emotionally sensitive or personal
- strategically important and ambiguous
