---
name: gmail-triage
description: Triages a Gmail inbox into Action Needed, FYI, and Low-Value, flags calendar-worthy mail, and prepares reply drafts without sending them. Use when the user says "triage my inbox", "check my email", "what needs my attention today", "sort my mail", "morning email review", or asks what arrived overnight that matters.
---

# Gmail Triage

Sort recent mail into three buckets, surface anything calendar-worthy, and prepare drafts. Report findings and stop for confirmation before any destructive or outbound action.

## Hard constraints

These are not defaults. They do not bend for convenience, batching, or an impatient user.

- **Never send an email.** Gmail drafts only.
- **Never trash or delete without explicit confirmation** in the current session, on a list the user has seen.
- **Never create a calendar event without explicit confirmation.**
- **Uncertainty defaults to FYI**, never Low-Value. Ambiguity does not earn deletion.

## Step 0 — Setup

Read `references/config.md` first. If it still carries the example values, **do not ask the user to fill it in** — not by editing the file, and not by answering questions from memory. Nobody can recall which addresses send them what. Their inbox already knows.

Requires Gmail and Google Calendar connectors. If either is missing, say so plainly rather than degrading silently.

### Look first

Scan 30 days of metadata before asking anything — senders, subjects, frequency. Do not open bodies. From that, draft all four lists:

- **`protected_senders`** — senders whose mail reads as security alerts, statements, invoices, receipts, payment or plan changes. Banks, government services, insurers, landlords, anything that bills them.
- **`forwarding_addresses`** — any address that repeatedly arrives carrying `FW:`/`Fwd:` subjects that duplicate mail from elsewhere.
- **`digest_senders`** — repeat senders with high volume and templated, self-contained subject lines.
- **`newsletter_senders`** — repeat senders with editorial subjects and unsubscribe footers.

Also derive the two settings rather than asking cold: read the calendar's configured timezone, and find the hour band where bulk mail actually lands in this inbox.

### Then confirm

Present each list as a numbered set for approval, one at a time, with a visible reason per line — `3. notifications@yourbank.example — 12 statements and 2 security alerts this month`. The reason is what makes confirmation possible; a bare address is a memory test again.

Accept corrections in whatever form they arrive: "drop 3 and 5", "the second one isn't a bank", "add my landlord", "all fine". Then move on.

Two questions genuinely need asking, and both carry a proposed answer so the user is agreeing or adjusting rather than composing:

1. **Timezone.** State what the calendar says and ask whether that's where they actually are. A calendar set to a former city is common and silently ruins event times.
2. **Run hour.** Propose one from the observed bulk-mail band and explain the reasoning: bulk mail lands overnight, real people write during their own working day, and mail that arrives while they're at their desk doesn't need an agent.

Leave `voice_profile` empty unless they insist. Derived-from-sent-mail beats self-description almost every time. Leave `overrides` empty always — it is not a setup field.

### Say what it can't know

Close setup by saying plainly that the config is a starting position, not a finished one — a 30-day scan cannot see a sender who writes quarterly, and no list survives first contact with a real inbox. What closes that gap is Step 6: correcting a call out loud, once, and the correction persisting.

Say it because it is true, and because a user who expects to correct the first week does correct it, while one who expected it to arrive finished reads the same mistakes as failure and stops running it.

Then write `references/config.md`, say what was written, and stop. Do not run a triage in the same breath — let them see the config first.

## Step 1 — Scan

Cover the window the user asked for; default to the last 24 hours. Cast three nets:

1. **Unread.** Search `is:unread after:YYYY/MM/DD -in:draft`.
2. **Unanswered humans.** Threads where a real person wrote last and the user has not replied — read or unread. This catches the message they opened on their phone and forgot.
3. **Starred.** Search `is:starred newer_than:7d`. Every result goes to **Action Needed, unconditionally.**

Net 2 surfaces an item only if something genuinely still needs an action: a person waiting, a live deadline, an unresolved account event.

**Net 3 gets no such filter, deliberately.** A manual star is the user telling you this matters. Evaluating whether they were right is not triage, it is second-guessing — and it defeats the one mechanism they have for overriding the classifier during the day. A star outranks every rule below.

If a starred thread already carries `Triage/FYI` or `Triage/Low-Value` from an earlier run, **move it**: remove the old label, apply `Triage/Action Needed`. A later star supersedes an earlier classification.

Bound the star net to a recent window (7 days is a reasonable default) so an existing backlog of old stars doesn't flood the first run. Check the size of that backlog before enabling this and tell the user what you found.

Ignore Gmail's automatic `IMPORTANT` flag entirely. In a busy inbox it lands on retail promotions constantly and carries no usable signal. Manual stars only.

Use `search_threads` metadata. Do not open message bodies except where Step 2 requires it.

## Step 2 — Classify

Apply the rules in `references/classification-rules.md`. First match wins. Every scanned thread gets exactly one of three labels:

- `Triage/Action Needed`
- `Triage/FYI`
- `Triage/Low-Value`

Create these labels if they do not exist (`create_label` supports nesting with `/`). Apply them with `label_thread`.

