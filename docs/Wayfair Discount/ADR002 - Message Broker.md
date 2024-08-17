# ADR002 Message Broker
Mainly an excuse to play around with Golang, but also, finally migrate the Next.js web app to be freely hosted in Vercel by properly queuing, and asynchronously performing discount checker requests thereby unlocking some asynchronous discount check delivery methods to explore (SMS, whatsApp, email, etc).

## Status
Complete

## Context
Presently, discount.mattrandell.com is a NextJ.js web application running on an old Macbook Air in my basement that uses a Puppeteer browser automation script to perform the web scraping to fetch discount information on a friend/family’s behalf. 

Some drawbacks of this approach:

* No static IP at home so I’ve needed to fix that on occasion.
* There are inherent security risks of having a web server on my home network.
* It is not TLS encrypted. (mainly out of laziness and intention to migrate to Vercel where it comes for free)
* The macbook air is slow for hosting a Next.js app.
* Currently manually deploying when there are changes.
* Multiple simultaneous discount requests are still synchronous, the request waits for the script to be free.
* Discount data is locally stored on this Macbook so if it dies then so does everyone’s history.

Note: The puppeteer script cannot easily be moved out of the basement without security risk, and increased likelihood of bot detection circumventing the entire strategy.

## Decision

The lookup process should be managed by a message broker. Discount requests should be placed on a queue/topic and complete immediately if enqueuing is successful. An asynchronous service (written in Go just because I want to) will then ingest the requests, perform the web scraping lookup, and reply with the result to be delivered to the user.

### Architecture

<iframe src="https://viewer.diagrams.net/?tags=%7B%7D&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Message%20Broker%20Architecture.drawio#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Frandellma%2Fdocs%2Fmain%2Fdocs%2FWayfair%2520Discount%2FMessage%2520Broker%2520Architecture.drawio" width="800" height="500"></iframe>

### Database Schema

**user**

* id: int
* email: text
* ...columns for Auth.js

**discount_result**

* id: int
* email: text
* txId: text
* data: text
* created_at: datetime
* updated_at: datetime

### To do

* Find a free-tiered message broker solution. (Vercel? Firebase? GCP?)
* Create a Go service to:
    * Await for discount requests
    * Trigger the puppeteer script
    * Send the results via message queue
* Determine how the Next.js app will receive the results and deliver them to the user.
    * Maybe polling to start?
    * Web socket?
* Migrate web app to vercel.
* Migrate file storage to postgres storage.

## Consequences

### Benefits
* Web app is TLS authenticated, and automatically deployed to Vercel when there are changes.
* Eliminates non-static IP difficulties - ingress traffic no longer needs to be allowed since it will be reaching out to the message broker instead.

### Drawbacks
* Increase complexity with message broker  and additional service needed.
* New language in the tech stack (but that’s kinda the point).
* The Go service and Puppeteer script still needs to be hosted in the basement.

## Post Implementation Notes

* Implemented and then abandoned the Go service as it was just adding complexity in calling the puppeteer script. Using a Node script to subscribe to pubsub and it can just call the script normally.
* Landed on using GCP pubsub for a message broker
    * Topics:
        * discount-request
        * discount-request-dlq
        * discount-result
        * discount-result-dlq
    * Subscriptions:
        * discount-request-sub-local - for local testing, uses env: local attribute as a filter
        * discount-request-sub-prod - for prod, uses env: prod attribute as a filter
        * discount-result-sub - consumes local and prod results, uses a push sub to the vercel hosted app.
* The move to Vercel hosting moved things to https, so locatstorage for users is lost. Not going to bother with a migration path.
* Delivering the results to users is done by polling from the browser - the txId is returned immediately for the web app once submitted to the request topic, then the web app polls the server until the result is available.
    * Not ideal, but good for now and gets around the Vercel function time limit.
* Fully migrated web-ui and discount.* domain to Vercel and no longer have web server running locally and no need for ingress port.
* Enabled authentication as I was worried that hosting at the Vercel IP would be more likely to attract anonymous traffic.
* Have not associated discount requests to users yet.
* Excluded auth for `/view` and `/api/discoutResult` routes. 
    * `/view` is needed for viewing discount results anonymously
    * `/api/discountResult` needed for pubsub push subscriptions but looking into enabling auth to validate the request comes from google.