# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

In short: iMessage could get more iPhone shoppers to reply and buy, but it costs about 20 times more than SMS, and Apple can block a number that looks like spam. So we test before we build. First we check that enough shoppers use iMessage. Then a small test in three stores counts who buys. If too few shoppers reply, it may not pay off. After seven weeks we decide whether it wins enough extra carts to pay for itself. Only then do we build it properly.

## 1. Provider: which one, and why

- **Today:** no provider yet. I compared five, from Blooio to running our own Macs.
- **Suggestion: Blooio**, for the test and the pilot.
- **Why:**
  - **Safe:** it blocks messages that break its rules, like a link before a reply. A bug on our side costs one message, not a blocked number.
  - **Fast to start:** a public price, no setup fee, live in days.
  - **Early warning:** it tells us when a number is at risk, so we can slow down.
- **Not Linq for now:** it only warns and leaves stopping to us. It's our backup.
- **Not Sendblue:** it sells cart recovery to stores, so it competes with us. And a 2026 data breach.

## 2. Risks: what can go wrong, and what we do

| Risk | What can go wrong | What we do |
|---|---|---|
| **Apple blocks a number** | Spam reports or sudden bursts get it blocked, and its chats stop. | New numbers start slowly. Only shoppers who agreed to texts, never at night. |
| **Two numbers** | Texts and iMessages share one app. A second store number looks like a scam. | Each shopper hears from one number. If a chat moves to SMS, it says it's the same store. |
| **No link at first** | iPhones turn off links from unknown senders. | The first message asks a short question. The link comes after the reply. |
| **Few replies** | A number nobody answers looks like a bot, and Blooio slows it down. | The first message invites a reply. If replies stay low: fewer chats, or Linq. |
| **One bad store** | It could get our whole account closed. | We check what a store sells before turning it on. A number that goes wrong switches off on its own. |
| **STOP and state law** | No phone carrier handles "STOP" for us. Some states limit sending hours. | We catch STOP ourselves and follow the same rules as SMS. |
| **Cost** | Each shopper costs about 20 times more than an SMS. | iMessage must win clearly more carts than SMS. The pilot measures that. |

### Edge cases: where iMessage needs different rules

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Channel check** | The lead's profile shows which channels work, like phone or email. | Before the AI writes, also check for iMessage, inside today's check if there is one. | Only a lookup can tell. |
| **Sending hours** | Every step waits for sending hours. | A reply goes out right away, but inside state legal hours. | A live chat can't wait until morning. Some states limit replies at night too. |
| **Reply limit** | After 5 replies, a person reviews and the AI stops. | Up to 30 AI replies, then a person. | Chats run long. Going silent mid-chat loses the lead. |
| **Calls instead of replying** | Not clear how it works: does the AI answer a call back to the number that texted? Asked the team. | Blooio forwards calls on the iMessage number to the store's support number. AI calls show the iMessage number. | One number for chat and calls. A number that doesn't answer loses the lead. |
| **Still typing** | The AI answers each message. | Wait a few seconds after the last message, with typing dots, then answer once. | People type in short bursts. Many answers look like a bot. |

## 3. Number health: how the team knows without a dashboard

- **Suggestion:** no extra job. Diagrams B and C check each number with every message, and Slack tells the team. A weak one starts fewer chats; a blocked one stops at once and is replaced within a day.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** each campaign turns its own channels on: calls, SMS, email, WhatsApp.
- **Suggestion:** only our admins, at first. Not every store yet: a new number needs a slow start and a content check, and one bad store could get everyone blocked. Not tied to a plan yet: the pilot first shows what it's worth. Later, it can be a paid add-on.

## 5. Which leads get it, and what happens to the rest

- **Suggestion:** only leads whose phone has iMessage. We check only where iMessage is turned on.
- **One number per lead:** with iMessage on, the campaign's texts go by iMessage. If one fails, the AI picks the next step, as after a failed SMS today.
- **Everyone else** (Android, no consent, number full for the day): the campaign as usual.

## 6. First version and time

- **Suggestion: test first, build after.** Three Blooio numbers cost about $1,730 until the decision.
  1. **Week 1:** check that enough shoppers use iMessage. Sign Blooio, give stores the required notice, and build a small test.
  2. **Week 2:** the test starts in three stores: one question, then the cart link after a reply. We count who buys.
  3. **Week 3:** stop if a number gets blocked or shoppers ask us to stop more than on SMS. Otherwise keep going.
  4. **Week 7:** decide. Go if iMessage wins about 2 more carts per 100 shoppers, then build the full version in a week or two.
