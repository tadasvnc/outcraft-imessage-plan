# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

**In short:** iMessage becomes a fifth channel, for iPhone leads. The rest stay on the other channels.

- **Step 0:** count reachable shoppers, test one line. **Then build:** 4–5 weeks, one engineer.
- **Pilot**, 3 US stores, a line each: random iPhone shoppers get iMessage. Decision in week 13.

## 1. Provider: which one, and why

- **Today:** no provider yet. Five compared in appendix A: Blooio, Linq, Sendblue, Photon, own Macs.
- **Suggestion: Blooio** for step 0 and the pilot, with a data agreement and minimal shopper data.
- **Why Blooio:** it enforces clear rules (no early links, few unanswered messages), so our mistakes can't get a line banned. Self-serve, known price.
- **Not Linq yet:** it only warns and leaves pausing to us. It's bigger, so we compare again at scale.
- **Not Sendblue:** it competes with us, selling iMessage cart recovery to stores. And a 2026 breach.

## 2. Risks: what breaks at volume, and what we do

- **Today:** SMS carriers handle STOP and spam. iMessage has none, so these risks are ours.

| Risk | Why it matters: what breaks | Suggestion: what we do |
|---|---|---|
| **Apple blocks a line** | Junk reports or bursts block the number. Its chats drop. | New chats ramp 10 → 25 → 50 a day. Consent only, no nights. |
| **Not enough lines** | A line starts about 50 new chats a day. Carts go cold in an hour. | One line per store in v1. Full → the fallback at once, no waiting. |
| **Failed iMessage** | A failed message waits while the cart goes cold. | Fallback at once. No delivery status → no resend; next steps as usual. |
| **No link before a reply** | iPhones disable links from strangers. | A short question first, the link after a reply. |
| **Lead not on iMessage** | About 4 in 10 US phones are Android. | We look it up first. Not on iMessage → the campaign as usual. |
| **Few replies** | One-way lines look like bots. | The first message asks a question. Replies halve → no new chats. |
| **One bad store** | It can get our account closed. | Admin check, a sub-account per store. |
| **STOP and state law** | No carrier catches STOP. Some states limit hours. | We catch STOP; SMS law checks apply. |
| **Cost** | $0.20 a lead, against $0.01 for SMS. | It pays off for us at about 15 more carts per 1,000 leads. |

### Edge cases: where iMessage needs other rules than today

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Sending hours** | Every step waits for sending hours. | Reply at once to a lead who just wrote, within the law. | Live chat: a next-morning answer loses the lead. |
| **Reply limit** | After 5 replies: a person. | Keep the chat going, up to 30 AI replies, then a person. | Going silent mid-chat loses the lead. |
| **Message writing** | AI writes, scheduler sends. | iMessage check, then two versions: iMessage, fallback. | Line known at send time; both ready, no AI call. |
| **Sender number** | Each store's own SMS number. | Every text step from one line, until a fallback. | Same app as SMS. Two numbers look like a scam. |
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
- **One number per lead:** SMS and iMessage share one app, so we text from one number.
- **The rest** (Android, no consent, no free slot): the campaign as usual. We log why.
- **Why:** iMessage works only on iPhones and costs 20× more. The pilot compares iPhones only.

## 6. First version and time

- **Today:** iMessage isn't built yet (the site lists it: question 1).
- **Suggestion, step 0 before building (2–3 weeks, ~$300):**
  - **Count:** carts a day with consent and US/CA number, and iPhone share (lookup on 500).
  - **Test one line, one store:** a throwaway script sends a fixed iMessage to random iPhone carts, up to the line's cap; the rest get SMS. Support answers replies in Slack (30 min a day).
  - **Result:** 2× replies, no flag → build. Flagged → retry slower, after missed calls. Else stop.
- **Then v1, in build order:** replies (diagram C): webhooks, AI answers and STOP, live before the first send. Then sending (diagram B): iMessage check, two versions, line limits, alerts.
- **Why this order:** $300 tests the two big unknowns before we build. Replies go live first.
- **How long:** 4–5 weeks of building for one engineer, 13 weeks from step 0 to the decision.
- **Blocks us:** data agreement, vendor notice, consent. **Could slip:** bans, SMS-only code.
- **What waits:** more lines per store, self-serve and billing, a dashboard (Slack does it), photos, a second provider, countries beyond US/CA.
- **Go or no-go**, against iPhone shoppers without iMessage: twice the replies in step 0, then 2 more carts recovered per 100 leads in the pilot (12, not 10).
- **Questions:** Is iMessage live? SMS consent enough? One send path? Does the AI write ahead?
