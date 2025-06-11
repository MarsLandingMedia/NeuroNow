# NeuroNow

**NeuroNow** is a fully integrated ServiceNow-based assistant framework powered by the OpenAI Assistant API. It allows ServiceNow developers to create conversational AI agents with contextual memory, tool usage, and record interaction capabilities—directly inside the Now Platform.

---

## Features & Capabilities

NeuroNow offers advanced automation through natural language and function-calling, including:

- Persistent Conversations: Each user’s interaction is retained and loaded using thread memory.
- Function Calling (Tool Use): Automatically executes mapped ServiceNow functions when the LLM determines they are required.
- Skill Execution: Uses the `gptSkillRunner` to dynamically evaluate JavaScript-based tools defined in the `u_neuronow_openai_skills` table.
- Reference Field Assistance:
  - Step-by-step guidance when selecting values for reference fields.
  - Shows available options from the referenced table.
  - Can resolve ambiguous instructions using fuzzy matching.
- Data Traversal & Reviews:
  - Can access and review records in any table.
  - Supports code reviews for Script Includes and other development artifacts.
- Multi-record Creation:
  - Populate multiple `sys_user` records using vague or natural descriptions.
  - Accurately assigns reference fields (e.g., `cmn_location`).
- Instructional Context Awareness:
  - Interprets advanced prompts like “choose 5 Marvel superheroes” and executes actions accordingly.
  - Maintains prior context between messages.
- Queued Async Execution:
  - Async logic can be adapted to a dedicated queue for scalability.
- Tool Extensibility:
  - Developers can expand tools to include catalog item creation, workflow building, page widget generation, and more.

NeuroNow is best paired with OpenAI models like GPT-4.1 mini or later for performance and contextual quality.

---

## Getting Started

### Step 1: Install NeuroNow

Install the scoped application via update set or from source control. It includes:

- Script Includes
- Tables
- Service Portal
- Widgets
- Supporting REST integrations
- Sample data

---

### Step 2: Configure System Properties

You must configure the following properties under **System Properties > All**:

| Property | Type | Description |
|---------|------|-------------|
| `x_neuronow.gpt.openai.api.key` | Password | Your OpenAI API key |
| `x_neuronow.gpt.openai.assistantid` | String | The Assistant ID from your OpenAI workspace |
| `x_neuronow.gpt.openai.agent.name` | String | Display name for your agent |
| `x_neuronow.gpt.chat.input.placeholder` | String | Placeholder text in chat input |
| `x_neuronow.gpt.portal.title` | String | Title shown at the top of the Service Portal interface |

---

### Step 3: REST Integration

NeuroNow includes all necessary REST Messages and HTTP Methods under the `OpenAI` REST Message configuration.  
These support all required OpenAI Assistant API operations, including:

- `createThread`
- `addMessage`
- `runThread`
- `runStatus`
- `getMessage`
- `functionResponse`
- `deleteThread`

No additional setup is required for the REST configuration itself.

To fully enable the project, ensure the following System Properties are populated:

- `x_neuronow.gpt.openai.api.key` — your OpenAI API Key  
- `x_neuronow.gpt.openai.assistantid` — your OpenAI Assistant ID

---

## Project Extras

This project includes additional optional components to enhance developer experience:

- `ScopedAppUtil` (global script include) — decrypts API keys.  
  Ensure the “Project Extras” are installed to access this.  
  [ScopedAppUtil Source (placeholder)](#)
- Sample `u_neuronow_openai_skills` — starter function scripts you can modify or extend.
- Sample assistant JSON + prompt files — upload these to OpenAI to match your instance tools.
- Fix script — clears the UI and thread data to reset the conversation.
- Custom field: `u_test_field` (on `sys_user`) — reference to `cmn_location`, useful for testing multi-user inserts via AI.

---

## Key Script Includes

| Name | Description |
|------|-------------|
| `x_neuronow.gpt` | Core logic that manages conversations, threads, and REST calls |
| `global.gptFunctions` | Utility functions for resolving fields, querying tables, etc. |
| `global.gptSkillRunner` | Dynamically executes user-defined skill scripts |
| `global.ScopedAppUtil` | (Extras) Decrypts secure properties like API key |

---

## Event-Driven Architecture

NeuroNow handles async conversation flow using event queues:

- `x_neuronow.gpt.openai.async.run` (Script Action)
  - Executes when an OpenAI message is queued
  - Triggers GPT function calling and response generation

Consider replacing or supplementing this with a custom Scheduled Job Queue or Flow for high-throughput needs.

---

## Tips for Developers

Here are a few ideas to expand on NeuroNow:

- Auto-generate catalog items using conversational definitions.
- Enable conversational creation of Knowledge Base articles or Flow Designer actions.
- Build new Service Portal widgets that dynamically hook into other LLM-driven flows.
- Extend the `u_neuronow_openai_skills` table with validated testing frameworks to simulate data population logic.

---

## Example Use Case

```text
"Add 3 users from Marvel comics, randomly pick them, and place them in different office locations. Set their test field based on their city."

→ NeuroNow will:
- Choose valid character names
- Create 3 sys_user records
- Assign each a unique cmn_location reference
- Populate custom reference fields with valid display values
