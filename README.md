# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md) · [Appendix: details, numbers, sources](appendix.md)

**In short:** a pilot with 2–3 US customers. Phase 1: leads who tap "prefer iMessage?" in an SMS or email write to us first, which warms up the lines. Phase 2: cart messages go out on the warm lines, tested 50/50 against SMS. Everyone else, Android included, gets SMS as today. Decision in about 2 months.

**Why it's hard:** Apple has no API for a business to message first. Providers run real Apple devices, and each line is one real number. iPhones disable links from unknown senders until the person replies. Lines that get "Report junk" taps are blocked, and providers cap a line at **50 new chats a day**. Apple tolerates these providers but doesn't approve them, so SMS fallback runs everywhere and a global kill switch exists.
*Assumptions to confirm:* the AI writes a step's message when it's due; consent is stored per lead and covers all texts, not only SMS; channels share one send path.

## 1. Provider
**Today:** no iMessage provider. **Compared:** Sendblue, Linq and Blooio on price, written limits, track record and conflicts. Rejected: our own Mac minis (bans and hardware would be ours) and Apple Messages for Business (the shopper must write first). **Suggestion:** Sendblue, if the contract bans it from using our customers' data; otherwise Linq. **Why:** all three cap a line at about 50 new chats a day, so terms decide. Sendblue publishes its limits as error codes and has the longest record (SOC 2, 5+ years), but sells its own cart recovery. Linq has the same limits and no competing product. Blooio is a year old and states no security record. Sales calls settle price and terms. Behind our own interface, switching takes a week.

## 2. Risks at volume
| Risk | What we do |
|---|---|
| **No link before a reply** (iOS and provider rule). Our cart message leads with the link. | First message: brand, "store assistant", one short question. Link after the reply. Two unanswered messages → SMS with the link. |
| **Apple blocks a line** after junk reports, bursts or one-way traffic. All its chats drop. | New chats per line ramp 10 → 25 → 50 a day over 3 weeks, max 15 an hour. Consent only, no night sends, a contact card after the first reply. One brand per line. |
| **Leads not on iMessage.** About 4 in 10 US phones are Android. Some numbers that moved to Android stay listed by Apple, and messages to them vanish. | Look up each number before the first send, cache 30 days. Not on iMessage → SMS, as today. Not delivered within an hour → clear the cache, next steps by SMS. |
| **Reply rate.** Few replies make a line look like a bot. Many mean more AI calls. | If a line's replies drop, it starts no new chats. Phase 1 warms lines with two-way chats. |
| **One customer's content** (adult, drugs, scams) can get our *whole* provider account closed. | Admins check what the brand sells. An automatic check on every AI message. A kill switch per customer. |
| **Not enough lines.** Carts go cold within an hour. | No free slot within 60 min → SMS. Per store: lines = eligible leads a day ÷ 40. |
| **State law.** Florida, Maryland, Oklahoma: sales texts only 8am–8pm, max 3 a day on one subject. Maine: AI disclosure. | The same contact-hour and frequency rules as SMS, by the shipping address state, else the area code. The pilot leaves out FL, MD and OK until legal says yes. |
| **Cost.** $195–$290 a line a month ($0.15–$0.25 a lead), against $0.01 for SMS. | The pilot tests it against SMS. A line pays for itself at 2 more carts recovered per 1,000 leads. |

| Topic | Today | Suggestion | Why |
|---|---|---|---|
| Sending hours | Every step waits for sending hours | Reply now to a lead active in the last 30 min, within state hours | Live chat, but the law still applies |
| Reply limit | 5, then a person | Cap of 30, then a person | Going silent loses the lead |
| Sender number | Each channel has its own number | One number per lead per campaign; it changes only on fallback or the lead's choice | A second number looks like a scam |
| Follow-ups | Run on schedule | A reply or purchase cancels pending steps, rechecked just before sending | No reminder after "yes, buying" |
| Bursts | AI answers each text | Typing dots, wait 15 s, answer once | People send 3 in a row |
| Thumbs-up, photo, voice note | SMS replies are text | No answer to a thumbs-up; media goes to a person | The AI can't hear a voice note |

## 3. Line health without a dashboard
**Today:** nothing per number. **Suggestion:** a job checks each line every 15 minutes: the provider's "line blocked" event, delivery rate, failures in a row, replies against sends, opt-outs and idle days. **Yellow** (delivery under 90% or opt-outs over 2%): halve the daily limit and alert Slack. **Red** (blocked, 5 failures in a row, or delivery under 70%): no new leads on the line, its chats move to SMS, and Slack and the customer are told. The on-call replaces the line within a working day. **Why:** the system acts first; people only replace lines.

## 4. Who turns it on
**Today:** customers switch channels on. **Suggestion:** only our admins in v1, per customer and campaign; we pay for pilot lines. Later, a paid add-on per line. **Why:** lines cost money monthly and need warm-up. One bad list or product can block lines, or our whole account. A price per line matches our cost.

## 5. Which leads get it
**Today:** every lead gets every channel in the campaign. **Suggestion:** leads with a US/CA number and consent, confirmed on iMessage, not opted out, not texting with us from another number, with a free line slot. Phase 1: only those who tap "prefer iMessage?", offered only to numbers confirmed on iMessage, so Android users never see it. Phase 2: half get iMessage, half SMS, so iPhone users are compared with iPhone users. **The rest**, Android included, go to the next channel, as today. We log why.

## 6. First version and time
**In, in build order:** first replies (diagram C): provider interface, webhooks with a catch-up job (the provider retries only 3 times), opt-out, chat rules, content check, typing dots. Then sending (diagram B): lookup with a 30-day cache, per-line limits, no-link rules, health alerts, SMS fallback.
**Waits, where we simplified:** self-serve and billing (pilot data first); moving chats between lines (SMS covers it); media and group chats (not needed for carts); a second provider; a dashboard (alerts cover it); countries beyond US/CA; RCS (reaches Android; its own project).

**Timeline:** week 0: provider calls, contract, data agreement, legal check. Weeks 1–2: build replies. Weeks 3–4: build sending and alerts, phase 1 live. Weeks 5–8: phase 2 live, fixes.
**How long:** 4 weeks of build for one engineer (3–5, depending on how channel-neutral our code is). **Blocks us:** the provider contract. **Could slip:** the contract, line setup, a ban during warm-up, legal answers. **Go/no-go:** replies at least 2× SMS (about 700 leads) and at least 2 points more carts recovered, for example 10% → 12% (about 7,700 leads, 4 weeks at 300 a day).

**Questions for the team:** The site lists iMessage: is it live, and what did we learn? How is pricing set? Does SMS consent cover iMessage? Do the contact-hour rules cover AI replies?
