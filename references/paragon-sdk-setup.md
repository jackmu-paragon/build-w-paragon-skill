# Paragon SDK Setup

## Overview
The Paragon SDK is used on nodejs-based frontend project. The Paragon SDK
is used to authenticate end-users with a JWT, and render an embedded UI 
component that takes users through the OAuth process for any given 3rd-party integration.

The high-level steps for setting up the Paragon SDK are:
- [ ] Install the Paragon SDK in the developer's nodejs project
- [ ] Grab `Paragon Project ID` and `Signing Key` from Paragon dashboard and set as environment variables
- [ ] Build a server-side endpoint to sign JWT with `Signing Key`
- [ ] Call the `paragon.authenticate` method with the end-user's unique JWT

These steps are further detailed in the **Checklist** below. A full **implementation example** is included after the Checklist.

## Checklist for setting up the Paragon SDK on Nodejs frontend apps
- [ ] Ensure the Paragon SDK is installed in the developer's nodejs project

```bash
npm install @useparagon/connect
```

- [ ] Prompt developer to grab their `Paragon Project ID` and `Signing Key` from the Paragon dashboard. 
    - Direct them to the docs if they need help: [Paragon Setup](https://docs.useparagon.com/getting-started/installing-the-connect-sdk)
    - Generally the `Paragon Project ID` and `Signing Key` should be environment variables

- [ ] Build a server-side endpoint to sign a JWT with the `Signing Key` with the following data

```typescript
{
	// Uniquely identifying key for a user or their company
	"sub": "the-user/company-id",
    // Audience claim, must match the project ID in the SDK configuration
    "aud": "useparagon.com/{paragon-project-id}",
	// Issue timestamp, should be the current time
	"iat": 1608600116,
	// Expiry timestamp for token, such as 1 hour from time of signature (iat)
	"exp": 1608603716,
}
```
  - This should NOT be done on the client. The `Signing Key` should only be exposed to server side applications

- [ ] On the client-side nodejs application, the Paragon autheticate method can now be used

```typescript
import { paragon } from '@useparagon/connect';

await paragon.authenticate(
	// Your Paragon Project ID
	"38b1f170-0c43-4eae-9a04-ab85325d99f7",
	// See previous step for how to encode your user token
	"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.ey..."
);
```

  - If the developer has an **on-prem/self-hosted** instance, they will also need to configure Paragon with their unique hostname

```typescript
paragon.configureGlobal({
  host: "mycompany.paragon.so",
});
```

  - The developer is now ready to get started [connecting integrations](connect-integrations.md)!

## Full example implementation
### Server side
```javascript
// server.js - Adding middleware to sign an authenticated user's token
const jwt = require('jsonwebtoken');

app.use((req, res, next) => {
  if (!req.user) {
    return next();
  }
  // JWT NumericDates specified in seconds:
  const currentTime = Math.floor(Date.now() / 1000);
  res.locals({
    paragonToken: jwt.sign(
      {
        sub: req.user.id,  // Your user's or their company's ID
        aud: `useparagon.com/${process.env.PARAGON_PROJECT_ID}`, 
        iat: currentTime,
        exp: currentTime + (60 * 60), // 1 hour from now
      },
      process.env.PARAGON_SIGNING_KEY,
      {
        algorithm: "RS256",
      }
    ),
  });
  next();
});
```

### Client side
```jsx
// layout.hbs - Include paragon.authenticate call in the view template
<body>
	<script type="text/javascript">
	  paragon.authenticate("project-id", "{{ paragonToken }}").then(() => {
		  // paragon.getUser() will now return the authenticated user
		});
	</script>
</body>
```
