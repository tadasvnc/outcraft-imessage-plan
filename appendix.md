# Appendix: details, numbers, sources

Back to the plan: [README.md](README.md) · [Diagrams](diagrams.md)

*Optional reading. The plan stands on its own; this is the detail behind it.*

## A. Provider comparison

| | Linq | Sendblue | Blooio | Photon |
|---|---|---|---|---|
| Outbound price | On request (reported about $250/line) | On request (the $100 plan is inbound only) | $289/line, $195 at 6+ lines | $250/line |
| New chats per line | About 50 in 24 hours, then the line is marked "at risk" (a warning, not a block) | 50 a day, 15 an hour, max 150 in a row without a reply. An error code for each rule. | No fixed number. Over 10 a day with under 40% replies → slowed, then paused. An error code for each rule. | Up to 50 a day |
| Line health in the API | Healthy / at risk / critical, a webhook on change, automatic move off a flagged line | A `line_blocked` webhook | A health endpoint and a safety-state webhook | A dashboard. API unclear. |
| Security | Says SOC 2 Type II (auditor not named). 99.95% uptime in the contract. | SOC 2, HIPAA. **Data breach in July 2026:** message content and phone numbers accessed for some customers, investigation ongoing. | No SOC 2 report yet, by its own blog | No claim on its own pages |
| Company | Raised about $20M (Feb 2026), 1,000+ teams, used by Lindy. Sells the API since about 2025. | The oldest, 5+ years | Founded 2024 | Small. Open-source core. |
| Conflict with us | None found | Sells its own iMessage cart recovery to Shopify brands | None found | None found |

