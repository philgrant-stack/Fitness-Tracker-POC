# 🏋️ Fitness Tracker — Project Run Sheet

A step-by-step build guide for deploying a shared fitness tracker using GitHub, GCP Cloud Functions, and Firestore.

---

## How to use this file
- Work through each phase in order — later phases depend on earlier ones
- Check off tasks as you complete them using `[x]`
- Save this file as `RUNSHEET.md` in the root of your GitHub repo

---

## Phase 1 — Accounts & Project Setup

> Goal: Get your core accounts and infrastructure placeholders in place.

- [ ] Create a [GitHub](https://github.com) account (if you don't have one)
- [ ] Create a new GitHub repository (e.g. `fitness-tracker`)
  - Set visibility to **Private** for now
  - Initialise with a `README.md`
- [ ] Create a [Google Cloud](https://console.cloud.google.com) account (if you don't have one)
- [ ] Create a new GCP **Project** (e.g. `fitness-tracker`)
  - Note your **Project ID** — you'll need it repeatedly
- [ ] Enable **billing** on your GCP project
  - Free tier is generous; you won't be charged for small usage
- [ ] Install the [Google Cloud CLI](https://cloud.google.com/sdk/docs/install) (`gcloud`) on your machine
- [ ] Run `gcloud auth login` and `gcloud config set project YOUR_PROJECT_ID`

---

## Phase 2 — Firestore Database

> Goal: Create the database that will store all fitness entries.

- [ ] In GCP Console, navigate to **Firestore**
- [ ] Select **Native mode** (not Datastore mode)
- [ ] Choose a region close to you (e.g. `australia-southeast1`)
- [ ] Create the database — leave the default name `(default)`
- [ ] Set Firestore **security rules** to allow read/write (for now, during development):
  ```
  rules_version = '2';
  service cloud.firestore.rules {
    match /databases/{database}/documents {
      match /{document=**} {
        allow read, write: if true;
      }
    }
  }
  ```
  > ⚠️ Lock these down before sharing the app publicly

---

## Phase 3 — Cloud Function (Python Backend)

> Goal: Write and deploy a Python function that handles reading and writing fitness data.

- [ ] Enable the **Cloud Functions API** in GCP Console
- [ ] Enable the **Cloud Build API** in GCP Console
- [ ] Create a local folder for your function (e.g. `functions/fitness/`)
- [ ] Create `main.py` with two endpoints:
  - `POST /log` — writes a fitness entry to Firestore
  - `GET /entries` — reads all entries from Firestore
- [ ] Create `requirements.txt` with:
  ```
  functions-framework==3.*
  google-cloud-firestore
  flask
  ```
- [ ] Test the function locally using `functions-framework`
- [ ] Deploy to GCP:
  ```bash
  gcloud functions deploy fitness-api \
    --runtime python311 \
    --trigger-http \
    --allow-unauthenticated \
    --region australia-southeast1
  ```
- [ ] Note the deployed **function URL** — you'll use this in the frontend

---

## Phase 4 — Frontend (HTML App)

> Goal: Connect your existing HTML fitness tracker to the live Cloud Function.

- [ ] Open your existing HTML app
- [ ] Replace any local/mock data calls with `fetch()` calls to your Cloud Function URL:
  - Log entry → `POST` to `YOUR_FUNCTION_URL/log`
  - Load entries → `GET` from `YOUR_FUNCTION_URL/entries`
- [ ] Test locally in your browser — confirm data appears in Firestore console
- [ ] Add your HTML file to the GitHub repo
- [ ] Decide on hosting for the frontend:
  - [ ] **GitHub Pages** (simplest — free, static hosting)
  - [ ] **GCP Cloud Storage** (static site bucket)
  - [ ] Other (Netlify, Vercel, etc.)

---

## Phase 5 — GitHub Actions (Automated Deployment)

> Goal: Set up CI/CD so pushing to GitHub automatically deploys your function and frontend.

- [ ] Create `.github/workflows/deploy.yml` in your repo
- [ ] Add a GCP service account key as a GitHub **secret** (`GCP_SA_KEY`)
- [ ] Configure the workflow to:
  - Trigger on push to `main`
  - Authenticate with GCP using the secret
  - Deploy the Cloud Function via `gcloud functions deploy`
  - (Optional) Deploy the frontend to GitHub Pages or GCS
- [ ] Push a test commit and confirm the Actions workflow runs successfully
- [ ] Check GCP Console to verify the new function version is live

---

## Phase 6 — Security & Sharing

> Goal: Tighten things up before sharing the link with friends.

- [ ] Update Firestore security rules to restrict access appropriately
- [ ] Add basic input validation to your Cloud Function
- [ ] (Optional) Add a simple name/identifier field so entries are attributed to each user
- [ ] Share the frontend URL with your friends
- [ ] Monitor usage in **GCP Console → Firestore** and **Cloud Functions logs**

---

## Reference Notes

| Item | Value |
|---|---|
| GCP Project ID | *(fill in)* |
| GCP Region | *(fill in)* |
| Cloud Function URL | *(fill in after Phase 3)* |
| Frontend URL | *(fill in after Phase 4/5)* |
| GitHub Repo URL | *(fill in)* |

---

*Generated as part of the fitness tracker pattern build. Work through phases in order and check off tasks as you go.*
