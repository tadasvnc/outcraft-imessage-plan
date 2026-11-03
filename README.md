# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md) · [Appendix: details, numbers, sources](appendix.md)

**In short:** a 6-week pilot for 2–3 US customers, turned on by our admins. Weeks 1–2: leads who tap "prefer iMessage?" write to us first. From week 3: cart messages go out on the now-warm lines. Each lead keeps one number. Everyone else gets SMS as today.

**Why it's hard:** Apple has no API for a business to message first. Providers run real Apple devices, and each line is one real number. iPhones disable links from unknown senders until the person replies. Lines that get "Report junk" taps are blocked, and every provider caps a line at **50 new chats a day**. Apple tolerates these providers but doesn't approve them, so SMS fallback runs everywhere and a global kill switch exists.
*Assumptions to confirm:* the AI writes a step's message when it's due; consent is stored per lead; channels share one send path.

## 1. Provider
**Today:** none. **Compared:** Sendblue, Linq, Blooio. Rejected: our own Mac minis (the bans and hardware would be ours) and Apple Messages for Business (the shopper must write first). **Suggestion:** sales calls with **Sendblue and Linq**, then pick on outbound price, limits written into the contract, how fast a blocked line is replaced, a data agreement (DPA), and whether their automatic SMS fallback can be turned off. **Why:** all providers share the 50-a-day limit, so contract terms decide. Sendblue sells its own cart-recovery product, so we'd sign only with a clause that bans using our data. Behind our own interface, switching takes about a week.

## 2. Risks at volume
| Risk | What we do |
|---|---|
| **No link before a reply** (iOS and provider rule). Our cart message leads with the link. | First message: brand, "store assistant", one short question. Link after the reply. Two unanswered messages → SMS with the link. |
| **Apple blocks a line** after junk reports, bursts or one-way traffic. All its chats drop. | Warm-up 10 → 25 → 50 a day, max 15 an hour, consent only, no night sends, a contact card after the first reply. One customer per line. |
| **One customer's content** (adult, drugs, scams) gets our *whole* provider account closed. | Admins check what the brand sells. An automatic check on every AI message. A kill switch per customer. |
| **Not enough lines.** Carts go cold within an hour. | No slot within 60 min → SMS. Lines = eligible leads a day ÷ 40. |
| **State law.** FL, MD, OK: no sales texts after 8pm, max 3 a day per subject. Maine: AI disclosure. | The same contact-hour and frequency rules as SMS. Pilot leaves out FL, MD and OK until legal says yes. |
| **Cost.** About $200–$290 a line a month ($0.15–$0.25 a lead), against $0.01 for SMS. | Pilot 50/50 against SMS. Line cost is recovered by 0.2% more carts recovered. |

| Topic | Today | Suggestion | Why |
|---|---|---|---|
| Sending hours | Every step waits | Reply now to a lead active in the last 30 min, within state hours | Live chat, but the law still applies |
| Reply limit | 5, then a person | Cap of 30 (FL/MD/OK after legal) | Going silent loses the lead |
| Sender number | Any from the pool | Same line for the whole chat | A new number looks like a stranger |
| Follow-ups | On schedule | A reply cancels pending steps | No reminder after "yes, buying" |
| Bursts | Each text wakes the AI | Typing dots, wait 15 s, answer once | People send 3 short texts |

## 3. Line health without a dashboard
**Today:** nothing per number. **Suggestion:** every 15 minutes a job checks the provider's "line blocked" event, delivery rate, failures in a row, send-to-receive ratio, opt-outs and idle days. **Yellow:** halve the daily limit and alert Slack. **Red:** no new leads on the line, its chats move to SMS, and Slack and the customer are told. The on-call replaces a red line within a working day. **Why:** the system acts first and people only decide on replacing a line.

## 4. Who turns it on
**Today:** customers switch channels on. **Suggestion:** our admins only in v1, later a paid add-on per line. **Why:** lines cost money monthly and need warm-up. One bad list or product can block lines, or our whole account. A price per line matches our cost.

## 5. Which leads get it
**Today:** every lead gets every channel in the campaign. **Suggestion:** only leads with a US/CA number, consent, confirmed on iMessage, not opted out, not already in an SMS chat, and with a free line slot. In weeks 1–2, only leads who tap "prefer iMessage?" in an SMS, email or call. They write first, so links work and the lines warm up. **The rest** go to the next channel. We log why.

## 6. First version and time
**In:** provider interface; lookup with a 30-day cache; per-line limits and the no-link rules; webhooks plus a catch-up job (the provider retries only 3 times); opt-out and the content check; typing dots; health alerts; SMS fallback.
**Waits:** self-serve and billing (pilot data first); moving chats between lines (SMS covers it); media and group chats (not needed for carts); a second provider; a dashboard (alerts cover it); countries beyond US/CA; RCS (official, reaches Android, worth its own look).

| Week | Work |
|---|---|
| 0 | Provider calls, contract, DPA, legal check on state rules |
| 1–2 | Interface, lookup, webhooks, chat rules. Inbound-first pilot. |
| 3 | Limits, content check, alerts. Outbound carts on warm lines. |
| 4–6 | Pilot against SMS |

**How long:** 3–5 weeks of build for one engineer, depending on how channel-neutral the code is. **Blocks us:** the provider contract. **Simplified:** admin-only, one provider, US/CA only, SMS instead of moving chats. **Could slip:** the contract, line setup, a ban during warm-up. **Go/no-go:** replies at least 2× SMS (about 700 leads) and at least 2 more points of carts recovered (about 7,700 leads, roughly 4 weeks).

**Questions for the team:** Is iMessage live somewhere already? How is pricing set? Do the contact-hour rules cover AI replies?
