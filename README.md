# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

- **Phase 1**, a pilot with 2–3 US stores: shoppers who tap "Prefer iMessage?" write to us first.
- **Phase 2:** we send cart messages first, tested against SMS. Then go or no-go.

## 1. Provider: which one, and why

- **Today:** none. **Compared:** Linq, Sendblue (2026 breach), Blooio (no SOC 2), Photon, own Macs.
- **Suggestion: Linq**, if it shows its SOC 2 Type II report.
- **Why:** limits are the same, so terms decide: a line-health API, 99.95% uptime, no rival product.

## 2. Risks: what breaks at volume, and what we do

| Risk | What breaks | What we do |
|---|---|---|
| **Apple blocks a line** | Junk reports or bursts block the number. Its chats drop. | New chats ramp 10 → 25 → 50 a day. Consent only, no nights. |
| **No link before a reply** | iPhones disable links from strangers. | One short question first. The link after a reply, or by SMS. |
| **Lead not on iMessage** | About 4 in 10 US phones are Android. | We look up each number first. Not on iMessage → SMS, as today. |
| **Few replies** | A line that only sends looks like a bot to Apple. | If a line's replies halve, it starts no new chats. |
| **One bad store** | It can get our whole provider account closed. | Admins check each store. A kill switch per store. |
| **STOP and state law** | No carrier catches STOP. Some states limit hours. | We catch STOP ourselves and use the SMS law checks. |
| **Cost** | $0.20 a lead, against $0.01 for SMS. | It pays off for us at about 15 more carts per 1,000 leads. |

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Sending hours** | Every step waits for sending hours. | Reply at once to a lead who just wrote, within the law. | Live chat: a next-morning answer loses the lead. |
| **Reply limit** | After 5 replies, a person takes over. | Keep the chat going, up to 30 AI replies, then a person. | Going silent mid-chat loses the lead. |
| **Sender number** | Each channel has its own number. | One number per lead, changed only on fallback. | A second number looks like a scam. |
| **Follow-ups** | Steps run on schedule. | A reply or purchase cancels them. | No "still thinking?" after "yes, buying". |

## 3. Line health: how the team learns without a dashboard

- **Today:** nothing is tracked per number.
- **Suggestion:** a job checks every line every 15 minutes and acts on its own:
  - **Yellow** (delivery under 90% or opt-outs over 2%): halve the daily limit, alert Slack.
  - **Red** (blocked, or delivery under 70%): no new leads, chats go to SMS, alert Slack.
- **Why:** the system acts first and Slack tells the team. People only replace lines.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** stores switch channels on in their settings.
- **Suggestion:** only our admins in v1, and we pay for pilot lines. Later, a paid add-on per line.
- **Why:** lines cost money and need warm-up, and one bad store can get us all banned.

## 5. Which leads get it, and what happens to the rest

- **Today:** every lead gets every channel the campaign has on.
- **Suggestion:** only leads with a US/CA number, consent and iMessage (we check first).
- **Phase 2:** half of them get iMessage and half SMS, picked at random.
- **The rest** (Android, no consent, no free line): the next channel, usually SMS. We log why.
- **Why:** iMessage works only on iPhones and costs 20× more. The split compares like with like.

## 6. First version and time

- **Step 0, before building:** count shoppers, then one line by hand for 2–3 weeks (~$300).
- **Step 1, in build order:** replies (diagram C), then sending (diagram B), then the 2–3 store pilot.
- **How long:** 4 weeks of building for one engineer, 12 weeks from step 0 to the decision.
- **Blocks us:** the contract, legal on consent. **Could slip:** a ban in warm-up, line setup.
- **Where we simplified:** no self-serve, Slack not a dashboard, SMS fallback, US/CA, no media.
- **Go:** replies at least 2× SMS, then 2 points more carts recovered.
- **Questions:** Is iMessage live? Does SMS consent cover it? Do channels share one send path?
