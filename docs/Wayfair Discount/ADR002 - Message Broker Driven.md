# ADR001 Message Broker Driven
Mainly an excuse to play around with Golang, but also, finally migrate the Next.js web app to be freely hosted in Vercel by properly queuing, and asynchronously performing discount checker requests thereby unlocking some asynchronous discount check delivery methods to explore (SMS, whatsApp, email, etc).

## Status
In Progress

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

## Consequences

### Benefits
* Web app is TLS authenticated, and automatically deployed to Vercel when there are changes.
* Eliminates non-static IP difficulties - ingress traffic no longer needs to be allowed since it will be reaching out to the message broker instead.

### Drawbacks
* Increase complexity with message broker  and additional service needed.
* New language in the tech stack (but that’s kinda the point).
* The Go service and Puppeteer script still needs to be hosted in the basement.