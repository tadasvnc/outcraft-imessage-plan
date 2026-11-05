# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

In short: iMessage could win more replies from iPhone shoppers, but it costs 20× more than SMS and Apple can ban a line. So we test before we build. First we count how many shoppers are on iMessage, and stop if there are too few. Then a small module sends iMessages in 3 stores, answers replies with the cart link, and counts who buys. Unless a line gets flagged or shoppers opt out more than on SMS, it keeps running, and in week 7 we decide on recovered carts: 2 more in 100 if lines run at full speed, about 6 if they're held back. Only then do we build it properly.

## 1. Provider: which one, and why

- **Today:** no provider yet. Five compared: Blooio, Linq, Sendblue, Photon, own Macs.
- **Suggestion: Blooio** for the test and the pilot, with a data agreement and minimal shopper data.
- **Why Blooio:** it refuses sends that break its rules (a link before a reply, too many unanswered messages), so a bug in our code costs one message, not a banned line. Self-serve, known price.
- **Not Linq yet:** it only warns, leaving pausing to us. Plan B if replies stay under 40%.
- **Not Sendblue:** it sells its own iMessage cart recovery to stores, and had a 2026 breach.

## 2. Risks: what breaks at volume, and what we do

| Risk | Why it matters: what breaks | Suggestion: what we do |
|---|---|---|
| **Apple blocks a line** | Junk reports or bursts block the number. Its chats drop. | New chats ramp 10 → 25 → 50 a day. Consent only, no nights. |
| **Two numbers** | SMS and iMessage share one app. A second store number looks like a scam. | One number per shopper. A chat moved to SMS says "same store". |
| **No link before a reply** | iPhones disable links from strangers. | A short question first, the link after a reply. |
| **Few replies** | One-way lines look like bots. Under 40% replies, Blooio holds a line to 10 new chats a day. | The first message asks a question. Under 40%: 10 a day, or Linq. |
| **One bad store** | It can get our whole Blooio account closed. | Admins check what a store sells first. If its line goes wrong, it switches off automatically. |
| **STOP and state law** | No carrier catches STOP. Some states limit hours. | We catch STOP; SMS law checks apply. |
| **Cost** | $0.20 a lead, against $0.01 for SMS. | It pays off for us at about 15 more carts per 1,000 leads; 4× that if a line is held to 10 a day. |

### Edge cases: where iMessage needs other rules than today

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Channel check** | Lead data shows which channels work: phone, email. | Before the AI writes, also check: is the number on iMessage? Reuse today's check if there is one. | Only a lookup shows iMessage. |
| **Message writing** | AI writes, scheduler sends. | Passed the check → an iMessage version too, next to the usual message. | A free slot is known only at send time. The campaign sets the fallback, so all are written early. |
| **Follow-ups and calls** | Steps and calls run on schedule. | A reply or purchase cancels them. No calls mid-chat. | No "still thinking?" after "yes, buying". |

## 3. Line health: how the team learns without a dashboard

- **Suggestion:** every 15 minutes a job checks each line. Weak (low delivery, many opt-outs): half the new chats, Slack alert. Blocked: no new chats, Slack alert, on-call replaces it within a day.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** each campaign switches its channels on: calls, SMS, email, WhatsApp.
- **Suggestion:** our admins only in v1. Not every store in its settings yet: a line needs warm-up and a content check, and one bad store can get us all banned. Not tied to a plan yet: the pilot first shows what it's worth. Later, a paid add-on per line.

## 5. Which leads get it, and what happens to the rest

- **Suggestion:** only leads whose number is on iMessage, looked up where it is on.
- **One number per lead:** SMS and iMessage share one app. So with iMessage on, a campaign's text steps are iMessage steps, each with a fallback (SMS or a call).
- **The rest** (Android, no consent, no free slot): the campaign as usual. We log why.

## 6. First version and time

- **Suggestion: test first, build after.**
  1. **Week 1:** count shoppers on iMessage, and stop if there are too few. Sign Blooio, give stores the required 10 days' notice, build a small test (3 days).
  2. **Day 11:** the test goes live in 3 stores. Shoppers get one question; a reply gets the cart link. We count who replies and who buys.
  3. **Week 3:** stop if a line gets flagged or shoppers opt out more than on SMS. Otherwise keep going.
  4. **Week 7:** decide. Go if iMessage recovers 2 more carts per 100. Then build the full version: 4 weeks, one engineer.
- **Cost until the decision:** 3 Blooio lines, about $1,730.
