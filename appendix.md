# Appendix: details, numbers, sources

Back to the plan: [README.md](README.md) · [Diagrams](diagrams.md)

*Optional reading. The plan stands on its own; this is the detail behind it.*

## A. Provider comparison

| | Linq | Sendblue | Blooio | Photon |
|---|---|---|---|---|
| Outbound price | On request (reported about $250/line) | On request (the $100 plan is inbound only) | $289/line, $195 at 6+ lines | $250/line |
| New chats per line | About 50 in 24 hours, then the line is marked "at risk" (a warning, not a block) | 50 a day, 15 an hour, max 150 in a row without a reply. An error code for each rule. | No fixed number. Over 10 a day with under 40% replies → slowed, then paused. An error code for each rule. | Up to 50 a day |
| Line health in the API | Healthy / at risk / critical, a webhook on change, automatic move off a flagged line | A `line_blocked` webhook | A health endpoint and a safety-state webhook | A dashboard. API unclear. |
| Security | Says SOC 2 Type II (auditor not named). 99.95% uptime in the contract. | SOC 2, HIPAA. **Data breach in July 2026:** message content and phone numbers accessed for some customers, investigation ongoing. | Not SOC 2 certified, by its own blog | No claim on its own pages |
| Company | Raised about $20M (Feb 2026), 1,000+ teams, used by Lindy. Sells the API since about 2025. | The oldest, 5+ years | Founded 2024 | Small. Open-source core. |
| Conflict with us | None found | Sells its own iMessage cart recovery to Shopify brands | None found | None found |

**Pick:** Linq, if it shows its SOC 2 Type II report and puts the pre-reply rules in the contract. Sendblue second: the most detailed rules, but the breach and its competing product. Blooio for the cheap manual test, since its prices are public. Photon doesn't fit: its pricing page sells cold outreach while its own guide says not to do it.

**Also looked at:**
- **Our own Mac minis:** rejected. The bans and the hardware would be ours, and one AI company (Lindy) was banned within half a day of launch on its own setup.
- **Apple Messages for Business:** official, but the shopper must write first, so it can't recover carts. A good fit later for inbound questions from Maps and Safari.
- **RCS for Business:** official and branded, and it reaches Android. It is not iMessage, so it gets its own look (section C).

**What goes in the contract:**
- the outbound price, and how fast new lines are set up;
- the pre-reply rules that apply to us, in writing. Sendblue's own cart guide puts a link, an image and a discount in the first message, while its limits page blocks exactly that;
- how fast a blocked line is replaced, and what it costs;
- a data agreement (DPA). Outcraft is based in Lithuania, so GDPR applies even to US leads, and the provider becomes a sub-processor;
- our customers' data is never used for the provider's own products or marketing;
- whether their automatic SMS fallback can be turned off. It would skip our own SMS consent and STOP handling;
- a sub-account and a kill switch per customer, so one brand can't get our whole account closed, and in writing that one store's ban can't affect the others;
- with Linq, a separate pool per store. If we choose the line ourselves, Linq's automatic failover turns off;
- the SOC 2 report itself, the auditor's name and the breach history.

**Provider details that shape the build.** From Linq's docs:
- A `phone_number.status_updated` webhook reports health changes (healthy, at risk, critical), so the health job uses Linq's own signal.
- When Linq picks the line, the recipients of a flagged line move to a fresh one (`failover_flagged`). A `continuation_message` can explain the new number.
- A flagged Linq number also stops sending SMS, by Linq's help page. One more reason to fall back through our own SMS path.
- Rate limits return 429 with `Retry-After`. The iMessage check allows one call per 10 seconds.

From Sendblue's docs, if we pick it:
- A `line_blocked` webhook exists, so the health job uses the provider's own signal.
- The lookup can answer "indeterminate". We treat that as "not on iMessage" and use SMS.
- Webhooks are retried only 3 times, with a 45 s timeout. If our endpoint is down for a few minutes, replies are lost. So a catch-up job pulls recent messages per line every 5 minutes (diagram C).
- Each message can have its own status URL. We put our message id in it, so a status that arrives before our database write still matches.
- Typing dots are one API call and add no outbound message, so they are in v1.

