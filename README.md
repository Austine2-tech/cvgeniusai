# CVGenius AI

AI-powered CV and document platform. Built by **@austohmgazadev**.

Status: **Phases 1 to 4 written.** Phase 2 adds the AI questionnaire, CV generation, 7 templates, the live editor, and PDF/DOCX export. Phase 3 adds document upload and six AI career tools. Phase 4 adds PDF tools, subscriptions and M-Pesa.

See `QUICKSTART.md` for the 10-minute first run.

## Structure

```
cvgenius-ai/
  frontend/   React + Vite + Tailwind + Framer Motion + Lucide
  backend/    Node + Express (AI calls, exports, secrets live here only)
  supabase/   schema.sql (already applied to your cvgenius-ai project)
```

## 1. Supabase

Run `supabase/schema.sql`, then the files in `supabase/migrations/` (`phase3_hardening.sql`, `phase4_billing.sql`). They are safe to re-run. On the live project they are already applied. Still needed in the Supabase dashboard:
1. Authentication -> Sign In / Providers -> Email: turn **OFF "Confirm email"**.
2. Authentication -> URL Configuration: Site URL `http://localhost:5173`, and add `http://localhost:5173/reset-password` to Redirect URLs.

## 2. Environment

`frontend/.env` is already filled in (public URL + public key).

`backend/.env` needs:
- `SUPABASE_SERVICE_ROLE_KEY` : from Supabase -> Project Settings -> API Keys. **Secret.**
- `AI_API_KEY` : your OpenAI key from https://platform.openai.com/api-keys. **Secret.**
- `AI_MODEL` : defaults to `gpt-4o-mini`. Any chat model on your account works.
- `AI_BASE_URL` : optional, only for OpenAI-compatible providers.

Never put these two keys in the frontend or commit them.

## 3. Run

```
cd backend  && npm install && npm run dev     # http://localhost:4000
cd frontend && npm install && npm run dev     # http://localhost:5173
cd backend  && npm test                       # all backend tests (56)
cd frontend && node --test test/fileCheck.test.js
```

`GET http://localhost:4000/api/health` reports `aiConfigured: true` once the key is set.

## 4. Test checklist

Phase 1: landing page, register (no email verification needed), login/logout, forgot password, dashboard, settings, private-route redirect.

Phase 2:
1. Dashboard -> Generate CV -> choose Professional or Freelancing.
2. Fill the questionnaire. Try **Skip this section**. Personal info (name, email) is required.
3. Refresh mid-way: your answers are restored (stored in your browser only).
4. Review step -> Generate my CV. You land in the editor.
5. Check the writing: it should only contain facts you entered. If you left a section blank it should be absent.
6. Editor: edit text, switch templates, change colour/font, reorder or hide sections, Save.
7. Download PDF and DOCX; open both and compare with the preview.
8. My documents: edit, download, delete a CV.
9. Create a second account: it must not see the first account's CVs.

Phase 3:
1. My documents -> upload a PDF or DOCX (5 MB max). Try a wrong file type and an oversized file: both should be refused with a message.
2. Download the uploaded file back, then delete it (it should disappear from storage too).
3. AI tools -> pick each of the six tools with a saved CV, an uploaded file, and pasted text.
4. CV Analyzer, ATS Review, Job Matching show scores and lists. ATS Review includes measured checks (email, phone, headings, length, bullets, dates) that do not depend on the AI.
5. Job Matching needs a pasted job description. Check that "missing keywords" are only suggestions and the advice never tells you to claim skills you lack.
6. CV Enhancer, Cover Letter: the result is editable text with Copy and Download .txt.
7. Try a scanned (image-only) PDF: you should get a message asking you to paste the text instead.
8. Second account: it must not be able to analyse the first account's CVs or files.

Phase 4:
1. PDF tools (all run in the browser, nothing is uploaded): Preview a PDF and flip pages. Merge two or more and reorder them. Split by ranges such as `1-3, 5` or every page. Compress in Light or Strong mode.
2. Billing page: shows trial end date, status and renewal date. Without M-Pesa credentials the Pay button is disabled with an explanation (this is intended; success is never faked).
3. After adding Daraja sandbox credentials (below): enter a test number, approve the prompt, and watch the page change to "Payment confirmed" by itself. Check Payment history and the renewal date.
4. Pay twice in a row: the second payment adds 30 days on top of the first.
5. Cancel the prompt on the phone: the page should say the request was cancelled and no access is added.
6. Admin price: see "Making yourself admin" below, then change the KES amount on the Billing page and confirm the next prompt uses it.

## Phase 3: documents and AI tools (setup and testing)

**Upload rules (enforced in three places: browser, backend, Supabase Storage/database)**
- PDF and DOCX only, 5 MB maximum, 20 files per user, no empty files.
- The backend checks the real file bytes, not just the name: PDF header, DOCX zip structure, encrypted or password-protected files, zip bombs and unsafe paths.
- The bucket itself rejects anything over 5 MB or with another type, even if someone bypasses the app.
- Browsers cannot write to Storage or to `uploaded_files`. Uploads go to `PUT /api/files?name=...` and deletes to `DELETE /api/files/:id`. Deleting removes the Storage object first, then the row.
- Users can only read their own folder. Every backend lookup filters by the signed-in user and checks the storage path prefix.

**Error messages people will see**

