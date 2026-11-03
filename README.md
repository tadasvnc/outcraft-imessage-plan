# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md) · [Appendix: details, numbers, sources](appendix.md) · [Questions for the team](appendix.md#k-questions-for-the-team)

- **Pilot:** 2–3 US stores for about 6 weeks, then go or no-go.
- **Phase 1:** shoppers who tap "Prefer iMessage?" write to us first. This warms up the lines.
- **Phase 2:** we send cart messages first, tested against SMS.
- **Why it's hard:** Apple has no API for this. A line is one real number, about 50 new chats a day.

## 1. Provider: which one, and why

- **Compared:** Linq, Sendblue, Blooio, Photon, our own Macs. All allow about 50 new chats a day.
- **Suggestion: Linq**, if it shows its SOC 2 Type II report.
- **Why:** line health in its API, a 99.95% uptime contract, no competing product.
- **Not the others:** Sendblue had a breach in July 2026 and sells cart recovery. Blooio has no SOC 2. Photon advises against messaging first. Our own Macs make every ban ours.

## 2. Risks: what breaks at volume, and what we do

| Risk | What we do |
|---|---|
| **Apple blocks a line** | Ramp new chats 10 → 25 → 50 a day, consent only. |
| **No link before a reply** | A short question first. The link after a reply, else SMS. |
| **Lead not on iMessage** | Look up each number first. If not → SMS, as today. |
| **Few replies look like a bot** | A line whose replies halve starts no new chats. |
| **One store gets us all banned** | Admins check each store. A kill switch per store. |
| **STOP and state law** | Our own STOP check, the SMS law checks, no FL/MD/OK. |
| **Cost:** $0.20 vs $0.01 a lead | Pays off for us at about 15 more carts per 1,000 leads. |

| Live chat | Today | Suggestion | Why |
|---|---|---|---|
| Hours | Steps wait | Reply at once, within state law | Live chat, but the law applies |
| Reply limit | 5 replies | 30, then a person | Silence loses the lead |
| Sender | Per channel | One number per lead | A new number looks fake |
| Follow-ups | On schedule | A reply or purchase stops them | No reminder after "buying" |
| Bursts | Each text | Wait 15 s, answer once | People send 3 in a row |
| Media | Text only | A person takes media | The AI can't hear voice notes |
| Calls | On schedule | No call during a chat | Two channels look broken |

## 3. Line health: how the team learns without a dashboard

- **Today:** nothing per number.
- **Suggestion:** a job checks every line every 15 minutes and acts on its own:
  - **Yellow** (delivery under 90% or opt-outs over 2%): halve the daily limit, alert Slack.
  - **Red** (blocked, or delivery under 70%): no new leads, chats go to SMS, alert Slack.
- **Why:** the system acts first and Slack tells the team. People only replace lines.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** stores switch channels on in their settings.
- **Suggestion:** only our admins in v1, and we pay for pilot lines. Later, a paid add-on per line.
- **Why:** lines cost money and need warm-up, and one bad store can get us all banned.

## 5. Which leads get it, and what happens to the rest

- **Today:** every lead gets every channel in the campaign.
- **Suggestion:** only US/CA numbers with consent, on iMessage. Phase 2 splits them at random against SMS.
- **The rest**, Android included, get the next channel, as today. We log why.

## 6. First version and time

- **Step 0, before building:** count eligible shoppers, then one line by hand for 2–3 weeks (~$300).
- **Step 1, in build order:** replies (diagram C), then sending (diagram B), then the 2–3 store pilot.
- **Waits:** self-serve, billing, a dashboard, media, RCS, countries beyond US/CA.
- **How long:** 4 weeks of building, 12 from step 0 to the decision. The contract blocks us.
- **Could slip:** a ban during warm-up, legal answers.
- **Go:** replies at least 2× SMS, then 2 points more carts recovered.
