# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

**In short:** iMessage becomes a fifth channel, for iPhone leads. The rest stay on the other channels.

- **Step 0:** count reachable shoppers, test one line. **Then build:** 4 weeks, one engineer.
- **Pilot**, 2–3 US stores: half the iPhone shoppers get iMessage, half don't. Go or no-go in week 12.

## 1. Provider: which one, and why

- **Today:** no provider yet.
- **Compared** on price, limits, health data, security: Blooio, Linq, Sendblue, Photon, own Macs.
- **Suggestion: Blooio**, with a data agreement and only the data a message needs. Plan B: Linq.
- **Why:** public prices ($195 a line at 6+), live in days, hard limits that catch our mistakes.
- **Not Sendblue:** it competes with us, selling iMessage cart recovery to stores. And a 2026 breach.

## 2. Risks: what breaks at volume, and what we do

| Risk | What breaks | What we do |
|---|---|---|
| **Apple blocks a line** | Junk reports or bursts block the number. Its chats drop. | New chats ramp 10 → 25 → 50 a day. Consent only, no nights. |
| **Not enough lines** | A line starts about 50 new chats a day. Carts go cold in an hour. | Lines full → the SMS version at once. About 1 line per 40 leads a day. |
| **Failed iMessage** | A failed message waits while the cart goes cold. | The SMS version goes at once. No receipt → no resend; the next step goes by SMS. |
| **No link before a reply** | iPhones disable links from strangers. | One short question first. The link after a reply, or on the next channel. |
| **Lead not on iMessage** | About 4 in 10 US phones are Android. | We look it up first. Not on iMessage → the next channel, usually SMS. |
| **Few replies** | One-way lines look like bots. | The first message asks a question. Replies halve → no new chats. |
| **One bad store** | It can get our account closed. | Admin check, a sub-account per store. |
| **STOP and state law** | No carrier catches STOP. Some states limit hours. | We catch STOP ourselves and use the SMS law checks. |
| **Cost** | $0.20 a lead, against $0.01 for SMS. | It pays off for us at about 15 more carts per 1,000 leads. |

### Edge cases: where iMessage needs other rules than today

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Sending hours** | Every step waits for sending hours. | Reply at once to a lead who just wrote, within the law. | Live chat: a next-morning answer loses the lead. |
| **Reply limit** | After 5 replies: a person. | Keep the chat going, up to 30 AI replies, then a person. | Going silent mid-chat loses the lead. |
| **Message writing** | The AI writes, then the scheduler sends. | If the lead can get iMessage, the AI writes two versions: iMessage and SMS. | A free line is only known at send time. With both ready, no AI call when sending. |
| **Sender number** | Each store's own SMS number. | Every text step from one line. Changed only on fallback. | Same app as SMS. Two numbers look like a scam. |
| **Follow-ups and calls** | Steps and calls run on schedule. | A reply or purchase cancels them. No calls mid-chat. | No "still thinking?" after "yes, buying". |

## 3. Line health: how the team learns without a dashboard

- **Today:** there are no iMessage lines yet, so there is nothing to watch.
- **Suggestion:** every 15 minutes a job reads the provider's warnings and our numbers, and acts:
  - **Yellow** (delivery under 90% or opt-outs over 2%): halve the daily limit, alert Slack.
  - **Red** (blocked, or delivery under 70%): no new leads, chats to SMS, alert Slack and the store.
- **Why:** the system acts first and Slack tells the team. On-call only replaces red lines, within a day.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** each campaign switches its channels on: calls, SMS, email, WhatsApp.
- **Suggestion:** only our admins in v1, and we pay for pilot lines. Later, a paid add-on per line.
- **Why:** lines cost money and need warm-up, and one bad store can get us all banned.

## 5. Which leads get it, and what happens to the rest

- **Today:** a lead is reached on every channel their campaign has switched on.
- **Suggestion:** only leads with a US/CA number, consent and iMessage (we check first).
- **One number per lead:** in the campaign, iMessage is a fifth channel. On the phone, it shares the Messages app with SMS, so a lead hears from one number at a time.
- **The rest** (Android, no consent, no free line): the next channel, usually SMS. We log why.
- **Why:** iMessage works only on iPhones and costs 20× more. The pilot compares iPhones only.

## 6. First version and time

- **Step 0, before building (2–3 weeks, ~$300):** a cheap test of the two biggest unknowns.
  - **Count:** carts a day with consent and US/CA number, and iPhone share (lookup on 500).
  - **Test one line, one store:** a throwaway script sends a fixed iMessage to half the iPhone carts; the other half get SMS. Replies go to Slack, and support answers (about 30 min a day).
  - **Result:** 2× replies, no flag → build. Flagged → plan B: shoppers write first. Too few → stop.
- **In v1, in build order:** replies (diagram C): webhooks, AI answers and STOP, live before the first send. Then sending (diagram B): iMessage check, two versions, line limits, alerts.
- **How long:** 4 weeks of building for one engineer, 12 weeks from step 0 to the decision.
- **Blocks us:** data agreement, 10 days' notice to stores for a new vendor, legal on consent. **Could slip:** a ban, few replies, SMS-only code.
- **Where we simplified, what waits:** self-serve and billing (needs pilot data), a dashboard (Slack does it), photos, a second provider, countries beyond US/CA.
- **Go or no-go**, against the half without iMessage: step 0 needs twice the replies. The pilot needs 2 more carts recovered per 100 leads, say 12 instead of 10.
- **Questions:** Is iMessage live? Does SMS consent cover it? Do channels share one send path? Does the AI write a step ahead?
