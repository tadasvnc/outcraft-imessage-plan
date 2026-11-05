# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

In short: iMessage could win more replies from iPhone shoppers, but it costs 20× more than SMS and Apple can ban a line. So we test before we build: a small script sends iMessages in 3 stores, and support answers by hand. If replies double, one engineer spends a week adding the must-haves (automatic STOP, counting recovered carts), and we run it 3 more weeks. In week 8 we decide whether it recovers 2 more carts in 100. Only then do we build it properly.

## 1. Provider: which one, and why

- **Today:** no provider yet. Five compared in appendix A: Blooio, Linq, Sendblue, Photon, own Macs.
- **Suggestion: Blooio** for step 0 and the pilot, with a data agreement and minimal shopper data.
- **Why Blooio:** it refuses sends that break its rules (a link before a reply, too many unanswered messages), so a bug in our code costs one message, not a banned line. Self-serve, known price.
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
  - **Week 1, count:** carts a day with consent, times each store's iPhone share. Pick 3 stores. Contract, data agreement, vendor notice.
  - **Weeks 2–3, test:** a throwaway script sends a fixed iMessage to random iPhone carts in 3 stores, a line each. Their campaign texts pause; the rest get the campaign as usual. Support answers by hand. The lines warm up meanwhile.
  - **End of week 3:** 2× replies, no flag → go on. Flagged → retry slower, after missed calls. Else stop.
  - **Week 4, one week of build:** automatic STOP, recovered carts counted per group, Slack alerts.
  - **Weeks 5–7, pilot:** the same setup on warm lines, until about 2,560 shoppers have had iMessage. Decision in week 8.
- **After a go, the full build (4 weeks, one engineer), in order:** replies (diagram C): webhooks, AI answers and STOP, live before the first send. Then sending (diagram B): channel check, versions, line limits, alerts.
- **Why this order:** nothing is built before shoppers reply, and only one week before we know they buy. Lines: 3, about $1,730 up to the decision.
- **Why 8 weeks:** the vendor notice (10 days), the warm-up (3 weeks) and the pilot (3 weeks) set it, not the build.
- **Blocks us:** data agreement, vendor notice, consent. **Could slip:** bans, stores with too few leads.
- **What waits:** more lines per store, self-serve and billing, a dashboard (Slack does it), photos, a second provider, countries beyond US/CA.
- **Go or no-go**, against iPhone shoppers without iMessage: twice the replies in step 0, then 2 more carts recovered per 100 leads in the pilot (12, not 10).
- **Questions:** Is iMessage live? SMS consent enough? One send path? Does the AI write ahead? A channel check before writing?
