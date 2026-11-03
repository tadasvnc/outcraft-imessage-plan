# iMessage as a new channel: plan

Tadas Vinckas · Outcraft engineering task · Diagrams: [diagrams.md](diagrams.md)

**In short:** a 6-week pilot for 2–3 US customers. Our admins turn it on. It goes only to opted-in leads we have confirmed are on iMessage, and each lead keeps one number for the whole chat. Everyone else gets SMS, as today. We scale only if iMessage beats SMS on revenue after line costs.

**Why it's hard:** Apple has no API for a business to message first. Providers run real Apple devices, and each "line" is one real number. People can tap "Report junk", and Apple blocks lines that look like spam. A line can safely start about **50 new chats a day**. Our campaigns today do the opposite: a link in the first message, high volume, any number from the pool.

**Should we build it?** Yes, but small and reversible. Apple's iCloud terms forbid automated and promotional use, and Apple has cut off unofficial iMessage access before (Beeper Mini, 2023). So iMessage must never be the only path. SMS fallback is everywhere, a global kill switch exists, and we never sell it as guaranteed. If time is tight, we run a manual pilot first: one line and one customer, driven from the provider's dashboard, to measure the lift before building.

**Assumptions** (to confirm with the team):
- the AI writes a step's message when it is due;
- each customer has its own SMS number;
- consent is stored per lead;
- channels share one send path.

## 1. Provider
**Today:** SMS and WhatsApp go through their own providers. iMessage has none.

| | Sendblue | Blooio |
|---|---|---|
| Outbound price | Price on request (the $100 plan is inbound only) | $289/line, $195 at 6+ lines |
| Limits | Published: 50 new/day, 15/hour, with an error code for each rule | "20–50 a day" guidance |
| Security, age | SOC 2, HIPAA, 5+ years | Not stated, about 1 year |
| iMessage lookup | Yes, 100–200 per line per day | Listed, limits unknown |

**Suggestion:** Sendblue, once a sales call confirms the price, how long new lines take, and how a blocked line is replaced. We build behind our own provider interface, so Blooio is a one-week switch.
**Why:** the limits are written down and returned as error codes, so we build against real rules, not guesses. We hold many customers' data, so SOC 2 matters. We skip the provider's SMS fallback, because our own SMS channel already handles consent and STOP.
**Also compared:**
- Linq and Photon: the same model.
- Our own Mac minis: rejected. The bans and the hardware would be ours, and one AI company was banned within half a day of launch.
- Apple Messages for Business: rejected. The shopper must write first, so it can't recover carts.
- RCS: official and branded, but it is not iMessage. Worth a separate look.

## 2. Risks at volume

| Risk | What breaks | What we do |
|---|---|---|
| **No link before a reply** | Until the lead replies, Sendblue blocks links and media, and limits messages to 300 characters and 6 in total. Our cart message leads with the checkout link. | The first iMessage is a short question with no link. The link comes after the reply. After 2 messages with no reply, the next step goes by SMS with the link. |
| **Apple blocks a line** | Junk reports, bursts and night sends block a line, and every open chat on it dies too. | Warm up each line at 10 → 25 → 50 new chats a day, at most 15 an hour, spread out. Consented leads only, no night sends, and a contact card after the first reply. Lines belong to one customer and are never shared. |
| **Low reply rate** | A line that only sends and rarely gets replies looks like a bot to Apple. | Track the send-to-receive ratio per line. When it drops, the line stops starting new chats. |
| **One customer's content** | If one brand sends adult, drug or scam content, the provider can close our *whole* account with no refund. | Admins check what the brand sells before turning iMessage on (no prescription drugs, adult, cannabis, gambling or finance). Every AI message is checked before sending. We ask for a sub-account per customer and a kill switch per customer. |
| **Apple ends the channel** | Apple blocks the provider's method for everyone, overnight. | Leads fall back to SMS automatically. We lose the blue bubble, not the leads. |
| **Not enough lines** | Carts go cold within an hour, and 50 a day fills up fast. | If no slot is free within 60 min, that step goes by SMS. Lines needed = eligible leads a day ÷ 40. For example, 300 carts at 55% iPhone needs 4–5 lines. |
| **Lead not on iMessage** | Android, iMessage turned off, numbers outside the US. | Look the number up before the first send and cache the result for 30 days. Update the cache if a message is downgraded. v1 covers US/CA only. |
| **Cost** | $195–$290 a line a month for about 1,200–1,500 new leads, so $0.13–$0.25 a lead. SMS is about $0.01. Long chats also cost more in AI calls. | The pilot splits leads 50/50 against SMS. A line pays for itself if it recovers just 0.2% more carts. |
| **Opt-out, webhooks** | iMessage has no carrier STOP. Webhooks can repeat, arrive late or arrive out of order. | We detect STOP ourselves and opt the lead out on all channels. Webhooks: check the signature, skip events we've seen, save then queue. A status stuck for 15 min is checked through the API. |

