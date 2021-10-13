# cypress-websocket-server

Cypress plugin to mock a websocket server during tests.

## Getting started

Install:

```bash
npm i cypress-websocket-server
```

Add `cy.setWebsocketWorkflow` command by adding the following to `cypress/support/index.js`:

```javascript
import { addCommand } from "cypress-websocket-server";
addCommand();
```

Add websocket server in `cypress/plugins/index.js`, accessible via endpoint `ws://localhost:1999`:

```javascript
const { startMockWsServer } = require("cypress-websocket-server");

module.exports = (on, config) => {
  startMockWsServer(config);
};
```

## Workflow file format

The fixture file JSON should contains an array of objects with properties:

- `type`:
  - `server`: a message that should be sent by the server
  - `client`: a message the server expects from the client
  - `reconnect`: an expected client reconnection
- `payload` (optional): message data, can be any object type

## Example workflow

In a scenario where a websocket exchange is expected after the user clicking a button, the websocket can be configured to follow a specific workflow like this:

```javascript
describe("Trigger WS workflow on button click", () => {
  it("Run new project", () => {
    cy.visit("/");
    cy.setWebsocketWorkflow("expected-workflow.json");
    cy.get("[data-cy=ws-trigger-button]").click();
  });
});
```

Being the contents of `expected-workflow.json` in fixtures folder:

```json
[
  {
    "type": "server",
    "payload": { "message": "connected" }
  },
  {
    "type": "client",
    "payload": { "action": "get-user" }
  },
  {
    "type": "server",
    "payload": {
      "message": "user-info",
      "data": {
        "id": 1,
        "name": "My User"
      }
    }
  }
]
```

The server will listen to any connections in `ws://localhost:1999` and:

- Send payload `{ "message": "connected" }` on client connection
- Expect client message containing payload `{ "action": "get-user" }`
- Send user info in payload

If the client doesn't send the expected message in the right order, a error will be printed to Cypress console.

## License

[MIT](LICENSE)
