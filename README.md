# Standard Procedure for MCP Server

## What is an MCP Server?

An MCP (Model Context Protocol) server exposes tools that any 
AI agent (Claude, ElevenLabs, etc.) can call to retrieve data.
Each tool does ONE specific thing and returns structured JSON.

---

## Naming Conventions

### Workflows
| Type | Format | Example |
|---|---|---|
| MCP Server | `MCP Server - [Name]` | `MCP Server - Prix Bétail` |
| Data source | `Source - [Name]` | `Source - Quebec Encans Prix` |
| Chatbot | `Chatbot [Name]` | `Chatbot Test` |

### Nodes
| Type | Format | Example |
|---|---|---|
| MCP tools | `get_[action]` | `get_latest_price` |
| HTTP requests | `[Verb] [What]` | `Get Dates Site` |
| Code nodes | `[Verb] [What]` | `Parse HTML Prix` |
| Conditions | `[Condition]?` | `Has Dates?` |

### Credentials
| Type | Format | Example |
|---|---|---|
| MCP Bearer | `MCP server [name]` | `MCP server météo` |
| AI models | `[Provider] account` | `Anthropic account` |

### Folders in n8n
```
Rag_fourrage/
  ├── Tool/          ← MCP Servers and templates
  ├── Animal price/  ← Animal pricing workflows
  └── Sub_wor.../    ← Sub-workflows (data sources)
```

---

## How to Create a New MCP Server

### Step 1: Duplicate the template

1. Open n8n → **Rag_fourrage / Tool**
2. Find **"MCP Server - [TEMPLATE]"**
3. Click the three dots **"..."** → **"Duplicate"**
4. Rename it: `MCP Server - [Your Name]`
5. Move it to the correct folder

### Step 2: Configure the MCP Server Trigger

1. Open the duplicated workflow
2. Click the **"MCP Server Trigger"** node
3. Fill in:

| Field | Value |
|---|---|
| Authentication | Bearer Auth |
| Credential | Select existing or create new |
| Path | `your-unique-path` (e.g. `prix-betail`) |

> ⚠️ The path must be unique across all MCP servers.
> Your MCP URL will be:
> `https://msfourrager.app.n8n.cloud/mcp/[your-path]`

### Step 3: Configure the data source workflow

1. Create or identify the sub-workflow that fetches your data
2. Open that sub-workflow → find the trigger node
3. Set **"Input data mode"** to **"Define using fields below"**
4. Add all input fields your tool needs:

| Field | Type | Description |
|---|---|---|
| `action` | String | Main action to execute |
| `categories` | String | Data category filter |
| `aggregation` | String | How to aggregate results |
| `periode_jours` | Number | Number of days to look back |

5. Save the sub-workflow

### Step 4: Configure the tool node

1. Click the **"get_[tool_name]"** node
2. Fill in:

| Field | Value |
|---|---|
| Name | `get_[action]` (e.g. `get_latest_price`) |
| Description | Explain WHEN to call this tool and WHAT parameters to pass |
| Workflow | Select your sub-workflow from the list |

3. Rename the node to match your tool name
4. Right-click → **"Rename"** → `get_[your_action]`

### Step 5: Add more tools (if needed)

For each specialized tool:
1. Click **"+"** on the Tools connection of the MCP Server Trigger
2. Search **"Call n8n Workflow Tool"**
3. Repeat Step 4 for each tool

> Best practice: Create one tool per question type.
> Avoid creating one tool that does everything.

### Step 6: Test the MCP Server

1. Open **Chatbot Test** workflow
2. Find the MCP Client tool connected to your server
3. Set the **Endpoint URL** to your MCP URL
4. Open the chat and test with real questions
5. Check the **Logs** panel to verify each tool is called correctly

### Step 7: Publish

1. Click **"Publish"** (top right)
2. Add the MCP URL to the team documentation in Slack

---

## Example: Livestock Pricing MCP

**URL:** `msfourrager.app.n8n.cloud/mcp/prix-betail`  
**Data source:** reseauencansquebec.com  
**Auth:** Bearer Token  

| Tool | When to call | Fixed params |
|---|---|---|
| `get_categories` | User doesn't know category name | action=categories |
| `get_latest_price` | "What is the price of X today?" | action=prix, aggregation=dernier |
| `get_price_trend` | "Is X going up or down?" | action=prix, aggregation=tendance |
| `get_price_by_site` | "Which site is cheapest?" | action=prix, aggregation=par_site |
| `get_weekly_average` | "Average price this week?" | action=prix, aggregation=par_date |

---

## Checklist before publishing

- [ ] MCP Server Trigger has unique path
- [ ] Bearer Auth credential is configured
- [ ] Each tool has a clear description
- [ ] Sub-workflow input fields are defined
- [ ] Tested in Chatbot Test with 3+ questions
- [ ] Responds correctly in FR / EN / ES
- [ ] Workflow is published and active
