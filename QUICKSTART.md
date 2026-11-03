# CVGenius AI: first run (about 10 minutes)

You need Node 18 or newer. M-Pesa is not needed yet.

1. **Backend keys.** In `backend/`, copy `.env.example` to `.env` and fill in:
   - `SUPABASE_URL` (already in `frontend/.env` as the project URL)
   - `SUPABASE_SERVICE_ROLE_KEY` (Supabase -> Project Settings -> API Keys, the secret one)
   - `AI_API_KEY` (your OpenAI key)
   Leave the `MPESA_*` lines as they are.
2. **Supabase dashboard.** Authentication -> Sign In / Providers -> Email: turn off "Confirm email". Authentication -> URL Configuration: Site URL `http://localhost:5173`, and add `http://localhost:5173/reset-password` to Redirect URLs.
3. **Start the backend.**
   `cd backend && npm install && npm run dev`
   Open http://localhost:4000/api/health. You want `supabaseConfigured: true` and `aiConfigured: true`. `mpesaConfigured: false` is expected.
4. **Start the frontend** (new terminal).
   `cd frontend && npm install && npm run dev`
   Open http://localhost:5173.
5. **Try it.** Register, generate a CV, edit it, export a PDF and a DOCX, upload a PDF, run an AI tool, and try the PDF tools.
6. **Run the tests.** `cd backend && npm test` (57 should pass).

If something fails, copy the exact error from the terminal or the browser console and send it to me.

When you have a Paybill or Till, fill in the `MPESA_*` lines, set `MPESA_ENV=sandbox` first, and test before going live.
