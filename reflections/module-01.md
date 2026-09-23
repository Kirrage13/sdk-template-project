# Module 1 — Reflection

## What I did

I forked the template repository, cloned my fork and set up the project locally.

I created the `.env` file, started the project with Docker Compose and checked that all services were running.

I also looked through `docker-compose.yml` and traced the `GET /products` request from the frontend to the PHP service, PostgreSQL and back to the browser.

I made a system diagram and wrote notes about how the services communicate.

## What I did not understand at first

At first I did not understand why Docker Compose could not start the project.

The problem was that the `.env` file was missing.

I also got confused why backend services use names like `database`, while the browser uses `localhost`.

Now I understand that containers use service names inside the Docker network, while the browser uses published ports on localhost.

## What I would do differently

Next time I would check the setup instructions and `.env.example` before trying to start the project.

I would also look at `docker-compose.yml` earlier, because it explains most of the connections between the services.

## How long this took me

About 3-4 hours.
