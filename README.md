# Agentic AI Workshop for Architecture Students
**University of Washington | Building AI Agents for CAD Environments**

## Overview
Build AI agents that interact with Rhino3D through natural language. This workshop explores the intersection between AI and automation, where traditional automation provides deterministic workflows with clear inputs and outputs, while AI agents bring contextual intelligence to mainstream these processes with thoughtful integration.

In architecture and construction workflows, AI agents can transform how we approach building performance analysis. Instead of manually filling dozens of input parameters, large language models understand context and generate thoughtful inputs automatically. This allows you to focus on comparative analysis of multiple outcomes rather than tedious data entry. The same principle applies to CAD interactions—tasks that are repetitive and manual become conversational and intuitive.

While agents offer powerful capabilities, setting up a functional system within current network ecosystems requires some technical groundwork. This workshop walks you through creating a running agent that interacts with Rhino3D in a controlled, defined manner. The core purpose is understanding the technical fundamentals of AI agents with minimal complexity, giving you a foundation to build more sophisticated automation tools for your design practice.

You'll learn the complete technical workflow: function creation → local hosting → public exposure → LLM integration.

---

## Quick Reference

**Important URLs:**
- Swiftlet Plugin: https://www.food4rhino.com/en/app/swiftlet
- ngrok Setup: https://ngrok.com/docs/guides/share-localhost/quickstart#windows
- Local MCP: `http://localhost:3001/mcp/`
- MCP Inspector: `http://localhost:6274`

**Default Ports:**
- Swiftlet Server: `3001`
- MCP Inspector: `6274`

---

---

## Prerequisites

### Required Software
You'll need **Rhino 8** installed on your machine, which automatically includes RhinoCompute. Install **Swiftlet** through Rhino's Package Manager and restart the application to activate it. Make sure **Node.js** is installed for running the MCP inspector. Finally, create a free **ngrok account** at ngrok.com—no billing information is required for the free tier.

**Installation Checklist:**
- [ ] Rhino 8 installed
- [ ] Swiftlet installed via Package Manager
- [ ] Rhino restarted after Swiftlet installation
- [ ] Node.js installed
- [ ] ngrok account created

### LLM Access
For **Claude**, a free account works perfectly fine. Download the desktop application and enable Developer Mode in settings. If you prefer **ChatGPT**, note that you'll need at least a Pro account to access Developer Mode functionality.

---

---

## Architecture Overview

```
┌─────────────┐      ┌──────────┐      ┌────────┐      ┌─────────┐
│  Rhino3D    │──────│ Swiftlet │──────│ ngrok  │──────│  Claude │
│  Functions  │      │  :3001   │      │ Tunnel │      │   LLM   │
└─────────────┘      └──────────┘      └────────┘      └─────────┘
     (GH)           (Local MCP)      (Public URL)    (AI Agent)
```

**Data Flow:** 
User prompt → LLM determines function → MCP protocol → Swiftlet routes → Grasshopper executes → Result returns to LLM

---

## Step 1: Create Functions (Tools)

**File:** `swiftlet_example.gh`

Open the Grasshopper file to define your functions with specific inputs and outputs. Swiftlet takes these Grasshopper definitions and converts them into callable endpoints that can be accessed over HTTP. This transformation turns your visual programming components into "tools" that large language models can invoke programmatically.

**Technical Details:**
- Host: `localhost:3001`
- Protocol: MCP (Model Context Protocol)
- Function Structure: Input parameters → Processing logic → Output confirmation

**Example Function:**
```
make_circle(radius, x, y)
```
This function accepts three parameters and creates a circle in Rhino at the specified coordinates, then returns a confirmation message.

> **Note:** Your functions will automatically be hosted on `localhost:3001` when Swiftlet is running in Rhino.

---

## Step 2: Inspect MCP Locally

### Install MCP Inspector

Run this command in your terminal:

```bash
npx @modelcontextprotocol/inspector
```

The inspector will automatically open at `localhost:6274`.

### Configure Connection

**MCP URL:**
```
http://localhost:3001/mcp/
```

**Connection Settings:**
- Transport Type: `Streamable HTTP`
- Connection Type: `Via Proxy`

### Verify Your Setup