## B. All risks

The plan lists the main ones. The full list:

| Risk | What breaks | What we do |
|---|---|---|
| **No link before a reply** | iOS disables links from unknown senders until the person replies. Sendblue also blocks links and media before a reply, with max 300 characters and 6 messages. Our cart message leads with the link. | The first message names the brand, says it's an assistant and asks one short question. For example: "Hi Sam, this is the Glow Shop assistant. You left the blue scarf in your cart. Want me to save it for you?" The link comes after the reply. Two unanswered messages → the next step goes by SMS with the link. |
| **Apple blocks a line** | Junk reports, bursts and night sends. Every open chat on the line dies too. | New chats per line ramp 10 → 25 → 50 a day over 3 weeks, max 15 an hour, spread out. Consent only, no night sends, a contact card after the first reply. One brand per line, never shared. |
| **Reply rate** | Too few replies: a line that only sends looks like a bot to Apple. Many replies: more AI calls, longer chats, more handoffs. | Track replies against sends per line. When they drop, the line stops starting new chats. Phase 1 starts every line with two-way chats. A cap of 30 AI replies per chat, then a person. |
| **One customer's content** | Adult, drug or scam content can get our *whole* provider account closed, with no refund. | Admins check what the brand sells before turning it on (list below). Every AI message passes a content check. Watch opt-outs and "who is this" replies per customer. Sub-account and kill switch per customer. |
| **Apple ends the channel** | Apple blocks the providers' method for everyone, overnight. | Leads fall back to SMS automatically. We lose the blue bubble, not the leads. See section I. |
| **Provider down** | Every send fails and retries 3 times, so carts wait. Replies sent while webhooks are down are lost. | If most sends fail for 5 minutes, a circuit breaker turns iMessage off for everyone and steps go by SMS. It turns back on after a test send works. The catch-up job collects missed replies. |
| **Not enough lines** | Carts go cold within an hour, and 50 a day fills fast. | No free slot within 60 min → that step goes by SMS. Size lines with the formula in section F. |
| **Lead not on iMessage** | Android (about 4 in 10 US phones), iMessage turned off, numbers outside the US, numbers that moved to Android but are still registered with Apple. | Look the number up before the first send and cache it for 30 days. A downgrade, or no receipt within an hour → clear the cache, next steps by SMS. v1 covers US/CA only. See section C. |
| **Cost** | Lines are a fixed monthly cost. Long chats also cost more in AI calls. | The pilot splits iPhone leads 50/50 against SMS. See section F. |
| **Opt-out** | iMessage has no carrier STOP. | We detect STOP, "unsubscribe" and similar ourselves, opt the lead out on all channels and send one confirmation. |
| **Webhooks** | Duplicates, late or out-of-order events, missed retries. | Check the signature, skip events we've seen, save then queue. A catch-up job every 5 min pulls missed events and messages with no receipt after an hour. |
| **State law** | Quiet hours and frequency limits, AI disclosure. | See section E. |
| **Idle lines** | Apple turns off lines with no traffic for about 2 months. | The health job flags idle lines. |

**Brands that don't get iMessage in v1:** prescription drugs (including erectile-dysfunction and weight-loss drugs), drug-like health claims, adult products, cannabis, CBD, vape, tobacco and alcohol, gambling, crypto, loans and debt, weapons, and anything counterfeit. These follow what US carriers already ban or restrict on SMS. Such brands keep email and calls.

