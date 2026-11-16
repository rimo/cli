# Rimo in Slack

[English](../en/rimo-in-slack.md) | [日本語](../ja/rimo-in-slack.md)

Ask about your meetings from Slack. Once the **Rimo** connector is on your Claude account, you can message Claude in Slack — *"What did we decide about pricing?"* — and get an answer read straight out of your Rimo meeting notes, without leaving Slack or opening Rimo.

You can ask what a meeting covered, what was decided, what the action items are, who attended, when it was held and how long it ran — or search back through past meetings for the one you only half-remember. See [Example things to ask](#example-things-to-ask).

**Start in a DM with Claude.** It's the easiest place to confirm the connector is working before you use it in a channel.

Claude runs in more than one place — **claude.ai** in the browser, the **Claude desktop app**, and **Claude in Slack** (also called **Claude Tag**). This page covers the **Slack** side.

> **In a channel, Claude asks before it uses your Rimo connector.** See [Use Rimo in a channel](#use-rimo-in-a-channel).

> Setting up the connector for the first time? Do that on [Rimo in Claude](rimo-in-claude.md) first — it walks through the connector directory and the Rimo sign-in. This page picks up from there. Using ChatGPT or Microsoft Copilot instead? See [Rimo in ChatGPT](rimo-in-chatgpt.md) or [Rimo in Microsoft Copilot](rimo-in-microsoft.md). On your own machine with a coding tool (Claude Code, Cursor, Codex)? See [Rimo in Coding Tools](setup-guide.md).

---

## Before you start

- A **Rimo account** — the same one you normally sign in with, in an organization.
- A **Claude account**. Claude Tag is available on the **Team** and **Enterprise** plans, **in beta**. To DM Claude, your seat must include Claude Code — or, on Enterprise, Cowork access on a Standard seat or a usage-based Chat seat.
- A **Slack workspace** with the **Claude app installed**. If it isn't installed, your Slack workspace administrator has to add it — see [If something doesn't work](#if-something-doesn-t-work).
- **The Rimo connector connected** to your Claude account — see [Rimo in Claude](rimo-in-claude.md).
- **Claude Tag set up for your workspace**, for channel use. Only a Claude **Primary Owner** or **Owner** can set it up — the plain **Admin** role can't. If you aren't an Owner, ask whoever manages your Claude workspace.

> You never type a Rimo password or key into Claude or Slack. You sign in on Rimo's own page, in your browser.

---

## Set up Claude in Slack

### 1. Install the Claude app in Slack

Add the **Claude** app to your Slack workspace from the [Slack Marketplace](https://claude.com/claude-for-slack). In most workspaces only a Slack administrator can install apps.

### 2. Find Claude in Slack

Search Slack for `claude`, then open the **Claude** app to start a direct message.

![Slack search with "claude" typed, showing the Claude agent in the results](../images/assets/slack-find-claude.png)

### 3. Connect your Claude account

The first time you open Claude in Slack, it asks you to connect your Claude account. Follow the prompt and **sign in with the same Claude account that has the Rimo connector connected** — otherwise Rimo won't be among Claude's tools, even though the setup looks complete.

![A DM with Claude in Slack showing the "Connect to Claude.ai" prompt and its Connect Account button](../images/assets/slack-connect-account.png)

### 4. Connect the Rimo connector

The connector is set up in **Claude's own settings**, not in Slack. If you haven't done it yet, open **[Customize → Connectors](https://claude.ai/customize/connectors)**, connect **Rimo Voice** (the Rimo connector's name in Claude's directory), then sign in to Rimo and pick an organization — [Rimo in Claude](rimo-in-claude.md) walks through it.

The same connector works in DMs and in channels. On the **Team** plan there's nothing extra to set up for channels. On the **Enterprise** plan, a Claude admin manages how personal connectors work in channels, and Claude Opus 5 must stay turned on for the organization — if Claude says it needs Claude Opus 5 to use your connectors, ask your Claude admin.

### 5. Check what Claude can reach

In the DM, ask:

```
Which connectors do you have available?
```

If **Rimo Voice** is in the list, Claude can reach your Rimo notes.

### 6. Ask your first question

In the DM, ask about a meeting the way you'd ask a colleague:

```
Search my Rimo notes for my latest meeting notes.
```

Claude finds the note and answers with what the meeting was, when it was held, how long it ran, who was there, the decisions, and your action items.

![A DM with Claude in Slack answering from the latest Rimo meeting note — the meeting's title, date, duration and attendees, followed by its decisions and action items](../images/assets/slack-rimo-query.png)

Claude replies in a thread and keeps working there, so you can follow up without repeating yourself. For decisions, action items, attendees, a specific topic, or an older meeting you need to dig up, see [Example things to ask](#example-things-to-ask).

You can ask from a channel too — see [Use Rimo in a channel](#use-rimo-in-a-channel).

---

## Use Rimo in a channel

In a channel, Claude uses your Rimo connector for **your own requests**, after you allow it.

1. In a channel where Claude is on, mention **@Claude** with your question:

   ```
   @Claude Search my Rimo notes for my latest meeting notes.
   ```

2. The first time, Claude asks in the thread — only you can see it — **"Can Claude use your connectors to work on that request?"** Pick one:
   - **Allow** — Claude starts right away and checks with you before posting anything that looks sensitive.
   - **Allow with review** — you approve every message before it posts.
   - **Don't allow** — Claude doesn't use your connectors for this request.

   With **Don't ask again in any channel** ticked, Claude remembers **Allow** or **Allow with review** for your later requests. Change it any time in the Claude app's **Home** tab, under **Personal connectors in channels**.

   ![A Claude thread in a Slack channel asking "Can Claude use your connectors to work on that request?" with Allow, Allow with review and Don't allow buttons](../images/assets/slack-channel-consent.png)

3. With **Allow with review**, click **Review**, read Claude's draft, then click **Post** or **Don't post**.

> **Everyone in the channel sees what Claude posts**, including people who can't open that note in Rimo. Ask in a DM when the notes shouldn't be shared.

Claude only uses your connectors for messages **you** post in that thread — a teammate's request doesn't use your Rimo access. To stop it in a thread, click **Stop** under Claude's message.

If Claude doesn't reply in a channel at all, see [Claude doesn't reply in a channel](#claude-doesn-t-reply-in-a-channel).

---

## How to ask in Slack

In a **DM with Claude**, or the Claude side panel from Slack's AI assistant header, just type your question — no mention needed.

From a **channel**, **tag @Claude** so it's sure to pick up your question. Once a thread is under way you **don't need to tag @Claude again** — it follows the thread and keeps the context of what you've already asked, so you can just reply:

```
Now group those action items by owner.
```

---

## Example things to ask

Ask about your meetings the way you'd ask a colleague:

- "Tell me about this meeting."
- "What were the main points discussed?"
- "What decisions were made?"
- "What were the action items?"
- "Who attended this meeting?"
- "How long did this meeting last?"
- "What did we discuss about the product launch?"
- "What did we decide about pricing?"
- "Find the meeting where we discussed the Q3 roadmap."
- "What did we discuss in our last meeting with the client?"

Say *Rimo* somewhere in the question — *"my Rimo notes"*, *"my last Rimo meeting"* — if Claude answers from the surrounding Slack messages instead of your meeting notes.

---

## Limitations

- **Answers in a channel are visible to everyone in it.** Ask in a DM when the notes shouldn't be shared — see [Use Rimo in a channel](#use-rimo-in-a-channel).
- **You only ever see your own notes.** Rimo shows Claude what **you** can see in Rimo; a note a colleague never shared with you won't appear.
- **One Rimo organization at a time** — the one the connector was signed in to.
- **Read-only.** Every Rimo tool only reads — nothing Claude does changes anything in Rimo.

---

## If something doesn't work

### I don't see Claude in Slack

The Claude app isn't installed in your workspace, or installing apps is restricted. **Contact your Slack workspace administrator** to have the Claude app added or approved — this isn't something you can enable from your own Slack account.

### Claude asks me to connect my account

Slack needs your **Claude account** linked before Claude can do anything. Follow the prompt and sign in, using the **same Claude account** that has Rimo connected.

### Claude answers but not from my Rimo notes

Work through these in order:

1. **Ask for Rimo explicitly.** Say *"search my Rimo notes for…"* or *"according to my Rimo notes"*. Without that, Claude may answer from the Slack conversation alone.
2. **Try it in a Claude DM.** A DM uses your connectors without asking, so it's the quickest way to verify Rimo access.
3. **Check the connector.** Confirm the Claude account Slack is linked to has **Rimo Voice** connected at **[Customize → Connectors](https://claude.ai/customize/connectors)**.
4. **In a channel, check your choice.** If you picked **Don't allow**, Claude answers without your connectors. Mention @Claude again and pick **Allow** or **Allow with review**. Your saved choice is in the Claude app's **Home** tab, under **Personal connectors in channels**.

### Claude doesn't reply in a channel

Claude isn't on in that channel yet. Ask a Claude **Owner** to set up Claude Tag for it at **[Admin settings → Claude Tag](https://claude.ai/admin-settings/claude-tag)**.

### Claude can't find a note I know exists

Claude only sees what your own Rimo account can see, in the one organization the connector was signed in to. If the note is in a different organization, reconnect and pick that one. A note a colleague never shared with you won't show up.

---

## Official Claude help

- [Claude Tag overview](https://claude.com/docs/claude-tag/overview)
- [What is Claude Tag?](https://support.claude.com/en/articles/15594475-what-is-claude-tag)
- [Personal connectors in channels](https://claude.com/docs/claude-tag/concepts/personal-connectors)
- [Use connectors to extend Claude's capabilities](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities)

---

## See also

- [Rimo in Claude](rimo-in-claude.md) — connect the Rimo connector (start here)
- [Rimo in Others](rimo-in-others.md) — every other tool Rimo works in
- [What you can do with MCP](mcp.md) — the tools and example prompts, once you are connected
- [Rimo in ChatGPT](rimo-in-chatgpt.md) — the same thing for ChatGPT
- [Rimo in Microsoft Copilot](rimo-in-microsoft.md) — connect through a Copilot Studio agent using DCR
- [Authentication](authentication.md) — how Rimo sign-in and accounts work
