# 📚 Literature Review Tracker

A clean, browser-based web app for organizing, analyzing, and exporting research papers during a literature review. Built with React (via CDN) — no installation or build step required. Just open `index.html` in any modern browser.

![Literature Review Tracker](https://img.shields.io/badge/Built%20with-React%2018-61dafb?style=flat-square&logo=react) ![License](https://img.shields.io/badge/License-MIT-green?style=flat-square) ![No Build Required](https://img.shields.io/badge/No%20build-required-purple?style=flat-square)

---

## ✨ Features

**Papers Table**
- Track papers across 12 fields: Author & Year, Title, Journal, Keywords, Research Question, Methodology, Sample Size, Findings, Research Gaps, Relevance, and Notes
- Add, Edit, and Delete papers via a modal form
- Methodology dropdown: Quantitative, Qualitative, Mixed Methods, Systematic Review, Meta-Analysis, Case Study, Experimental, Survey
- Relevance tagging: High, Medium, Low

**Search & Filter**
- Live full-text search across title, author, journal, keywords, and findings
- Filter by methodology and relevance level independently
- One-click clear filters

**Dashboard**
- Methodology distribution bar chart
- Relevance breakdown donut chart with percentage breakdown
- High Relevance paper cards with findings preview
- Research Gaps summary collected from all papers

**Stat Cards** (always visible)
- Total Papers
- Average Sample Size
- Top Methodology
- High Relevance count

**Export to Excel**
- Downloads a `.xlsx` file with two sheets:
  - **Literature Review** — full paper data table with all fields
  - **Dashboard Summary** — aggregate stats, relevance breakdown, methodology distribution

---

## 🗄️ How Data Is Stored

Data is stored in **Firebase Firestore** — a free, real-time cloud database by Google. Every add, edit, or delete is saved instantly to the cloud.

**Structure in Firestore:**
```
users/
  {userId}/
    papers/
      {paperId}/   ← one document per paper
        author, title, journal, keywords,
        researchQuestion, methodology, sampleSize,
        findings, gaps, relevance, notes,
        createdAt, updatedAt
```

Each user's papers are stored under their own unique user ID, so data is completely private — no user can see another's papers.

**What this means in practice:**
- Papers are saved automatically — no manual save button needed
- Refreshing the page loads your papers back instantly
- Open the app on any device, any browser — your data is always there
- The "Synced to cloud ✓" toast confirms every save

The free Firebase Spark plan includes 1 GB storage and 50,000 reads/day — far more than any literature review will ever use.

---

## 🚀 Getting Started

1. Clone or download this repository
2. Open `index.html` in any modern browser (Chrome, Firefox, Edge, Safari)
3. No npm install, no build step, no server needed

```bash
git clone https://github.com/your-username/literature-review-tracker.git
cd literature-review-tracker
# Then just open index.html in your browser
```

---

## 🔐 Firestore Security Rules

Once you're ready to go beyond test mode, update your Firestore rules in the Firebase console to lock down access properly:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/papers/{paperId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

This ensures each user can only read and write their own papers.

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| React 18 (CDN) | UI and state management |
| Babel Standalone (CDN) | JSX transpilation in-browser |
| Firebase Auth | Google Sign-In |
| Firebase Firestore | Cloud database (real-time sync) |
| SheetJS / xlsx (CDN) | Excel export (.xlsx) |
| Google Fonts — Playfair Display + Inter | Typography |
| Plain CSS (CSS variables) | Styling and theming |

No bundler, no Node.js, no dependencies to install.

---

## 📁 Project Structure

```
literature-review-tracker/
└── index.html      # The entire app — HTML, CSS, and React all in one file
└── README.md       # This file
```

---

## 🎨 Design

- Color palette: **purple** (`#6d28d9`) and **green** (`#059669`) accents
- Headings: **Playfair Display** (serif)
- Body: **Inter** (sans-serif)
- Responsive layout — works on tablet and desktop screens

---

## 📄 License

MIT — free to use, modify, and share.