**The labels are the user's navigation, not internal bookkeeping.** A chat report cannot be scrolled, searched, or previewed; a Gmail label can. Label everything scanned so the user can browse each bucket in Gmail's own interface.

### Cost discipline

Bulk senders — job boards, newsletters, retail — send enormous HTML emails, routinely 150–270KB each. Opening them is the single largest cost in this workflow and it is almost always unnecessary: the subject line already carries what the user needs.

Open a message body only when the classification genuinely turns on content the subject does not reveal, or when preparing a draft reply. Never open bulk mail to "check" a classification the sender alone determines.

## Step 3 — Detect calendar items

Run detection on **Action Needed and FYI only**. Never on Low-Value — promotional deadlines are real dates that belong nowhere near a calendar.

**Tier A — confirmed, dated, the user is a participant.** They are enrolled, registered, booked, invited, or interviewing. Propose a timed event.

**Tier B — a deadline they must act by.** No meeting, but a date with consequences. Propose an all-day event.

**Tier C — open invitation with no commitment.** Public webinars, "upcoming events" roundups. Mention in the report; propose nothing.

Suppress dates that appear inside newsletter *content*, and marketing urgency ("expires today", "last chance").

### Timezone handling

**Always set `timeZone` explicitly on every event created.** A calendar's own default timezone may differ from where the user actually lives, and an unpinned event silently lands at the wrong hour.

Note also that pinning fixes storage, not display: a calendar renders its grid in *its own* configured timezone. If the two disagree, say so, and tell the user the fix is in their Google Calendar settings. Do not let them discover this by missing an interview.

When the source email states no timezone, infer it from the sender's country and show the reasoning before asking for confirmation. When it states no end time, assume one hour and say the assumption is yours. When it implies a series ("Day 1", "Session 1 of 6"), propose only the session with a confirmed date.

## Step 4 — Prepare drafts

For Action Needed items that need a reply, write a draft and save it with `create_draft`. Never send.

Match the user's voice. If `references/config.md` records a voice profile, follow it. Otherwise read 10–15 of their sent messages (`in:sent`) and derive: greeting style, whether they open with a warmth line, paragraph length, how directly they state an ask, sign-off, and which language they use for which kind of recipient. Preserve their quirks — mixed contractions, comma splices, an unusual sign-off. Cleaning these up makes the draft sound like a machine.

Offer one warmer and one shorter alternative register alongside each draft rather than choosing for them.

If nothing needs a reply, say so plainly. Reporting zero drafts is an accurate outcome, not a failure to be papered over.

## Step 5 — Report

Keep it tight. The report is a summary and a decision point; the labels are where reading happens.

```
Triage — <date> · <n> threads scanned

ACTION NEEDED (n)
  1. <subject> · <sender> · <one line on why>
     https://mail.google.com/mail/u/0/#all/<threadId>

CALENDAR
  Propose: <title> · <date, time, timezone>  → confirm?
  Mention only: <Tier C items>

DRAFTS PREPARED (n)

FYI (n) — https://mail.google.com/mail/u/0/#label/Triage%2FFYI
  <digests grouped by sender, subject lines only; newsletters as one line>

LOW-VALUE (n) — https://mail.google.com/mail/u/0/#label/Triage%2FLow-Value
  <grouped by type> — delete list, awaiting approval

SUPPRESSED (n) — duplicates dropped
```

Link every Action Needed item directly to its thread. Link each label so the user can browse the whole bucket.

**End by asking for confirmation on (a) proposed calendar events and (b) the deletion list. Take no action on either until the user replies.**

When the user approves deletions, use `trash_thread`. Trash is recoverable for 30 days; permanent deletion is not offered by this skill.

## Step 6 — Corrections

When the user disputes a classification, two things happen, in this order.

**Fix it now.** Move the label. Do not argue the rule that produced it.

**Then offer to keep it.** Ask whether this should hold from now on. If yes, write an entry to the `overrides` block in `references/config.md` — sender, bucket, today's date, and a one-line note on the reasoning. Add `subject_contains` when the correction applies to one kind of message from a sender rather than all of it, and `prepare_draft: true` when the complaint was a missing draft rather than a wrong bucket.

Ask once, accept the answer, and don't raise it again for that sender.

**This step is the difference between a tool and a chore.** Without it the same correction is made every week and nothing accumulates. Treat a correction the user bothered to type as the most valuable signal in the run.

Read the complaint for what it actually is:

- *"This should have been Action Needed"* → a bucket override
- *"Why didn't you draft a reply to this?"* → `prepare_draft: true`, same bucket
- *"Stop showing me these"* → a Low-Value override, not a deletion request
- *"That one's fine, but the other mail from them isn't"* → two entries, the specific one carrying `subject_contains`

When a correction contradicts an existing override, say which one it replaces before rewriting it.

## Reporting honestly

Flag borderline calls rather than burying them. If a classification was close, if a sender's category is ambiguous, if a rule produced a result that looks wrong — say so in the report. A triage agent the user cannot correct is worse than no triage agent, because the mistakes become invisible.
