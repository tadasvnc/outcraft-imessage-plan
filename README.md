# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

In short: iMessage could get more iPhone shoppers to reply and buy, but it costs about 20 times more than SMS, and Apple can block a number that looks like spam. So we test before we build. First we check that enough shoppers use iMessage. Then a small test in three stores counts who buys. If too few shoppers reply, it may not pay off. After seven weeks we decide whether it wins enough extra carts to pay for itself.

## 1. Provider: which one, and why

- **Today:** no provider yet. I compared five, from Blooio to running our own Macs.
- **Suggestion: Blooio**, for the test and the pilot.
- **Why:**
  - **Safe:** it blocks messages that break its rules, like a link before a reply. A bug on our side costs one message, not a blocked number.
  - **Fast to start:** $289 a month per number, no setup fee, live in days.
  - **Early warning:** it tells us when a number is at risk, so we can slow down.
- **Not Linq for now:** it only warns and leaves stopping to us. It's our backup.
- **Not Sendblue:** it sells cart recovery to stores, so it competes with us. And a 2026 data breach.

## 2. Risks: what can go wrong, and what we do

| Risk | What can go wrong | What we do |
|---|---|---|
| **Apple blocks a number** | Spam reports or sudden bursts get it blocked, and its chats stop. | New numbers start slowly. Only shoppers who agreed to texts, never at night. |
| **Two numbers** | Texts and iMessages share one app. A second store number looks like a scam. | Each shopper hears from one number. If a chat moves to SMS, it says it's the same store. |
| **First message** | To stop scams, an iPhone won't let shoppers tap a link from a number they haven't saved or replied to. A number that sends a lot but gets few replies looks like a bot. | A short question first, the link after the reply. If replies stay low: fewer chats, or Linq. |
| **Same first message** | Blooio restricts every number when half our first messages start the same way. | Each first message is personal: store, product, varied wording. |
| **One bad store** | It could get our whole account closed. | We check what a store sells before turning it on. A number that goes wrong switches off on its own. |
| **STOP and state law** | No phone carrier handles "STOP" for us. Some states limit sending hours. | We catch STOP ourselves and follow the same rules as SMS. |

### Edge cases: where iMessage needs different rules

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Channel check** | The lead's profile shows which channels work, like phone or email. | Before the AI writes, also check for iMessage, inside today's check if there is one. | Only a lookup can tell. |
| **Calls instead of replying** | A call back to the number that texted reaches the AI agent. | Blooio forwards the iMessage number's calls to the campaign number. AI calls show the iMessage number. | One number for chat and calls, as with SMS today. |
| **Still typing** | The AI answers each message. | Wait a few seconds after the last message, with typing dots, then answer once. | People type in short bursts. Many answers look like a bot. |

## 3. Number health: how the team knows without a dashboard

- **Suggestion:** Slack tells the team when a number goes wrong. No extra job is needed: we check the number with every message. A blocked number stops at once and is replaced within a day.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** each campaign turns its own channels on: calls, SMS, email, WhatsApp.
- **Suggestion:** our admins turn it on, one store at a time.
  - **Not by stores themselves yet:** each new number needs a slow start and a content check, and one bad store could get everyone blocked.
  - **Not in a price plan yet:** the pilot first shows what it's worth. Later, it can be a paid add-on.

## 5. Which leads get it, and what happens to the rest

- **Suggestion:** a lead gets iMessage only if all of these are true:
  - iMessage is on for the store and the campaign;
  - the lead agreed to texts, hasn't asked us to stop, and isn't on Do Not Call;
  - the number is from the US or Canada, and the lookup says it has iMessage;
  - our iMessage number has room for a new chat today.
- **One number per lead:** with iMessage on, the campaign's texts go by iMessage. If one fails, the AI picks the next step, as after a failed SMS today.
- **The rest** get the campaign as today, without iMessage. We log why.

## 6. First version and time

- **Suggestion: test first, build after.** Three Blooio numbers cost about $1,730 until the decision.
  1. **Week 1:** check that enough shoppers use iMessage. Sign Blooio, give stores the required notice, and build a small test.
  2. **Week 2:** the test starts in three stores: a short, personal question, then the cart link after a reply. We count who buys.
  3. **Week 3:** stop if a number gets blocked or shoppers ask us to stop more than on SMS. Otherwise keep going.
  4. **Week 7:** decide. Go if iMessage wins about 2 more carts per 100 shoppers, enough to pay for the numbers. Then build the full version in a week or two.
