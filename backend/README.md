# Prescripto Backend

Auth + appointment booking with automated SMS (Twilio) and email (Nodemailer/Gmail)
reminders sent 12 hours before each appointment.

## What changed from the original files

- **Database**: `authStore.js` and `appointmentsStore.js` now persist to **MongoDB**
  via Mongoose instead of in-memory arrays (`models/User.js`, `models/Appointment.js`,
  `db.js`). Data now survives restarts and works across multiple server instances.
- **Password security**: passwords are now hashed with **bcrypt** before being stored.
  The original code stored plain-text passwords — that's fixed.
- **Email reminders**: new `emailService.js` (Nodemailer + Gmail SMTP). The
  scheduler now sends SMS *and* email (if an `email` was given on the appointment).
  Both channels fall back to console logging if not configured, so everything
  still runs end-to-end without real credentials.
- **Validation**: a few extra guards (appointment can't be booked in the past,
  password minimum length, required fields) since this now hits a real DB.
- **Tests**: updated to run against an in-memory MongoDB
  (`mongodb-memory-server`) instead of the plain-array store, plus new tests
  for the appointment/reminder-window logic.

## Setup

```bash
npm install
cp .env.example .env
```

Fill in `.env`:

- `MONGODB_URI` — local Mongo (`mongodb://localhost:27017/prescripto`) or an
  Atlas connection string.
- `TWILIO_ACCOUNT_SID` / `TWILIO_AUTH_TOKEN` / `TWILIO_FROM_NUMBER` — from the
  [Twilio console](https://console.twilio.com). Leave blank during dev; SMS
  will just be logged to the console.
- `EMAIL_USER` / `EMAIL_PASS` — a Gmail address and a **Gmail App Password**
  (not your normal password — Google blocks plain SMTP logins). Create one at
  https://myaccount.google.com/apppasswords (requires 2-Step Verification).
  Leave blank during dev; email will just be logged to the console.

## Run

```bash
npm start          # production
npm run dev         # auto-restart on file changes
```

The server starts on `http://localhost:3001` and the reminder scheduler
(checks every minute for appointments entering their 12-hour window) starts
automatically.

## Test

```bash
npm test
```

Uses `mongodb-memory-server` to spin up a throwaway in-memory MongoDB for the
test run — no real database needed. Note: the first run downloads a MongoDB
binary, so it needs normal internet access (won't work in network-restricted
sandboxes).

## API

| Method | Route             | Body                                                                 |
|--------|--------------------|-----------------------------------------------------------------------|
| POST   | `/api/auth`        | `{ mode: 'signup', name, email, password }` or `{ mode: 'login', email, password }` |
| POST   | `/api/appointments`| `{ doctorName, date: 'YYYY-MM-DD', time: 'HH:mm', patientName, phone, email? }` |
| GET    | `/api/appointments`| —                                                                     |
| POST   | `/api/reset`       | dev-only, wipes users (disabled when `NODE_ENV=production`)           |

`email` on the appointment is optional — omit it and only SMS goes out.
`phone` is always required, since it's the primary reminder channel.
