# ReachOut AI — Automated HR Outreach Platform

**Live:** https://applyagent.netlify.app/

An end-to-end job outreach platform that lets candidates send personalised application emails to HR contacts at scale, with AI-assisted email writing, open/reply tracking, scheduling, and a Kanban pipeline to manage applications.

---

## Table of Contents

1. [Tech Stack](#tech-stack)
2. [Authentication](#authentication)
3. [Campaigns](#campaigns)
   - [Create Campaign](#create-campaign)
   - [Contacts — PDF Upload](#contacts--pdf-upload)
   - [Contacts — Enter Manually](#contacts--enter-manually)
   - [AI Email Generation](#ai-email-generation)
   - [Resume for AI](#resume-for-ai)
   - [Compose Email](#compose-email)
   - [Save Draft / Send / Schedule](#save-draft--send--schedule)
   - [Campaign History & Contact Management](#campaign-history--contact-management)
4. [App Board (Kanban)](#app-board-kanban)
5. [Schedule](#schedule)
6. [Analytics](#analytics)
7. [Job Toolkit](#job-toolkit)
8. [Candidate Profile](#candidate-profile)
9. [Pricing — Free vs Pro](#pricing--free-vs-pro)
10. [Email Tracking](#email-tracking)
11. [Follow-up Emails](#follow-up-emails)
12. [Environment Variables](#environment-variables)
13. [Running Locally](#running-locally)

---

## Tech Stack

| Layer     | Technology |
|-----------|------------|
| Frontend  | React 18, Vite, Tailwind CSS, React Router |
| Backend   | Node.js (ESM), Express |
| Database  | MongoDB (Mongoose) |
| Email     | Nodemailer + Gmail SMTP |
| AI        | Groq LLM — `llama-3.3-70b-versatile` (free tier) |
| Auth      | JWT + Google OAuth 2.0 |
| Deploy    | Netlify (frontend) + custom server (backend) |

---

## Authentication

- **Email / Password** — register, login, forgot password, reset password via email link
- **Google OAuth** — one-click sign-in with Google account
- **JWT sessions** — token stored in `localStorage`, sent on every API request
- **Protected routes** — dashboard is gated behind auth; unauthenticated users are redirected to login

---

## Campaigns

The core feature. A campaign bundles a set of HR contacts with an email template and tracks every send, open, and reply.

### Create Campaign

1. Give the campaign a **name** (used in history and the database).
2. Add contacts via PDF upload or manually.
3. Optionally attach your resume for AI generation.
4. Select the target role and customise the email subject.
5. Write or AI-generate the email body.
6. Save as draft or send immediately.

---

### Contacts — PDF Upload

- Drag-and-drop or click to upload a PDF file containing HR contact details.
- The backend extracts all email addresses automatically using `pdfjs-dist`.
- Extracted emails appear as **removable chips** below the upload zone.
- Each chip shows the email; tapping × removes just that contact before sending.
- "Clear all" removes the full extracted list at once.

---

### Contacts — Enter Manually

- A **tag-input** box — type one or more email addresses separated by commas.
- Press **Enter** or **,** to add a contact as a chip instantly.
- **Backspace** on an empty input removes the last chip.
- Each chip has an × button to remove it individually.
- Duplicate emails are automatically ignored.
- Incomplete / invalid addresses are highlighted in amber.
- **Job Description toggle** — a checkbox reveals a textarea to paste the JD, which is used later by the AI to write a tailored email.

---

### AI Email Generation

Powered by **Groq LLM** (free tier, `llama-3.3-70b-versatile`).

**Flow:**

| Step | What you do |
|------|-------------|
| Step 01 (Manual mode) | Check "Add Job Description" → paste the JD |
| Step 01 (PDF mode) | JD textarea appears inside the Compose section |
| ★ Resume step | Upload your resume PDF or paste a summary |
| Step 03 | Click **✨ Generate with AI** |

The AI receives:
- Full job description (up to 4 000 chars)
- Target role
- Resume summary (up to 1 000 chars, if provided)

It returns a tailored **subject line** and **email body** which are injected directly into the editor. The editor re-mounts so the generated content always appears immediately.

---

### Resume for AI

An optional step between contacts and role selection.

**Upload PDF tab**
- Click-to-upload zone — PDF files only, validated both client-side (`accept=".pdf"`) and server-side (MIME type check).
- Backend extracts up to 4 000 characters of plain text from the PDF.
- A 300-character preview is shown after successful extraction.
- Click the zone again to replace the file.

**Enter Summary tab**
- Free-form textarea — paste key skills, highlights, or a written summary.
- Character counter shown (max 4 000).

---

### Compose Email

- **Rich text editor** with a full toolbar: bold, italic, underline, heading, align left/centre, bullet list, numbered list, insert link, horizontal divider.
- WYSIWYG `contentEditable` editor — what you see is what recipients receive.
- When AI generates an email the editor remounts with the new content immediately.
- The email subject line is a fully editable input — pre-filled from the role dropdown but freely customisable.

---

### Save Draft / Send / Schedule

| Action | Behaviour |
|--------|-----------|
| **Save as Draft** | Campaign stored in DB with status `draft`; can be sent later from History |
| **Create & Send** | Campaign created and emails dispatched immediately (concurrent sends via `p-limit`) |
| **Schedule** (from History) | Pick a date + time; campaign fires automatically via the background scheduler |

Each sent email includes:
- A **1×1 invisible tracking pixel** for open detection
- The candidate's **resume PDF** attached automatically

---

### Campaign History & Contact Management

Listed in reverse-chronological order. Each campaign card shows:

- Name, role, status badge (Draft / Running / Completed / Scheduled / Failed)
- Contact count, sent, failed, opened, and replied counts
- Created date and scheduled fire time (if applicable)

**Per-campaign actions:**

| Action | Description |
|--------|-------------|
| **View contacts** | Expand the card to see all contacts as rows with colour-coded status badges |
| **Remove contact** | Hover a contact row → × button → permanently removes that contact from the campaign |
| **Send** | Available on Draft and Failed campaigns with pending contacts |
| **Schedule** | Inline date/time picker; free plan: 1 scheduled campaign at a time |
| **Delete campaign** | Removes the campaign and all its data from the database |

---

## App Board (Kanban)

A visual board to track every application through hiring stages:

```
Applied → Opened → Replied → Screening → Interview → Offer → Rejected
```

Each card shows the contact email, company name, and last activity. Move cards between columns to keep your pipeline up to date.

---

## Schedule

Dedicated view of all upcoming scheduled campaigns. A background Node cron job (runs every minute) checks for campaigns whose `scheduledAt` time has passed and triggers the send automatically — no manual action needed.

---

## Analytics

Overview metrics across all campaigns:

| Metric | Description |
|--------|-------------|
| Total sent | All emails successfully dispatched |
| Open rate | % of sent emails where the tracking pixel fired |
| Reply rate | % of sent emails where a reply was detected via IMAP |
| Failed | Emails that bounced or errored during send |

Includes a daily trend chart (last 7 or 30 days), per-campaign breakdown table, and response category distribution (Interview Invitation, Assessment Round, HR Discussion, Rejection, Offer, No Response).

---

## Job Toolkit

A library of ready-to-use outreach templates for 9 engineering roles:

| Role | Templates included |
|------|--------------------|
| Frontend Engineer | HR email, cover letter, LinkedIn DM, cold email, cold DM, referral request, follow-up, interview intro |
| Backend Engineer | Same set |
| Full Stack Engineer | Same set |
| Data Engineer | Same set |
| ML / AI Engineer | Same set |
| DevOps / Cloud | Same set |
| Mobile Engineer | Same set |
| QA Engineer | Same set |

Templates are role-aware — they auto-fill stack keywords, domain context, and a realistic highlight bullet.

---

## Candidate Profile

Save your personal details once; they are used across the platform for context:

- Full name, current title, years of experience
- Skills list
- LinkedIn and GitHub URLs
- Phone number
- Resume PDF upload (stored server-side)

---

## Pricing — Free vs Pro

| Limit | Free | Pro |
|-------|------|-----|
| Campaigns per day | 1 | Unlimited |
| Contacts per campaign | 5 | Unlimited |
| Scheduled campaigns | 1 active at a time | Unlimited |
| AI email generation | ✅ | ✅ |
| Analytics | ✅ | ✅ |
| Kanban board | ✅ | ✅ |
| Job Toolkit | ✅ | ✅ |

Free-plan limits are enforced on both frontend (UI banners, disabled buttons) and backend (HTTP 403 with a `FREE_LIMIT_*` error code).

---

## Email Tracking

Each outgoing email embeds a hidden 1×1 PNG tracking pixel:

```
GET /api/track/:trackingId
```

When the recipient's email client loads the image, the backend:
1. Marks the contact's status as `opened`
2. Records `openedAt` timestamp
3. Increments the campaign's `openCount`

**Reply detection** runs via Gmail IMAP (`imapflow`) polling every 3 minutes. When a reply is matched to a sent email, the contact is moved to `replied`, `replyCount` is incremented, and a response category can be assigned (e.g. "Interview Invitation").

---

## Follow-up Emails

Send a follow-up to contacts who received an email but have not replied after N days:

- Configurable days threshold (default: 3 days)
- Tracks `followUpSent` and `followUpSentAt` per contact
- Second-attempt follow-ups use a slightly different, gentle tone
- Campaign `followUpCount` incremented per successful send

---

## Environment Variables

Create `backend/.env`:

```env
MONGO_URI=mongodb+srv://...
PORT=5050
CLIENT_ORIGIN=http://localhost:5173

JWT_SECRET=your_long_random_jwt_secret

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your@gmail.com
SMTP_PASS=your_16_char_app_password

GROQ_API_KEY=gsk_...        # Free at console.groq.com

MAX_CONCURRENCY=5

GOOGLE_CLIENT_ID=...        # console.cloud.google.com
SERVER_URL=http://localhost:5050
```

---

## Running Locally

```bash
# 1. Clone the repository
git clone <repo-url>
cd hire-job-reach

# 2. Start the backend
cd backend
npm install
cp .env.example .env        # fill in all values
npm run dev                 # node --watch  →  http://localhost:5050

# 3. Start the frontend (new terminal)
cd ../frontend
npm install
npm run dev                 # Vite  →  http://localhost:5173
```

Open **http://localhost:5173** — the frontend proxies all `/api/*` requests to the backend automatically via the Vite dev server config.
