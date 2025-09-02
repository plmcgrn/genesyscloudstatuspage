# LogicApp Flow
This diagram shows the high level processing for the LogicApp flow.  In general, it

- Receives a payload from StatusPage
- Parses it for components and the type of update (for incidents, this is "Resolved" or one of their open statuses, like "Investigating".)
- Gets credentials for Genesys, stored somewhere securely like Azure KeyVault
- Gets the data table rows from Genesys mapping to your queues
- Checks for matches on the Component ID's from the payload against the column in your data table rows
- Selectively enables or disables the outage message flag on each row with a foreach loop for updating the data table rows

```mermaid
graph TD
  A[Receive Payload] --> B[Send 200 OK]
  B --> C[Parse StatusPage payload]
  C --> D[Get Genesys creds]
  D --> E[Auth to Genesys]
  E --> F[Fetch Data Table Rows]
  F --> G[Match rows against Payload Components]
  G --> |Yes| H[Set outage flag]
  G --> |No| I[Clear outage flag]
  H --> J[Process each row match]
  I --> J[Process each row match]
  J --> K[Send Genesys row update]
  K --> |More rows| J
  K --> |No more rows| L[Process complete]