# CleanCircle

CleanCircle is a small, demonstration-ready community cleanliness reporting system aligned with **UN Sustainable Development Goal 11: Sustainable Cities and Communities**. It uses Express, EJS, MongoDB/Mongoose, server-side sessions, bcryptjs, and Multer.

## Features

- Citizen self-registration, login, profile, issue reporting, image upload, status tracking, and feedback after resolution.
- Citizen report edits and deletion are limited to reports in `Pending`.
- Citizens can pin a report with browser GPS or by clicking/dragging a map marker; workers see a report map and direct-to-location directions.
- Admin dashboard with status metrics and Chart.js category/status charts, filters, CSV export, worker management, assignment, and verification decisions.
- Worker task list limited to assigned reports, with start/rework and after-cleaning proof submission.
- Status history is appended to every workflow transition.

## Requirements

- Node.js 18+
- MongoDB running locally or an accessible MongoDB connection string

## Setup

1. Install dependencies:

   ```bash
   npm install
   ```

2. Copy `.env.example` to `.env` and configure `MONGODB_URI` and a strong `SESSION_SECRET`. In PowerShell:

   ```powershell
   Copy-Item .env.example .env
   ```

   For MongoDB Atlas, replace the local `MONGODB_URI` with the connection string shown under **Connect → Drivers** in Atlas. Replace the username, password, and cluster host with your database user's values. In Atlas **Network Access**, allow your current IP address. Do not commit `.env` or share the connection string; `.env` is ignored by Git.
3. Optionally set `ADMIN_NAME`, `ADMIN_EMAIL`, and `ADMIN_PASSWORD`. On startup, CleanCircle creates this initial administrator only when no admin account exists. Otherwise, create an admin directly in the database using a bcrypt hash.
4. Ensure your configured MongoDB instance is running/reachable, then run:

   ```bash
   npm run dev
   ```

   Open <http://localhost:3000>. Use `npm start` for normal startup.

For a quick college demonstration without installing MongoDB or configuring Atlas, run `npm run demo`. This starts a temporary in-memory MongoDB; accounts, reports, and sessions are erased when the command stops. The first run downloads the MongoDB binary. Use the configured, persistent database with `npm run dev` or `npm start` for normal use.

## Deploy to Netlify

The `netlify.toml` configuration publishes static assets from `public/` and routes application requests to the Express function in `netlify/functions/server.js`. Deploy the repository root as the Netlify base directory. In the Netlify site settings, add these environment variables for the Production context (and Deploy Previews too if needed):

- `MONGODB_URI`: a reachable MongoDB Atlas connection string. Do not use the local `127.0.0.1` example on Netlify.
- `SESSION_SECRET`: a random secret with at least 32 characters.
- `GOOGLE_MAPS_API_KEY`: optional; required only for Google Maps rendering.
- `ADMIN_EMAIL` and `ADMIN_PASSWORD`: optional; used to seed the initial admin account.

Configure Atlas network access to allow connections from your deployment, then redeploy after changing environment variables. Never commit `.env` or paste database credentials into source code. Netlify Functions use temporary local storage, so production report-image uploads need durable object storage; files written to the function filesystem are not persistent.

For local development, start the Express app with `npm run demo` or `npm run dev`, then open <http://localhost:3000>. Netlify deploys use the public site URL and the configured serverless function instead.

If startup reports that it could not connect to MongoDB, verify the URI and Atlas Network Access IP allowlist. The application cannot start its database-backed session store or seed an administrator until MongoDB is reachable.

The initial admin credentials are the values configured in `.env`. Admins create worker accounts from **Workers**. Citizen accounts are created through the registration page. Never use the example password or development session secret in a deployed environment.

## Report workflow

`Pending → Assigned → In Progress → Awaiting Verification → Resolved`

An administrator can instead mark a submitted task `Rework Required` or `Rejected`, and must add a note. Rework returns the task to its assigned worker, who can begin it again. Only the admin can assign reports or resolve work.

## Project layout

```text
config/       MongoDB connection
controllers/  Authentication, citizen reports, admin, and worker behavior
middleware/   Session role checks, async errors, image uploads
models/       User, Report, and Feedback schemas
public/       CSS and browser JavaScript
routes/       Role-specific Express routes
uploads/      Locally stored report photos (created as needed)
views/        EJS pages and shared header/footer
server.js     Express application entry point
```

Uploaded photos are limited to JPG, PNG, and WEBP files up to 5 MB. Files are served from `/uploads`. Bootstrap, Font Awesome, and Chart.js load from their public CDNs.

Location pins use browser geolocation, which requires user permission and a secure browser context (HTTPS or localhost). If GPS is unavailable, citizens can click or drag a pin on the map or enter coordinates. Workers can open driving directions in Google Maps. Leaflet map tiles use OpenStreetMap data.
To use Google Maps for location maps, set `GOOGLE_MAPS_API_KEY` in `.env`. Enable the Maps JavaScript API and billing in Google Cloud, and restrict the key to your application's HTTP referrers. If the key is omitted or Google Maps fails to load, the app falls back to Leaflet with OpenStreetMap/CARTO tiles.
