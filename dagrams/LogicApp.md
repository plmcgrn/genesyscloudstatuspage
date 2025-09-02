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