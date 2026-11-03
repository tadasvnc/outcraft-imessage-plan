# iMessage as a new channel: plan

Tadas Vinckas · Outcraft engineering task · Diagrams: [diagrams.md](diagrams.md)

**In short:** a 6-week pilot for 2–3 US customers. Our admins turn it on. It goes only to opted-in leads we have confirmed are on iMessage, and each lead keeps one number for the whole chat. Everyone else gets SMS, as today. Before we scale, we test whether the extra replies pay for the lines.

**What makes iMessage different.** Apple has no API for a business to message first. Its own Messages for Business works only when the shopper writes first, so it can't recover carts. Providers like Sendblue and Blooio run real Apple devices. Each "line" is one real number. Apple lets people tap "Report junk", and too many reports or bot-like sending get the number blocked. So a line can take about **50 new leads a day**, and we can lose it overnight. Our campaigns today do the opposite: a link in the first message, high volume, any number from the pool.

## 1. Provider

| | Sendblue | Blooio |
|---|---|---|
| Outbound plan | "Blue Ocean", price on request. The $100 plan is inbound only. | $289/line, from $195 at 6+ lines |
| Limits | Published: 50 new contacts/line/day, 15/hour, error codes per rule | "20–50 a day" guidance only |
| iMessage lookup | Yes, 100–200 per line per day | Listed, limits unknown |
| Security | SOC 2, HIPAA | Not stated |
| Age | 5+ years | About 1 year |

**Suggestion: Sendblue, after one sales call confirms the price per line, how long lines take to set up, and how a blocked line is replaced. We build it behind our own provider interface, with Blooio as plan B.**
**Why:** its limits are written down and returned as error codes, so we build against real rules instead of guessing. We hold data for many customers, so SOC 2 counts. With the interface, switching costs about a week. We don't use the provider's SMS fallback. Our own SMS channel already handles consent and STOP.

## 2. Risks at volume

| Risk | What breaks | What we do |
|---|---|---|
| **No link before a reply** | Until the lead replies, Sendblue blocks links and media, and limits messages to 300 characters and 6 in total. Today our first cart message carries the checkout link. | On iMessage, the AI's first message is a short question with no link. The link goes after the lead replies. If there are 2 messages and no reply, the next step goes by SMS with the link. |
| **Apple blocks the line** | Junk reports, bursts, cold leads and night sends get a line blocked. Every open chat on it dies too. | New-contact limits per line: warm up 10 → 25 → 50 a day, at most 15 an hour, sends spread out. Consented leads only. No night sends. Send a contact card after the first reply. Lines belong to one customer and are never shared, so one bad list can't burn another customer's numbers. |
| **Not enough lines** | Carts go cold within the hour. 50 a day per line fills up fast. | If a lead can't get a slot within 60 min, that step goes by SMS. Size lines as eligible leads a day ÷ 40. For example, 300 carts a day at 55% on iPhone needs 4–5 lines. |
| **Lead not on iMessage** | Android, iMessage turned off, numbers outside the US. | Check with the lookup before the first send and cache the result for 30 days. If the provider reports the message as downgraded, we update the cache. v1 covers US/CA numbers only. |
| **Cost** | A line is a fixed $100–$290 a month for about 1,200–1,500 new leads, so $0.10–$0.30 per lead. SMS is about $0.01. Long chats add AI cost. | Pilot: half the eligible leads get iMessage, half get SMS. Compare revenue per lead after line cost. Scale only if iMessage wins. |
| **Reply rate** | More replies mean more AI calls and more handoffs to people. | A hard cap of 30 AI replies per chat. Summarise long chats. Alert if the handoff queue grows. |
| **Opt-out** | No carrier STOP on iMessage, but US consent law still applies. | We detect STOP-type replies ourselves and opt the lead out on all channels. |
| **Webhooks** | Duplicates, wrong order, outages | Check the signature, skip events we have seen, save then return 200, and process on a queue. A status stuck for 15 min gets checked through the API. |

| Topic | Today | Suggestion | Why |
|---|---|---|---|
| Sending hours | Every step waits | A reply to a lead active in the last 30 min goes out now. A first contact still waits. | It's a live chat, but a 3am cold message looks like a bot. |
| Reply limit | 5, then a person | Keep chatting, hard cap of 30 | Going silent mid-chat loses the lead |
| Sender number | Any number from the pool | The same line for the whole chat | A new number reads as a stranger and draws "Report junk" |
| Follow-ups | Run on schedule | A reply cancels the pending steps | A reminder after "yes, buying" looks broken |
| Message bursts | Each message wakes the AI | Wait 15 s, then answer once | People send 3 short texts in a row |

## 3. Line health without a dashboard
A job runs every 15 minutes and checks each line for:
- delivery rate over the last 100 sends;
- failures in a row;
- "blocked" or "declined" errors;
- the opt-out rate;
- reply rate against the line's own 7-day average;
- days with no sends (Apple turns off lines that stay idle for about 2 months).

**Yellow** (delivery under 90%, or opt-outs above 2%): the line's daily limit is halved and a Slack alert goes out. **Red** (a blocked error, 5 failures in a row, or delivery under 70%): the line takes no new leads, its open chats move to SMS, and we alert Slack and email the customer. A weekly Slack summary catches slow decline. The system acts first and tells the team what it did. A person only decides whether to replace the line.

## 4. Who turns it on
**Today:** customers switch channels on themselves. **Suggestion:** in v1, only our admins, per customer. After the pilot, a paid add-on per line. **Why:** every line is a fixed monthly cost and needs warm-up. One bad lead list can get it blocked in a day, so we want to see each early customer's consent and lead source first. A price per line matches what it costs us.

## 5. Which leads get it
Only leads that meet all of these: a US/CA number, marketing consent, confirmed on iMessage, not opted out, and a free slot on one of the customer's lines. During the pilot, half of these get iMessage and half get SMS. **The rest** go to the campaign's next channel, as today, so no lead is dropped. We log why each one skipped iMessage, so we know how many we lose to Android, missing consent or full lines.

## 6. First version and time
**In:**
- one provider behind our interface, with lines added by admins;
- lookup with a cache;
- a send queue with per-line limits, where each lead keeps its line;
- the no-link-before-reply rules;
- status and reply webhooks;
- the chat rules above;
- opt-out;
- health alerts;
- fallback to SMS.

**Waits:**
- self-serve and billing (needs pilot numbers);
- moving a chat to a new line (v1 falls back to SMS);
- pictures, reactions and typing dots (not needed to win a reply);
- group chats;
- a second provider;
- a dashboard (alerts do the job);
- countries beyond US/CA.

| Week | Work |
|---|---|
| 0 | Provider call and contract, order 2 lines (runs alongside week 1) |
| 1 | Provider interface, lookup and cache, the send queue with limits |
| 2 | Webhooks, opt-out, replies to the AI, chat rules. Line warm-up starts. |
| 3 | Health alerts, SMS fallback, tests on our own phones |
| 4–6 | Pilot with 2–3 customers, compared against SMS |

That is about 3 weeks of build for one backend engineer, plus a 3-week pilot. **It could slip if:**
- the provider contract or outbound approval is slow (most likely);
- lines take a long time to set up;
- a line is blocked during warm-up;
- SMS is built into the code in more places than expected.

**Questions for the team:** Is there a channel interface today? Where is consent stored? Is iMessage already live somewhere, and what did we learn from it?
