# plan-trip-backend

API for **plann.er**, a group trip planner. The organizer creates a trip, invites people by email, and the group builds an activity schedule and a list of useful links (bookings, tickets, etc.) together.

## Stack

- **Node.js** + **TypeScript**
- **Fastify 4** with **Zod** (`fastify-type-provider-zod`) for route validation and typing
- **Prisma** ORM with **SQLite**
- **Nodemailer** for confirmation emails (uses [Ethereal](https://ethereal.email) test accounts in development)
- **Day.js** for dates

## Flow

1. `POST /trips` creates the trip with the owner already confirmed and the guests pending, and emails the owner a confirmation link.
2. Opening `GET /trips/:tripId/confirm` confirms the trip and sends each guest their own confirmation link.
3. `GET /participants/:participantId/confirm` confirms a participant.

Confirmation links redirect to the frontend (`WEB_BASE_URL`).

## Data model

```
Trip ─┬─< Participant   (name, email, is_confirmed, is_owner)
      ├─< Activity      (title, occurs_at)
      └─< Link          (title, url)
```

Schema in [`prisma/schema.prisma`](./prisma/schema.prisma), migrations in `prisma/migrations`.

## Endpoints

| Method | Route | Description |
|---|---|---|
| POST | `/trips` | Creates a trip (`destination`, `starts_at`, `ends_at`, `owner_name`, `owner_email`, `emails_to_invite`) |
| GET | `/trips/:tripId` | Trip details |
| PUT | `/trips/:tripId` | Updates destination and dates |
| GET | `/trips/:tripId/confirm` | Confirms the trip and sends the invites |
| POST | `/trips/:tripId/invites` | Invites a new participant by email |
| GET | `/trips/:tripId/participants` | Lists participants |
| GET | `/participants/:participantId` | Participant details |
| GET | `/participants/:participantId/confirm` | Confirms a participant |
| POST | `/trips/:tripId/activities` | Creates an activity (`title`, `occurs_at`) |
| GET | `/trips/:tripId/activities` | Activities grouped by day of the trip |
| POST | `/trips/:tripId/links` | Adds a link (`title`, `url`) |
| GET | `/trips/:tripId/links` | Lists links |

### Rules and errors

- Dates are validated on the server: a trip can't start in the past or end before it starts, and activities must fall within the trip dates.
- A central error handler turns Zod errors into `400` responses with the invalid fields, and business-rule errors (`ClientError`) into `400` responses with the message.

## Running locally

Requirement: Node.js 20.6+ (the `dev` script uses `--env-file`).

```bash
npm install
cp .env.example .env
npx prisma migrate dev
npm run dev
```

The API runs at `http://localhost:3333`. Sent emails can be viewed on Ethereal; to browse the database, run `npx prisma studio`.

### Environment variables

| Variable | Example | Purpose |
|---|---|---|
| `DATABASE_URL` | `file:./dev.db` | SQLite database |
| `API_BASE_URL` | `http://localhost:3333` | Base URL for confirmation links in emails |
| `WEB_BASE_URL` | `http://localhost:3000` | Where confirmation links redirect to |
| `PORT` | `3333` | API port |
