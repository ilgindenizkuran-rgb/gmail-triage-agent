# Configuration

Edit this file before the first run. Every value below is an example — replace them with your own.

The classification rules are deliberately generic. This file is where your inbox becomes specific. Getting it wrong mostly produces noise; getting `protected_senders` wrong is the one that can actually cost you something, so start there.

---

## protected_senders

**The most important list here.** Anything matching is never classified Low-Value and never appears on a deletion list, regardless of what other rules conclude.

Include: your banks, government services, tax authority, health insurer, landlord or mortgage provider, account-security addresses for the platforms you depend on, and anything that bills you.

```yaml
protected_senders:
  - "*@yourbank.com"
  - "*@gov.example"
  - "noreply-accounts@google.com"
  - "*@appleid.apple.com"
  - "billing@*"
```

Note the distinction the rules draw: a *security or billing event* from these senders is escalated to Action Needed, while *marketing* from the same institution falls through to normal handling. Your bank's fraud alert and your bank's weekly market newsletter are not the same thing.

---

## forwarding_addresses

Accounts that auto-forward into this inbox — an old university address, a former employer, a secondary account. Used to suppress duplicates when the same message also arrives directly.

```yaml
forwarding_addresses:
  - "you@university.example"
  - "you@oldcompany.example"
```

Leave empty if you don't forward anything.

---

## digest_senders

High-volume senders whose **subject lines already carry the payload**. Filed as FYI, presented as a scannable list, and their bodies are never opened.

This is the cost rule. Digest mail is the heaviest thing in most inboxes — job boards, listing sites, order and shipping notifications, alert feeds, score and result summaries. Opening a day's worth to extract what the subject line already said is where an agent like this gets expensive enough that you stop running it.

```yaml
digest_senders:
  - "jobalerts-noreply@linkedin.com"
  - "*@indeed.com"
  - "*@propertysite.example"
  - "noreply@shipping.example"
  - "alerts@*"
```

The test for adding a sender: **would reading the body tell you anything the subject line didn't?** If no, it belongs here. If you'd genuinely want the contents summarised, it's a newsletter — put it in the section below.

Leave empty if nothing in your inbox fits. The rule simply won't fire.

## newsletter_senders

Subscriptions you actually chose. Filed as FYI and collapsed to one line, rather than treated as promotional junk.

```yaml
newsletter_senders:
  - "*@nytimes.com"
  - "*@substack.com"
  - "noreply@medium.com"
```

---

## voice_profile

Used when preparing draft replies. Leave empty and the skill will derive a profile from your sent mail on first use, which is usually better than describing yourself.

If you do fill it in, record observable habits rather than aspirations — how you *actually* write, including the parts you might edit out.

```yaml
voice_profile:
  greeting: "Hi <first name>,"
  opener: "usually a short warmth line before the ask"
  paragraphs: "1-3 sentences, blank line between"
  sign_off: "Best, <first name>"
  languages: "English for professional mail, <other> for personal"
  quirks: "mixes contractions with full forms; no em-dashes"
```

---

## timezone

**Set this.** Your calendar's configured timezone may not match where you live, and an unpinned event lands at the wrong hour.

```yaml
timezone: "Europe/Istanbul"
```

If your Google Calendar's own timezone setting differs from this value, the skill will flag it. Events will still be stored correctly, but the calendar *grid* renders in its own timezone, so times will read an hour or two off. Fix that in Google Calendar → Settings → Time zone.

---

## schedule

Optional. Only affects guidance the skill gives about when to run.

```yaml
schedule: "09:00 daily"
```

Pick the hour by watching your own inbox rather than by instinct. Bulk mail usually lands overnight; real people write during their working day, which may be in a different timezone from yours. A run timed to catch the overnight pile is doing the job — you'll see the mail that arrives while you're at your desk without any help.

---

## overrides

**You do not fill this in by hand.** It starts empty and grows as you correct the agent.

When a classification is wrong, say so in chat. The agent applies the fix immediately and asks whether to make it permanent. If you say yes, it writes a line here — and every run after that respects it.

This is the difference between an agent you correct daily and one that learns your inbox once.

```yaml
overrides:
  - sender: "noreply@medium.com"
    bucket: "Low-Value"
    added: "2026-01-15"
    note: "algorithmic digest, not curated editorial"

  - sender: "*@hiringplatform.example"
    subject_contains: "has messaged you"
    bucket: "Action Needed"
    added: "2026-01-18"
    note: "real person waiting, answered on their platform not by email"

  - sender: "*@hiringplatform.example"
    bucket: "Action Needed"
    prepare_draft: true
    added: "2026-01-18"
    note: "named signer, replies by email"
```

Three things to know about how these are read:

- **Overrides are checked before every other rule.** They outrank the classification rules entirely.
- **More specific wins.** An entry with `subject_contains` is matched before a bare sender entry from the same domain, so one sender can route two ways.
- **`prepare_draft: true`** forces a draft even when the message carries no explicit question.

The one case the agent will stop and ask about: an override that would send a `protected_senders` match to Low-Value. It will confirm once before writing that, then honour it.

To undo an entry, say so in chat — "stop treating Medium as Low-Value" — or delete the line yourself.
