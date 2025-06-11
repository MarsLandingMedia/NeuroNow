# NeuroNow

NeuroNow is a conversational AI integration for ServiceNow, powered by OpenAI’s Assistants API. It enables developers and admins to interact with ServiceNow through natural language to automate processes, query data, build content, and populate records.

---

## Features

### Conversational Intelligence with Context
- Persistent threads and messages stored in custom tables.
- User identity and session context retained throughout multi-step interactions.

### Tool (Skill) Execution via Assistant API
- Dynamically runs functions defined in the `u_neuronow_openai_skills` table.
- Supports input/output chaining via `submit_tool_outputs`.
- Tools are validated for security and scoped execution.

### Reference Field Resolution
- Step-by-step guidance for selecting and resolving reference values.
- Supports fuzzy matching and exploration of options before committing.
- Demonstrates intelligent behavior even with vague prompts.

### Record Traversal & Code Review
- Ask the system to review or explain Script Includes or other code records.
- Readable feedback and structured suggestions are returned in real time.

### Advanced Record Querying
- Use natural prompts to query any table and return custom field lists.
- Supports both encoded query and field:value formats.
- Results can be returned in raw or display value format.

### Structured Async Processing
- Asynchronous execution supported via Event system (`x_neuronow.gpt.openai.async.run`).
- Good candidate for scale-out solutions using Queue-based workers.

### Multi-User & Dynamic Record Creation
- Add multiple users or records simultaneously.
- Use vague criteria ("Pick 5 Marvel heroes") and let the LLM generate values.

### Developer-Oriented Use Cases
- Create catalog items, update widgets, or review code with prompts.
- Explore system fields, schema, and record content dynamically.
- Easily extensible with additional skill records and assistant tools.

---

## Installation

### Step 1: Import the Application

- Install NeuroNow via update set or clone this repository into a ServiceNow scoped app.
- Ensure all script includes, tables, and widgets are present and active.

### Step 2: Configure System Properties

Set the following properties in **System Properties** or use scripting to apply them.

| Property | Type | Description |
|---------|------|-------------|
| `x_neuronow.gpt.openai.api.key` | password | Your OpenAI API Key |
| `x_neuronow.gpt.openai.assistantid` | string | Your Assistant ID |
| `x_neuronow.gpt.chat.input.placeholder` | string | Placeholder for the portal chat input |
| `x_neuronow.gpt.openai.agent.name` | string | Display name shown for the AI |
| `x_neuronow.gpt.portal.title` | string | Title used in the Service Portal header |

### Step 3: REST Message Setup

NeuroNow includes a REST Message named `OpenAI` with preconfigured methods:

- `createThread`
- `addMessage`
- `runThread`
- `runStatus`
- `getMessage`
- `functionResponse`
- `deleteThread`

No further configuration is needed aside from ensuring the system properties are set.

---

## Assets & Artifacts

### Script Includes

- `x_neuronow.gpt` — Orchestrator for thread/message/run logic
- `gptFunctions` — Server-side tool functions callable by the assistant
- `gptSkillRunner` — Executes stored skills dynamically
- `ScopedAppUtil` — Global utility (requires [Project Extras](#project-extras))

### Script Action

- `x_neuronow.gpt.openai.async.run` — Executes `processAsyncRequest` for queued async handling

### Tables

- `x_neuronow_openai_conversation`
- `x_neuronow_openai_conversation_message`
- `u_neuronow_openai_skills` — Tool definition and execution metadata

### Service Portal

- Portal: **GPT**
- Page: **gpt_ui**
- Widget: **GPT Module**

### Custom Fields

- `u_test_field` on `sys_user` (reference to `cmn_location`)  
  Used for testing the LLM's ability to populate references accurately.

---

## Project Extras

Additional content such as:

- Predefined tools and assistant JSON
- Sample system prompts
- Global Script Include `ScopedAppUtil`
- Fix scripts to reset portal threads

Available in a separate repository:  
**[NeuroNow Extras Repository](https://github.com/your-org/neuronow-extras)**

---

## Example Prompt and Tool Call

> **Prompt:**  
> "Add a user to the sys_user table with the real name Bruce Wayne. Use the username bwayne, title him CEO, and set the email to bruce.wayne@wayneenterprises.com."

The assistant interprets this and generates a structured tool call like the following:

```json
{
  "type": "function_call",
  "name": "create_sys_user",
  "description": "Creates a new user in the sys_user table in ServiceNow.",
  "parameters": {
    "type": "object",
    "properties": {
      "first_name": {
        "type": "string",
        "description": "The user's first name."
      },
      "last_name": {
        "type": "string",
        "description": "The user's last name."
      },
      "user_name": {
        "type": "string",
        "description": "The user's login name."
      },
      "email": {
        "type": "string",
        "description": "The user's email address."
      },
      "title": {
        "type": "string",
        "description": "The user's job title."
      }
    },
    "required": ["first_name", "last_name", "user_name"],
    "additionalProperties": false
  },
  "arguments": {
    "first_name": "Bruce",
    "last_name": "Wayne",
    "user_name": "bwayne",
    "email": "bruce.wayne@wayneenterprises.com",
    "title": "CEO"
  }
}
