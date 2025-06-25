# NeuroNow

NeuroNow is a conversational AI integration for ServiceNow, powered by OpenAI’s API. It enables developers and admins to interact with ServiceNow through natural language to automate processes, query data, build content, and populate records.

---

## Note

You will require an OpenAi account and you can put as little as $5 USD on it for a few days worth of heavy use....maybe more. In the last 30 days I've spent $7.75 for 16.5M input tokens. That's me testing.

---

## How It Works

NeuroNow connects ServiceNow to OpenAI's Assistants using a structured tool-execution framework. Here's the flow:

1. **User Enters Prompt in Service Portal**  
   Through the GPT Portal UI widget, a user submits a natural language prompt (e.g., "Add a user named Bruce Wayne").

2. **Assistant Matches the Prompt to a Tool**  
   In OpenAI, a pre-configured assistant uses embedded functions (tools) to match prompts to specific function calls (e.g., `create_sys_user`).  
   These tools must be installed in your OpenAI assistant — see **[NeuroNow Extras](https://github.com/MarsLandingMedia/NeuroNow-Extras)**.

3. **Tool Call Sent to ServiceNow**  
   The matched tool call (including its arguments) is routed back to ServiceNow through the API and handled by NeuroNow logic.

4. **ServiceNow Skill is Executed**  
   NeuroNow maps the tool call to a matching script in the `u_neuronow_openai_skills` table (based on the function name).  
   The script is executed using `gptSkillRunner`, and the result is returned to OpenAI and shown in the portal.

5. **System Responds and Holds Context**  
   Threads and messages are persisted so the LLM can continue the conversation with full context.

---

## Features

### Conversational Intelligence with Context
- Persistent threads and messages stored in custom tables.
- User identity and session context retained throughout multi-step interactions.

### Tool (Skill) Execution via API
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

[![Watch the video](https://img.youtube.com/vi/BatQu_sBEFQ/0.jpg)](https://www.youtube.com/watch?v=BatQu_sBEFQ)

### Step 1: Import the Application

- Install NeuroNow by cloning this repository into a ServiceNow scoped app.
- Ensure all script includes, tables, and widgets are present and active.

### Step 2: Install ServiceNow Functions update set from the NeuroNow Extras repo. 
** IT WILL NOT WORK WITHOUT THESE **
- **[NeuroNow Extras](https://github.com/MarsLandingMedia/NeuroNow-Extras)**
- You can build your own, but on initial install the portal will not work if these are not there.
- ** YOU MUST HAVE CORESPONDING OPENAI FUNCTIONS/TOOLS THAT CALL YOUR SERVICENOW FUNCTIONS. ** These are also included in the 'extra' package. 

### Step 3: Configure System Properties

Set the following properties in **System Properties** or use scripting to apply them.

| Property                                      | Type     | Description                              | Notes                                                                 |
|----------------------------------------------|----------|------------------------------------------|-----------------------------------------------------------------------|
| `x_neuronow.gpt.openai.api.key`              | password | Your OpenAI API Key                      |                                                                       |
| `x_neuronow.gpt.openai.assistantid`          | string   | Your Assistant ID                        |                                                                       |
| `x_neuronow.gpt.chat.input.placeholder`      | string   | Placeholder for the portal chat input    | Already set by default.                                                                      |
| `x_neuronow.gpt.openai.agent.name`           | string   | Display name shown for the AI            | Already set by default.                                                                      |
| `x_neuronow.gpt.portal.title`                | string   | Title used in the Service Portal header  | Already set by default.                                                                     |


---

## Assets & Artifacts

### Script Includes

- `x_neuronow.gpt` — Orchestrator for thread/message/run logic
- `gptFunctions` — Server-side tool functions callable by the assistant
- `gptSkillRunner` — Executes stored skills dynamically
- `ScopedAppUtil` — Global utility (requires [NeuroNow Extras](https://github.com/MarsLandingMedia/NeuroNow-Extras))

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

## NeuroNow/Project Extras

Additional content such as:

- Predefined tools and assistant JSON
- Sample system prompts
- Global Script Include `ScopedAppUtil`
- ServiceNow skills records to match the OpenAi tools/functions.

Available in a separate repository:  
**[NeuroNow Extras](https://github.com/MarsLandingMedia/NeuroNow-Extras)**

---

## Example Prompt and Tool Call

> **Prompt:**  
> "Add a user to the sys_user table with the real name Bruce Wayne. Use the username bwayne, title him CEO, and set the email to bruce.wayne@wayneenterprises.com."

The assistant interprets this and generates a structured OpenAi tool call like the following:

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
```
The object is processed in the corresponding ServiceNow skills function like so:
```javascript
(function() {
    var user = new GlideRecord('sys_user');
    user.initialize();
    user.first_name = input.first_name;
    user.last_name = input.last_name;
    user.user_name = input.user_name;

    if (input.email)
        user.email = input.email;

    if (input.title)
        user.title = input.title;

    user.insert();

    return JSON.stringify({
        success: true,
        sys_id: user.getUniqueValue(),
        message: "User '" + input.first_name + " " + input.last_name + "' created successfully."
    });
})();
```
---

## Disclaimers

<span style="color:#0366d6"><strong>Legal and Use Notice</strong></span>  
This project is licensed under the <a href="https://www.gnu.org/licenses/gpl-3.0.html" target="_blank">GNU Affero General Public License</a> and is a product of <strong>Mars Landing Media LLC</strong>. It is provided <em>solely for demonstration and educational purposes</em>.  

No warranties, maintenance, or official support of any kind are provided. Mars Landing Media LLC assumes no responsibility for any damages, loss of data, or operational disruption resulting from incorrect, inappropriate, or enterprise use of this product or any part of it.  
This code is not production-ready and may contain errors or incomplete logic. It is intended to serve as a reference or starting point only. Use this solution at your own risk.


<span style="color:#d73a49"><strong>ServiceNow® Notice</strong></span>  
ServiceNow is a registered trademark of ServiceNow, Inc. This project is <strong>not affiliated with, endorsed by, or sponsored by ServiceNow</strong>. Any references to ServiceNow are intended for educational and integration demonstration purposes only.  

Use of this project assumes that you remain in compliance with ServiceNow’s Terms of Service and applicable usage policies.  


<span style="color:#6f42c1"><strong>OpenAI® Notice</strong></span>  
This project uses the OpenAI API. OpenAI and ChatGPT are trademarks of OpenAI. This project is <strong>not affiliated with or endorsed by OpenAI</strong>. All API usage must comply with OpenAI's Terms of Use.


<span style="color:#6f42c1"><strong>GlideRecordSecure Notice</strong></span>  
This project does not utilize `GlideRecordSecure`, and therefore does not enforce full ACL-based access controls. Data access in scripts assumes a trusted and sandboxed development environment, and may not meet ServiceNow security best practices.