**Line health, starting values** (tuned in the pilot):
- Checked every 15 min: the provider's `line_blocked` event, blocked or declined errors, delivery rate over the last 100 sends, failures in a row, replies against sends, opt-out rate, days with no sends.
- **Yellow:** delivery under 90%, or opt-outs above 2%. The daily new-chat cap halves, and Slack gets an alert.
- **Red:** the provider says blocked, 5 failures in a row, or delivery under 70%. No new leads on the line, its chats move to SMS, and Slack and the customer are told. The ops on-call replaces the line within one working day.
- A replacement line starts its warm-up again, so the store has less iMessage capacity for 2–3 weeks. More of its leads go by SMS meanwhile.
- A weekly Slack summary per line catches slow decline.

## C. Android

About 4 in 10 US phones in use are Android (StatCounter, mid-2026: iOS 58–61%, Android 39–42%). The share differs a lot by store: about 87–88% of US teens own an iPhone (Piper Sandler, 2025).

- **iMessage can only reach the iPhone share.** At best it lifts about 6 in 10 US leads. The rest stay on SMS, and we sell it that way.
- **Pilot stores with iPhone-heavy shoppers.** We ask each candidate store for its iPhone share of mobile visits; any analytics tool shows it. A line priced per month pays off poorly for a store at 40%.
- **Compare iPhone with iPhone.** See section G.
- **Android users never see "prefer iMessage?".** If they tapped it, they would start an SMS chat with the iMessage line, outside our SMS rules. The offer goes only to numbers the lookup confirmed.
- **Numbers that moved to Android.** Apple can keep a number registered for iMessage after its owner leaves iPhone, until they deregister it. Messages to it vanish, or land on an old iPad or Mac. The 30-day cache makes this worse. No receipt within an hour clears the cache and moves the lead to SMS. The "two unanswered, then SMS" rule catches the rest.
- **SMS to Android is changing too.** Google Messages warns about links from unknown senders and hides suspected scams. It was piloted in 2024 in a few countries, with a wider rollout planned. Our SMS results on Android may get weaker either way.
- **RCS is the answer for Android.** Business RCS now runs on all major US carriers, through Google Messages on Android. It shows a verified brand name and logo, and allows links and buttons. Carrier approval takes 2–6 weeks. It is the natural next channel for the leads iMessage can't reach. Not confirmed: which US carriers show business RCS on iPhone.
- **Other markets:** Lithuania is about 63–70% Android, and Australia about 60–64% iPhone. iMessage fits the US, Canada and Australia. In most of the EU, WhatsApp, already a channel, matters more.

## D. Edge cases beyond the plan's table

- **A reply or a purchase while a step waits in the queue.** The step is checked again right before it is sent, and dropped if it's no longer wanted (diagram B).
- **The lead writes to the other number.** After a fallback, a lead may still write to the old iMessage line. We answer where the lead last wrote, with the whole chat history.
- **A line is blocked mid-chat.** Its chats move to the store's SMS number. The first SMS says it's the same store and why the number changed.
- **The lead's iMessage uses an email address.** Some people start chats from their Apple ID email, not their number. The "prefer iMessage?" link pre-fills a short code, so we can still match the lead (diagram C).
- **An opted-out lead writes again.** A person sees it. No automatic messages until the lead opts back in.
- **"Is this a bot?"** The AI says it's the store's assistant and offers a person.
- **Thumbs-up and other reactions.** They count as engagement in line stats, but get no answer.
- **One company, several brands.** Lines belong to a brand, so a lead always sees one brand per number.
- **Bursts.** People send 3 messages in a row. Typing dots, wait 15 s, then one answer.
- **Photos and voice notes.** They go to a person. The AI can't hear a voice note.
- **Opt-out scope.** An opt-out covers every channel of that store, not other stores on Outcraft, because consent is given per brand.

## E. Law

