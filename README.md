# 📚 Literature Review Tracker

A cloud-synced, browser-based research paper management tool built for academics conducting systematic and structured literature reviews. I built this to solve a real problem I faced — tracking dozens of papers across multiple devices while maintaining a clear picture of methodologies, research gaps, and relevance to my work.

**Live App:** [storied-khapse-9a663c.netlify.app](https://storied-khapse-9a663c.netlify.app)

![Built with React](https://img.shields.io/badge/Built%20with-React%2018-61dafb?style=flat-square&logo=react)
---

## 💡 Motivation

Conducting a rigorous literature review is one of the most demanding parts of academic research. According to best practices recommended by university research libraries, a well-structured literature review requires tracking not just paper details, but also **search strategies, eligibility criteria, methodological patterns, and research gaps** across your entire body of sources.

I wanted something lightweight, visual, and purpose-built for tracking the *content* of papers — not just their citations. This app fills that gap.

---

## ✨ Features

### 📄 Papers Table
- Log papers across **12 structured fields**: No., Author & Year, Title, Journal, Keywords, Research Question, Methodology, Sample Size, Findings, Research Gaps, Relevance, and Notes
- **Add, Edit, Delete** papers via a clean modal form
- Methodology dropdown aligned with standard academic approaches: Quantitative, Qualitative, Mixed Methods, Systematic Review, Meta-Analysis, Case Study, Experimental, Survey
- **Relevance tagging** (High / Medium / Low) with color-coded badges for quick scanning

### 📊 Dashboard Analytics
- **Methodology distribution** — horizontal bar chart showing the breakdown of research approaches across your library
- **Relevance donut chart** — visual split of High / Medium / Low relevance papers with percentage
- **High Relevance paper cards** — at-a-glance view of your most critical sources with findings preview
- **Research Gaps summary** — all identified gaps aggregated from every paper in one place, supporting gap analysis aligned with PRISMA-style review practices

### 🔢 Stat Cards
Always-visible summary: Total Papers, Average Sample Size, Top Methodology, High Relevance count

### 🔍 Search & Filter
- Live full-text search across title, author, journal, keywords, and findings
- Dropdown filters for methodology and relevance
- One-click clear all filters

### 📥 Export to Excel
- Downloads a `.xlsx` file with two sheets:
  - **Literature Review** — complete paper data with all fields
  - **Dashboard Summary** — aggregate stats, methodology and relevance breakdown

### 🔐 Authentication & Cloud Sync
- Secure **email/password** authentication via Firebase Auth
- Each user's papers are stored privately — complete data isolation
- **Real-time sync** via Firestore — papers save instantly and are available on any device, any browser

---

## 🏗️ System Design

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        BROWSER (Client)                      │
│                                                              │
│  ┌─────────────────────┐     ┌──────────────────────────┐   │
│  │     React 18 UI     │────►│   Firebase Auth SDK v9   │   │
│  │   (index.html)      │     │  (Email/Password Auth)   │   │
│  └──────────┬──────────┘     └──────────────────────────┘   │
│             │                                                │
│  ┌──────────▼──────────┐     ┌──────────────────────────┐   │
│  │  In-memory State    │◄───►│  Firestore SDK v9        │   │
│  │  (React useState)   │     │  (Real-time onSnapshot)  │   │
│  └─────────────────────┘     └──────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
          │                                │
          ▼                                ▼
  ┌──────────────┐              ┌────────────────────┐
  │   Netlify    │              │   Firebase Cloud   │
  │  (Hosting)   │              │  Auth + Firestore  │
  └──────────────┘              └────────────────────┘
```

### Data Flow — Adding or Editing a Paper

```
User submits form
      │
      ▼
React validates input
      │
      ▼
Firestore write → users/{uid}/papers/{paperId}
      │
      ▼
onSnapshot listener fires automatically
      │
      ▼
React state updates → UI re-renders instantly
      │
      ▼
"Synced to cloud ✓" toast appears
```

### Authentication Flow

```
User enters email + password
      │
      ├─ Sign Up ──► createUserWithEmailAndPassword()
      │                    └──► Firestore namespace created for user
      │
      └─ Sign In ──► signInWithEmailAndPassword()
                          └──► onAuthStateChanged() fires
                                    └──► Firestore listener attaches
                                              └──► Papers load in real time
```

---

## 🗄️ Database Design

Data is stored in **Firebase Firestore** — a scalable, serverless NoSQL document database. The structure follows a per-user namespace pattern, ensuring complete data isolation between users.

```
Firestore Database
│
└── users/                            ← top-level collection
    └── {userId}/                     ← one document per registered user
        └── papers/                   ← sub-collection of research papers
            └── {paperId}/            ← auto-generated document ID per paper
                ├── author            (string)  — e.g. "Smith et al. (2023)"
                ├── title             (string)  — full paper title
                ├── journal           (string)  — publication name
                ├── keywords          (string)  — comma-separated
                ├── researchQuestion  (string)  — core research question
                ├── methodology       (string)  — one of 8 standard types
                ├── sampleSize        (number)  — 0 if not applicable
                ├── findings          (string)  — key results
                ├── gaps              (string)  — identified limitations
                ├── relevance         (string)  — High | Medium | Low
                ├── notes             (string)  — personal annotations
                ├── createdAt         (timestamp) — auto server timestamp
                └── updatedAt         (timestamp) — auto server timestamp
```

### Why Firestore?

- **Real-time** — `onSnapshot` listeners push updates to all open tabs/devices instantly
- **Offline-capable** — Firestore SDK caches data locally, so the app still works briefly without internet
- **Scalable** — the free Spark plan includes 1 GB storage and 50,000 reads/day, far exceeding any literature review's needs
- **Serverless** — no backend infrastructure to manage or pay for

---

## 🔐 Security

### Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/papers/{paperId} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }
  }
}
```

This enforces:
- ✅ Only authenticated users can access data
- ✅ Each user can only read/write their own papers (matched by `uid`)
- ✅ No cross-user data access — not even readable, not even in error logs
- ✅ All data in transit is encrypted via HTTPS

---

## 🛠️ Tech Stack

| Layer | Technology | Reason for Choice |
|---|---|---|
| UI Framework | **React 18** (CDN, no bundler) | Component model, hooks, virtual DOM — without build complexity |
| Transpiler | **Babel Standalone** (CDN) | Runs JSX in-browser, no Node.js needed |
| Authentication | **Firebase Auth v9** | Managed auth with email/password, session persistence |
| Database | **Firebase Firestore v9** | Real-time NoSQL, per-user isolation, free tier |
| Excel Export | **SheetJS / xlsx** (CDN) | Client-side .xlsx generation with multi-sheet support |
| Typography | **Google Fonts** (Playfair Display + Inter) | Academic serif for headings, clean sans-serif for data |
| Styling | **Plain CSS** with CSS variables | Zero dependencies, full design control, easy theming |
| Hosting | **Netlify** | Free static hosting, instant deploys, HTTPS out of the box |
| Version Control | **GitHub** | Source history, collaboration-ready |

**No bundler. No npm. No Node.js.** The entire application is a single `index.html` file that loads dependencies from CDN and runs directly in any modern browser.

---

## 📁 Project Structure

```
literature-review-tracker/
│
├── index.html                  ← The entire application (HTML + CSS + JS)
│   │
│   ├── <head>
│   │   ├── Google Fonts (Playfair Display, Inter)
│   │   ├── React 18 + ReactDOM (CDN)
│   │   ├── Babel Standalone (CDN)
│   │   ├── SheetJS xlsx (CDN)
│   │   └── Firebase App, Auth, Firestore (CDN)
│   │
│   ├── <style>
│   │   ├── CSS custom properties (design tokens)
│   │   ├── Layout (header, tabs, grid, table)
│   │   ├── Components (cards, badges, modal, toolbar)
│   │   ├── Dashboard (bars, donut, paper cards, gap list)
│   │   └── Responsive breakpoints
│   │
│   └── <script type="text/babel">
│       ├── Firebase initialization
│       ├── LoginScreen      — email/password sign in + sign up form
│       ├── PaperModal       — add/edit modal with all 12 fields
│       ├── PapersTable      — data table with search, filter, actions
│       ├── Dashboard        — analytics charts and summaries
│       ├── exportExcel()    — two-sheet .xlsx builder
│       └── App              — root: auth state, Firestore listener, routing
│
└── README.md                   ← This file
```

### Key Components

| Component | Lines of Responsibility |
|---|---|
| `App` | Auth state management, Firestore real-time listener, tab routing, CRUD handlers |
| `LoginScreen` | Email/password sign-in and sign-up, form validation, Firebase error mapping |
| `PapersTable` | Renders all paper rows, search highlighting, edit/delete actions |
| `PaperModal` | Controlled form for adding and editing papers with all 12 fields |
| `Dashboard` | Methodology bar chart, relevance donut SVG, high-relevance cards, gap list |
| `exportExcel()` | Builds multi-sheet .xlsx using SheetJS with column widths and summary stats |

---

## 🚀 Running Locally

```bash
# Clone the repo
git clone https://github.com/sweta816/Literature-review-tracker.git
cd Literature-review-tracker

# Option 1: Open directly in browser
# Just double-click index.html — but note Firebase Auth won't work on file://

# Option 2: Serve locally (recommended)
npx serve .
# Open http://localhost:3000
```

### Deploy Your Own Instance

1. Create a Firebase project at [firebase.google.com](https://firebase.google.com)
2. Enable **Firestore Database** (production mode) and **Email/Password Authentication**
3. Replace the `firebaseConfig` object in `index.html` with your project's config
4. Deploy to Netlify by dragging the project folder to [netlify.com](https://netlify.com)
5. Add your Netlify domain to Firebase → Authentication → Settings → Authorized domains
6. Set Firestore security rules as shown above

---

## 📊 How the Dashboard Works

The dashboard computes all analytics client-side from the papers currently in your Firestore collection — no pre-aggregation needed.

- **Methodology bars** — counts papers by methodology, sorts descending, calculates percentage of total
- **Relevance donut** — SVG circle with `strokeDasharray` segments computed from counts, no chart library needed
- **Research Gaps** — filters papers with non-empty `gaps` field and renders them as a numbered list, making cross-paper gap synthesis easy
- **High Relevance cards** — filters `relevance === "High"` papers, shown with title, author, journal, methodology badge, and truncated findings

---

## 🔮 Planned Features

- [ ] BibTeX / RIS import (from Zotero, Mendeley, Google Scholar)
- [ ] Citation generator (APA 7th, MLA, Chicago)
- [ ] PRISMA flow diagram auto-generator
- [ ] Paper tagging and grouping by theme
- [ ] Collaborative shared libraries (invite co-researchers)
- [ ] AI-powered research gap detection and synthesis suggestions
- [ ] Annotation and highlights per paper
- [ ] Dark mode

---

*Built to bring structure, clarity, and analytical insight to the literature review process.*