| Topic | Today | Suggestion | Why |
|---|---|---|---|
| Sending hours | Every step waits | A reply to a lead active in the last 30 min goes out now. A first contact still waits. | It's a live chat, but a 3am cold message looks like a bot |
| Reply limit | 5, then a person | Keep chatting, hard cap of 30 | Going silent mid-chat loses the lead |
| Sender number | Any from the pool | The same line for the whole chat | A new number reads as a stranger and gets "Report junk" |
| Follow-ups | Run on schedule | A reply cancels the pending steps | A reminder after "yes, buying" looks broken |
| Message bursts | Each message wakes the AI | Wait 15 s, then answer once | People send 3 short texts in a row |

## 3. Line health without a dashboard
**Today:** no per-number health checks. **Suggestion:** a job runs every 15 minutes and checks each line for:
- the provider's own "flagged" status;
- blocked or declined errors;
- delivery rate over the last 100 sends;
- failures in a row;
- the send-to-receive ratio;
- opt-out rate;
- days with no sends (Apple turns off idle lines after about 2 months).

**Yellow** (delivery under 90%, or opt-outs above 2%): the daily limit halves and Slack gets an alert. **Red** (flagged, 5 failures in a row, or delivery under 70%): the line takes no new leads, its chats move to SMS, and Slack and the customer are alerted. The ops on-call replaces a red line within one working day. A weekly Slack summary catches slow decline. **Why:** the system acts first and tells people what it did. All thresholds are starting values, tuned during the pilot.

## 4. Who turns it on
**Today:** customers switch channels on themselves. **Suggestion:** only our admins in v1, per customer. After the pilot, a paid add-on per line. **Why:** each line is a fixed monthly cost and needs warm-up. A bad lead list can get a line blocked in a day, and one brand selling the wrong thing can get our whole provider account closed. We check each early customer's consent, lead source and products first. Pricing per line matches our real cost.

## 5. Which leads get it
**Today:** every lead goes through every channel the campaign has switched on. **Suggestion:** only leads that meet all of these:
- a US/CA number and marketing consent;
- confirmed on iMessage;
- not opted out;
- not already in an SMS chat (we don't switch channels mid-conversation);
- a free slot on one of the customer's lines.

In the pilot, half of these get iMessage and half get SMS. **The rest** go to the campaign's next channel, so no lead is dropped. We log why each one skipped iMessage, so we see how many we lose and why.

## 6. First version and time
**In:**
- one provider behind our interface, lines added by admins;
- lookup with a cache;
- per-line limits, with each lead keeping its line;
- the no-link rules;
- status and reply webhooks;
- the chat rules above;
- opt-out and the content check;
- health alerts;
- SMS fallback and kill switches.

**Waits, and why:**
- **Self-serve and billing:** needs pilot numbers first.
- **Moving a chat to a new line:** SMS covers it in v1.
- **Pictures, reactions, typing dots:** not needed to win a reply.
- **Group chats:** carts are one-to-one.
- **A second provider:** the interface keeps the door open.
- **A dashboard:** alerts do the job.
- **Countries beyond US/CA:** iPhone share is lower and lines are US numbers.

**Next, if the pilot works:**
- let leads write first (a tap-to-text link in SMS, email or a call), which means no junk banner and lower risk;
- use iMessage for warm segments (upsell, win-back, subscriptions);
- RCS as the official path.

| Week | Work |
|---|---|
| 0 | Provider call, contract and 2 lines (alongside week 1) |
| 1 | Provider interface, lookup and cache, send queue with limits |
| 2 | Webhooks, opt-out, replies to the AI, chat rules. Line warm-up starts. |
| 3 | Health alerts, content check, SMS fallback, tests on our phones |
| 4–6 | Pilot with 2–3 customers against SMS |

**How long:** 3–5 weeks of build for one backend engineer, depending on how channel-neutral the code is today. Then a 3-week pilot.
**Blocks us:** the provider contract for outbound sending. Everything else runs in parallel.
**Where I simplified:**
- a blocked line falls back to SMS instead of moving the chat;
- admin-only;
- one provider;
- US/CA only;
- alerts instead of a dashboard.

**Could slip if:**
- the provider is slow;
- line setup takes time;
- a line is banned during warm-up;
- SMS is wired into more places than expected.

**Go/no-go after the pilot** (starting values, to agree with the team):
- reply rate at least 2× SMS;
- revenue per lead after line costs above SMS;
- opt-outs no higher than SMS;
- under 1 blocked line per 10 line-months.

**Questions for the team:** Is iMessage already live somewhere, and what did we learn? Is there a common channel interface? Where is consent stored? Have we looked at RCS?
