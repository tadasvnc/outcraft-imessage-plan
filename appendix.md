# Appendix: details, numbers, sources

Back to the plan: [README.md](README.md) · [Diagrams](diagrams.md)

*Optional reading. The plan stands on its own; this is the detail behind it.*

## A. Provider comparison

| | Sendblue | Linq | Blooio |
|---|---|---|---|
| Outbound price | On request (the $100 plan is inbound only) | On request (reported about $250/line) | $289/line, $195 at 6+ lines |
| New chats per line | 50 a day, 15 an hour, an error code for each rule | 50 a day, max 150 outbound in a row | "20–50 a day" guidance |
| Track record | SOC 2, HIPAA, 5+ years | Enterprise focus, used by AI-agent companies (Lindy) | Security not stated, about 1 year |
| Conflict with us | Sells its own iMessage cart recovery to Shopify brands | None found | None found |

**Also looked at:**
- **Photon:** the same model as the three above.
- **Our own Mac minis:** rejected. The bans and the hardware would be ours, and one AI company (Lindy) was banned within half a day of launch on its own setup.
- **Apple Messages for Business:** official, but the shopper must write first, so it can't recover carts. A good fit later for inbound questions from Maps and Safari.
- **RCS for Business:** official and branded. It works on iPhone since iOS 18.1 and on Android too. Attentive and Postscript use it. It is not iMessage, so it gets its own look.

**What goes in the contract:**
- the outbound price, and how fast new lines are set up;
- the pre-reply rules that apply to us, in writing. Sendblue's own cart guide puts a link, an image and a discount in the first message, while its limits page blocks exactly that;
- how fast a blocked line is replaced, and what it costs;
- a data agreement (DPA). Outcraft is based in Lithuania, so GDPR applies even to US leads, and the provider becomes a sub-processor;
- our customers' data is never used for the provider's own products or marketing;
- whether their automatic SMS fallback can be turned off. It would skip our own SMS consent and STOP handling;
- a sub-account and a kill switch per customer, so one brand can't get our whole account closed.