| Situation | Message gist |
|---|---|
| Wrong type / `.doc` | Only PDF and DOCX; save old Word files as .docx |
| Over 5 MB | Larger than the 5 MB limit |
| Renamed or damaged file | Named .pdf/.docx but is not one |
| Password protected | Remove the password and upload again |
| Scanned / image only | Uploads, but flagged "No readable text"; paste text instead (no OCR yet) |
| AI key missing | Banner on the Tools page, button disabled, `ai_not_configured` from the API |
| No subscription | 402 with a link to Billing |

**Automated tests** (no network or keys needed): `cd backend && npm test` covers file validation, upload/delete/ownership, all six tools with a saved CV, an uploaded file and pasted text, missing AI credentials, and cross-user access.

**Manual checklist** (needs `npm install`, the backend running and a real AI key)
1. Upload a normal PDF and DOCX: both appear, and Tools can read them.
2. Upload a renamed `.exe` as `.pdf`, a 6 MB PDF, a password-protected PDF or DOCX, and a `.doc`: each is refused with a clear message.
3. Upload a scanned PDF: it is kept, shows "No readable text", and the Tools page suggests pasting.
4. Delete a file, then check Supabase -> Storage -> documents: the object is gone.
5. With two accounts, confirm account B cannot see, download or analyse account A's files (it gets "not found").
6. Blank `AI_API_KEY`, restart the backend: the Tools page shows the amber banner and the button is disabled.
7. Run each of the six tools on a saved CV, an uploaded file and pasted text.

## Phase 5: deployment and maintenance

**Backend (Render, Railway, Fly or any Docker host).** `backend/Dockerfile` is included. Set these environment variables on the host (never in the code): `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `AI_API_KEY`, `AI_MODEL`, `FRONTEND_URL` (your exact site address), `TRUST_PROXY=1`, and the `MPESA_*` values. Then open `https://your-backend/api/health` and confirm `supabaseConfigured` and `aiConfigured` are `true`.

**Frontend (Vercel or Netlify).** Root folder `frontend`, build command `npm run build`, output `dist`. Set `VITE_API_URL` to your backend address, plus the existing public `VITE_SUPABASE_*` values. `vercel.json` makes page refreshes work.

**Supabase.** In Authentication -> URL Configuration, change Site URL to your live address and add `https://your-site/reset-password` to Redirect URLs.

**M-Pesa live.** Set `MPESA_ENV=production`, your own shortcode and passkey, and `MPESA_CALLBACK_BASE_URL` to the public backend address. Safaricom must be able to reach `/api/billing/mpesa/callback/<secret>`.

**Cleaning up deleted accounts.** Deleting an account removes its database rows but not its Storage files. Run `cd backend && node scripts/sweep-orphans.js` to list them, and add `--delete` to remove them.

**Before launch.** Run every checklist in this README with two real accounts, and rotate any key that was ever pasted anywhere outside `backend/.env`.

## M-Pesa (Daraja) setup

1. Create an account at https://developer.safaricom.co.ke, create an app with "Lipa Na M-Pesa Sandbox" enabled, and copy the Consumer Key, Consumer Secret and the test Passkey.
2. Put them in `backend/.env` (see `.env.example`). Generate `MPESA_CALLBACK_SECRET` with the command shown there.
3. Safaricom must be able to reach your backend over **public HTTPS**. While developing locally, expose it with a tunnel such as `cloudflared tunnel --url http://localhost:4000` or ngrok, and set `MPESA_CALLBACK_BASE_URL` to that address. Restart the backend afterwards.
4. For live payments set `MPESA_ENV=production` and use your own Paybill or Till credentials from Safaricom (go-live is approved by Safaricom, not by this app).

How payments are protected:
- The browser never decides a payment succeeded. Only the server activates a subscription, through a database function that works exactly once per payment.
- The callback URL contains a secret. Even a valid-looking callback is re-checked directly with Safaricom (STK query) before access is granted, and the paid amount must match.
- If a callback never arrives, the status check recovers the result by asking Safaricom directly.
- An M-Pesa receipt number can only ever be used once.

## Making yourself admin

Register your account first, then in Supabase -> SQL Editor run:
```
update public.profiles set role = 'admin' where email = 'you@example.com';
```
Admins see an extra panel on the Billing page to set the KES price and days per payment. Normal users cannot change their own role (blocked in the database).

## How the AI stays honest

- Names, employers, dates, institutions, links and certifications are copied from your answers by the server, never taken from the AI.
- The AI only writes the summary, bullet wording and polish. Skills it returns that you did not list are discarded.
- If the AI returns something malformed, your own wording is kept and you are told.
- Prompts ask for plain, specific language and ban common AI filler words. Read the result before you send it anywhere.

## Notes

- Templates are single-column on purpose so applicant tracking systems parse them correctly.
- PDFs use standard fonts (Helvetica, Times, Courier) and only Latin characters render. Names with other scripts may lose characters in PDF; DOCX is unaffected.
- AI and export endpoints require an active trial or paid subscription (`402` otherwise).
- Tools read a CV by id on the server and always filter by the signed-in user, so other users' documents are unreachable.
- Scanned or image-only PDFs have no readable text. OCR is not included; paste the text instead.
- Deleting a whole account removes the database rows but not the Storage files. Use `backend/scripts/sweep-orphans.js` to clean up.
- PDF tools: Light compression rarely shrinks files much. Strong compression converts pages to images, so text is no longer selectable and ATS systems cannot read the result. Do not use Strong on a CV you plan to submit.
- The CV Enhancer returns plain text. Turning it back into a designed PDF/DOCX is a manual step for now.

## Roadmap

Done in this repo for Phase 5: Supabase security advisor (no issues), deployment files, orphan cleanup. Still needs you: a real browser run of every checklist and a live deploy.
