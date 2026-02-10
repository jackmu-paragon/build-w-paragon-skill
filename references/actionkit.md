# ActionKit for 3rd-party integration actions

## ActionKit Overview
ActionKit is an API that provides useful abstrated actions that work with your users' integration and run on their behalf.

Some examples of ActionKit actions are: 
- `NOTION_CREATE_PAGE`
- `SALESFORCE_UPDATE_RECORD`
- `SLACK_SEND_DIRECT_MESSAGE`

Popular use cases for ActionKit include
1. Tools for AI agents
2. Steps for Workflow Builders
3. UI components like buttons that should perform a 3rd-party action

## ActionKit Endpoints
The ActionKit API has two main endpoints:

1. The `LIST ACTIONS` endpoint to retrieve available actions based off the users' connected integrations.

```bash
curl --request GET \
  --url 'https://actionkit.useparagon.com/projects/{project_id}/actions' \
  --header 'Authorization: Bearer {PARAGON_JWT}'
```

The `LIST ACTIONS` API will return a result in this format:

```
{
  "actions": {
    "hubspot": [
      {
        "type": "function",
        "function": {
          "name": "HUBSPOT_CREATE_RECORD_CONTACTS",
          "description": "Create a contact in HubSpot CRM",
          "parameters": {
            "type": "object",
            "properties": {
              "field-email": {
                "type": "string",
                "description": "Email"
              },
            },
            "required": [
              "field-email"
            ],
            "additionalProperties": false,
          }
        }
      }
    ]
  },
  "errors": []
}
```

2. The `RUN ACTION` endpoint to perform an action on your users' behalf

```bash
curl --request POST \
  --url https://actionkit.useparagon.com/projects/{project_id}/actions \
  --header 'Authorization: Bearer {PARAGON_JWT}' \
  --header 'Content-Type: application/json' \
  --data '
{
  "action": "SLACK_LIST_CHANNELS",
  "parameters": {}
}
'
```

The `RUN ACTION` output will depend on the action run.

The ActionKit API can be used anywhere across any framework or programming language

NOTE: If the developer does not have a way to create `PARAGON JWT` for their users, refer to the [Paragon SDK setup](paragon-sdk-setup.md) and the [Connecting Integrations](connect-integrations.md)

## ActionKit API Additional Parameters
### List Actions
The `LIST ACTIONS` endpoint by default returns all actions for integrations a user has connected to. Here are additional query parameters to filter and include relevant actions:

- `format=` can be set to `json_schema` for LLM tools or `paragon` for workflow builder use cases
- `limit_to_available=` can be set to `true` or `false` to include only integrations a user has connected to vs all available integrations; default is `true`
- `integrations=` can be set to a comma separated list of integration names i.e. "salesforce,hubspot,slack"
- `categories=` can be set to a comma separated list of integration categories i.e. "crm,project_management"

### Run Action
The `RUN ACTION` endpoint has two required body parameters:

- `action` with the action name (this can be found in the `LIST ACTIONS` payload)
- `parameters` with an object with all required parameters (an action's parameters is also found in the `LIST ACTIONS` payload)

```bash
curl --request POST \
  --url https://actionkit.useparagon.com/projects/{project_id}/actions \
  --header 'Authorization: Bearer {PARAGON_JWT}' \
  --header 'Content-Type: application/json' \
  --data '
{
  "action": "SLACK_SEARCH_MESSAGES",
  "parameters": {
    "query": "search-term"
  }
}
'
```

## Example Implementation Agent Tools
A popular use case for ActionKit is tools for AI Agents. The output schema for the `LIST ACTIONS` endpoint has LLM-readable descriptions that be easily plugged into any LLM API or framework.

Here's an example using Vercel's AI-SDK
```typescript
import { generateText, jsonSchema, tool } from "ai";
import { openai } from "@ai-sdk/openai";

const response = await fetch("https://actionkit.useparagon.com/projects/<Project ID>/actions", {
    method: "GET",
    headers: {
      Authorization: `Bearer ${paragonUserToken}`,
    },
});

const { actions, errors } = await response.json();

if (errors.length === 0) {
  await generateText({
    model: openai("gpt-4o"),
    tools: Object.fromEntries(
      actions.map((action) => [
        action.function.name,
        tool({
          description: tool.function.description,
          parameters: jsonSchema(tool.function.parameters),
          execute: async (params: any, { toolCallId }) => {
            try {
              const response = await fetch(
                `https://actionkit.useparagon.com/projects/<Project ID>/actions`,
                {
                  method: "POST",
                  body: JSON.stringify({
                    action: tool.function.name,
                    parameters: params,
                  }),
                  headers: {
                    Authorization: `Bearer ${session.paragonUserToken}`,
                    "Content-Type": "application/json",
                  },
                }
              );
              const output = await response.json();
              if (!response.ok) {
                throw new Error(JSON.stringify(output, null, 2));
              }
              return output;
            } catch (err) {
              if (err instanceof Error) {
                return { error: { message: err.message } };
              }
              return err;
            }
          },
        }),
      ])
    ),
    toolChoice: "auto",
    temperature: 0,
    system: "You are a helpful assistant. Be as concise as possible.",
    prompt: "Help me create a new task in Jira.",
  });
}
```
