---
date: 2025-02-01
title: Stardelve - Read & Publish SciFi Stories
weight: 1
tags: ["webdev", "typescript", "go", "react", "sql"]
image: "stardelve.png"
---

[Stardelve](https://stardelve.com/) is a fiction writing website along the veins of Wattpad, RoyalRoad, etc. designed specifically for hosting SciFi stories. It was my first major dive back in to web development with a fullstack project

<!--more-->

I built this website not only to gain deeper experience in web development, but as a place that could specialise and host science fiction stories that would otherwise go unnoticed. While many of the existing websites out there allow any kind of story on their platform, it results in a focus either on the romance genre (Wattpad, AO3, etc) or the litrpg & fantasy genre (RoyalRoad, Scribblehub)

# Frontend

Built using
- Typescript
- Tailwind via NativeWind
- Expo
- React Native
- React Native for Web
- React Query (Tanstack Query)
- React Hook Form

- Developed as a SPA application which communicates using JSON & REST API with the backend
- The website UI/UX was largely based on RoyalRoad but with a more Scifi appropriate darker minimalism
- All UI components and styles were made myself alongside using Nativewind rather than a UI component framework/library
- There is currently no mobile app available. Using React Native as a base should allow for that in the future
- It has a mobile-responsive design using flexbox

Caching is handled via React Query, with manual cache invalidation strategies on the frontend to handle authors creating and editting their work.

# Backend

Built using
- Go
- golang-migrate
- snowflake
- pgx
- Echo Router
- SQL (Postgres)

It's a Monolith with clear separation between different features within the codebase to allow it to scale in the future if needed as a Distributed Monolith

- Discord OAuth integration for login & registration
- A handwritten AuthN & AuthZ 'service' that supports multiple profiles and multiple login providers, all linked under a single account
    - Session cookie stored on the frontend
    - Lots of security considerations taken, e.g Secure HTTP-only cookies, CSRF (inc. Login CSRF), CSP, CORS
- Uses LCG (Linear Conguential Generator) & XOR to Obfuscate Database ID's and generate URL-safe shortcodes for stories and chapters, by uniquely mapping each Database ID to a separate ID within the same bitrange before converting to Base62
- Created a Geo2IP 'service' to determine the location of IP addresses connecting to the backend for security and logging, with automated daily updates of the Geo2IP database, as well as leveraging country information passed along by Cloudflare
- A modified Snowflake ID system was used as a basis for all backend ID's
- Uses 'bruno', a Postman alternative, to test endpoints. Also has automated creation of 'dev' pages when built in development mode that allow for easily accessing and debugging endpoints via the browser
- Custom validation for user-provided data via endpoints
- Anonymously stores user data via a rotating hash of IP & Useragent to comply with GDPR
- Uses an LRU in-memory and in-application cache to minimize calls to the database and improve endpoint response times
    - Includes a lot of logic for handling caching and invalidation correctly
    - Allows the service to scale to many users without the database being overloaded, as it's hosted on Supabase, so now it can remain on the free or cheapest paid tier for a long time
- Handwritten SQL queries for all DB operations, with transactions and logging of DB actions taken
- Uses a three-layer architecture across each internal service - Handlers, Business Logic, and Repository

# Hosting

- The frontend SPA is served by the backend as a combined Webserver + API endpoint, with Cloudflare caching where possible
- Cloudflare is used primarily as a proxy and firewall
- The backend is running on a VPS provided using Hetzner, set up manually with best practices for security and using Docker with Dokploy and Traefik to manage the dashboard and proxy requests to the correct service
- The Postgres Database is hosted on Supabase
- Analytics are self-hosted and provided by Umami Analytics while complying with GDPR

# Database

- Full normalisation
- A soft-delete approach was taken for data entries
- Audit table for stories and chapters
    - The audit table is automatically filled in by triggers that run and convert table entries to JSON as before/after column data
- Uses views to handle accessing tables correctly (non-deleted entries and publically published entries, for example)
- Primary ID's are based on Snowflake ID's
