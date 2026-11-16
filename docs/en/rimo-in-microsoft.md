# Rimo in Microsoft Copilot

[English](../en/rimo-in-microsoft.md) | [日本語](../ja/rimo-in-microsoft.md)

Connect Rimo to a **Microsoft Copilot Studio agent** and ask about your meeting notes in Microsoft Teams or Microsoft 365 Copilot. The agent looks things up in Rimo and answers:

- *"What did we decide in Monday's call, according to my Rimo notes?"*
- *"Summarise last week's client meeting from Rimo."*

You add Rimo to a Copilot Studio agent as a **tool** — a hosted MCP server. A maker adds Rimo once. After the agent is published, each employee opens it and signs in with their own Rimo account.

**The server URL to add:**

```
https://mcp.rimo.app/mcp
```

> **Already have a Copilot Studio agent?** You don't need to build a new one — jump straight to **[Add Rimo MCP to your agent](#add-rimo-mcp-to-your-agent)**, then come back for testing and publishing.

> Using Claude, ChatGPT, or Gemini instead? See [Rimo in Claude](rimo-in-claude.md), [Rimo in ChatGPT](rimo-in-chatgpt.md), or [Rimo in Gemini](rimo-in-gemini.md).

---

## How Microsoft organization setup works

There are three separate parts:

1. A **maker** (the person who builds and manages the agent) creates a Copilot Studio agent and adds the Rimo MCP as a tool.
2. An **administrator** approves the published agent so the whole organization can find it.
3. Each **end user** opens the agent and signs in to Rimo.

