# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md) · [Appendix: details, numbers, sources](appendix.md) · [Questions for the team](appendix.md#k-questions-for-the-team)

- **Pilot:** 2–3 US stores for about 6 weeks, then go or no-go.
- **Phase 1:** shoppers who tap "Prefer iMessage?" write to us first. This warms up the lines.
- **Phase 2:** we send cart messages first, tested against SMS.

## 1. Provider: which one, and why

- **Today:** no iMessage provider. **Compared:** Linq, Sendblue, Blooio, Photon, our own Macs.
- **Suggestion: Linq**, if it shows its SOC 2 Type II report.
- **Why:** limits are the same, so terms decide: a line-health API, 99.95% uptime, no rival product.
- **Not the others:** Sendblue (2026 breach, rival product), Blooio (no SOC 2), Photon, own Macs.

## 2. Risks: what breaks at volume, and what we do

| Risk | What breaks | What we do |
|---|---|---|
| **Apple blocks a line** | Junk reports or bursts block the number. Its chats drop. | New chats ramp 10 → 25 → 50 a day. Consent only, no nights. |
| **No link before a reply** | iPhones disable links from strangers. | One short question first. The link after a reply, or by SMS. |
| **Lead not on iMessage** | About 4 in 10 US phones are Android. | We look up each number first. Not on iMessage → SMS, as today. |
| **Few replies** | A line that only sends looks like a bot to Apple. | If a line's replies halve, it starts no new chats. |
| **One bad store** | It can get our whole provider account closed. | Admins check each store. A kill switch per store. |
| **STOP and state law** | No carrier catches STOP. Some states limit hours. | We catch STOP ourselves and use the SMS law checks. |
| **Cost** | $0.20 a lead, against $0.01 for SMS. | It pays off for us at about 15 more carts per 1,000 leads. The pilot tests it. |

| Topic | Today | Suggestion | Why |
|---|---|---|---|
| **Sending hours** | Every step waits for sending hours. | Reply at once to a lead who just wrote, within state law. | A live chat, but some states ban late texts. |
| **Reply limit** | After 5 replies, a person takes over. | Keep chatting up to 30 AI replies, then a person. | Going silent mid-chat loses the lead. |
| **Sender number** | Each channel has its own number. | One number per lead, changed only on fallback. | A second number looks like a scam. |
| **Follow-ups** | Steps run on schedule. | A reply or purchase cancels them. | No "still thinking?" after "yes, buying". |

More edge cases (bursts, photos and voice, calls during a chat): [appendix D](appendix.md#d-edge-cases-beyond-the-plans-table).

## 3. Line health: how the team learns without a dashboard

- **Today:** nothing per number. **Suggestion:** a job checks each line every 15 min and acts:
  - **Yellow** (delivery under 90% or opt-outs over 2%): halve the daily limit, alert Slack.
  - **Red** (blocked, or delivery under 70%): no new leads, chats go to SMS, alert Slack.
- **Why:** the system acts first and Slack tells the team. People only replace lines.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** stores switch channels on in their settings.
- **Suggestion:** only our admins in v1, and we pay for pilot lines. Later, a paid add-on per line.
- **Why:** lines cost money and need warm-up, and one bad store can get us all banned.

## 5. Which leads get it, and what happens to the rest

- **Today:** every lead gets every channel in the campaign.
- **Suggestion:** only US/CA iPhone numbers with consent, split at random against SMS.
- **The rest**, Android included, get the next channel, as today. We log why.
- **Why:** the split compares iPhone users with iPhone users, and nobody gets less than today.

## 6. First version and time

- **Step 0, before building:** count shoppers, then one line by hand for 2–3 weeks (~$300).
- **Step 1, in build order:** replies (diagram C), then sending (diagram B), then the 2–3 store pilot.
- **How long:** 4 weeks of building for one engineer, 12 weeks from step 0 to the decision.
- **Blocks us:** the provider contract, legal on consent. **Could slip:** a ban in warm-up.
- **Where we simplified:** no self-serve, Slack not a dashboard, SMS fallback, US/CA, no media.
- **Go:** replies at least 2× SMS, then 2 points more carts recovered.