- **Treat iMessage exactly like SMS.** Courts treat texts as "calls" under the TCPA, and there is no ruling that iMessage is different.
- **Which state:** the shipping address if we have it, else the area code. If they differ, the stricter rule applies.
- **Quiet hours:** 21 states set hours for sales texts. Florida, Maryland, Oklahoma, Washington and others allow 8am–8pm local time. Texas allows 9am–9pm, and from noon on Sundays. Postscript reads Florida's law as covering a brand's replies too. So "reply right away" stays inside state hours (diagrams B and C).
- **Frequency:** Florida, Maryland and Oklahoma allow at most 3 messages per 24 hours on the same subject. A chat of up to 30 replies may break that. The pilot leaves these three states out until legal decides.
- **Opt-out (FCC, since 11 April 2025):** an opt-out sent by any reasonable means must be honoured within 10 business days. The "revoke all" part is delayed to 31 January 2027. We opt out at once and on all channels, so we already meet it.
- **Texas SB 140 (since 1 September 2025):** marketing texts now fall under the Texas solicitation law. There is a registration duty, with exemptions for current or former customers, and a private right of action reported at $500–$10,000 per violation. A cart abandoner who never bought may not count as a former customer. Legal check needed.
- **AI disclosure:** Maine (since September 2025) requires clear notice when a consumer could think they are talking to a human. California's SB 1001 covers bots that hide being a bot to sell. So the first message names the brand and says it's an assistant.
- **GDPR:** Outcraft is established in Lithuania, so GDPR covers its processing even for US leads (Art. 3(1)). We need a DPA with the provider and list it as a sub-processor.

## F. Sizing and cost

- **Lines needed per store** = eligible leads a day ÷ 40. The cap is 50, and the rest is headroom for spreading sends out and for retries.
- **Example:** a store with 300 carts a day with consent and a US/CA number. If 55–70% are on iMessage, that is 165–210 leads a day, so **5–6 lines**. In the pilot only half of them get iMessage, so about 3 lines.
- During warm-up a line takes only 10, then 25, new chats a day. Phase 1 covers part of this period.
- **Cost per line:** about $195–$290 a month (Blooio list price, Linq reported at about $250, Sendblue on request). At about 1,200 new chats a line a month, that is $0.15–$0.25 a lead, against about $0.01 for SMS.
- **Break-even:** $200 ÷ (1,200 leads × $80 average order) ≈ 0.2 percentage points, or **2 more carts recovered per 1,000 leads**. The $80 is an assumption. That counts all the store's recovered revenue. If Outcraft pays for the line and keeps only its share (say 15%), it needs about **14–17 more carts per 1,000 leads** ($200–$250 a line), about 1.5 points. If the store pays for the line, it needs about 2.5 per 1,000 counting revenue, or about 5 counting profit at a 50% margin. The go bar of 2 points (20 per 1,000) covers every case.
- **Pilot cost:** 6–10 lines for about 2 months, roughly $2,500–$6,000. We pay it, so stores can join easily.
- **Other costs:** longer chats mean more AI calls, ops time for line swaps, and support when a line is blocked.

## G. Pilot and go/no-go

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
- **Both halves are iPhone leads.** Leads are split only after the lookup confirms iMessage. iPhone and Android shoppers differ in age and income, so comparing iMessage with all SMS leads would flatter iMessage.
- **The SMS half gets no "prefer iMessage?" offer** in phase 2, or the comparison stops being clean.
- **If lines are the bottleneck,** the SMS group can be up to 3× the iMessage group. For 10% → 12%, that cuts the iMessage leads needed from about 3,840 to about 2,560.
- **Phase 1 leads are not in the test.** They chose iMessage themselves, so they would flatter it too. Phase 1 measures how many tap the offer and how those chats go.
- Warm-up limits volume early. If the pilot stores have fewer than about 300 eligible leads a day, we run longer or add a store.

## H. Rollout

