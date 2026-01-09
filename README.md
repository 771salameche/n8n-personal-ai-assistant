# Personal AI Assistant: n8n vs. MCP Implementation

This project provides two distinct implementations of a Personal AI Assistant, showcasing different architectural approaches for building multi-agent systems. The goal is to compare a traditional, workflow-based approach using [n8n](https://n8n.io/) with a modern, decentralized approach using the [Model Context Protocol (MCP)](https://github.com/mcp-works/spec).

## Table of Contents
- [Overview](#overview)
- [Architecture Comparison](#architecture-comparison)
- [Project Structure](#project-structure)
- [Technology Stack](#technology-stack)
- [Setup & Installation](#setup--installation)
- [Configuration](#configuration)
- [Features Comparison](#features-comparison)
- [Usage Examples](#usage-examples)
- [Performance & Resource Usage](#performance--resource-usage)
- [Pros & Cons](#pros--cons)
- [When to Choose Which Approach](#when-to-choose-which-approach)
- [Troubleshooting](#troubleshooting)
- [Future Improvements](#future-improvements)


## Overview

This repository contains the blueprints for a personal AI assistant capable of managing calendars, emails, research tasks, and more. It was developed to explore and contrast two powerful methodologies for AI agent development:

1.  **n8n Workflow Automation:** A "classic" implementation where a main n8n workflow orchestrates specialized sub-workflows, all running within a single n8n instance. This represents a tightly-coupled, monolithic architecture.
2.  **MCP (Model Context Protocol) Servers:** A decentralized, microservices-style implementation. Specialized AI agents are exposed as independent MCP servers, and a main agent communicates with them over a network. This represents a loosely-coupled, distributed architecture.

By comparing these two approaches, developers can gain insights into the trade-offs of each and make informed decisions for their own AI projects.

## Architecture Comparison

The fundamental difference lies in how the agents are structured and how they communicate.

| Aspect | n8n Workflow Approach | MCP Server Approach |
| :--- | :--- | :--- |
| **Paradigm** | Monolithic / Tightly-Coupled | Microservices / Loosely-Coupled |
| **Communication** | Internal function calls between n8n workflows. | Network requests (HTTP) between agents. |
| **Deployment** | Single n8n instance containing all workflows. | Main agent plus multiple independent agent servers. |


### n8n Workflow Architecture
The `Personal Assistant` workflow acts as the central brain. When it receives a request (e.g., via Telegram), its core agent decides which specialized "tool" is needed. These tools are other n8n workflows (`Calendar_Agent`, `Email_Agent`, etc.) that are invoked directly using the `toolWorkflow` node. All processing happens within the same environment.

### MCP Server Architecture
The `Main_Agent` workflow also serves as an entry point. However, instead of calling other workflows directly, it communicates with external MCP servers using the `mcpClientTool` node. Each MCP server (`Calendar_MCP_Agent`, `Email_MCP_Agent`) is a self-contained n8n workflow with an `mcpTrigger` node, making it an accessible endpoint on the network. This allows for greater separation of concerns and deployment flexibility.

## Project Structure

### `Personal Assistant AI Agent` (n8n Approach)
```
/Personal Assistant AI Agent
├── Personal_Assistant.json  # Main orchestrator agent
├── Calendar_Agent.json      # Sub-workflow for calendar tasks
├── Email_Agent.json         # Sub-workflow for email tasks
├── Projects_Agent.json      # Sub-workflow for project management
└── Research_Agent.json      # Sub-workflow for research tasks
```

### `MCP_Personnal_Assistant` (MCP Approach)
```
/MCP_Personnal_Assistant
├── Main_Agent.json            # Main agent that calls MCP servers
├── Calendar_MCP_Agent.json    # Standalone agent for calendar tasks
├── Email_MCP_Agent.json       # Standalone agent for email tasks
└── Research_MCP_Agent.json    # Standalone agent for research tasks
```

## Technology Stack

| Component | n8n Workflow Approach | MCP Server Approach |
| :--- | :--- | :--- |
| **Core Framework** | n8n | n8n + MCP |
| **AI Agent Logic** | `@n8n/n8n-nodes-langchain` | `@n8n/n8n-nodes-langchain` |
| **LLM Provider** | OpenRouter (e.g., GPT-4o) | OpenRouter (e.g., GPT-4o) |
| **User Interface** | Telegram | Telegram |
| **Tool Integrations**| Google Workspace (Sheets, Calendar, Gmail), SerpAPI, Wikipedia | Same |
| **Agent Communication**| n8n `toolWorkflow` node | MCP `mcpClientTool` and `mcpTrigger` nodes |
| **External Tunneling**| Not required | ngrok (or similar) for exposing MCP servers |

## Setup & Installation

**Prerequisites:**
*   An active n8n instance (cloud or self-hosted).
*   Credentials for OpenRouter, Google Workspace APIs, SerpAPI, and Telegram.
*   (For MCP) [ngrok](https://ngrok.com/) or another tunneling service to expose local n8n instances.

### n8n Workflow Approach
1.  **Import Workflows:** In your n8n workspace, import all five JSON files from the `Personal Assistant AI Agent` directory.
2.  **Configure Credentials:** Go to the "Credentials" section in n8n and add the necessary API keys and OAuth connections for Google, OpenRouter, and Telegram.
3.  **Assign Credentials:** Open each workflow and assign the correct credentials to the relevant nodes (e.g., connect the `Telegram Trigger` to your Telegram credentials).
4.  **Activate Workflows:** Activate all imported workflows. The `Personal_Assistant` workflow is the main entry point.

### MCP Server Approach
1.  **Deploy MCP Agents:**
    *   Import `Calendar_MCP_Agent.json`, `Email_MCP_Agent.json`, and `Research_MCP_Agent.json` into one or more n8n instances.
    *   Assign credentials to all nodes within these workflows.
    *   Activate the workflows. The `mcpTrigger` node in each makes it a live server.
2.  **Expose MCP Endpoints:**
    *   For each running MCP agent, find its webhook URL in the `mcpTrigger` node.
    *   Use ngrok to expose this local webhook URL to the internet. Example: `ngrok http <your-n8n-instance-url>`.
    *   You will get a public URL for each agent (e.g., `https://random-name.ngrok-free.app/mcp/...`).
3.  **Deploy Main Agent:**
    *   Import `Main_Agent.json` into an n8n instance (can be the same or different from the MCP agents).
    *   Assign credentials for Telegram, Google Sheets, and OpenRouter.
4.  **Configure MCP Clients:**
    *   In the `Main_Agent` workflow, open each `mcpClientTool` node (`EmailMCP`, `CalendarMCP`, `ResearchMCP`).
    *   Paste the corresponding public ngrok URL from step 2 into the "Endpoint URL" field.
5.  **Activate Main Agent:** Activate the `Main_Agent` workflow.

## Configuration

### n8n Workflow Credentials
All API keys and OAuth connections are managed within the n8n "Credentials" store. You must link these credentials to the nodes that require them.

*   **OpenRouter:** Used in all agents for LLM access.
*   **Google (OAuth):** Used for Gmail, Calendar, and Sheets tools.
*   **SerpAPI:** Used by the Research Agent.
*   **Telegram:** Used for the trigger and response nodes.

### MCP Server Endpoints
The key configuration for the MCP approach is linking the `mcpClientTool` in the `Main_Agent` to the public URLs of the running MCP server agents. Any changes to the ngrok tunnel URL will require updating the client tool configuration.



## Features Comparison

| Feature | n8n Workflow Approach | MCP Server Approach | Notes |
| :--- | :--- | :--- | :--- |
| **Email Management** | ✅ | ✅ | Send/receive emails via Gmail. |
| **Calendar Management** | ✅ | ✅ | Create/view events in Google Calendar. |
| **Web Research** | ✅ | ✅ | Wikipedia, Hacker News, and Google Search (SerpAPI). |
| **Contact Lookup** | ✅ | ✅ | Reads contact data from a Google Sheet. |
| **Project Management** | ✅ | ❌ | Can read/update project status from a Google Sheet. |
| **Knowledge Base** | ✅ | ❌ | Includes a vector store for company information. |
| **Calculator**| ✅ | ❌ | A simple calculator tool is included. |

## Usage Examples

Interaction with both assistants is done via Telegram. The main difference is the underlying mechanism.

**Use Case: "Schedule a meeting with jane@example.com tomorrow at 2 pm to discuss the Q3 report."**

*   **n8n Workflow:**
    1.  The `Personal_Assistant` agent receives the message.
    2.  It determines the task requires the Calendar tool.
    3.  It invokes the `Calendar_Agent` sub-workflow, passing the details.
    4.  The `Calendar_Agent`'s `Create Event with Attendee` node connects to the Google Calendar API and creates the event.
    5.  The success message is passed back to the main agent, which responds via Telegram.

*   **MCP Server:**
    1.  The `Main_Agent` receives the message.
    2.  It determines the task requires the calendar agent.
    3.  It makes an HTTP request to the `CalendarMCP` server's public URL, sending the details in the payload.
    4.  The `Calendar_MCP_Agent` workflow is triggered, and its `Create Event with Attendee` node connects to the Google Calendar API.
    5.  The result is sent back over HTTP to the `Main_Agent`, which then responds via Telegram.

## Performance & Resource Usage

| Metric | n8n Workflow Approach | MCP Server Approach |
| :--- | :--- | :--- |
| **Latency** | Lower. Internal calls are faster than network requests. | Higher. Adds network latency for each MCP call. |
| **Resource (CPU/RAM)**| Concentrated on a single n8n instance. Can be high. | Distributed across multiple instances. More scalable. |
| **Reliability** | A failure in the main instance affects everything. | More resilient. One MCP agent can fail without affecting others. |
| **Scalability** | Vertically scalable (more resources to the instance). | Horizontally scalable (add more agent instances). |

## Pros & Cons

### n8n Workflow Approach
**Pros:**
*   **Simplicity:** Easier to set up and debug as it's all in one place.
*   **Lower Latency:** No network overhead between agents.
*   **Unified Management:** All workflows are managed from a single n8n UI.

**Cons:**
*   **Tightly Coupled:** Changes to a sub-workflow can potentially break the main agent.
*   **Less Scalable:** All load is handled by a single n8n instance.
*   **Not Reusable:** Hard to reuse sub-workflows outside of the n8n environment.

### MCP Server Approach
**Pros:**
*   **Highly Modular & Reusable:** Agents are independent services that can be used by any MCP-compliant client.
*   **Scalable & Resilient:** Agents can be deployed, scaled, and updated independently.

*   **Technology Agnostic:** An MCP agent could be written in Python, Node.js, or any other language, not just n8n.
*   **Clear Separation of Concerns:** Enforces a clean, distributed architecture.

**Cons:**
*   **Increased Complexity:** Requires managing multiple services and network configurations (ngrok).
*   **Higher Latency:** Network requests add overhead.
*   **Distributed Debugging:** Tracing a request across multiple services can be more challenging.

## When to Choose Which Approach

*   **Choose the n8n Workflow approach for:**
    *   Rapid prototyping and internal projects.
    *   Simpler agent systems where scalability is not a primary concern.
    *   When you want to keep everything managed within a single, unified interface.

*   **Choose the MCP Server approach for:**
    *   Complex, production-grade agent systems.
    *   Scenarios requiring high scalability and resilience.
    *   When you want to build a library of reusable, independent AI agents that can be leveraged across multiple applications.
    *   Team environments where different developers are responsible for different agents.

## Troubleshooting

*   **n8n `toolWorkflow` not found:** Ensure the sub-workflow is active and the correct Workflow ID is used in the main agent.
*   **MCP `mcpClientTool` errors:**
    *   Verify the ngrok tunnel is active and the URL is correct.
    *   Check that the target MCP agent workflow is active in its n8n instance.
    *   Inspect the MCP agent's execution logs for internal errors.
*   **Credential Errors:** Double-check that all required credentials have been created in n8n and correctly assigned to each node.

## Future Improvements

### n8n Implementation
*   Break down the monolithic `Personal_Assistant` into more specialized orchestrators.
*   Add more robust error handling and state management.

### MCP Implementation
*   Add the missing `Projects` and `Knowledge Base` agents as new MCP servers.
*   Create a proper deployment setup using a service like Docker or Kubernetes instead of ngrok for production use.
*   Implement a service discovery mechanism for the main agent to find MCP servers dynamically.


