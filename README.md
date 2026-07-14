# AI-Career-Path-Recommender
# 🧭 Intern Career Path Recommender

An AI-powered tool that suggests career paths for interns based on their
skills and interests, recommends personalized learning paths and job
roles, and tracks their progress over time. Built with **Python**,
**TensorFlow**, and the **OpenAI API**, with a **Streamlit** dark-themed
UI.

---

## ✨ Features

- **Profile input** — pick skills and interests from a taxonomy (or add
  your own).
- **AI-personalized recommendations** — a TensorFlow neural network
  scores every known job role against the intern's profile; the OpenAI
  API turns the top matches into a readable, personalized narrative
  explaining *why* each role fits and how to prioritize the learning
  path.
- **Progress tracking** — check off learning-path steps per role and
  see completion percentage plus a score-history chart over time,
  stored locally in SQLite.
- **Dark theme UI** out of the box.
- **Works without an API key** — if `OPENAI_API_KEY` isn't set, the app
  falls back to clear, rule-based explanations so it's still fully
  usable offline/for demos.

---

## 🏗️ Architecture

```
career_recommender/
├── app.py              # Streamlit UI (dark theme) — entry point
├── ml_model.py          # TensorFlow role-matching model (RoleMatcher)
├── ai_advisor.py         # OpenAI API integration (narrative generation)
├── database.py           # SQLite persistence (profiles, scores, progress)
├── data/
│   └── careers.json       # Skill/interest taxonomy + job role definitions
├── models/                # Auto-created: cached trained TensorFlow model
├── db/                     # Auto-created: SQLite database file
├── .streamlit/
│   └── config.toml          # Dark theme configuration
└── requirements.txt
```

**How a request flows:**

1. The intern selects skills/interests in the sidebar.
2. `ml_model.RoleMatcher` turns that into a multi-hot vector and scores
   it against every role using a small dense neural network trained on
   synthetic examples derived from `data/careers.json` (see note
   below), blended with a simple skill-overlap score for
   interpretability.
3. `ai_advisor.generate_career_narrative` sends the top matches plus
   the intern's profile to the OpenAI API (`gpt-4o-mini` by default),
   asking for a short personalized explanation and a re-prioritized
   learning path per role.
4. Results are shown in the UI and saved via `database.py` so progress
   and score history can be tracked over time.

> **Note on the ML model:** There's no large dataset of real intern →
> career outcomes to train on, so `ml_model.py` synthesizes training
> data from each role's defined required skills/interests (with noise
> for generalization). This makes the matcher genuinely learn
> similarity patterns rather than hard-coding rules, while being
> transparent that it isn't trained on real historical placement data.
> Swap `_synthesize_training_data()` for real labeled data
> (intern profile → role they succeeded in) when you have it, and the
> rest of the pipeline (scoring, caching, inference) stays the same.

---

## 🚀 Setup

### 1. Install dependencies

```bash
cd career_recommender
python3 -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. (Optional but recommended) Set your OpenAI API key

```bash
export OPENAI_API_KEY="sk-..."        # macOS/Linux
setx OPENAI_API_KEY "sk-..."           # Windows
```

Without this, the app still runs and shows rule-based explanations
instead of AI-generated narratives.

### 3. Run the app

```bash
streamlit run app.py
```

Open the URL Streamlit prints (usually `http://localhost:8501`).

---

## 🖱️ Using the tool

1. **Get Recommendations tab** — enter your name, pick your skills and
   interests in the sidebar, choose how many roles to see, and click
   **Get My Career Recommendations**.
2. Review the matched roles, match scores, personalized "why this
   fits" explanation, and the suggested learning path for each.
3. **My Progress tab** — select your profile, pick a role's learning
   path, and check off steps as you complete them. Your completion
   percentage and score history over time are shown automatically.

Recommendations are re-saved every time you submit the form, so score
history builds up as your skills/interests evolve — re-run the
recommendation step periodically (e.g., monthly) to see your matches
shift as you learn.

---

## 🧩 Extending the system

- **Add roles/skills**: edit `data/careers.json` — no code changes
  needed; the ML model retrains automatically the first time it's
  needed after the schema changes (delete `models/role_matcher.keras`
  to force a retrain).
- **Swap the LLM**: change `MODEL_NAME` in `ai_advisor.py`.
- **Swap the database**: `database.py` isolates all persistence behind
  simple functions (`upsert_intern`, `save_recommendations`,
  `set_step_progress`, etc.) — replace the SQLite calls with a call to
  Postgres/MySQL/etc. without touching `app.py`.
- **Real training data**: once you have historical intern → role
  outcome data, replace `_synthesize_training_data()` in `ml_model.py`
  with a loader for that dataset for a more accurate matcher.

---

## ⚠️ Notes & limitations

- This is a starter/demo-grade system: the ML model is trained on
  synthetic data derived from role definitions, not real outcomes —
  treat scores as directional, not authoritative.
- The OpenAI API call incurs usage costs on your account; monitor
  usage if deploying to many users.
- No authentication is included — profiles are keyed only by name in
  SQLite. Add proper auth before deploying beyond a local/internal
  demo.
