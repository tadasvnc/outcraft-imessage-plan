# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

- **Step 0:** count how many shoppers we can reach, then test one line by hand. Only then we build.
- **Pilot**, 2–3 US stores: we send cart messages by iMessage to half of the iPhone shoppers. The other half get the campaign without iMessage. Then go or no-go.

## 1. Provider: which one, and why

- **Today:** no provider yet. **Compared:** Blooio, Linq, Sendblue (2026 breach), Photon, own Macs.
- **Suggestion: Blooio**, with a data agreement, and its SOC 2 done before we scale.
- **Why:** public prices ($195 a line at 6+), live in days, hard limits that catch our mistakes.

## 2. Risks: what breaks at volume, and what we do

| Risk | What breaks | What we do |
|---|---|---|
| **Apple blocks a line** | Junk reports or bursts block the number. Its chats drop. | New chats ramp 10 → 25 → 50 a day. Consent only, no nights. |
| **No link before a reply** | iPhones disable links from strangers. | One short question first. The link after a reply, or on the next channel. |
| **Lead not on iMessage** | About 4 in 10 US phones are Android. | We look up each number first. Not on iMessage → the next channel, usually SMS. |
| **Few replies** | One-way lines look like bots. | The first message asks a question. Replies halve → no new chats. |
| **One bad store** | It can get our account closed. | Admin check, a sub-account per store. |
| **STOP and state law** | No carrier catches STOP. Some states limit hours. | We catch STOP ourselves and use the SMS law checks. |
| **Cost** | $0.20 a lead, against $0.01 for SMS. | It pays off for us at about 15 more carts per 1,000 leads. |

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Sending hours** | Every step waits for sending hours. | Reply at once to a lead who just wrote, within the law. | Live chat: a next-morning answer loses the lead. |
| **Reply limit** | After 5 replies, a person takes over. | Keep the chat going, up to 30 AI replies, then a person. | Going silent mid-chat loses the lead. |
| **Sender number** | Each store texts from its SMS number. | A lead on iMessage gets every text step, SMS ones too, from one line. Changed only on fallback. | SMS and iMessage share one app. A second number looks like a scam. |
| **Follow-ups** | Steps run on schedule. | A reply or purchase cancels them. | No "still thinking?" after "yes, buying". |

## 3. Line health: how the team learns without a dashboard

- **Today:** there are no iMessage lines yet, so there is nothing to watch.
- **Suggestion:** a job checks every line every 15 minutes and acts on its own:
  - **Yellow** (delivery under 90% or opt-outs over 2%): halve the daily limit, alert Slack.
  - **Red** (blocked, or delivery under 70%): no new leads, chats move to the next channel, alert Slack.
- **Why:** the system acts first and Slack tells the team. People only replace lines.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** each campaign switches its channels on: calls, SMS, email, WhatsApp.
- **Suggestion:** only our admins in v1, and we pay for pilot lines. Later, a paid add-on per line.
- **Why:** lines cost money and need warm-up, and one bad store can get us all banned.

## 5. Which leads get it, and what happens to the rest

- **Today:** a lead is reached on every channel their campaign has switched on.
- **Suggestion:** only leads with a US/CA number, consent and iMessage (we check first).
- **One number per lead:** in the campaign, iMessage is a fifth channel. On the phone, it shares the Messages app with SMS, so a lead hears from one number at a time.
- **Pilot:** half of them get iMessage, half the campaign without it, picked at random.
- **The rest** (Android, no consent, no free line): the next channel, usually SMS. We log why.
- **Why:** iMessage works only on iPhones and costs 20× more. The split compares like with like.

## 6. First version and time

- **Step 0, before building:** count shoppers, then one line by hand for 2–3 weeks (~$300).
- **Plan B:** if cold messages get that line flagged, leads write to us first, from a link in our SMS and email.
- **Step 1, in build order:** replies (diagram C), then sending (diagram B), then the 2–3 store pilot.
- **How long:** 4 weeks of building for one engineer, 12 weeks from step 0 to the decision.
- **Blocks us:** the data agreement, legal on consent. **Could slip:** a ban, few replies.
- **Where we simplified:** one pilot, no self-serve, Slack not a dashboard, existing channels as fallback, US/CA, no media.
- **Go:** against leads without iMessage, 2× the replies, then 2 points more carts recovered.
- **Questions:** Is iMessage live? Does SMS consent cover it? Do channels share one send path?
