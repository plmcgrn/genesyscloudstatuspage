# Automating Genesys Cloud IVR messaging with StatusPage

## Overview
In this write-up, we will discuss how we built automation to take actionable events from StatusPage, and turn them into automated messaging in Genesys Cloud inbound voice flows.  This overall solution would work with any contact center platform that has API-based tooling, as well as any monitoring platform that can trigger notification-based events (webhook payloads).

## Components

### Genesys Cloud
- Inbound Flow
- Data Table
- OAuth client

### StatusPage
- Webhook-based page subscription

### Azure
- LogicApps (can also use Power Automate)

## Overall Strategy

For this atuomation, the goal was to "subscribe" an IVR call flow to StatusPage, so that when an incident is open, the IVR will play relevent messaging for the impacted product(s).  Callers dialing in during an active service interruption would hear messaging noting that, with the goal of having them call back at a later time.

## LogicApps (or Power Automate)
For this part, you will need to set up a webhook listener.  We do this first as the StatusPage subscription will require this webhook.  Your app does not need to be functional, yet.

In Azure LogicApps, create a new app
For Trigger, choose HTTP(S).  You can change the request type to POST, though this is already the default for security.
Save the app.  You will come back to it later.

## StatusPage
Visit your desired StatusPage site, click Subscribe, and choose the webhook integration.  You will be asked to provide an email address, as this subscription type requires verification and you will need that confirmation email to make any component/group modifications to the desired subscription.

Follow the on-screen process to activate the webhook subscription.  You will use the webhook from LogicApps here.

## Genesys Cloud (initial)
- Create a new OAuth client.  The client will need to have edit permissions on Data Table rows, as well as Read permissions on Data Tables and Rows.
- Create a data table to house your outage toggle configuration.  At a minimum, you will need
  - Row name/id -  This can be whatever you want, but the LogicApp will need to be able to find the row(s) based on these
  - Boolean field for "outage is active"
  - (Optional) Text field to house messaging passed from StatusPage
  - StatusPage component mapping(s) text field.  This will house the component IDs from StatusPage, so your LogicApp can match against them when receiving a webhook payload.
- Inbound flow - This is where it becomes business-specific.  In your inbound flow, you will
  - Check the data table for active outage row(s) related to your overall IVR experience (product, department, etc.)
  - If an outage is active, play back TTS or a pre-recorded User Prompt
  - (Optionally) play back TTS for the custom StatusPage message if you opted to store that.

## LogicApps
In the previously-created LogicApp, you will need to configure logic to handle a few high level tasks

- Authenticate to Genesys Cloud API
- Get a list of rows from your outage data table
- Match row(s) based on your StatusPage payload
- Update status of those rows based on whether the StatusPage Incident is in an "open" status or a "resolved" one.

This will be business-specific, so I will provide general strategy and links to the relevant API documentation for both platforms.

[Genesys data table API](https://developer.genesys.cloud/routing/architect/data-tables)

[StatusPage incident payload guide](https://support.atlassian.com/statuspage/docs/enable-webhook-notifications/) (this will be the structure you receive when StatusPage sends the payload to your LogicApp webhook)

In my specific build, my LogicApp does the following
1. Queries Gensys Cloud data table API to get a list of all rows in the outage-related data table
2. Sets a variable noting whether the StatusPage payload is an "open" or "resolved" notification
3. Uses LogicApp expression logic to match rows in the table against components from StatusPage
4. If "open", calls the Genesys API for each data table row that matched, setting the outage field to "true"
5. If "resolved", calls the Genesys API for each data table row that matched, setting the outage field to "false".

## Wrapping up
This brief write-up serves as a guide on how to integrate two systems together, in order to achieve real time automation of service interruption messaging.  The overall solution removes the need for operational support teams to manage outage messaging by hand, and leverages existing service monitoring (StatusPage) to drive this process automatically.  Since manage organizations have separate teams managing their support and application/delivery environments, it also ensures no dependencies are introduced on cross-team coordination while actively managing incidents purely for the purpose of customer notifications.

