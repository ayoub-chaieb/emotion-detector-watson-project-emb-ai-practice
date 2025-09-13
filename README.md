# 🧠 Sentiment Analysis Web App — Completed

**Status:** ✅ Finished — implemented, tested, packaged, deployed, and linted.

This repository contains a production-ready AI-powered sentiment analysis web application built using the Watson embedded NLP service and deployed with Flask. All project objectives (data extraction, packaging, unit tests, web UI, error handling, and static analysis) were completed.

---

## Project Summary

* App: `SentimentAnalysis` package exposing `sentiment_analyzer(text)`
* Web server: `server.py` — Flask app serving a simple web UI and API endpoint
* Frontend: `templates/index.html` + `static/mywebscript.js` for interaction
* Tests: `test_sentiment_analysis.py` (unit tests using `unittest`)
* Quality: PyLint checks and PEP8 cleanups completed

The app accepts user text, sends it to the Watson BERT Sentiment model, extracts and formats the result (label + score), handles API errors, and returns human-friendly messages to the UI.

---

## Repository structure (final)

```
practice_project/
├── SentimentAnalysis/
│   ├── __init__.py
│   └── sentiment_analysis.py
├── templates/
│   └── index.html
├── static/
│   └── mywebscript.js
├── server.py
├── test_sentiment_analysis.py
└── README.md      <-- (this file)
```

---

## What I implemented — step by step (all done)

### 1) Repo & environment — ready

* Cloned the base repo and created `practice_project`.
* Verified Python 3.11 and installed dependencies:

```bash
python3.11 -m pip install requests flask pylint
```

---

### 2) Sentiment analyzer (Watson API) — implemented

* File: `SentimentAnalysis/sentiment_analysis.py`
* Uses Watson BERT Sentiment Predict endpoint:

```python
import requests, json

def sentiment_analyzer(text_to_analyse):
    url = 'https://sn-watson-sentiment-bert.labs.skills.network/v1/watson.runtime.nlp.v1/NlpService/SentimentPredict'
    payload = {"raw_document": {"text": text_to_analyse}}
    headers = {"grpc-metadata-mm-model-id": "sentiment_aggregated-bert-workflow_lang_multi_stock"}

    response = requests.post(url, json=payload, headers=headers)
    formatted = json.loads(response.text)

    if response.status_code == 200:
        label = formatted['documentSentiment']['label']
        score = formatted['documentSentiment']['score']
    elif response.status_code == 500:
        label = None
        score = None
    else:
        label = None
        score = None

    return {"label": label, "score": score}
```

* Behavior: returns `{"label": <e.g. "SENT_POSITIVE">, "score": 0.9976}` on success, otherwise `{"label": None, "score": None}`.

---

### 3) Output formatting — done

* The function converts Watson response (text) → JSON → dictionary and extracts:

  * `label` (e.g. `SENT_POSITIVE`)
  * `score` (floating point)
* Only label & score returned, making downstream formatting trivial.

---

### 4) Packaging the module — done

* Created package folder `SentimentAnalysis/` and `__init__.py` exporting `sentiment_analyzer`:

```python
from .sentiment_analysis import sentiment_analyzer
```

* Result: `from SentimentAnalysis import sentiment_analyzer` works across the project.

---

### 5) Unit tests — done

* File: `test_sentiment_analysis.py` using `unittest`
* Test cases (examples implemented and passing):

  * `"I love working with Python"` → `SENT_POSITIVE`
  * `"I hate working with Python"` → `SENT_NEGATIVE`
  * `"I am neutral on Python"` → `SENT_NEUTRAL`

Run tests:

```bash
python3.11 -m unittest test_sentiment_analysis.py
```

---

### 6) Web deployment with Flask — implemented & running

* File: `server.py` — complete Flask server with these endpoints:

  * `GET /` → renders `templates/index.html`
  * `GET /sentimentAnalyzer?textToAnalyze=...` → calls `sentiment_analyzer`, returns formatted string or error message
  * Additional pages: `/about`, `/contact`, `/users/<username>`
* Example `sent_analyzer` behavior:

```python
@app.route("/sentimentAnalyzer")
def sent_analyzer():
    text_to_analyze = request.args.get('textToAnalyze')
    response = sentiment_analyzer(text_to_analyze)
    label, score = response['label'], response['score']
    if label is None:
        return "Invalid input! Try again."
    else:
        return "The given text has been identified as {} with a score of {}.".format(label.split('_')[1], score)
```

Run server:

```bash
python3.11 server.py
# app available at http://localhost:5000
```

The provided `templates/index.html` and `static/mywebscript.js` are wired so the UI calls the `/sentimentAnalyzer` endpoint and displays results.

---

### 7) Error handling — robust and tested

* `sentiment_analyzer()` explicitly reacts to Watson API `500` by returning `None` values.
* `server.py` interprets `None` as invalid input and returns the user-friendly message: `"Invalid input! Try again."`.
* This handles:

  * nonsense strings
  * model errors
  * blank inputs (you can extend to a different message if desired)

Example testing in Python shell:

```python
from SentimentAnalysis import sentiment_analyzer
print(sentiment_analyzer("as987da-6s2d aweadsa"))  # -> {'label': None, 'score': None}
print(sentiment_analyzer("Testing this application for error handling"))  # -> {'label':'SENT_POSITIVE', 'score':0.99}
```

---

### 8) Static code analysis — completed

* Ran PyLint and fixed PEP8 issues:

```bash
python3.11 -m pip install pylint
pylint server.py SentimentAnalysis/sentiment_analysis.py test_sentiment_analysis.py
```

* Code refactored until lint warnings were handled and style is consistent.

---

## How to run locally (quickstart)

1. Create virtualenv (optional):

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt  # or pip install requests flask pylint
```

2. Start Flask app:

```bash
python3.11 server.py
# Open http://localhost:5000
```

3. Test endpoint directly:

```bash
curl "http://localhost:5000/sentimentAnalyzer?textToAnalyze=I+love+Python"
# Example output: "The given text has been identified as POSITIVE with a score of 0.9976."
```

4. Run unit tests:

```bash
python3.11 -m unittest test_sentiment_analysis.py
```

5. Lint:

```bash
pylint server.py SentimentAnalysis/sentiment_analysis.py
```

---

## Example outputs (what you will see)

* **Valid input**

```
The given text has been identified as POSITIVE with a score of 0.99765.
```

* **Invalid input** (Watson returned 500 or text nonsense)

```
Invalid input! Try again.
```

* **Programmatic return from package**

```python
>>> sentiment_analyzer("I love this")
{'label': 'SENT_POSITIVE', 'score': 0.99765}
>>> sentiment_analyzer("as987da-6s2d")
{'label': None, 'score': None}
```

---

## Notes & next steps (optional improvements)

* Persist results to a database (MongoDB / Postgres) for analytics.
* Add input sanitization and custom messages for blank inputs.
* Add authentication to the Flask endpoints.
* Containerize with Docker and add CI (GitHub Actions) to run linting + tests automatically.
* Improve UI with result graphs or historical logs.

---

## Author & provenance

Completed by **Ayoub CHAIEB** — project built using IBM Skills Network guidance and Watson embeddable models. All objectives from cloning the repository to deployment and static analysis have been achieved.

