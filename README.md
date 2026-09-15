# Gmail Triage Agent

A Claude plugin that sorts a Gmail inbox into three buckets each morning, flags anything that belongs on your calendar, and prepares reply drafts — without sending, deleting, or scheduling anything on its own.

## What it does

Every run scans the previous day's mail and files each thread into exactly one bucket:

| Bucket | Meaning |
|---|---|
| **Action Needed** | Someone is waiting, a deadline is live, or something happened on one of your accounts |
| **FYI** | Worth knowing, nothing required — digests, newsletters, notifications |
| **Low-Value** | Promotions, marketing, platform noise |

The buckets become **nested Gmail labels**, which is the point: a chat summary can't be searched or previewed, but a Gmail label can. You read the report to decide, then browse the label to actually look at things.

It also:

- **Flags calendar-worthy mail** and asks before creating any event
- **Prepares reply drafts in your voice**, saved as drafts only
- **Lists deletion candidates** and waits for your approval
- **Keeps your corrections.** Tell it a call was wrong and it offers to make the fix permanent

## What it will not do

These are enforced in the skill, not left to judgment:

- **Never sends email.** Drafts only.
- **Never deletes without confirmation**, on a list you've seen.
- **Never creates a calendar event without confirmation.**
- **Ambiguity defaults to FYI**, never Low-Value. A misfiled FYI costs a glance; a wrongly binned email can cost an opportunity.

## Install

Add the marketplace and install:

```
/plugin marketplace add ilgindenizkuran-rgb/gmail-triage-agent
/plugin install gmail-triage-agent
```

Requires the **Gmail** and **Google Calendar** connectors. Enable both before the first run.

## Set up

Run it. It reads 30 days of your inbox first — senders, subjects, frequency, no bodies — and drafts the config itself: what must never be buried, what forwards in from another address, which senders you only skim, which newsletters you actually chose. Then it shows you each list with a reason per line and you approve or strike entries. Two things it asks outright, both with a suggested answer: your real timezone, and what hour to run.

You are confirming, not composing. Nobody remembers which address sends their bank statements.

You can edit `skills/gmail-triage/references/config.md` by hand instead. Nothing stops you. But the file exists to be filled in, not to be a prerequisite.

The config will be incomplete on day one regardless — a 30-day scan can't see a sender who writes quarterly. That gap closes by correcting it, not by getting setup perfect.

The section that matters most is `protected_senders` — your bank, government services, account security, anything that bills you. Nothing matching it can ever be classified Low-Value or land on a deletion list. Get the other lists wrong and you get noise; get this one wrong and you can lose something real.

## Correcting it

Say so in chat. "This should have been Action Needed." "Stop showing me these." "Why no draft for that one?"

It fixes the label, then asks whether to keep the rule. If you say yes it writes an entry to `overrides` in your config, and every run after that respects it — including the awkward cases, like one sender whose notification emails need your attention and whose marketing doesn't.

Expect to do this a handful of times in the first two weeks and then rarely. That's the design: corrections are supposed to accumulate, not repeat.

## Use

On demand:

```
/gmail-triage
```

Or as a scheduled task, which is what it's designed for. Pick the hour by watching your own inbox: bulk mail tends to land overnight, while real people write during *their* working day — which may be a timezone or two from yours. A run timed to sweep the overnight pile is doing the job. Mail that arrives while you're at your desk doesn't need an agent to tell you about it.

## Design notes

Three decisions shaped this more than the rest, and they're the interesting part.

**Digest mail is never opened.** Job boards, listing sites, delivery notifications — this category routinely runs 150–270KB of HTML per message, and reading a day's worth can cost ten to twenty times the entire rest of the run. To extract what? A company and a title, a tracking status, three addresses and their prices. The subject line already gave you all of it. So the rule reads subjects and stops. This single constraint is most of why the agent is cheap enough to run every day, which is the only schedule at which triage is worth anything.

**Ambiguity is not allowed to earn deletion.** The rules are ordered, first-match-wins, with a protected-sender class that overrides everything below it. Anything the classifier is unsure about goes to FYI. The asymmetry is deliberate: the cost of over-filing is a wasted glance, and the cost of under-filing is unbounded.

**Labels are navigation, not bookkeeping.** Early versions treated the chat report as the deliverable. It isn't — you can't search it, preview a message from it, or come back to it in the afternoon. The report is where you decide; Gmail is where you read. It also means you can file something yourself: star it, or apply the Action Needed label by hand, and the next run respects it.

**A correction is worth more than a rule.** The generic ruleset is a starting position, not an answer — every inbox disagrees with it somewhere. So the interesting question isn't how good the rules are out of the box, it's whether disagreeing with them costs you anything. Corrections are written to config and outrank every rule below them, which means the second week is measurably better than the first without anyone editing a file.

**A star is an instruction, not a hint.** The starred net has no "does this really need action?" filter, unlike every other net. Evaluating whether the user was right to star something isn't triage, it's second-guessing — and it would break the one lever they have for overriding the classifier during the day.

## Limitations

- **Gmail only.** The classification logic would port, the tooling wouldn't.
- **One run sees one day.** Your corrections persist, but the mail itself doesn't — it has no memory of what you dealt with yesterday unless that's recorded in a label.
- **Gmail's auto-`IMPORTANT` flag is close to useless** as a signal in a busy inbox — it lands on retail promotions constantly. Manual stars are treated as the stronger signal.
- **First-week accuracy depends entirely on config.** The two failure modes to watch: something you wanted buried in the FYI digest, and something in Low-Value that should never have been there. Both are worth correcting out loud on the day you notice them.
- **Overrides are per-sender, not conceptual.** It can learn that a given address is noise. It can't learn that you've lost interest in a topic.

## License

MIT — see [LICENSE](LICENSE).