**Provider details that shape the build** (from Sendblue's docs; to check the same for Linq):
- A `line_blocked` webhook exists, so the health job uses the provider's own signal.
- The lookup can answer "indeterminate". We treat that as "not on iMessage" and use SMS.
- Webhooks are retried only 3 times, with a 45 s timeout. If our endpoint is down for a few minutes, replies are lost. So a catch-up job lists recent messages per line every 5 minutes (diagram C).
- Each message can have its own status URL. We put our message id in it, so a status that arrives before our database write still matches.
- Typing dots are one API call and add no outbound message, so they are in v1.

## B. All risks

The plan lists the top six. The full list:

| Risk | What breaks | What we do |
|---|---|---|
| **No link before a reply** | iOS disables links from unknown senders until the person replies. Sendblue also blocks links and media before a reply, with max 300 characters and 6 messages. Our cart message leads with the link. | The first message names the brand, says it's an assistant and asks one short question. The link comes after the reply. Two unanswered messages → the next step goes by SMS with the link. |
| **Apple blocks a line** | Junk reports, bursts and night sends. Every open chat on the line dies too. | Warm-up 10 → 25 → 50 new chats a day, max 15 an hour, spread out. Consent only, no night sends, a contact card after the first reply. One customer per line, never shared. |
| **Low reply rate** | A line that only sends looks like a bot to Apple. | Track send-to-receive per line. When it drops, the line stops starting new chats. Inbound-first weeks start every line with two-way traffic. |
| **One customer's content** | Adult, drug or scam content can get our *whole* provider account closed, with no refund. | Admins check what the brand sells before turning it on (see the list below). Every AI message passes a content check. Watch opt-outs and "who is this" replies per customer. Sub-account and kill switch per customer. |
| **Apple ends the channel** | Apple blocks the providers' method for everyone, overnight. | Leads fall back to SMS automatically. We lose the blue bubble, not the leads. See section G. |
| **Not enough lines** | Carts go cold within an hour, and 50 a day fills fast. | No free slot within 60 min → that step goes by SMS. Size lines with the formula in section D. |
| **Lead not on iMessage** | Android, iMessage turned off, numbers outside the US. | Look the number up before the first send and cache it for 30 days. Update the cache when a message is downgraded. v1 covers US/CA only. |
| **Cost** | Lines are a fixed monthly cost. Long chats also cost more in AI calls. | The pilot splits leads 50/50 against SMS. See section D. |
| **Opt-out** | iMessage has no carrier STOP. | We detect STOP, "unsubscribe" and similar ourselves, opt the lead out on all channels and send one confirmation. |
| **Webhooks** | Duplicates, late or out-of-order events, missed retries. | Check the signature, skip events we've seen, save then queue. Catch-up job every 5 min. A status stuck for 15 min is checked through the API. |
| **State law** | Quiet hours and frequency limits, AI disclosure. | See section C. |
| **Idle lines** | Apple turns off lines with no traffic for about 2 months. | The health job flags idle lines. |

**Brands that don't get iMessage in v1:** prescription drugs (including erectile-dysfunction and weight-loss drugs), drug-like health claims, adult products, cannabis, CBD, vape, tobacco and alcohol, gambling, crypto, loans and debt, weapons, and anything counterfeit. These follow what US carriers already ban or restrict on SMS. Such brands keep email and calls.

**Line health, starting values** (tuned in the pilot):
- Checked every 15 min: the provider's `line_blocked` event, blocked or declined errors, delivery rate over the last 100 sends, failures in a row, send-to-receive ratio, opt-out rate, days with no sends.
- **Yellow:** delivery under 90%, or opt-outs above 2%. The daily new-chat cap halves, and Slack gets an alert.
- **Red:** the provider says blocked, 5 failures in a row, or delivery under 70%. No new leads on the line, its chats move to SMS, and Slack and the customer are told. The ops on-call replaces the line within one working day.
- A weekly Slack summary per line catches slow decline.

## C. Law

**Rule of thumb:** treat iMessage exactly like SMS. Courts treat texts as "calls" under the TCPA, and there is no ruling that iMessage is different.

- **Quiet hours:** 21 states set hours for sales texts. Florida, Maryland, Oklahoma, Washington and others allow 8am–8pm local time. Texas allows 9am–9pm, and from noon on Sundays. Postscript reads Florida's law as covering a brand's replies too. So "reply right away" stays inside state hours (diagrams B and C).
- **Frequency:** Florida, Maryland and Oklahoma allow at most 3 messages per 24 hours on the same subject. A chat of up to 30 replies may break that. The pilot leaves these three states out until legal decides.
- **Opt-out (FCC, since 11 April 2025):** an opt-out sent by any reasonable means must be honoured within 10 business days. The "revoke all" part is delayed to 31 January 2027. We opt out at once and on all channels, so we already meet it.
- **Texas SB 140 (since 1 September 2025):** marketing texts now fall under the Texas solicitation law. There is a registration duty, with exemptions for current or former customers, and a private right of action reported at $500–$10,000 per violation. A cart abandoner who never bought may not count as a former customer. Legal check needed.
- **AI disclosure:** Maine (since September 2025) requires clear notice when a consumer could think they are talking to a human. California's SB 1001 covers bots that hide being a bot to sell. So the first message names the brand and says it's an assistant.
- **GDPR:** Outcraft is established in Lithuania, so GDPR covers its processing even for US leads (Art. 3(1)). We need a DPA with the provider and list it as a sub-processor.

## D. Sizing and cost

- **Lines needed** = eligible leads a day ÷ 40. The cap is 50, and the rest is headroom for spreading sends out and for retries.
- **Example:** 300 carts a day with consent and a US/CA number. About 55–60% of US phones in use are iPhones (69–75% of new sales), and online shoppers likely skew higher. At 55–70% that is 165–210 leads a day, so **5–6 lines**.
- During warm-up a line takes only 10, then 25, new chats a day. The inbound-first weeks cover this period.
- **Cost per line:** about $195–$290 a month (Blooio list price, Linq reported at about $250, Sendblue on request). At about 1,200 new chats a line a month, that is $0.15–$0.25 a lead, against about $0.01 for SMS.
- **Break-even:** $200 ÷ (1,200 leads × $80 average order) ≈ **0.2 points** more carts recovered. The $80 is an assumption.
- **Costs that aren't on the invoice:** longer chats mean more AI calls, ops time for line swaps, and support when a line is blocked.

## E. Pilot and go/no-go

Two-sided test, 5% significance, 80% power. The baselines (6% SMS reply rate, 10% recovery) are assumptions until we see Outcraft's own data.

| What we want to prove | Leads needed (both groups) | Days at 100 eligible a day | Days at 300 a day |
|---|---|---|---|
| Reply rate 6% → 12% (2×) | 712 | 8 | 3 |
| Reply rate 6% → 9% | 2,418 | 25 | 9 |
| Recovery 10% → 15% (vendor claim) | 1,372 | 14 | 5 |
| Recovery 10% → 12% | 7,682 | 77 | 26 |
| Recovery 10% → 11% | 29,502 | 296 | 99 |

- **Go:** reply rate at least 2× SMS, and recovery at least 2 points above SMS.
- **Guardrails:** opt-outs no higher than SMS, and under 1 blocked line per 10 line-months.
- **Revenue after line costs** is tracked, but proving a 1-point lift needs about 30,000 leads, so it shows direction only.
- Warm-up limits volume early. If the pilot customers have fewer than about 300 eligible leads a day, we run longer or add a customer.

## F. Rollout order

- **Weeks 1–2, inbound-first:** every SMS, email or AI call offers "prefer iMessage? tap here". That's a link that opens a chat with the customer's iMessage line. The lead writes first, so:
  - links work, and there is no junk banner;
  - none of the pre-reply rules apply;
  - Apple stops filtering a sender once the person has replied 3 times;
  - real two-way chats warm the lines up and keep the send-to-receive ratio healthy.
- **From week 3:** outbound cart messages on lines that are already warm, with all the safeguards. The riskiest part, the cold first message, starts last and on the healthiest lines.
- **If the team wants less risk:** v1 is inbound-first only, and outbound comes in v2.
- **Cheapest test of all:** one line and one customer, run by hand from the provider's dashboard, to measure the lift before any build.

## G. Platform risk, in proportion

- Apple's iCloud terms say the service is for personal use, and forbid automated access and promotional messages. Every iMessage provider runs real Apple IDs against these terms. Outcraft isn't a party to them, but the channel exists only while Apple tolerates it.
- Precedents: Apple cut off Beeper Mini in December 2023 (a consumer app using fake credentials, a different case), and Lindy's own account was banned within half a day.
- I found no sign of Apple shutting down managed providers in 2025–26, and Sendblue has run for 5+ years.
- **So: low chance, high impact.** SMS fallback runs everywhere, a global kill switch exists, and we never sell iMessage as guaranteed.

## H. What waits, and ideas for later

**Waits in v1, and why:**
- **Self-serve and billing:** needs pilot numbers first.
- **Moving a chat to a new line:** SMS covers it in v1.
- **Pictures and reactions:** not needed to win a reply.
- **Group chats:** carts are one-to-one.
- **A second provider:** our interface keeps the switch to about a week.
- **A dashboard:** alerts do the job.
- **Countries beyond US/CA:** iPhone share is lower, and lines are US numbers.
- **RCS:** official and reaches Android, so it's worth its own look.

**If the pilot works:**
- **Warm segments:** post-purchase upsell, saving subscription cancellations, win-back. These leads know the brand, so junk reports are rarer.
- **Contact card in the order confirmation email or on the thank-you page:** once it's saved, there is no junk banner.
- **RCS for Business** as the official path, next to iMessage.
- **Apple Messages for Business** for inbound questions.
- **Pricing:** a best-effort premium channel with automatic fallback, priced per line.

## I. Questions for the team

1. The site lists iMessage. Is it live, with which provider, and what has worked?
2. How is pricing set, and who would pay for lines?
3. Do the contact-hour and frequency controls already cover state rules (FL, MD, OK, TX)? Do they apply to AI replies?
4. Where is consent stored, and does it cover iMessage?
5. Is there a common channel interface?
6. Have you looked at RCS for Business?
7. How much downside is acceptable if Apple cut iMessage off overnight?

## Sources

- **Providers:** [Sendblue pricing](https://sendblue.com/pricing) · [Sendblue limits](https://docs.sendblue.com/limits/) · [Sendblue API summary](https://docs.sendblue.com/llms.txt) · [Sendblue Recover](https://recover.sendblue.com/) · [Sendblue cart guide](https://www.sendblue.com/blog/imessage-ecommerce-abandoned-cart) · [Sendblue vs Blooio](https://www.sendblue.com/compare/sendblue-vs-blooio) · [Messages for Business vs API](https://www.sendblue.com/blog/apple-messages-for-business-vs-imessage-api) · [Blooio pricing](https://blooio.com/integrations/api/pricing) · [Blooio vs Sendblue](https://blooio.com/alternatives/sendblue) · [Linq rate limits](https://docs.linqapp.com/guides/platform/rate-limits/index.md) · [Linq vs Sendblue](https://linqapp.com/s/linq-vs-sendblue) · [Linq flagged number](https://help.linqapp.com/flagged-number) · [Photon deliverability](https://photon.codes/docs/best-practices/imessage-deliverability)
- **Apple:** [iCloud terms](https://www.apple.com/legal/internet-services/icloud/) · [Unknown senders, iOS 26](https://support.apple.com/en-in/125068) · [Twilio: iOS 26 and business messaging](https://www.twilio.com/en-us/blog/insights/trends/potential-ios-26-update-communications-implications) · [iMessage link protection (Bitdefender)](https://bitdefender.in/iphone-users-tricked-into-disabling-apple-imessages-phishing-protection) · [Beeper Mini (MacRumors)](https://www.macrumors.com/2023/12/10/apple-confirms-it-shut-down-beeper-mini/) · [Lindy: three rewrites, one Apple ban](https://www.lindy.ai/blog/imessage-api-three-rewrites-one-apple-ban-and-what-actually-works)
- **Law:** [Postscript: Florida rules](https://postscript.io/blog/florida-compliance-changes) · [State quiet hours (Referrizer)](https://support.referrizer.com/article/tcpa-quiet-hours-and-state-specific-rules-for-text-campaigns) · [FCC revoke-all delay (BHFS)](https://www.bhfs.com/insight/fcc-extends-compliance-deadline-for-revoke-all-rule/) · [Texas SB 140 (Morgan Lewis)](https://www.morganlewis.com/pubs/2025/09/texas-telephone-solicitation-law-now-covers-text-messages) · [AI disclosure laws (DLA Piper)](https://www.dlapiper.com/en/insights/publications/2026/01/ai-disclosure-laws-on-chatbots-are-on-the-rise-key-takeaways-for-companies)
- **Market:** [US iPhone share (MacRumors, Counterpoint)](https://www.macrumors.com/2026/05/13/apple-q1-market-share-growth/) · [RCS on iOS 18.1 (Twilio)](https://www.twilio.com/en-us/blog/insights/trends/rcs-business-messaging-apple-update) · [Apple and RCS (Sinch)](https://sinch.com/blog/apple-support-rcs/) · [Attentive RCS](https://ecommercenews.com.au/story/attentive-launches-ai-tools-for-rcs-message-rollout)
