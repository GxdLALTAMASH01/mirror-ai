# Mirror AI

Capture a single frame from your webcam and get a quick, specific read on
your mood and style — plus one concrete observation. Full-stack app: React
(Vite) frontend styled as a camera studio viewfinder, FastAPI backend, and
**OpenRouter** as the only model provider — no other AI API is called.

## How it works

1. The frontend opens your camera inside a viewfinder frame with corner
   brackets, like a camera UI.
2. Click **Capture reading** — a still frame is taken, brackets pulse and a
   scan line sweeps while it's analyzed.
3. The backend sends the frame to a vision model through OpenRouter
   (`https://openrouter.ai/api/v1/chat/completions`) and gets back a short
   structured reading.
4. The result appears as a spec-sheet card: mood, style, and one detail.

## Setup

### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env          # then add your OPENROUTER_API_KEY
uvicorn main:app --reload --port 8000
```

Get a free API key at https://openrouter.ai/keys (no credit card needed —
the default model, `openrouter/free`, auto-picks a free vision-capable
model for each request, so you can run this at zero cost). Want a specific
model instead? Set `OPENROUTER_MODEL` in `.env`, e.g.
`google/gemma-3-27b-it:free`, or a paid one like `anthropic/claude-3.5-sonnet`.

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open http://localhost:5173 and allow camera access.

## Deploying (getting a real link)

**Backend → Render.com (free tier)**
1. Push this repo to GitHub (if you haven't already).
2. On [render.com](https://render.com), New → Web Service → connect this repo.
3. Root directory: `backend`
4. Build command: `pip install -r requirements.txt`
5. Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
6. Add environment variables:
   - `OPENROUTER_API_KEY` — your key
   - `FRONTEND_URL` — your Vercel URL once you have it (step below), e.g. `https://mirror-ai.vercel.app`
7. Deploy. You'll get a URL like `https://mirror-ai-backend.onrender.com`.

**Frontend → Vercel (free tier)**
1. On [vercel.com](https://vercel.com), Add New → Project → import this repo.
2. Root directory: `frontend`
3. Add environment variable: `VITE_API_URL` = your Render backend URL from above (no trailing slash).
4. Deploy. You'll get a URL like `https://mirror-ai.vercel.app` — that's your shareable link.
5. Go back to Render and set `FRONTEND_URL` to this Vercel URL, then redeploy the backend so CORS allows it.

## Notes

- The backend calls **only** OpenRouter — there's no Anthropic, OpenAI, or
  any other SDK in `requirements.txt`. Every model request is routed
  through `https://openrouter.ai/api/v1/chat/completions` using the plain
  `httpx` HTTP client.
- CORS is set to only allow `http://localhost:5173` plus whatever you set
  in `FRONTEND_URL` (see `main.py`).
- The system prompt explicitly avoids commenting on protected
  characteristics (race, disability, body shape, etc.) and sticks to what's
  visibly chosen — expression, posture, clothing, styling. Worth reviewing
  `SYSTEM_PROMPT` in `backend/main.py` before you extend this.
- No images are stored — each frame is sent to OpenRouter and discarded.
- Free OpenRouter models are rate-limited (roughly 20 requests/minute, 200/day)
  and can change availability over time — check https://openrouter.ai/models
  if `openrouter/free` ever stops finding a vision model.