Rimo uses **per-user authentication**. Each user can only access notes their own Rimo account can see. The first time someone asks a question that needs Rimo, Copilot Studio asks them to sign in to Rimo; after that the connection stays active, and they're only asked again if it expires or is disconnected. See [Microsoft's user-authentication guide](https://learn.microsoft.com/en-us/microsoft-copilot-studio/configure-enduser-authentication).

The Microsoft accounts must belong to the **same Microsoft Entra tenant** for the organization rollout described here. Each user also needs their **own Rimo account**. The Microsoft email and Rimo email don't need to match — if they happen to match, that's fine too; it makes no difference.

---

## Create a Copilot Studio agent

Skip this section if you already have an agent — go to [Add Rimo MCP to your agent](#add-rimo-mcp-to-your-agent).

1. Sign in to [Microsoft Copilot Studio](https://copilotstudio.microsoft.com/) and select the environment you want.
2. Open **Agents** in the left menu, then select **New agent**. Copilot Studio creates an untitled draft and opens it in the **Build** view.
3. Select the name at the top and type the agent's name (for example, `Rimo`). In **Instructions**, describe how it should behave — see [Recommended agent instructions](#recommended-agent-instructions) below.
4. Copilot Studio configures the agent for **Microsoft Entra ID** authentication automatically. Leave it on.

See [Create and edit agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-first-bot) for the full walkthrough.

### Recommended agent instructions

Clear instructions keep the agent grounded in Rimo and stop it from inventing details. Paste something like this into the agent's **Instructions**, then adjust to taste:

```text
You help the signed-in user find and understand information from their Rimo meeting notes. You have read-only access to Rimo through the connected tools; the user's Rimo account and organization decide what you can see.

- For anything about the user's meetings, notes, transcripts, decisions, participants, or documents, call a Rimo tool before answering.
- Base every answer only on what the Rimo tools return. If a detail isn't in a tool result, say you don't have it — never guess or invent one.
- If nothing relevant is found, say so plainly. Rimo access is read-only; never claim you created, edited, shared, or deleted a note.
- If the meeting, person, team, or date is ambiguous, ask one short clarifying question first.
- Answer concisely: give the answer first, then brief supporting detail, and name the note title and meeting date you used.
```

---

## Add Rimo MCP to your agent

The following setup is done once by the maker, in the agent's **Build** view.

1. In the right-hand panel, select **+** next to **Tools**. The **Add a tool** dialog opens.

   ![The agent Build view with the Tools section marked in the right-hand panel](../images/assets/microsoft-open-tools.png)

2. In the dialog, select **+ Add** (top right), then choose **Model Context Protocol (MCP)**.

   ![The Add a tool dialog with the Add menu open and Model Context Protocol (MCP) marked](../images/assets/microsoft-add-tool-mcp.png)

3. Fill in the server details:
   - **Icon:** select the icon box and upload the Rimo logo (PNG, under 30 KB).
   - **Server name:** `Rimo`
   - **Server description:** `Rimo — list, read, search, and ask across your meeting notes, transcripts, and documents.`
   - **Server URL:** `https://mcp.rimo.app/mcp`
   - **Authentication:** **OAuth 2.0**
   - **Configuration type:** **Dynamic (with discovery)**

   ![The Add MCP server dialog with the Rimo icon, name, description, URL, OAuth 2.0, and Dynamic (with discovery) selected](../images/assets/microsoft-mcp-details.png)

4. Select **Add**. When Copilot Studio asks for a connection, create one (see [Sign in to Rimo as the maker](#sign-in-to-rimo-as-the-maker)), then select **Add** again.

See [Connect an existing MCP server to a Copilot Studio agent](https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent).

---

## Sign in to Rimo as the maker

Copilot Studio asks the maker to create a Rimo connection while adding the tool.

1. When the connection panel shows **Not connected**, open the dropdown and select **Create new connection**.

   ![The Select a connection dialog with the connection dropdown open and Create new connection marked](../images/assets/microsoft-select-connection.png)

2. Select **Create**.

   ![The Create a connection to use Rimo dialog with the Create button](../images/assets/microsoft-create-connection.png)

3. A Rimo page opens. Sign in with your usual Rimo account, choose the **organization** the connection may access, and select **Allow access**. The button stays disabled until you choose an organization.

   ![The Rimo consent page with the organization selector and the Allow access button](../images/assets/microsoft-rimo-approve.png)

4. Back in Copilot Studio the connection shows a green check. Select **Add**.

This maker connection is only for building and testing. It is **not** shared as the identity for all employees — each end user signs in with their own Rimo account after deployment.

---

## Set the tool to per-user sign-in

Confirm the Rimo tool signs each employee in with **their own** Rimo account, not the maker's.

1. In the right-hand panel under **Tools**, select **Rimo**. The **Edit tool** dialog opens — there is no separate **Edit** button.
2. In the top-right **Authentication** dropdown, choose **End user account**.

   ![The Edit tool dialog with Authentication set to End user account](../images/assets/microsoft-auth-mode-user.png)

**End user account** means every employee signs in with their own Rimo account and only sees their own notes. **Shared account** would make everyone share the maker's single connection — do not use it. Select **Done**.

---

## Test before publishing

1. Open the agent's **Preview** tab and start a new conversation.
2. Ask:

   ```text
   Show my latest Rimo notes.
   ```

3. The first time, Copilot may show a **Permission Required** card — select **Allow**. The agent then returns your notes.

   ![The Preview tab showing a table of returned Rimo notes](../images/assets/microsoft-test-panel.png)

---

## Choose your Microsoft plan

You've built and tested the agent — to **publish** it you need a publish-capable Microsoft plan. This table is about **Microsoft** — what your organization needs in Copilot Studio to build, publish, and run the agent. It is **not** a Rimo plan: Rimo doesn't charge for this and there's no special Rimo plan to buy — every user just signs in with their existing Rimo account.

| Microsoft setup | Who can build | Can publish? | What end users need | How usage is paid |
|---|---|---:|---|---|
| **Copilot Studio trial** | The trial user | No | Not applicable | Test panel only |
| **Microsoft 365 Copilot** — the add-on, Microsoft 365 Copilot Business, or Microsoft 365 E7 entitlement | A licensed user with environment access | Yes | For the assured Microsoft 365 Copilot experience, assign an eligible Microsoft 365 Copilot entitlement to each user | Included for authenticated employee-facing use, subject to fair-use limits |
| **Standalone Copilot Studio credit pack** | A maker with the free Copilot Studio User License | Yes | No special Copilot Studio license for users of the published agent; they still need access to the channel, such as Teams | Tenant Copilot Credits |
| **Copilot Studio pay-as-you-go or a pre-purchase plan** | A maker in the Copilot Studio Author group | Yes | No special Copilot Studio license for users of the published agent; they still need access to the channel | Tenant billing plan / Copilot Credits |
| **Copilot Studio Teams plan** | Classic-agent makers | Teams only | Teams access | **Not covered by this guide.** The Teams plan lacks Power Platform connectors, while Copilot Studio MCP access relies on them |

Important plan details:

- A **trial can create and test** the agent, but it **cannot publish** it.
- With a standalone credit pack, the administrator purchases tenant capacity and assigns the `$0` **Copilot Studio User License** to each maker.
- With pay-as-you-go or a pre-purchase plan, the administrator links billing to the environment and grants makers the **Copilot Studio Author** role.
- Users of a **published** agent do not need a special Copilot Studio license. If they don't have Microsoft 365 Copilot, their use consumes the tenant's Copilot Studio capacity or pay-as-you-go meter.

> **Runtime capacity for a published agent.** Building and testing draws almost nothing, but once the agent is live for the whole organization its use consumes Copilot Credits. Before you roll out widely, make sure the environment has **pay-as-you-go** enabled or a **Copilot Credit pack** allocated — otherwise users can hit *"This agent is currently unavailable. It has reached its usage limit."*

See [Copilot Studio licensing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing), [assign licenses and manage access](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-licensing), [Microsoft 365 Copilot licensing models](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/prerequisites#agent-capabilities-and-licensing-models), and the [June 2026 Copilot Studio Licensing Guide](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/bade/documents/products-and-services/en-us/bizapps/Microsoft-Copilot-Studio-Licensing-Guide-June-2026-PUB.pdf).

---

## Roles for publishing to the organization

The maker does not have to be a tenant administrator.

| Task | Minimum role or permission |
|---|---|
| Build, configure, and publish the agent | Agent Owner/Editor plus the Copilot Studio User License or Copilot Studio Author entitlement described above |
| Approve and manage the agent in the Microsoft 365 admin center | **AI Administrator** |
| Manage Teams app availability, automatic installation, and pinning | **Teams Administrator** |
| Use the published agent | Normal tenant user with access to the agent and its channel |

---

## Publish to Teams and Microsoft 365 Copilot

You need a publish-capable plan. A trial stops at the previous test step.

1. In the right-hand panel, select **+** next to **Channels**, then choose **Teams + Microsoft 365**.

   ![The Add a channel dialog with Teams + Microsoft 365 marked](../images/assets/microsoft-add-channel.png)

2. Under **Availability**, leave **Microsoft Copilot and Microsoft Teams** selected so the agent works in Microsoft 365 Copilot as well as Teams, then select **Add channel**.

   ![The Availability tab with Microsoft Copilot and Microsoft Teams selected](../images/assets/microsoft-publish-channel.png)

3. Select **Publish** (top right), then **Publish agent**. Wait for *"Your agent published successfully"*.

4. Select the **Teams + Microsoft 365** channel in the right-hand panel and, under **About info**, add the name, description, icon, privacy statement, and terms of use your organization requires. Publish again after any content change.

See [Connect and configure an agent for Teams and Microsoft 365 Copilot](https://learn.microsoft.com/en-us/microsoft-copilot-studio/publication-add-bot-to-microsoft-teams).

---

## Share the Rimo connector with your organization

This step is easy to miss and end users **cannot connect without it**.

Adding the Rimo tool creates a **custom connector** in your Power Platform environment, owned by the maker. By default no one else can use it, so end users hit *"Connector information unavailable"* and cannot create their Rimo connection. Share the connector once:

1. Open [Power Apps](https://make.powerapps.com/), select the environment that contains your agent, and go to **Custom connectors** in the left menu.

   ![The Power Apps Custom connectors list with the Rimo connector](../images/assets/microsoft-custom-connectors.png)

2. Open the Rimo connector, go to the **Share** tab, and grant access to your organization — **Share with org**, or add a security group that contains everyone who will use the agent.

   ![The Rimo custom connector Share tab with Share with org marked](../images/assets/microsoft-connector-share.png)

Leave the existing **Power Virtual Agents Service** entry in place — it's added automatically by the system and is required; don't remove it.

---

## Try it as an end user first

Before submitting the agent to the whole organization, test it the way a normal employee would. Use two accounts:

- **Account A — maker/admin:** a publish-capable Copilot Studio entitlement (not a trial), access to the agent's environment, and **AI Administrator** if it will also approve the org deployment.
- **Account B — normal end user:** an ordinary account in the same Entra tenant with no maker/admin role, Teams access, and its own Rimo account. (To test in the Microsoft 365 Copilot app rather than Teams, give it a Microsoft 365 Copilot license.)

### Maker / Account A

1. Select the **Share** icon in the top bar. The **Share** dialog opens.
2. In **Add a name, group, or email**, add Account B (or a security group that includes it) — the install link only works for users who have access.
3. Select **Copy sharing link** and send it to Account B.

Your tenant must allow Power Platform apps in Teams. If the link is blocked, ask a Teams Administrator to allow the agent.

### End user / Account B

> [!IMPORTANT]
> **Allow the sign-in pop-up.** Selecting **Connect** opens Rimo sign-in in a pop-up window. If your browser blocks it, allow the blocked pop-up — the Rimo sign-in page then opens. Leave it blocked and sign-in fails with *"Unable to sign in. Please try again."*

1. Open the agent (from the link, or find **Rimo** in the Microsoft 365 or Teams agent store) and select **Add**.

   ![The Rimo listing in the Agent Store with the Add button](../images/assets/microsoft-store-add.png)

2. Ask a question that needs Rimo, such as *"Can you list my recent Rimo notes?"* The agent shows a Rimo card: *"Connect your account, then select "Continue"."* Select **Connect**.

   ![The Rimo chat showing the Connect your account card with Connect and Continue](../images/assets/microsoft-enduser-connect-card.png)

3. In the window that opens, select **Create**.

   ![The Rimo connection dialog with the Create button](../images/assets/microsoft-enduser-connect-rimo.png)

4. Sign in with **Account B's own Rimo account**, choose an organization, and select **Allow access**.

> [!IMPORTANT]
> **Allow the sign-in pop-up.** Rimo sign-in opens in a pop-up window at this step. If your browser blocks it, allow the blocked pop-up — the Rimo sign-in page then opens. Leave it blocked and sign-in fails with *"Unable to sign in. Please try again."*

5. Return to the chat and select **Continue** on the card. The first time, an **Allow** card may appear — select **Allow**. The agent then returns Account B's notes.

   ![The Rimo chat returning a table of the end user's recent notes](../images/assets/microsoft-enduser-notes.png)

---

## Make the agent available to the organization

After the end-user test succeeds, the maker submits the agent and an administrator approves it.

### Step 1 — the maker submits it

1. Select the **Share** icon in the top bar and turn on **Org-wide sharing for use access**. This grants everyone permission to *use* the agent — a required step, because a user who finds the agent still can't chat with it until they have access.

   ![The Share Rimo dialog with Org-wide sharing for use access turned on](../images/assets/microsoft-share-dialog.png)

2. Select the **Teams + Microsoft 365** channel in the right-hand panel and open **Use and share**.
3. Under **Share your agent**, select **Submit to org catalog** and confirm. This sends the agent for admin approval; once approved, it's listed org-wide under **Built by your org** in the Microsoft 365 Agent Store (and **Built for your org** in Teams).

   ![The Teams + Microsoft 365 channel on the Use and share tab with Submit to org catalog marked](../images/assets/microsoft-submit-catalog.png)

   > **Note:** After submitting, keep **Org-wide sharing for use access** on — turning it off later stops users from chatting with the agent, even after they've installed it.

### Step 2 — an administrator approves it

Using an account with the **AI Administrator** role:

1. Open the [Microsoft 365 admin center](https://admin.microsoft.com/) and go to **Agents → All agents → Requests**.

   ![The Microsoft 365 admin center on the Requests tab with the pending Rimo agent](../images/assets/microsoft-admin-requests.png)

2. Open the pending Rimo agent, review its publisher, capabilities, data access, tools, security, and permissions, then select **Publish to store** to approve it.

   ![The pending Rimo agent detail with Publish to store marked](../images/assets/microsoft-admin-approve.png)

3. In the **Publish agent to selected users** wizard, choose whether users can find and install the agent (**All users** or specific users/groups) and who gets it pre-installed (optional), then select **Next** through the remaining steps and **Publish**.

   ![The Select users step of the Publish agent to selected users wizard](../images/assets/microsoft-admin-publish-users.png)

After approval, users find the agent under **Built for your org** in Teams or **Built by your org** in the Microsoft 365 Agent Store. A Teams Administrator can also install and pin it automatically for selected users.

See [Agent Store in Microsoft 365 Copilot](https://learn.microsoft.com/en-us/microsoft-365/copilot/copilot-agent-store) and [set agent availability](https://learn.microsoft.com/en-us/microsoft-365/copilot/agent-essentials/agent-lifecycle/agent-availability).

---

## What each employee does

After the administrator deploys the agent, each employee only needs to:

1. Open the Rimo agent in Teams or Microsoft 365 Copilot (add it first if it wasn't deployed automatically).
2. Ask a question that needs Rimo.
3. Connect and sign in to Rimo, choose an organization, and allow access — once.

### The first time you use it

> [!IMPORTANT]
> **Allow the sign-in pop-up.** Selecting **Connect** opens Rimo sign-in in a pop-up window. If your browser blocks it, allow the blocked pop-up — the Rimo sign-in page then opens. Leave it blocked and sign-in fails with *"Unable to sign in. Please try again."*

- After you allow access on the Rimo page, return to the chat and select **Continue**. If nothing happens, **send your message again** — the connection takes a moment to activate.
- The first tool call may show a one-time **Permission Required → Allow** card. After you allow it, later messages and new chats work without prompting.

---

## Updating the agent

- **Content or tool changes:** publish the agent again. Existing users receive the update; a new organization approval isn't normally required.
- **Store details such as the icon or description:** submit the agent for admin approval again.
- **New Rimo tools don't appear:** open the Rimo tool in Copilot Studio, refresh or recreate the connection/tool discovery, then publish again.

---

## Example things to ask

- "Show me my Rimo notes from this week."
- "Which Rimo meetings did I attend last sprint?"
- "Get the transcript of my last Rimo meeting."
- "Who was in that meeting?"
- "Find my Rimo notes about pricing strategy."
- "Find my notes about onboarding, even if they don't use that exact word."
- "Looking at my Rimo notes, what did we decide about the Q3 release?"

Rimo only ever shows the agent what the signed-in user can see in the Rimo organization they approved.

---

## Switching organizations & turning it off

- **One Rimo organization at a time.** To use another organization, open your connection settings, disconnect Rimo, and connect again while choosing the other organization.
- **Remove a user's access.** The user can revoke or refresh the Rimo connection from their connection page.
- **Stop organization access.** An administrator can block, remove, or undeploy the agent in the Microsoft 365 admin center.
- **Take the agent offline.** The maker can remove the Teams and Microsoft 365 Copilot channel, but an admin-approved store listing must also be removed by an administrator.

---

## If something doesn't work

**You can test but can't publish.** A Copilot Studio trial cannot publish. Use Microsoft 365 Copilot, a standalone Copilot Studio credit pack, pay-as-you-go, or another publish-capable entitlement.

**Adding the tool fails.** Confirm you selected **OAuth 2.0 → Dynamic (with discovery)** and used exactly `https://mcp.rimo.app/mcp`.

**The end user is seeing the maker's Rimo notes.** Open the Rimo tool and confirm **Authentication** is **End user account**, not **Shared account**. Every employee must have their own Rimo connection.

**The end user gets "Connector information unavailable."** The Rimo custom connector wasn't shared. Share it with the organization — see [Share the Rimo connector with your organization](#share-the-rimo-connector-with-your-organization).

**"Unable to sign in. Please try again." when connecting.** The browser is blocking the sign-in pop-up. Allow pop-ups for the connection page and try **Connect** again.

**Rimo sign-in succeeds but the connection stays "Not connected."** This happens when the agent lives in a dedicated (non-Default) Power Platform environment: each user also needs a role in that environment before their connection can be saved. Ask the maker to grant users an **Environment Maker** role in the agent's environment — for the whole organization, via a security group rather than one user at a time.

**"This agent is currently unavailable. It has reached its usage limit."** The environment ran out of Copilot Credits. The maker or admin needs to enable pay-as-you-go or allocate a Copilot Credit pack — see [Runtime capacity](#choose-your-microsoft-plan).

**The end user can't install the agent.** Confirm both Microsoft accounts are in the same tenant, the user is included in the agent's access assignment, Power Platform apps are allowed in Teams, and the agent is published with the Teams and Microsoft 365 Copilot channel.

**The agent doesn't appear in the organization store.** A maker submission isn't enough. An AI Administrator must approve it under **Microsoft 365 admin center → Agents → All agents → Requests**.

---

## Official Microsoft help

- [Connect an existing MCP server to a Copilot Studio agent](https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent)
- [Configure user authentication for tools](https://learn.microsoft.com/en-us/microsoft-copilot-studio/configure-enduser-authentication)
- [Copilot Studio licensing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing)
- [Assign licenses and manage access](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-licensing)
- [Publish to Teams and Microsoft 365 Copilot](https://learn.microsoft.com/en-us/microsoft-copilot-studio/publication-add-bot-to-microsoft-teams)
- [Agent management roles and permissions](https://learn.microsoft.com/en-us/microsoft-365/admin/manage/agent-roles-perms?view=o365-worldwide)
- [Agent Store in Microsoft 365 Copilot](https://learn.microsoft.com/en-us/microsoft-365/copilot/copilot-agent-store)

---

## See also

- [Rimo in Claude](rimo-in-claude.md) — connect Rimo to Claude
- [Rimo in ChatGPT](rimo-in-chatgpt.md) — connect Rimo to ChatGPT
- [Rimo in Gemini](rimo-in-gemini.md) — connect Rimo to Gemini Spark
- [What you can do with MCP](mcp.md) — available Rimo tools and examples
- [Authentication](authentication.md) — how Rimo sign-in and accounts work