Once connected, navigate to the "Tools" tab in the inspector to view all available functions. Test each one by providing sample inputs and verifying that outputs return correctly. 

> **Why This Matters:** This validation step ensures your MCP structure is properly configured before you expose it publicly to external services.

---

## Step 3: Expose Local Server Publicly

### The Problem
Modern LLM services implement strict security policies that block direct access to local servers running on your machine.

### The Solution: ngrok Tunnel

Create a secure tunnel that bridges your localhost with the public internet.

**Setup (One-time):**

1. Create a free account at https://ngrok.com
2. Download ngrok for your operating system
3. Authenticate with your token:

```bash
ngrok config add-authtoken <YOUR_TOKEN>
```

**Start the Tunnel:**

```bash
ngrok http 3001 --host-header=rewrite
```

**Expected Output:**
```
Forwarding: https://[random-name].ngrok-free.dev → localhost:3001
```

Copy your unique ngrok URL from the output. This URL remains active as long as the ngrok process is running.

### Verify Public Access

Test your public endpoint in the MCP Inspector:

```
https://[your-ngrok-url].ngrok-free.dev/mcp
```

> **Technical Note:** The `--host-header=rewrite` flag ensures Swiftlet accepts the forwarded requests despite them arriving from a different domain.

---

## Step 4: Connect to LLM

### For Claude (Desktop App)

1. Open Claude desktop application
2. Navigate to **Settings** → Enable **Developer Mode**
3. Create a new **Custom Connector**:
   - **Name:** `Rhino Agent` (or any name you prefer)
   - **URL:** `https://[your-ngrok-url].ngrok-free.dev/mcp`
4. Save and activate the connector

### For ChatGPT (Pro Account Required)

1. Open ChatGPT settings
2. Enable **Developer Mode** (requires Pro subscription)
3. Create a **Custom App**:
   - **Name:** `CAD Assistant` (or any name you prefer)
   - **MCP URL:** `https://[your-ngrok-url].ngrok-free.dev/mcp`
4. Authorize and enable the app

> **Important:** Replace `[your-ngrok-url]` with your actual ngrok forwarding URL from Step 3.

---

## Testing Prompts

Try these conversational commands in your connected LLM:

```
"Create 5 random circles in Rhino"
```

```
"Make a cylinder at origin with radius 10 and height 50"
```

```
"Generate a grid of points 10x10"
```

### Behind the Scenes

When you send a prompt, here's what happens:

1. The LLM parses your intent and selects the appropriate function from your MCP server
2. It generates the necessary parameters (radius values, coordinates, etc.) based on context
3. The LLM calls the function through the ngrok tunnel
4. ngrok routes the request to Swiftlet on your local machine
5. Swiftlet executes the corresponding Grasshopper component
6. The result returns through the chain back to the LLM
7. The LLM formulates a natural language response confirming the action

---

## Troubleshooting

### Common Issues and Solutions

**MCP Inspector won't connect to local server**
- Verify Swiftlet is running in Rhino (check Grasshopper component is active)
- Confirm port 3001 is not being used by another application
- Restart Rhino and reopen the Grasshopper file

**ngrok tunnel fails to start**
- Re-run the authentication command: `ngrok config add-authtoken <YOUR_TOKEN>`
- Check if you have an active internet connection
- Verify you haven't exceeded free tier limits (1 tunnel for free accounts)

**Functions not appearing in MCP Inspector**
- Refresh the MCP Inspector browser page
- Verify the Grasshopper component is properly configured
- Check that Swiftlet shows "Server Running" status

**LLM cannot access the MCP server**
- Update your connector/app URL with the current ngrok forwarding address
- Verify the ngrok process is still running (it stops if terminal is closed)
- Test the public URL directly in MCP Inspector first
- Ensure Developer Mode is enabled in your LLM settings

**ngrok URL keeps changing**
- Free ngrok accounts generate new URLs each session
- Paid accounts offer static domains
- Remember to update your LLM connector URL when ngrok restarts

> **Pro Tip:** Keep three terminals open during the workshop:
> 1. MCP Inspector (`npx @modelcontextprotocol/inspector`)
> 2. ngrok tunnel (`ngrok http 3001 --host-header=rewrite`)
> 3. General purpose terminal for testing


