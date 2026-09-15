# Classification rules

Evaluated top to bottom. **First match wins.** Every scanned thread gets exactly one bucket.

Values in `CONFIG:` references come from `config.md`.

---

## R-1 — User overrides

Checked before everything else. Entries in `CONFIG: overrides` are corrections the user has made and chosen to keep.

Match on `sender`, and on `subject_contains` where present. **The more specific entry wins** — an entry carrying `subject_contains` is matched before a bare sender entry for the same domain. This is what lets one sender route two ways: a platform's "X has messaged you" notification and the same platform's marketing both exist, and they are not the same thing.

An override assigns the bucket directly. No rule below is consulted. If the entry carries `prepare_draft: true`, prepare a draft for that thread regardless of whether it reads as reply-expecting.

**These are instructions, not hints.** The user already saw the classification that produced this entry and disagreed with it. Re-evaluating whether they were right is not triage.

One exception, and only one: if an override would place a `CONFIG: protected_senders` match into Low-Value, confirm once before writing the entry. After it is written, honour it without asking again.

Report overrides that fired as a count in the run summary. The user should be able to see their own rules working.

---

## R0 — Deduplicate forwards

Many people auto-forward a second account (school, old work) into their main inbox, so the same message arrives twice.

Drop a forwarded copy when **all** hold:

- the sender matches `CONFIG: forwarding_addresses`
- the subject carries a forward prefix (`FW:`, `Fwd:`, `RE: FW:`)
- the same original sender arrived directly within 48 hours

If no original arrived, classify the forward normally — it is the only copy.

Report suppressed duplicates as a count. Never silently vanish mail.

---

## R1 — Protected senders

Banks, government services, account security, and billing. Matched against `CONFIG: protected_senders`.

**Never Low-Value, under any rule below.** These are exactly the messages a triage agent must not bin.

**R1a → Action Needed.** An event occurred *on the user's account*: new-device or new-browser login, password or security change, price or plan change, failed payment, third-party data access they did not initiate.

**R1b → FYI.** Routine authentication noise: sign-in-with-Google/OAuth notices for services they use daily. Real, but firing on every login makes it wallpaper.

**R1c → falls through to R6.** The same institution *marketing* to them. A bank's daily market bulletin is not an account event. The distinguishing question: **does this report something that happened to their account, or is it selling and informing generally?**

---

## R2 — Digest senders → FYI, always

Senders in `CONFIG: digest_senders`.

**Do not open these bodies.** The subject line already carries the payload — `"Associate Project Manager at FIRST"`, `"Your package is out for delivery"`, `"3 new listings in Kadıköy"`. Expanding it adds nothing. Present them as a scannable list, grouped by sender, one line each.

This rule exists for cost as much as clarity. Digest mail is the heaviest category in most inboxes, routinely 150–270KB of HTML per message; opening a day's worth can cost ten to twenty times the entire rest of the run, to surface information already sitting in the subject. Job boards are the most common instance, not the only one.

If a user wants genuine filtering inside this mail — role matching, price thresholds — that is a separate deliberate workflow with a stated budget, not a step smuggled into daily triage.

## R3 — People versus platforms

**A real person expecting a reply → Action Needed.** Individually addressed, non-`noreply`, and a response would be expected.

**Direct-message notifications → Action Needed.** `"X messaged you"` from a social or professional network means a human is waiting, even though the email itself is automated.

**All other platform nudges → Low-Value.** Invitation accepted, profile views, "people you may know", session replays, engagement prompts. These report activity, not obligation.

---

## R4 — Commitments already made → Action Needed

Something the user signed up for, paid for, registered for, or booked, which is now happening or expiring.

The test: **are they already a participant, or is this an invitation to become one?**

- A session tonight for a course they enrolled in → Action Needed
- A saved job posting expiring Friday → Action Needed
- 90% off that same course → not this rule; falls to R6
- "Last chance to register" for something they never joined → not this rule

---

## R5 — Subscribed newsletters → FYI

Senders in `CONFIG: newsletter_senders`, plus recognisable subscription mail with genuine editorial content. Collapse to one line listing subjects. The user chose to receive these; they are not junk, and they are not urgent.

### R5b — Human-signed mail from a product they use → FYI

A named individual writing to make a genuine ask — a founder requesting feedback, a support engineer following up.

**Does not apply** to bulk marketing wearing a person's name: an instructor "announcement" carrying a coupon and an unsubscribe footer is a promotion regardless of the signature. The test is whether an individual is actually asking something, not whether a name appears at the top.

---

## R6 — Everything else → Low-Value

Retail and travel promotions, discount codes, course and learning-platform marketing, cold bulk outreach, mass surveys, automated digests without editorial substance.

---

## Overriding rules

These outrank everything above.

1. **Uncertainty → FYI.** Never Low-Value. A misfiled FYI costs a glance; a wrongly deleted email can cost an opportunity.
2. **Protected senders are never deletable**, whatever any other rule concludes.
3. **Anything from a real person is never Low-Value.**
4. **Nothing is deleted without explicit confirmation**, on a list the user has seen, in the current session.

---

## Tuning

Expect the first week to be wrong in small ways. The two failure modes worth watching:

- **A genuinely wanted email buried in the FYI digest.** Usually means a sender needs promoting in `config.md`.
- **Something in Low-Value that should never have been there.** Usually means a missing entry in `protected_senders`.

Both are config fixes, not rule rewrites. Change the rules only when the same misclassification survives a correct config.

For a single sender behaving wrongly, the faster path is an override: correct it in chat and accept the offer to keep it. Reach for `protected_senders` and the other lists when a whole *class* of mail is misfiled, and for overrides when one sender is.
