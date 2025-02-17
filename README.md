# Debrief on Improvements

## Code Quality And Readability

1. Adopt TypeScript to enforce better type safety, reduce runtime errors, and improve maintainability.
1. Implement automated tests using Jest or Mocha to validate logic and prevent regressions.
1. Improve modularization by breaking down responsibilities into smaller, reusable functions.
1. Standardize logging using a structured format with a logging library like winston for better debugging and analysis.
1. Enhance error handling, adding custom error classes and more contextual messages for easier debugging.

## Project Architecture

1. Create a dedicated layer for external requests (HubSpot and MongoDB) to separate HTTP and database interactions from business logic.
1. Separate business logic from integration code, ensuring that services process data before storing it in the database.
1. Adopt a more modular project structure, organizing code into services/, repositories/, and workers/ for better maintainability.

## Code Performance

1. Move token management to a separate worker

- A dedicated worker would handle token refresh and store the valid token in Redis.
- The main worker fetching data from HubSpot would simply retrieve the token from Redis, avoiding unnecessary refresh logic in multiple places.

2. Use a more robust queuing system

- Instead of manually managing queues with an array and async.queue, a dedicated system like Redis (BullMQ) or Google Cloud Pub/Sub would improve scalability and message persistence.

3. Optimize HubSpot API requests

- Implement batch requests to minimize the number of separate API calls.
  Use caching (e.g., Redis) for contact associations to avoid redundant API requests.

4. Webhooks (if possible)

- The system currently uses polling to fetch updates from HubSpot, but it could leverage webhooks for real-time processing, reducing requests and improving efficiency.

## Bugs & Issues Encountered

1. `lastPulledDates` was inaccessible

- Fixed by converting the Mongoose document into a plain object using `.toObject()`.

2. Error when fetching contacts from HubSpot (`operator IN requires values`)

- Solved by removing duplicate IDs and ensuring the filter used `values` instead of `value`.

3. Issues with access token management

- API requests failed when the token expired. Storing tokens in Redis could solve this issue.

# Screenshot Output

---

# API Sample Test

## Getting Started

This project requires a newer version of Node. Don't forget to install the NPM packages afterwards.

You should change the name of the `.env.example` file to `.env`.

Run `node app.js` to get things started. Hopefully the project should start without any errors.

## Explanations

The actual task will be explained separately.

This is a very simple project that pulls data from HubSpot's CRM API. It pulls and processes company and contact data from HubSpot but does not insert it into the database.

In HubSpot, contacts can be part of companies. HubSpot calls this relationship an association. That is, a contact has an association with a company. We make a separate call when processing contacts to fetch this association data.

The Domain model is a record signifying a HockeyStack customer. You shouldn't worry about the actual implementation of it. The only important property is the `hubspot`object in `integrations`. This is how we know which HubSpot instance to connect to.

The implementation of the server and the `server.js` is not important for this project.

Every data source in this project was created for test purposes. If any request takes more than 5 seconds to execute, there is something wrong with the implementation.