**Pick: Blooio**, from step 0 through the pilot, so the lines warmed in step 0 carry into the pilot.
- **Why:** public prices ($289 a line, no setup fees), live in days, and hard limits with an error code for each rule, so the provider catches our own mistakes. It falls back to SMS only through our own Twilio, so we leave that off and fall back through our own channels.
- **Conditions:** a data agreement (DPA) before any shopper data flows, and only the data a message needs: first name, phone number, cart item. Its SOC 2 is "in progress" by its trust page. That matters only if our customers' security reviews ask for it. Then we move to Linq; our provider interface keeps that to about a week.
- **Plan B: Linq.** It claims SOC 2 Type II, has a 99.95% uptime contract and automatic failover, but no public price.
- **Not Sendblue: it competes with us.** It sells its own iMessage cart recovery to Shopify and WooCommerce stores ([Sendblue Recover](https://recover.sendblue.com/)), which is our market. As our provider it would see our customers' shoppers, messages and volumes. A no-data-use clause helps, but it can't be checked. It also had a data breach in July 2026. **Not Photon:** its pricing page sells cold outreach while its own guide says not to do it.

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
- with Blooio, when its SOC 2 will be done, in case our customers ask, and whether a ban in one store's sub-account slows the others (its `sibling_ban` rule slows every number on an account for 72 hours);
- with Linq (plan B), a separate pool per store. If we choose the line ourselves, Linq's automatic failover turns off;
- any SOC 2 report, the auditor's name and the breach history.

**Provider details that shape the build.** From Blooio's docs (our pick):
- Before a written reply, a chat allows 3 plain-text messages, and links and media are blocked (`conversation_content_restricted`). Our no-link rule and "two unanswered → next channel" stay inside that.
- The `one_way` rule: over 10 new chats a day (3-day average) with under 40% written replies slows the line, then pauses new chats. Sizing has to allow for it (section F).
- Its 429 errors such as `conversation_awaiting_reply` aren't temporary, and retrying won't help. Our retry logic reads the error key, not just the status code, and tells the AI at once, which picks the next step.
- The `safety.state_changed` webhook and the health endpoint (polled every 5 minutes) feed our health job.
- One banned number slows every number on the account for 72 hours (`sibling_ban`), so each store gets its own sub-account.
- Its SMS fallback runs only through our own Twilio account. We leave it off, and the AI picks the next step instead.

From Linq's docs (plan B):
- A `phone_number.status_updated` webhook reports health changes (healthy, at risk, critical), so the health job uses Linq's own signal.
- When Linq picks the line, the recipients of a flagged line move to a fresh one (`failover_flagged`). A `continuation_message` can explain the new number.
- A flagged Linq number also stops sending SMS, by Linq's help page. One more reason to fall back through our own channels.
- Rate limits return 429 with `Retry-After`. The iMessage check allows one call per 10 seconds.

From Sendblue's docs, if we pick it:
- A `line_blocked` webhook exists, so the health job uses the provider's own signal.
- The lookup can answer "indeterminate". We treat that as "not on iMessage" and use the next channel.
- Webhooks are retried only 3 times, with a 45 s timeout. If our endpoint is down for a few minutes, replies are lost. So a catch-up job pulls recent messages per line every 5 minutes (diagram C).
- Each message can have its own status URL. We put our message id in it, so a status that arrives before our database write still matches.
- Typing dots are one API call and add no outbound message, so they are in v1.

## B. All risks

The plan lists the main ones. The full list:

| Risk | What breaks | What we do |
|---|---|---|
| **No link before a reply** | iOS disables links from unknown senders until the person replies. Blooio also blocks links and media before a written reply, and allows 3 plain-text messages until then. Our cart message leads with the link. | The first message names the brand, says it's an assistant and asks one short question. For example: "Hi Sam, this is the Glow Shop assistant. You left the blue scarf in your cart. Want me to save it for you?" The link comes after the reply. Two unanswered messages → the next step goes on the next channel, with the link. |
| **Apple blocks a line** | Junk reports, bursts and night sends. Every open chat on the line dies too. | New chats per line ramp 10 → 25 → 50 a day over 3 weeks, max 15 an hour, spread out. Consent only, no night sends, a contact card after the first reply. One brand per line, never shared. |
| **Reply rate** | Too few replies: a line that only sends looks like a bot to Apple. Many replies: more AI calls, longer chats, more handoffs. | Track replies against sends per line. When they drop, the line stops starting new chats. The first message asks a question, to invite a reply. Step 0 shows the real rate by week 3, before we build. A cap of 30 AI replies per chat, then a person. |
| **One customer's content** | Adult, drug or scam content can get our *whole* provider account closed, with no refund. | Admins check what the brand sells before turning it on (list below). Every AI message passes a content check. Watch opt-outs and "who is this" replies per customer. Sub-account and kill switch per customer. |
| **Apple ends the channel** | Apple blocks the providers' method for everyone, overnight. | Leads fall back to the next channel automatically. We lose the blue bubble, not the leads. See section I. |
| **Provider down** | Every send fails and retries 3 times, so carts wait. Replies sent while webhooks are down are lost. | If most sends fail for 5 minutes, a circuit breaker turns iMessage off for everyone and steps go on the next channel. It turns back on after a test send works. The catch-up job collects missed replies. |
| **Not enough lines** | Carts go cold within an hour, and 50 a day fills fast. | One line per store in v1. If its daily or hourly cap is full, the AI picks the next step at once, with no waiting. The pilot sizes the iMessage group to fit the line (section F). |
| **Failed iMessage** | A failed message waits while the cart goes cold, or a resend reaches the lead twice. | If we know the iMessage didn't arrive (refused after 10 s, 30 s and 60 s retries, failed, not on iMessage), the AI gets it at once and picks the next step, as after a failed SMS today. If we don't know (no delivery status in 15 minutes), we don't resend: the lead is marked, and the next steps run as usual. Live chat answers go first in the queue, before normal steps. |
| **Lead not on iMessage** | Android (about 4 in 10 US phones), iMessage turned off, numbers outside the US, numbers that moved to Android but are still registered with Apple. | Look the number up before the first send and cache it for 30 days. "Not on iMessage" → the AI picks the next step at once. No delivery status within 15 minutes → no resend; the cache is cleared and the next steps run as usual. If the lookup itself fails, the lead gets the campaign as usual. v1 covers US/CA only. See section C. |
| **Cost** | Lines are a fixed monthly cost. Long chats also cost more in AI calls. | The pilot splits iPhone leads 50/50, with and without iMessage. See section F. |
| **Opt-out** | iMessage has no carrier STOP. | We detect STOP, "unsubscribe" and similar ourselves, opt the lead out on all channels and send one confirmation. |
| **Webhooks** | Duplicates, late or out-of-order events, missed retries. | Check the signature, skip events we've seen, save then queue. A catch-up job every 5 min pulls missed events and messages with no delivery status after 15 minutes. |
| **State law** | Quiet hours and frequency limits, AI disclosure. | See section E. |
| **Idle lines** | Apple turns off lines with no traffic for about 2 months. | The health job flags idle lines. |

**Brands that don't get iMessage in v1:** prescription drugs (including erectile-dysfunction and weight-loss drugs), drug-like health claims, adult products, cannabis, CBD, vape, tobacco and alcohol, gambling, crypto, loans and debt, weapons, and anything counterfeit. These follow what US carriers already ban or restrict on SMS. Such brands keep email and calls.

**Line health, starting values** (tuned in the pilot):
- Checked every 15 min: the provider's `line_blocked` event, blocked or declined errors, delivery rate over the last 100 sends, failures in a row, replies against sends, opt-out rate, days with no sends.
- **Yellow:** delivery under 90%, or opt-outs above 2%. The daily new-chat cap halves, and Slack gets an alert.
- **Red:** the provider says blocked, 5 failures in a row, or delivery under 70%. No new leads on the line, its chats move to the next channel, and Slack and the customer are told. The ops on-call replaces the line within one working day.
- A replacement line starts its warm-up again, so with one line per store, the store is back to 10 new chats a day for 2–3 weeks. More of its leads get the campaign as usual meanwhile.
- A weekly Slack summary per line catches slow decline.

## C. Android

About 4 in 10 US phones in use are Android (StatCounter, mid-2026: iOS 58–61%, Android 39–42%). The share differs a lot by store: about 87–88% of US teens own an iPhone (Piper Sandler, 2025).

- **iMessage can only reach the iPhone share.** At best it lifts about 6 in 10 US leads. The rest stay on the campaign's other channels, and we sell it that way.
- **Pilot stores with iPhone-heavy shoppers.** We ask each candidate store for its iPhone share of mobile visits; any analytics tool shows it. A line priced per month pays off poorly for a store at 40%.
- **Compare iPhone with iPhone.** See section G.
- **Numbers that moved to Android.** Apple can keep a number registered for iMessage after its owner leaves iPhone, until they deregister it. Messages to it vanish, or land on an old iPad or Mac. The 30-day cache makes this worse. No delivery status within 15 minutes clears the cache, and the next steps run as usual. The "two unanswered, then next channel" rule catches the rest.
- **SMS to Android is changing too.** Google Messages warns about links from unknown senders and hides suspected scams. It was piloted in 2024 in a few countries, with a wider rollout planned. Our SMS results on Android may get weaker either way.
- **RCS is the answer for Android.** Business RCS now runs on all major US carriers, through Google Messages on Android. It shows a verified brand name and logo, and allows links and buttons. Carrier approval takes 2–6 weeks. It is the natural next channel for the leads iMessage can't reach. Not confirmed: which US carriers show business RCS on iPhone.
- **Other markets:** Lithuania is about 63–70% Android, and Australia about 60–64% iPhone. iMessage fits the US, Canada and Australia. In most of the EU, WhatsApp, already a channel, matters more.

## D. Edge cases beyond the plan's table

- **A reply or a purchase while a step waits in the queue.** The step is checked again right before it is sent, and dropped if it's no longer wanted (diagram B).
- **The lead writes to the other number.** After a fallback, a lead may still write to the old iMessage line. We answer where the lead last wrote, with the whole chat history.
- **A line is blocked mid-chat.** Its chats move to the next channel. If that's SMS, the first SMS says it's the same store and why the number changed.
- **The lead replies from an Apple ID email.** Rare, because we start the chat on their number. If the reply can't be matched, it goes to the inbox and the store links it (diagram C).
- **An opted-out lead writes again.** A person sees it. No automatic messages until the lead opts back in.
- **"Is this a bot?"** The AI says it's the store's assistant and offers a person.
- **Thumbs-up and other reactions.** They count as engagement in line stats, but get no answer.
- **One company, several brands.** Lines belong to a brand, so a lead always sees one brand per number.
- **Bursts.** People send 3 messages in a row. Typing dots, wait 15 s, then one answer.
- **Photos and voice notes.** They go to a person. The AI can't hear a voice note.
- **Calls during a chat.** Today AI calls run on schedule. No call while the lead is chatting on iMessage, because two channels at once looks broken.
- **Opt-out scope.** An opt-out covers every channel of that store, not other stores on Outcraft, because consent is given per brand.

## E. Law

- **Treat iMessage exactly like SMS.** Courts treat texts as "calls" under the TCPA, and there is no ruling that iMessage is different.
- **Which state:** the shipping address if we have it, else the area code. If they differ, the stricter rule applies.
- **Quiet hours:** 21 states set hours for sales texts. Florida, Maryland, Oklahoma, Washington and others allow 8am–8pm local time. Texas allows 9am–9pm, and from noon on Sundays. Postscript reads Florida's law as covering a brand's replies too. So "reply right away" stays inside state hours (the scheduler for B, diagram C for live replies).
- **Who checks:** the scheduler fires steps only inside sending hours, and those must sit inside state law. The default 9:00–17:00, Mon–Fri, in the lead's timezone fits every rule above. If the scheduler doesn't check state law itself, pilot campaigns keep the default hours, and our admins check this when they turn iMessage on. Live replies skip the scheduler, so diagram C checks legal hours itself, with the same rule.
- **Frequency:** Florida, Maryland and Oklahoma allow at most 3 messages per 24 hours on the same subject. A chat of up to 30 replies may break that. The pilot leaves these three states out until legal decides.
- **Opt-out (FCC, since 11 April 2025):** an opt-out sent by any reasonable means must be honoured within 10 business days. The "revoke all" part is delayed to 31 January 2027. We opt out at once and on all channels, so we already meet it.
- **Texas SB 140 (since 1 September 2025):** marketing texts now fall under the Texas solicitation law. There is a registration duty, with exemptions for current or former customers, and a private right of action reported at $500–$10,000 per violation. A cart abandoner who never bought may not count as a former customer. Legal check needed.
- **AI disclosure:** Maine (since September 2025) requires clear notice when a consumer could think they are talking to a human. California's SB 1001 covers bots that hide being a bot to sell. So the first message names the brand and says it's an assistant.
- **GDPR:** Outcraft is established in Lithuania, so GDPR covers its processing even for US leads (Art. 3(1)). We need a DPA with the provider and list it as a sub-processor.

## F. Sizing and cost

- **v1: one line per store**, like its one SMS number. A warm line takes about 40 new chats a day: the cap is 50, and the rest is headroom for spreading sends out and for retries. More lines per store wait until pilot data shows they pay off, because they need line choice and pinning each lead to one line.
- **With Blooio's reply rule:** if a line's new chats get under 40% written replies, Blooio stops its new chats until it is back to about 10 a day, and a line costs about 4× more per lead. Then the pilot needs about 6 more carts per 100, not 2 (section H). The test shows which case applies by week 3.
- **Example:** a store with 300 carts a day with consent and a US/CA number. If 55–70% are on iMessage, that is 165–210 leads a day. Its one line takes about 40 of them; the rest are the pilot's comparison group. At full scale it would need 5–6 lines.
- During warm-up a line takes only 10, then 25, new chats a day. The 3 lines warm up in step 0, so the pilot starts at full volume.
- **Cost per line:** about $195–$290 a month (Blooio list price, Linq reported at about $250, Sendblue on request). At about 1,200 new chats a line a month, that is $0.15–$0.25 a lead, against about $0.01 for SMS.
- **Break-even:** $200 ÷ (1,200 leads × $80 average order) ≈ 0.2 percentage points, or **2 more carts recovered per 1,000 leads**. The $80 is an assumption. That counts all the store's recovered revenue. If Outcraft pays for the line and keeps only its share (say 15%), it needs about **14–17 more carts per 1,000 leads** ($200–$250 a line), about 1.5 points. If the store pays for the line, it needs about 2.5 per 1,000 counting revenue, or about 5 counting profit at a 50% margin. The go bar of 2 points (20 per 1,000) covers every case.
- **Pilot cost:** 3 lines, one per store, for about 2 months (step 0 and the pilot): roughly $1,730 at $289 a line. We pay it, so stores can join easily.
- **Other costs:** longer chats mean more AI calls, ops time for line swaps, and support when a line is blocked.

## G. Pilot and go/no-go

Two-sided test, 5% significance, 80% power. The baselines without iMessage (6% reply rate, 10% recovery) are assumptions until we see Outcraft's own data.

| What we want to prove | Leads needed (both groups) | Days at 100 eligible a day | Days at 300 a day |
|---|---|---|---|
| Reply rate 6% → 12% (2×) | 712 | 8 | 3 |
| Reply rate 6% → 9% | 2,418 | 25 | 9 |
| Recovery 10% → 15% (vendor claim) | 1,372 | 14 | 5 |
| Recovery 10% → 12% | 7,682 | 77 | 26 |
| Recovery 10% → 11% | 29,502 | 296 | 99 |

- **What we compare:** the iMessage group against iPhone leads without it, who get the campaign without iMessage, usually SMS. So the test answers the product question: does adding iMessage recover more carts?
- **Go:** recovery at least 2 points higher with lines at full speed, or about 6 points if Blooio holds them at 10 a day (section H). Replies only decide the setup, not the go.
- **Guardrails:** opt-outs no higher than without iMessage, and under 1 blocked line per 10 line-months.
- **Revenue after line costs** is tracked, but proving a 1-point lift needs about 30,000 leads, so it shows direction only.
- **Both groups are iPhone leads.** Leads are split only after the lookup confirms iMessage. iPhone and Android shoppers differ in age and income, so comparing with all leads would flatter iMessage.
- **One line per store sets the split.** The iMessage group is what the line takes (about 40 a day per store), and the group without iMessage is bigger. At 3×, for 10% → 12%, that cuts the iMessage leads needed from about 3,840 to about 2,560: about 4 weeks from the first message with 3 stores, warm-up included. With fewer stores, the pilot runs longer.
- Test shoppers count too, since the setup doesn't change. If the pilot stores have fewer than about 300 eligible leads a day, we run longer or add a store.

## H. Rollout

| Weeks | Phase | The question it answers |
|---|---|---|
| 1 | **Count**, paperwork, the test module (~3 days) | Are there enough eligible shoppers? |
| 2–3 | **Test:** the module sends fixed messages; replies get the cart link | Do they reply? |
| 4–6 | **Pilot:** the same setup keeps running, lines at full speed | Does adding iMessage recover more carts? |
| 7 | **Decision** | Go or no-go |
| After a go | **Full build:** replies (diagram C), then sending (diagram B), 4 weeks | — |

**Why this order:** only a 3-day module is built before we know shoppers reply and buy. The test and the pilot are one run with two checkpoints, so shoppers from the test count toward the pilot. The pace is set by the vendor notice (10 days), the warm-up (3 weeks) and the shoppers needed, not by the build.

**Week 1: count and set up**
- Provider contract, data agreement (DPA), legal check on state rules. Blooio goes on Outcraft's sub-processor list on day 1, because that must be 10 days before any shopper data reaches it (Outcraft's DPA, clause 7.4.3).
- **Count the eligible shoppers** (a few days): carts with consent and a US/CA number, times each store's iPhone share from its analytics. A lookup on about 500 numbers confirms it once the notice has run. Stop if a store has fewer than ~30 a day. Pick 3 stores.
- **Build the test module** (about 3 days), inside our app behind an on/off switch.

**The test module**
- **Sending:** it takes the store's abandoned-cart event. It checks consent, a US/CA number, not opted out, Do Not Call, not in FL, MD or OK, and 9:00–17:00 local time. It asks Blooio whether the number is on iMessage, and sends a fixed message to a random group of iPhone carts, up to the line's daily cap (10 a day at first, about 40 after 3 weeks). Once Blooio accepts the message, the shopper's campaign texts pause, so nobody hears from two numbers. The rest get the campaign as usual.
- **Four fixed messages, nobody answers by hand:**
  1. **First message:** "Hi Sam, this is the Glow Shop assistant. You left the blue scarf in your cart. Want me to save it for you?"
  2. **The shopper replies:** "Here's your cart: [link]. Questions? The Glow Shop team is at [store's support email]."
  3. **The shopper writes again,** once only: "Thanks! The Glow Shop team can help at [email]."
  4. **STOP and similar words:** the shopper is opted out on all of the store's channels through Outcraft's existing opt-out code, with one confirmation. Unclear cases ("please stop texting me") count as STOP.
- **Counting:** each shopper is tagged with their group. Replies, clicks and purchases come from Outcraft's existing tracking, so counting is a report, not new code. Every reply is saved, so we can read what shoppers ask.
- **Alerts:** Blooio's safety webhook posts line warnings to Slack.
- **Against diagrams B and C:**
  - **Kept from B:** the checks before a first message, the iMessage lookup, the random group, the line's daily and hourly cap, our message id against double sends. If Blooio refuses a send, the shopper simply stays on the campaign.
  - **Kept from C:** the reply webhook with its signature check and repeat skipping, automatic STOP, the two fixed answers.
  - **Waits for the full build:** AI writing and AI chat (typing dots, the 15 s wait, the 30-reply cap, handoff to a person), the fallback after a failed delivery (rare after the lookup, so only counted), retries, the health job and circuit breaker, the catch-up job, legal hours for replies (FL, MD and OK stay out of the test), iMessage steps and switches in campaigns.
- The module is switched off and deleted once the full build replaces it.

**Test (from about day 11) and checkpoint (end of week 3)**
- 3 US stores, one Blooio line each, switched on by our admins. We pay for the lines.
- **Stop only for harm:** a line flagged or banned, or opt-outs above SMS.
- **Why carts, not replies:** a shopper can see the message, open the store's app and buy without replying. Replies matter only for the line, through Blooio's rule: a line averaging more than 10 new chats a day with under 40% written replies is slowed, and after 48 hours (or at once under 20%) its new chats stop until it is back to 10 a day or 40% replies.
- **The reply rate picks the setup.** The lines pass 10 new chats a day in week 2, so it shows by week 3:

| Written replies | Setup | Pilot bar (more carts per 100) |
|---|---|---|
| 40% or more | Blooio at full speed, ~40 new chats a line a day | 2 |
| Under 40% | Blooio held at 10 a day, where the rule doesn't apply; about 4× the cost per shopper | about 6 |
| Under 40% | or Linq at full speed: no reply-rate rule, but a higher ban risk and new lines (~3 weeks of warm-up) | 2 |

- **The pilot still fits.** A bigger lift needs fewer shoppers: about 6 more carts per 100 needs only ~330 shoppers with iMessage, about 11 days at 30 a day.
- **Early win:** by week 3 about 700 shoppers have had iMessage, enough to spot a big lift (around 5 points) but not a 2-point one. A big early lift is a strong go signal; otherwise the week-7 decision stands.
- **Limit:** very low replies may also look like spam to Apple, whose thresholds nobody publishes. That's why a flagged line still means stop.
- **If cold messages get a line flagged:** one more try on a new line, with a slower ramp, and iMessage only right after a missed call, when the shopper has just heard from the brand. Flagged again → stop.
- **Otherwise stop**, or look at RCS.

**Pilot (weeks 4–6)**
- The same setup keeps running. The lines reach full speed, about 40 new chats a day each.
- A random split among iPhone shoppers only, sized to each store's line: about 40 a day get iMessage, the rest the campaign without it (section G). It runs until about 2,560 shoppers have had iMessage, test shoppers included.
- Replies get the same fixed answers. A real AI chat would also answer questions in the thread, so the pilot slightly understates iMessage. That's the safe direction: if a plain link wins, the full product should do better.
- **We measure:** replies, clicks, recovered carts, opt-outs, blocked lines, lookup accuracy, how often a line was full, and what shoppers ask.

**Decision (week 7)**, once the last pilot carts have had a few days to buy.
- **Go if**, against iPhone shoppers without iMessage: recovery at least 2 points higher at full speed (about 6 if lines were held at 10 a day), opt-outs no higher, and under 1 blocked line per 10 line-months (section G).
- **Go →** the full build, then a paid add-on per line, more stores, later self-serve.
- **No-go →** stop, or look at RCS.
- **Before growing beyond the pilot:** if our customers' security reviews ask for SOC 2 and Blooio still has none, we move to Linq.

**Full build after a go (4 weeks, one engineer)**
- **Replies first (diagram C), weeks 1–2:** provider interface, webhooks with the catch-up job, STOP handling, chat rules (typing dots, 15 s wait, a cap of 30), content check.
- **Then sending (diagram B), weeks 3–4:** the channel check before writing, the iMessage message, iMessage text steps, failures reported to the AI, per-line limits, no-link rules, health alerts, the random split.
- **SMS-only code:** if the shared send check sits inside the SMS code, we pull it out first, about one more week.
- The test module keeps running meanwhile, so the 3 stores don't lose iMessage.

**Other notes:**
- **If Outcraft already runs iMessage:** skip the test and use that data.
- **Effort:** a 3-day module before the decision, 4 weeks after a go. Lines: 3, about $1,730 up to the decision.

## I. Platform risk

- Apple's iCloud terms say the service is for personal use, and forbid automated access and promotional messages. Every iMessage provider runs real Apple IDs against these terms. Outcraft isn't a party to them, but the channel exists only while Apple tolerates it.
- Precedents: Apple cut off Beeper Mini in December 2023 (a consumer app using fake credentials, a different case), and Lindy's own account was banned within half a day.
- I found no sign of Apple shutting down managed providers in 2025–26, and Sendblue has run for 5+ years.
- **So: low chance, high impact.** Fallback to the next channel runs everywhere, a global kill switch exists, and we never sell iMessage as guaranteed.

## J. What waits, and ideas for later

**Waits in v1, and why:**
- **Self-serve and billing:** needs pilot numbers first.
- **More than one line per store:** needs line choice and pinning each lead to one line. One line fits the pilot.
- **Waiting for a free slot:** in v1, no slot on line X tells the AI at once, like every other "no". If the pilot shows many of these (for example, when sending hours open), we add a short wait for the next slot, at most an hour.
- **Moving a chat to a new line:** the next channel covers it in v1.
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
3. Does the SMS consent we collect cover iMessage? **If not:** we add iMessage to the consent text first, and the pilot waits for it.
4. Do the contact-hour and frequency controls already cover state rules (FL, MD, OK, TX)? Do they apply to AI replies? **If not:** pilot campaigns keep the default 9–17, Mon–Fri, which fits every state rule we found (section E), and C checks legal hours for AI replies.
5. Where is consent stored?
6. Is there a common channel interface? **If not:** before the full build we pull the shared send check out of the SMS code, about one more week (the "SMS-only code" risk).
7. When an SMS fails today, does the AI get it and pick the next step? We assume so, from the flow in the brief. **If not:** iMessage failures go wherever failed SMS go today.
8. Before the AI writes, does Outcraft check which channels a lead can get? **If yes:** iMessage joins that check. **If not:** we add one step before writing, the channel check.
9. What share of our stores' shoppers use iPhones?
10. Have you looked at RCS for Business?
11. How much downside is acceptable if Apple cut iMessage off overnight?
12. Do you already use Blooio, or have you talked to them?
13. Does Outcraft record recovered carts per shopper? **If yes:** the test only tags each shopper's group and counting is a report. **If not:** the module records purchases itself.

## Sources

- **Providers:** [Sendblue pricing](https://sendblue.com/pricing) · [Sendblue limits](https://docs.sendblue.com/limits/) · [Sendblue API summary](https://docs.sendblue.com/llms.txt) · [Sendblue Recover](https://recover.sendblue.com/) · [Sendblue cart guide](https://www.sendblue.com/blog/imessage-ecommerce-abandoned-cart) · [Sendblue vs Blooio](https://www.sendblue.com/compare/sendblue-vs-blooio) · [Messages for Business vs API](https://www.sendblue.com/blog/apple-messages-for-business-vs-imessage-api) · [Blooio pricing](https://blooio.com/integrations/api/pricing) · [Blooio vs Sendblue](https://blooio.com/alternatives/sendblue) · [Linq rate limits](https://docs.linqapp.com/guides/platform/rate-limits/index.md) · [Linq vs Sendblue](https://linqapp.com/s/linq-vs-sendblue) · [Linq flagged number](https://help.linqapp.com/flagged-number) · [Photon deliverability](https://photon.codes/docs/best-practices/imessage-deliverability) · [Sendblue incident report](https://www.sendblue.com/blog/sendblue-security-incident-update) · [Linq phone reputation](https://docs.linqapp.com/guides/phone-numbers/phone-reputation/) · [Linq sending and failover](https://docs.linqapp.com/channel/imessage/guides/messaging/sending-messages/index.md) · [Linq iMessage API](https://linqapp.com/imessage-api) · [Blooio messaging safety](https://docs.blooio.com/guides/messaging-safety) · [Blooio API pricing](https://blooio.com/integrations/api/pricing) · [Photon pricing](https://photon.codes/pricing)
- **Apple:** [iCloud terms](https://www.apple.com/legal/internet-services/icloud/) · [Unknown senders, iOS 26](https://support.apple.com/en-in/125068) · [Twilio: iOS 26 and business messaging](https://www.twilio.com/en-us/blog/insights/trends/potential-ios-26-update-communications-implications) · [iMessage link protection (Bitdefender)](https://bitdefender.in/iphone-users-tricked-into-disabling-apple-imessages-phishing-protection) · [Deregister iMessage (Apple)](https://selfsolve.apple.com/deregister-imessage) · [Beeper Mini (MacRumors)](https://www.macrumors.com/2023/12/10/apple-confirms-it-shut-down-beeper-mini/) · [Lindy: three rewrites, one Apple ban](https://www.lindy.ai/blog/imessage-api-three-rewrites-one-apple-ban-and-what-actually-works)
- **Android and RCS:** [US mobile OS share (StatCounter)](https://gs.statcounter.com/os-market-share/mobile/united-states-of-america) · [Teen iPhone ownership (MacRumors, Piper Sandler)](https://www.macrumors.com/2025/04/09/teen-iphone-ownership-continues-to-soar/) · [Lithuania mobile OS share (StatCounter)](https://gs.statcounter.com/os-market-share/mobile/lithuania) · [Australia mobile OS share (StatCounter)](https://gs.statcounter.com/os-market-share/mobile/australia) · [Google Messages protections (Google Security Blog)](https://security.googleblog.com/2024/10/5-new-protections-on-google-messages.html) · [RCS on all US carriers (Twilio)](https://www.twilio.com/en-us/blog/products/RCS-ATT-Integration) · [RCS on iOS 18.1 (Twilio)](https://www.twilio.com/en-us/blog/insights/trends/rcs-business-messaging-apple-update) · [Apple and RCS (Sinch)](https://sinch.com/blog/apple-support-rcs/) · [Attentive RCS](https://ecommercenews.com.au/story/attentive-launches-ai-tools-for-rcs-message-rollout)
- **Law:** [Postscript: Florida rules](https://postscript.io/blog/florida-compliance-changes) · [State quiet hours (Referrizer)](https://support.referrizer.com/article/tcpa-quiet-hours-and-state-specific-rules-for-text-campaigns) · [FCC revoke-all delay (BHFS)](https://www.bhfs.com/insight/fcc-extends-compliance-deadline-for-revoke-all-rule/) · [Texas SB 140 (Morgan Lewis)](https://www.morganlewis.com/pubs/2025/09/texas-telephone-solicitation-law-now-covers-text-messages) · [AI disclosure laws (DLA Piper)](https://www.dlapiper.com/en/insights/publications/2026/01/ai-disclosure-laws-on-chatbots-are-on-the-rise-key-takeaways-for-companies)
- **Market:** [US iPhone sales share (MacRumors, Counterpoint)](https://www.macrumors.com/2026/05/13/apple-q1-market-share-growth/)