- **Week 0:** provider calls, contract, DPA, legal check on state rules. Lines ordered.
- **Weeks 1–2:** build replies (diagram C). That is all phase 1 needs.
- **Weeks 3–4, phase 1:** SMS and email steps offer "prefer iMessage? tap here", only to numbers confirmed on iMessage. The link opens a chat with the store's line and pre-fills a short code. The lead writes first, so:
  - links work, and there is no junk banner;
  - none of the pre-reply rules apply;
  - Apple stops filtering a sender once the person has replied 3 times;
  - real two-way chats warm the lines up.

  Meanwhile we build sending (diagram B).
- **Weeks 5–8, phase 2:** outbound cart messages on the warm lines, 50/50 against SMS. The riskiest part, the cold first message, starts last and on the healthiest lines.
- **If the team wants less risk:** stop after phase 1. v1 is then replies only, and outbound comes later.
- **Cheapest test of all:** one line and one customer, run by hand from the provider's dashboard, to measure the lift before any build. Dashboard sends skip our consent, STOP and hours checks, so someone checks those by hand. It also gives an early cart-recovery number during the build, which phase 1 can't.
- **Effort:** about 4 weeks of build for one engineer, 3–5 depending on how channel-neutral our code is.

## I. Platform risk

- Apple's iCloud terms say the service is for personal use, and forbid automated access and promotional messages. Every iMessage provider runs real Apple IDs against these terms. Outcraft isn't a party to them, but the channel exists only while Apple tolerates it.
- Precedents: Apple cut off Beeper Mini in December 2023 (a consumer app using fake credentials, a different case), and Lindy's own account was banned within half a day.
- I found no sign of Apple shutting down managed providers in 2025–26, and Sendblue has run for 5+ years.
- **So: low chance, high impact.** SMS fallback runs everywhere, a global kill switch exists, and we never sell iMessage as guaranteed.

## J. What waits, and ideas for later

**Waits in v1, and why:**
- **Self-serve and billing:** needs pilot numbers first.
- **Moving a chat to a new line:** SMS covers it in v1.
- **Sending pictures:** not needed to win a reply.
- **Group chats:** carts are one-to-one.
- **A second provider:** our interface keeps the switch to about a week.
- **A dashboard:** alerts do the job.
- **Countries beyond US/CA:** iPhone share is lower in most of them, and lines are US numbers.
- **RCS:** official and reaches Android, but needs carrier approval. Its own project (section C).

**If the pilot works:**
- **Warm segments:** post-purchase upsell, saving subscription cancellations, win-back. These leads know the brand, so junk reports are rarer.
- **Contact card in the order confirmation email or on the thank-you page:** once it's saved, there is no junk banner.
- **RCS for Business** for Android, next to iMessage.
- **Apple Messages for Business** for inbound questions.
- **Pricing:** a best-effort premium channel with automatic fallback, priced per line.

## K. Questions for the team

1. The site lists iMessage. Is it live, with which provider, and what has worked?
2. How is pricing set, and who would pay for lines?
3. Does the SMS consent we collect cover iMessage?
4. Do the contact-hour and frequency controls already cover state rules (FL, MD, OK, TX)? Do they apply to AI replies?
5. Where is consent stored?
6. Is there a common channel interface?
7. What share of our stores' shoppers use iPhones?
8. Have you looked at RCS for Business?
9. How much downside is acceptable if Apple cut iMessage off overnight?

## Sources

