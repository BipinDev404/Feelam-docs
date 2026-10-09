# Feelam — Project Documentation


<img width="825" height="404" alt="image" src="https://github.com/user-attachments/assets/a670b9e5-daa4-4ff6-908e-cad55964f7b8" />


> **Feelam** is an ultra-minimal, high-performance movie & web series search engine and streaming catalog. Inspired by Google's clean search interface and Apple's design aesthetics, Feelam focuses on instant discovery, AI-assisted mood curation, and seamless content management.

---

## Table of Contents

1. [System Overview & Architecture](#system-overview--architecture)
2. [Tech Stack](#tech-stack)
3. [Key Features](#key-features)
   - [Instant Search & Fuzzy Matching](#1-instant-search--fuzzy-matching)
   - [AI Vibe & Mood Search](#2-ai-vibe--mood-search)
   - [Catalog & Series Management](#3-catalog--series-management)
   - [User Request System](#4-user-request-system)
   - [Broken Link Reporting](#5-broken-link-reporting)
   - [Search Trends & Analytics](#6-search-trends--analytics)
   - [Admin Portal & Security](#7-admin-portal--security)
4. [Data Models & Schemas](#data-models--schemas)
5. [Backend API Reference](#backend-api-reference)
6. [Frontend Structure](#frontend-structure)
7. [Environment Variables & Configuration](#environment-variables--configuration)
8. [Installation & Deployment](#installation--deployment)

---
<img width="637" height="536" alt="image" src="https://github.com/user-attachments/assets/c20d2a49-b910-433a-a903-03587694c229" />
<img width="1119" height="933" alt="image" src="https://github.com/user-attachments/assets/c6b4e3b1-8abd-47a4-b059-3db64326bb4b" />


## System Overview & Architecture

Feelam operates as a modern **full-stack web application** running on Express and Vite:
┌────────────────────────────────────────────────────────┐
│ Feelam Frontend │
│ React 18 • TypeScript • Tailwind CSS • Lucide │
└───────────────────────────┬────────────────────────────┘
│
REST APIs / JSON Payloads
│
┌───────────────────────────▼────────────────────────────┐
│ Express Server │
│ Vite Dev Middleware / Production Static Host │
└──────┬────────────────────┬────────────────────┬───────┘
│ │ │
┌──────▼───────┐ ┌───────▼───────┐ ┌───────▼───────┐
│ Google Cloud │ │ Gemini 2.5 │ │ TMDB │
│ Firestore │ │ Flash AI SDK │ │ Public API │
│ (DB Engine) │ │ Mood Engine │ │ (Auto-enrich) │
└──────────────┘ └───────────────┘ └───────────────┘
code
Code
### Multi-Tier Data Persistence Strategy
Feelam ensures 100% offline and uninterrupted operation through a 3-tier cascade:
1. **Primary**: Google Cloud Firestore for real-time cloud persistence.
2. **Server Local Storage**: File-backed storage in `data/movies.json` and `data/requests.json`.
3. **Client Cache**: Synchronized browser `localStorage` ensuring instant search execution (< 5ms).

---

## Tech Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React 18, TypeScript, Tailwind CSS, Lucide Icons, Recharts |
| **Backend** | Node.js, Express, tsx |
| **Database** | Google Cloud Firestore (Firebase SDK), Local JSON Fallback |
| **AI Engine** | Google GenAI TypeScript SDK (`@google/genai`), Gemini 2.5 Flash |
| **External APIs** | The Movie Database (TMDB) API v3 |
| **Email Service** | Nodemailer (SMTP notifications for fulfilled user requests) |
| **Build & Tooling**| Vite, ESLint, TypeScript Compiler |

---

## Key Features

### 1. Instant Search & Fuzzy Matching
- Real-time search by title, cast members, directors, release year, genres, keywords, and audio languages.
- Microsecond client-side cache indexing with sub-string ranking and match scoring.
- Keyboard navigation (Arrow keys `Up`/`Down`, `Enter` to select, `Esc` to reset).

### 2. AI Vibe & Mood Search
- Natural language query processor powered by **Gemini 2.5 Flash**.
- Users can search with prompts such as *"late night rainy neon cyberpunk thriller"* or *"comfort anime for a cold Sunday"*.
- Returns curated matches from the Feelam catalog with personalized justification summaries.

### 3. Catalog & Series Management
- **Movies**: Single-title indexing with resolution tags (4K UHD, 1080p FHD, 720p HD, 480p) and multi-host mirror support (Google Drive, Direct, Mega).
- **Web Series / TV Shows**: Multi-season and episodic structure with season download groups, network badges (Netflix, HBO, AMC, etc.), and episode counts.
- **Responsive Layout**: Category discovery pills for **Recently Released**, **Popular Movies**, and **Web Series** positioned above a dedicated **Request Movie / Series** trigger.

### 4. User Request System
- Community members can request missing titles with optional email notification.
- Automated notification pipeline dispatches transactional emails via Nodemailer when an admin marks a request as completed.

### 5. Broken Link Reporting
- Users can flag non-working mirrors or dead streams directly from the movie detail page.
- Tracks exact URL, resolution, and specific issue (dead link, audio desync, corrupted file).
- Admin dashboard allows marking reports as *Investigating*, *Resolved*, or *Dismissed*.

### 6. Search Trends & Analytics
- Live search query telemetry tracking frequency, timestamp, and result count.
- Visualized with **Recharts** (Bar & Line charts) showing top search terms.
- Includes a **1-click "Add to Catalog"** button on missing search terms that auto-queries TMDB and opens the Add Movie modal.

### 7. Admin Portal & Security
- **Authentication**:
  - Encrypted Passkey login with brute-force lockout protection (5 failed attempts locks authentication for 15 minutes).
  - Google Admin OAuth verification for approved administrator emails.
- **Discovery & Importer**:
  - Live TMDB integration: auto-completes title, poster, backdrop, cast, synopsis, runtime, and trailer URLs in one click.
  - Bulk JSON Importer / Exporter with validation reports and schema reconciliation.
- **Inline Editing**:
  - Instant quick edit dialogs directly on search cards for administrators.

---

## Data Models & Schemas

### Movie
```typescript
interface Movie {
  id: string;                      // URL slug (e.g., "oppenheimer-2023")
  title: string;                   // Movie title
  year: number;                    // Release year
  director?: string;               // Director or creator
  rating: number;                  // 0.0 - 10.0 scale
  runtime?: number;                // Duration in minutes
  genres: string[];                // e.g. ["Sci-Fi", "Drama"]
  cast?: string[];                 // Main cast members
  description?: string;            // Overview synopsis
  poster?: string;                 // TMDB / external poster URL
  backdrop?: string;               // TMDB / external backdrop URL
  trailer?: string;                // YouTube trailer URL or ID
  language?: string;               // Primary spoken language
  country?: string;                // Production country
  mediaType?: 'movie' | 'series';  // Catalog categorization
  totalSeasons?: number;           // For TV series
  totalEpisodes?: number;          // For TV series
  networks?: string[];             // Broadcaster or streaming platforms
  downloadLinks?: DownloadLink[];  // Mirror hosts
  seasonDownloads?: SeasonDownloadGroup[];
}
MovieRequest
code
TypeScript
interface MovieRequest {
  id: string;
  title: string;
  requesterEmail?: string;
  notes?: string;
  status: 'pending' | 'completed' | 'rejected';
  createdAt: string;
  completedAt?: string;
  notifiedAt?: string;
}
BrokenLinkReport
code
TypeScript
interface BrokenLinkReport {
  id: string;
  movieId: string;
  movieTitle: string;
  linkUrl: string;
  linkQuality?: string;
  reportedIssue: string;
  status: 'pending' | 'investigating' | 'resolved' | 'dismissed';
  createdAt: string;
}
Backend API Reference
Method	Endpoint	Description
GET	/health	Container & service health check
GET	/favicon.ico	Cinema vector SVG browser favicon
GET	/api/movies	Fetch all indexed movies
GET	/api/movies/search?q=...	Server-side fuzzy search
GET	/api/movies/:id	Get movie details by slug ID
POST	/api/movies	Create or update a movie document
DELETE	/api/movies/:id	Delete a single movie
DELETE	/api/movies/batch	Bulk delete movie documents
DELETE	/api/movies/all	Wipe all movies (Admin only)
POST	/api/movies/import	Batch import JSON movie arrays
GET	/api/stats	Database counts, genre distributions, and backend status
POST	/api/ai/mood-search	Gemini 2.5 Flash natural language mood recommender
GET	/api/requests	Fetch user title requests
POST	/api/requests	Submit a new title request
PATCH	/api/requests/:id	Update request status (pending, completed, rejected)
POST	/api/requests/:id/notify	Send email confirmation via Nodemailer to requester
GET	/api/reports	Fetch broken link reports
POST	/api/reports	Report a dead streaming or download link
PATCH	/api/reports/:id	Update broken link report status
GET	/api/analytics/searches	Fetch query frequency and top searched terms
POST	/api/analytics/searches	Log a user search query event
GET	/api/tmdb/search?query=...	Query TMDB database for auto-fill data
Frontend Structure
code
Code
src/
├── components/
│   ├── AdminSettingsTab.tsx        # System settings & TMDB API key config
│   ├── AiMoodResults.tsx           # Semantic AI search presentation
│   ├── BrokenLinkModal.tsx         # User broken link reporting modal
│   ├── EmptyState.tsx              # Zero-results state with request CTA
│   ├── Footer.tsx                  # Minimalist legal & info footer
│   ├── Header.tsx                  # Top bar with wordmark & stealth portal
│   ├── InfoModal.tsx               # Privacy, terms, and developer credits
│   ├── LoadingState.tsx            # Clean animated skeleton loader
│   ├── MovieDetail.tsx             # Full movie page with mirrors & trailer
│   ├── QuickEditMovieModal.tsx     # Admin inline editing overlay
│   ├── RequestMovieModal.tsx       # User request submission modal
│   ├── SearchAnalyticsChart.tsx    # Recharts analytics graphs
│   ├── SearchBar.tsx               # Instant debounced input with AI toggle
│   ├── SearchResults.tsx           # Fast responsive grid of movie cards
│   └── SeriesSeasonDownloadManager.tsx # Episode & season download links editor
├── firebase/
│   ├── auth.ts                     # Admin credentials & passkey verification
│   ├── config.ts                   # Firestore initialization
│   ├── movies.ts                   # Database CRUD and local storage sync
│   ├── reports.ts                  # Broken link report handlers
│   └── requests.ts                 # User request handlers
├── hooks/
│   ├── useDebounce.ts              # Input debounce hook
│   ├── useMovieSearch.ts           # Central search & filter controller
│   └── useSearchTracker.ts         # Query analytics logging & hook
├── pages/
│   ├── Admin.tsx                   # Comprehensive administrator management panel
│   └── Home.tsx                    # Main portal and search landing page
├── types/
│   ├── brokenLink.ts               # Broken link TypeScript types
│   ├── movie.ts                    # Movie & season download schema types
│   ├── request.ts                  # Title request schema types
│   └── tmdb.ts                     # TMDB API payload types
└── utils/
    ├── downloads.ts                # Default download link generators
    ├── sanitizeMovie.ts            # Schema normalizer and cleaner
    ├── slug.ts                     # URL slug generator
    └── tmdb.ts                     # TMDB API client & transformer
Environment Variables & Configuration
Create or update your .env file in the root directory:
code
Env
# Server Port
PORT=3000

# The Movie Database API Key (v3 auth)
TMDB_API_KEY=your_tmdb_api_key_here

# Google Gemini API Key (for server-side AI mood search)
GEMINI_API_KEY=your_gemini_api_key_here

# Administrator Passkey for Admin Portal
ADMIN_PASSKEY=Feelam@Admin2025!

# SMTP Configuration for Request Fulfillment Emails (Optional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password
SMTP_FROM="Feelam Movies" <no-reply@feelam.com>
Installation & Deployment
1. Prerequisites
Node.js (version 18 or higher)
npm or yarn
2. Install Dependencies
code
Bash
npm install
3. Run in Development Mode
Starts the Express server with Vite middleware mounted on port 3000:
code
Bash
npm run dev
4. Build for Production
Compiles the React frontend using Vite and builds the TypeScript assets:
code
Bash
npm run build
5. Start Production Server
code
Bash
npm start
The server will bind to 0.0.0.0:3000 and serve static production assets with full API support.
