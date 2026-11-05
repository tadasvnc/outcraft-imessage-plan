# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

In short: iMessage could win more replies from iPhone shoppers, but it costs 20× more than SMS and Apple can ban a line. So we test before we build. First we count how many shoppers are on iMessage, and stop if there are too few. Then a small module sends iMessages in 3 stores, answers replies with the cart link, and counts who buys. Unless a line gets flagged or shoppers opt out more than on SMS, it keeps running, and in week 7 we decide on recovered carts: 2 more in 100 if lines run at full speed, about 6 if they're held back. Only then do we build it properly.

## 1. Provider: which one, and why

- **Today:** no provider yet. Five compared in appendix A: Blooio, Linq, Sendblue, Photon, own Macs.
- **Suggestion: Blooio** for the test and the pilot, with a data agreement and minimal shopper data.
- **Why Blooio:** it refuses sends that break its rules (a link before a reply, too many unanswered messages), so a bug in our code costs one message, not a banned line. Self-serve, known price.
- **Not Linq yet:** it only warns and leaves pausing to us. Plan B if replies stay under 40%, since it doesn't hold lines back. We compare again at scale.
- **Not Sendblue:** it competes with us, selling iMessage cart recovery to stores. And a 2026 breach.

## 2. Risks: what breaks at volume, and what we do

| Risk | Why it matters: what breaks | Suggestion: what we do |
|---|---|---|
| **Apple blocks a line** | Junk reports or bursts block the number. Its chats drop. | New chats ramp 10 → 25 → 50 a day. Consent only, no nights. |
| **Not enough lines** | A line starts about 50 new chats a day. Carts go cold in an hour. | One line per store in v1. Full → the fallback at once, no waiting. |
| **Failed iMessage** | A failed message waits while the cart goes cold. | Fallback at once. No delivery status → no resend; next steps as usual. |
| **No link before a reply** | iPhones disable links from strangers. | A short question first, the link after a reply. |
| **Lead not on iMessage** | About 4 in 10 US phones are Android. | We look it up first. Not on iMessage → the campaign as usual. |
| **Few replies** | One-way lines look like bots. Under 40% replies, Blooio holds a line to 10 new chats a day. | The first message asks a question. Under 40%: 10 a day, or Linq. |
| **One bad store** | It can get our whole Blooio account closed. | Admins check what a store sells first. If its line goes wrong, it switches off automatically. |
| **STOP and state law** | No carrier catches STOP. Some states limit hours. | We catch STOP; SMS law checks apply. |
| **Cost** | $0.20 a lead, against $0.01 for SMS. | It pays off for us at about 15 more carts per 1,000 leads; 4× that if a line is held to 10 a day. |

### Edge cases: where iMessage needs other rules than today

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Sending hours** | Every step waits for sending hours. | Reply at once to a lead who just wrote, within the law. | Live chat: a next-morning answer loses the lead. |
| **Reply limit** | After 5 replies: a person. | Keep the chat going, up to 30 AI replies, then a person. | Going silent mid-chat loses the lead. |
| **Channel check** | Lead data shows which channels work: a phone number, an email. | One more check before the AI writes: is the number on iMessage? Reuse today's check if there is one. | Only a lookup shows iMessage, and the AI must know whether to write for it. |
| **Message writing** | AI writes, scheduler sends. | Passed the check → an iMessage version too, next to the campaign's usual message (none for a call). | A free slot is known only at send time. The campaign sets the fallback, so all are written early: no AI call when sending. |
| **Sender number** | Each store's own SMS number. | With iMessage on, every text step is an iMessage step from one line, until a fallback. | Same app as SMS. Two numbers look like a scam. |
| **Follow-ups and calls** | Steps and calls run on schedule. | A reply or purchase cancels them. No calls mid-chat. | No "still thinking?" after "yes, buying". |

## 3. Line health: how the team learns without a dashboard

- **Today:** no iMessage lines to watch. For SMS, carriers and Twilio handle number health.
- **Suggestion:** every 15 minutes a job reads the provider's warnings and our numbers, and acts:
  - **Yellow** (delivery under 90% or opt-outs over 2%): halve the daily limit, alert Slack.
  - **Red** (blocked, or delivery under 70%): no new leads, its chats go to the fallback, alert Slack.
- **Why:** the system acts first and Slack tells the team. On-call only replaces red lines, within a day.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** each campaign switches its channels on: calls, SMS, email, WhatsApp.
- **Suggestion:** our admins only in v1, one line per store, paid by us. Later, a paid add-on per line.
- **Why:** lines cost money and need warm-up, and one bad store can get us all banned.

## 5. Which leads get it, and what happens to the rest

- **Today:** a lead is reached on every channel their campaign has switched on.
- **Suggestion:** leads on iMessage. Checked only where iMessage is on.
- **One number per lead:** SMS and iMessage share one app. So with iMessage on, a campaign's text steps are iMessage steps, each with a fallback (SMS or a call).
- **The rest** (Android, no consent, no free slot): the campaign as usual. We log why.
- **Why:** iMessage works only on iPhones and costs 20× more. The pilot compares iPhones only.

## 6. First version and time

- **Today:** iMessage isn't built yet (the site lists it: question 1).
- **Suggestion: test first, build after.**
  1. **Week 1:** count shoppers on iMessage; stop if a store has under ~30 a day. Sign Blooio, send the vendor notice, build a small test module (~3 days).
  2. **Day 11, live in 3 stores:** a fixed question; a reply gets the cart link and the store's email; STOP opts out automatically. Their SMS pauses. Replies and purchases are counted per group.
  3. **Week 3 checkpoint:** stop only if a line is flagged or opt-outs beat SMS. Replies under 40% → lines held at 10 a day, or Linq.
  4. **Week 7 decision:** go if iMessage recovers 2 more carts per 100 (about 6 if lines are held back).
  5. **After a go:** the full build, 4 weeks, one engineer: AI chat, fallbacks, health job (diagram C, then B).
- **Why 7 weeks:** the vendor notice (10 days) and the warm-up (3 weeks) set the pace, not the build. Lines: 3, about $1,730.
- **Blocks us:** data agreement, vendor notice, consent. **Could slip:** bans, too few shoppers, a move to Linq.
- **Questions:** Is iMessage live? SMS consent enough? Recovered carts tracked per shopper? One send path?
