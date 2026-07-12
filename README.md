# StudyAI — Smart AI Study Companion & Quiz Generator

An AI-powered study platform: upload notes/PDF/DOCX → get AI summaries,
flashcards, adaptive quizzes, weak-topic analysis, and a personalized
7-day study schedule.

**Stack:** React.js · Python Flask · Groq (Llama 3.3 70B Versatile) · Firebase Firestore (optional, with local JSON fallback)

---

## 1. Project structure

```
studyai/
├── backend/
│   ├── app.py                  # Flask API (all routes)
│   ├── services/
│   │   ├── ai_service.py       # Groq prompts: summary, flashcards, quiz, schedule
│   │   ├── storage_service.py  # Firestore OR local JSON storage
│   │   └── file_processor.py   # PDF / DOCX / TXT text extraction
│   ├── data/                   # local JSON "database" (auto-created)
│   ├── requirements.txt
│   └── .env.example
└── frontend/
    ├── src/
    │   ├── components/         # Dashboard, Upload, Summary, Flashcards, Quiz, Schedule, Navbar
    │   ├── api/api.js          # Axios calls to the Flask backend
    │   └── App.js
    └── package.json
```

---

## 2. Get a free Groq API key (required)

1. Go to **https://console.groq.com**
2. Sign up / log in (free).
3. Open **API Keys** in the left sidebar → **Create API Key**.
4. Copy the key (starts with `gsk_...`). You won't see it again.

## 3. (Optional) Firebase Firestore

The app works completely fine **without** this — it automatically falls
back to local JSON files in `backend/data/`. Only set this up if you
specifically need cloud storage for your submission.

1. Go to https://console.firebase.google.com → Create a project.
2. Build → Firestore Database → Create database (test mode is fine for a college project).
3. Project Settings (gear icon) → Service Accounts → **Generate new private key**.
4. Save the downloaded JSON file somewhere on your machine (do **not** commit it).
5. Point `FIREBASE_CREDENTIALS_PATH` at it in your `.env` (step 4 below).

---

## 4. Backend setup (Flask)

```bash
cd backend
python -m venv venv

# activate it:
venv\Scripts\activate        # Windows
source venv/bin/activate     # macOS/Linux

pip install -r requirements.txt

# create your real env file
cp .env.example .env         # macOS/Linux
copy .env.example .env       # Windows
```

Open `backend/.env` and paste in your real key:

```
GROQ_API_KEY=gsk_your_real_key_here
GROQ_MODEL=llama-3.3-70b-versatile
FIREBASE_CREDENTIALS_PATH=
FLASK_PORT=5000
FLASK_DEBUG=True
```

Run it:

```bash
python app.py
```

You should see `Running on http://0.0.0.0:5000` and a storage log line
telling you whether it's using Firestore or local JSON.

Test it's alive: open **http://localhost:5000/api/health** → `{"status": "ok"}`

---

## 5. Frontend setup (React)

Open a **second terminal** (leave Flask running in the first):

```bash
cd frontend
npm install
npm start
```

This opens **http://localhost:3000** automatically. It talks to the
Flask backend at `http://localhost:5000/api` by default (see
`frontend/src/api/api.js` — override with a `REACT_APP_API_URL` env var
if you deploy the backend elsewhere).

---

## 6. Using the app

1. **Dashboard** — see all uploaded materials + analytics (score trend, weak topics).
2. **Upload Material** — drag in a PDF/DOCX/TXT, or paste raw text.
3. Inside a material, use the tabs:
   - **Summary** — structured Markdown summary (overview, key concepts, definitions, principles, revision notes).
   - **Flashcards** — flip cards, mark known, shuffle, track mastery.
   - **Quiz** — choose MCQ / True-False / Short Answer, take it, get scored + weak topics.
   - **Schedule** — generates a 7-day plan that prioritizes topics you got wrong.

---

## 7. Common issues

| Problem | Fix |
|---|---|
| Frontend shows "Could not reach the StudyAI backend" | Make sure `python app.py` is running on port 5000 |
| `GROQ_API_KEY is not set` error | Edit `backend/.env`, restart `python app.py` |
| PDF upload fails | Only text-based PDFs work well; scanned/image-only PDFs have no extractable text |
| CORS errors in browser console | Confirm `flask-cors` is installed and `CORS(app)` is in `app.py` (already included) |

---

## 8. Push to GitHub (for submission)

```bash
cd studyai
git init
git add .
git commit -m "StudyAI: AI-powered study companion and quiz generator"
git branch -M main
git remote add origin https://github.com/<your-username>/studyai.git
git push -u origin main
```

`.gitignore` is already set up so your `.env`, `node_modules/`, and any
Firebase credential files never get committed.

---

## 9. Tech notes (for your report/documentation)

- **AI model:** Groq API, `llama-3.3-70b-versatile` — chosen for strong reasoning, high-quality educational content, and fast inference speed.
- **Prompt engineering:** each feature (summary, flashcards, quiz, schedule) uses a dedicated system + user prompt in `backend/services/ai_service.py`, with `response_format: json_object` for structured features so the frontend gets reliable, parseable data.
- **Weak topic identification:** computed from quiz results grouped by topic (`identify_weak_topics`), then fed back into the schedule generator so later study plans prioritize what the student actually got wrong.
- **Storage:** `storage_service.py` abstracts Firestore vs. local JSON behind the same functions (`save_doc`, `get_doc`, `list_docs`, `delete_doc`), so the rest of the app never needs to know which backend is active.
