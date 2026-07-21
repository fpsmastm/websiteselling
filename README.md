# J&N Website Creators

A complete Next.js 16 marketing site + admin dashboard for J&N Website Creators, a branding and web design studio for new businesses.

**Stack:** Next.js 16 (App Router, TypeScript) · Tailwind CSS v4 · Drizzle ORM · PostgreSQL · Zod · Resend / SMTP (nodemailer) · signed-cookie admin auth (jose)

---

## What's included

- **Marketing site** — hero, services, about, pricing (flexible/negotiable), process, newsletter signup, and a full contact form, all built around a terracotta (`#c5622d`) brand identity with a blueprint-grid/ampersand design motif.
- **Contact form** → validates with Zod → saves to PostgreSQL (`inquiries` table) → emails a notification to `Jandnwebsitecreators@gmail.com` via **Resend or SMTP** (your choice).
- **Newsletter signup** → saves subscribers to PostgreSQL (`newsletter_subscribers` table).
- **`/admin` dashboard** — passcode-protected (no user accounts needed). View every inquiry and newsletter subscriber, expand messages, mark as read/unread, reply by email.

---

## 1. Install dependencies

```bash
npm install
```

## 2. Set up your environment

```bash
cp .env.example .env.local
```

Then fill in `.env.local`:

| Variable | What it's for |
|---|---|
| `DATABASE_URL` | Your PostgreSQL connection string (Neon, Supabase, Railway, RDS, or local Postgres all work). |
| `ADMIN_PASSCODE` | The passcode for `/admin`. Make it long and random — this is the only thing standing between the public and your inquiries. |
| `ADMIN_SESSION_SECRET` | Signs the admin session cookie. Generate one with `openssl rand -hex 32`. |
| `EMAIL_PROVIDER` | `resend` or `smtp` — pick one. |
| `NOTIFY_EMAIL` | Where inquiry/notification emails are sent (defaults to `Jandnwebsitecreators@gmail.com`). |
| `EMAIL_FROM` | The "from" address on outgoing emails. |
| `RESEND_API_KEY` | Required if `EMAIL_PROVIDER=resend`. Get one at resend.com. |
| `SMTP_HOST` / `SMTP_PORT` / `SMTP_SECURE` / `SMTP_USER` / `SMTP_PASSWORD` | Required if `EMAIL_PROVIDER=smtp`. For Gmail, use an **App Password** (not your regular password) — generate one at myaccount.google.com/apppasswords, with `SMTP_HOST=smtp.gmail.com`, `SMTP_PORT=465`, `SMTP_SECURE=true`. |

## 3. Set up the database

Run the included migration to create the two tables (`inquiries`, `newsletter_subscribers`):

```bash
npm run db:migrate
```

If you change `src/db/schema.ts` later, generate a fresh migration first:

```bash
npm run db:generate
npm run db:migrate
```

You can also browse your data visually with Drizzle Studio:

```bash
npm run db:studio
```

## 4. Run it

```bash
npm run dev
```

- Site: http://localhost:3000
- Admin: http://localhost:3000/admin/login

## 5. Deploy

Works on Vercel, Railway, Render, or any Node host:

1. Push to a git repo and import it into your host of choice.
2. Set all the environment variables from `.env.local` in your host's dashboard.
3. Run `npm run db:migrate` once against your production database (or run it as a build/release step).
4. Deploy.

---

## Project structure

```
src/
  app/
    page.tsx                # Homepage — assembles all sections
    admin/
      login/page.tsx        # Admin passcode login
      dashboard/page.tsx     # Protected inquiries + subscribers view
    api/
      contact/route.ts       # Contact form submission
      newsletter/route.ts    # Newsletter signup
      admin/login/route.ts   # Passcode check → sets session cookie
      admin/logout/route.ts
      admin/inquiries/[id]/read/route.ts
  components/                # Hero, Services, About, Pricing, Process, etc.
  db/
    schema.ts                # Drizzle schema (inquiries, newsletter_subscribers)
    index.ts                 # Drizzle client
    migrate.ts                # Standalone migration runner
  lib/
    validation.ts             # Zod schemas (shared client + server)
    email.ts                  # Resend / SMTP email sending
    auth.ts / session.ts      # Admin passcode + signed session cookie
  proxy.ts                    # Route protection for /admin/* (Next.js 16's
                               # renamed "middleware" convention)
drizzle/                      # SQL migrations
```

## Notes

- The contact form still confirms success to the visitor even if the notification email fails to send — their inquiry is safely in the database either way, and you'll see it in `/admin`.
- A honeypot field (`company`) on both forms quietly discards bot submissions without showing an error.
- The admin login endpoint is rate-limited (8 attempts per 10 minutes per IP) to slow down brute-force guessing.
- Swap `EMAIL_PROVIDER` between `resend` and `smtp` any time — no code changes needed, just environment variables.
