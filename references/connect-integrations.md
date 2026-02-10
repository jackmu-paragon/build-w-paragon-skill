# Connecting Integrations

## Overview
This reference covers how to connect integrations **AFTER the Paragon SDK has been setup**.
Connecting integrations means:

- [ ] Step 0 (Optional): On the Paragon dashboard, enable and configure 3rd-party integrations and apps
- [ ] Step 1: developer can retrieve enabled integrations they configured with Paragon
- [ ] Step 2: developer can render the "Connect Portal" in their frontend application
  - The Connect Portal is an embedded UI that takes the end-user through the OAuth process to authenticate to any 3rd-party integration (i.e. Google Drive auth, Notion auth, etc.)

## How to Connect Integrations
### Step 0 (Optional) Enabling and configuring integrations

If the developer has not done so, visit the [Paragon Dashboard](https://dashboard.useparagon.com) to enable 3rd-party integrations they want to connect to.

Provide the developer the [docs on adding integrations](https://docs.useparagon.com/getting-started/adding-an-integration) for more details.

For development, the developer does NOT need to create & configure a 3rd-party OAuth application for their integration (i.e. GitHub OAuth app, Slack OAuth app, etc). Paragon can provide developer credentials for most integrations.

In production, the developer WILL NEED to create & configure their own 3rd-party OAuth application.

### Step 1: Retrieving enabled integrations**

Use the Paragon SDK's `Get Integration Metadata` method to retrieve data on all of the developer's enabled integrations 

```typescript
import { paragon } from '@useparagon/connect';

paragon.getIntegrationMetadata();
```

This method will return data with this schema:

```json
[
  {
    type: 'salesforce',
    name: 'Salesforce',
    brandColor: '#057ACF',
    icon: 'https://cdn.useparagon.com/2.35.0/dashboard/public/integrations/salesforce.svg'
  },
  {
    type: 'hubspot',
    name: 'Hubspot',
    brandColor: '#F67600',
    icon: 'https://cdn.useparagon.com/2.35.0/dashboard/public/integrations/hubspot.svg'
  },
]
```

In typescript:

```typescript
import { paragon, IntegrationMetadata } from "@useparagon/connect";

const integrationMetadata: IntegrationMetadata[] = await paragon.getIntegrationMetadata();
```

The developer can use the integration metadata to build catalogs in their frontend if desired:

```tsx
<div className="flex flex-wrap">
  {integrationMetadata.map((integration: IntegrationMetadata) => {
    <div key={integraiton.type} className="flex gap-4">
      <div>{integration.name}</div>
      <button onClick={() => paragon.connect(integration.type)} />
    </div>
  })}
</div>
```

### Step 2: Render the embedded Connect Portal
The Paragon Connect Portal is a pre-built UI component that is embedded in the user's frontend app.

The `paragon.connect` method is used to bring up the Connect Portal

```typescript
paragon.connect("salesforce"); //replace salesforce with any integration type (found in the integration metadata
```

After a user connects an integration, you can now use ActionKit, Managed Sync, or Workflows to interact with 
the 3rd-party API on their behalf!

If a user is interested in 
- Building Workflows: direct them to Paragon's [Workflow docs](https://docs.useparagon.com/workflows/overview)
- Using ActionKit to perform CRUD-like actions: see [ActionKit reference](actionkit.md)
- Using Managed Sync to ingest & sync large volumes of 3rd-party data: see [Managed Sync reference](managed-sync.md)
