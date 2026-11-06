# iMessage as a new channel: plan

Tadas Vinckas · [Diagrams](diagrams.md)

In short: iMessage could get more iPhone shoppers to reply and buy. But it costs much more than SMS, and Apple can block a line that looks like spam. So we test before we build. First we check that enough shoppers use iMessage. Then a small test in three stores shows whether they reply and buy. Without enough replies, it won't pay off. After seven weeks we decide whether it can win enough extra carts over SMS to pay for its higher cost. If it can, we build the full version.

## 1. Provider: which one, and why

- **Today:** no provider yet. I compared five, from Blooio to running our own Macs.
- **Suggestion: Blooio**, for the test and the pilot.
- **Why:**
  - **Safe:** it blocks messages that break its rules, like a link before a reply. A bug on our side costs one message, not a blocked line.
  - **Fast to start:** $289 a month per line (one number), no setup fee, live in days.
  - **Early warning:** it tells us when a line is at risk, so we can slow down.
- **Not Linq for now:** it only warns and leaves stopping to us. It's our backup.
- **Not Sendblue:** it sells cart recovery to stores, so it competes with us. Also a 2026 breach.
- **Not our own Macs:** nothing would catch our mistakes, and every ban would be ours to fix.

## 2. Risks: what can go wrong, and what we do

| Risk | What can go wrong | What we do |
|---|---|---|
| **Apple blocks a line** | Spam reports or sudden bursts get it blocked, and its chats stop. | New lines start slowly. Only shoppers who agreed to texts, never at night. |
| **Two numbers** | Texts and iMessages share one app. A second store number looks like a scam. | Each shopper hears from one number. If a chat moves to SMS, it says it's the same store. |
| **First message** | An iPhone blocks links from unknown numbers until the shopper replies. Many messages with few replies look like a bot. | A short question first, the link after the reply. If replies stay low: fewer chats, or Linq. |
| **Same first message** | Blooio restricts every line when half our first messages start the same way. | Each first message is personal: store, product, varied wording. |
| **One bad store** | It could get our whole account closed. | We check what a store sells before turning it on. A line that goes wrong switches off on its own. |
| **STOP and state law** | No phone carrier handles "STOP" for us. Some states limit sending hours. | We catch STOP ourselves and follow the same rules as SMS. |

### Edge cases: where iMessage needs different rules

| Edge case | Today | Suggestion | Why |
|---|---|---|---|
| **Channel check** | The lead's profile shows which channels work. | Before the AI writes, check for iMessage and that the line can send. If not, write for the next channel. | No AI message is wasted. |
| **Calls instead of replying** | A call back to the number that texted reaches the AI agent. | Blooio forwards the line's calls to the campaign number. AI calls show the line's number. | One number for chat and calls, as with SMS today. |
| **Short texts in a row** | The AI probably waits a few seconds for more texts. | Wait a bit longer than on SMS, about 15 seconds, then answer once. If we see them typing, keep waiting. | It feels like chat, so shoppers send more short texts. Our dots make the wait feel fine. |
| **Typing dots** | SMS has no typing dots. | Show the shopper our typing dots while we wait and write. | They see a reply is coming and stay in the chat. |

## 3. Line health: how the team knows without a dashboard

- **Suggestion:** Slack tells the team when a line goes wrong. We check it with every message, so no extra job is needed. A blocked line stops at once and is replaced in a day.

## 4. Who turns it on: our admins, every store, or a plan

- **Today:** each campaign turns its own channels on: calls, SMS, email, WhatsApp.
- **Suggestion:** our admins turn it on, one store at a time. Not by stores yet: each new line needs a slow start and a content check, and one bad store could get everyone blocked.
- **In the pilot: no fee, only a share of the sales it wins.** We pay for the lines, so stores join fast and pay only when it works. Once the pilot shows the extra carts, we set the final price: a share or a paid add-on.

## 5. Which leads get it, and what happens to the rest

- **Suggestion:** before the AI writes, we check: iMessage is on for the store and campaign, the lead agreed to texts and has iMessage, and the line is healthy and under its daily chat limit.
- **Right before sending, we check the line again.** If it went down, no retry: the AI picks the next step, as after a failed SMS.
- **The rest** get the campaign as today, without iMessage. We log why.

## 6. First version and time

- **Suggestion: test first, build after.** Three Blooio lines cost about $1,730 until we decide.
  1. **Week 1:** check that enough shoppers use iMessage. Sign Blooio, give stores the required notice, and build a small test.
  2. **Week 2:** the test starts in three stores: a short, personal question, then the cart link after a reply. We count who buys.
  3. **Week 3:** stop if a line gets blocked or shoppers ask us to stop more than on SMS.
  4. **Week 7:** decide. Go if iMessage wins enough extra carts over SMS to pay for its higher cost. Then build the full version in a week or two.