- **Providers:** [Sendblue pricing](https://sendblue.com/pricing) · [Sendblue limits](https://docs.sendblue.com/limits/) · [Sendblue API summary](https://docs.sendblue.com/llms.txt) · [Sendblue Recover](https://recover.sendblue.com/) · [Sendblue cart guide](https://www.sendblue.com/blog/imessage-ecommerce-abandoned-cart) · [Sendblue vs Blooio](https://www.sendblue.com/compare/sendblue-vs-blooio) · [Messages for Business vs API](https://www.sendblue.com/blog/apple-messages-for-business-vs-imessage-api) · [Blooio pricing](https://blooio.com/integrations/api/pricing) · [Blooio vs Sendblue](https://blooio.com/alternatives/sendblue) · [Linq rate limits](https://docs.linqapp.com/guides/platform/rate-limits/index.md) · [Linq vs Sendblue](https://linqapp.com/s/linq-vs-sendblue) · [Linq flagged number](https://help.linqapp.com/flagged-number) · [Photon deliverability](https://photon.codes/docs/best-practices/imessage-deliverability) · [Sendblue incident report](https://www.sendblue.com/blog/sendblue-security-incident-update) · [Linq phone reputation](https://docs.linqapp.com/guides/phone-numbers/phone-reputation/) · [Linq sending and failover](https://docs.linqapp.com/channel/imessage/guides/messaging/sending-messages/index.md) · [Linq iMessage API](https://linqapp.com/imessage-api) · [Blooio messaging safety](https://docs.blooio.com/guides/messaging-safety) · [Blooio API pricing](https://blooio.com/integrations/api/pricing) · [Photon pricing](https://photon.codes/pricing)
- **Apple:** [iCloud terms](https://www.apple.com/legal/internet-services/icloud/) · [Unknown senders, iOS 26](https://support.apple.com/en-in/125068) · [Twilio: iOS 26 and business messaging](https://www.twilio.com/en-us/blog/insights/trends/potential-ios-26-update-communications-implications) · [iMessage link protection (Bitdefender)](https://bitdefender.in/iphone-users-tricked-into-disabling-apple-imessages-phishing-protection) · [Deregister iMessage (Apple)](https://selfsolve.apple.com/deregister-imessage) · [Beeper Mini (MacRumors)](https://www.macrumors.com/2023/12/10/apple-confirms-it-shut-down-beeper-mini/) · [Lindy: three rewrites, one Apple ban](https://www.lindy.ai/blog/imessage-api-three-rewrites-one-apple-ban-and-what-actually-works)
- **Android and RCS:** [US mobile OS share (StatCounter)](https://gs.statcounter.com/os-market-share/mobile/united-states-of-america) · [Teen iPhone ownership (MacRumors, Piper Sandler)](https://www.macrumors.com/2025/04/09/teen-iphone-ownership-continues-to-soar/) · [Lithuania mobile OS share (StatCounter)](https://gs.statcounter.com/os-market-share/mobile/lithuania) · [Australia mobile OS share (StatCounter)](https://gs.statcounter.com/os-market-share/mobile/australia) · [Google Messages protections (Google Security Blog)](https://security.googleblog.com/2024/10/5-new-protections-on-google-messages.html) · [RCS on all US carriers (Twilio)](https://www.twilio.com/en-us/blog/products/RCS-ATT-Integration) · [RCS on iOS 18.1 (Twilio)](https://www.twilio.com/en-us/blog/insights/trends/rcs-business-messaging-apple-update) · [Apple and RCS (Sinch)](https://sinch.com/blog/apple-support-rcs/) · [Attentive RCS](https://ecommercenews.com.au/story/attentive-launches-ai-tools-for-rcs-message-rollout)
- **Law:** [Postscript: Florida rules](https://postscript.io/blog/florida-compliance-changes) · [State quiet hours (Referrizer)](https://support.referrizer.com/article/tcpa-quiet-hours-and-state-specific-rules-for-text-campaigns) · [FCC revoke-all delay (BHFS)](https://www.bhfs.com/insight/fcc-extends-compliance-deadline-for-revoke-all-rule/) · [Texas SB 140 (Morgan Lewis)](https://www.morganlewis.com/pubs/2025/09/texas-telephone-solicitation-law-now-covers-text-messages) · [AI disclosure laws (DLA Piper)](https://www.dlapiper.com/en/insights/publications/2026/01/ai-disclosure-laws-on-chatbots-are-on-the-rise-key-takeaways-for-companies)
- **Market:** [US iPhone sales share (MacRumors, Counterpoint)](https://www.macrumors.com/2026/05/13/apple-q1-market-share-growth/)
