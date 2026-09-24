# Terribly Made Notes App

[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB-7-green?logo=mongodb)](https://www.mongodb.com/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker)](https://www.docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

An intelligent audio-to-notes platform that transcribes lectures, voice memos, and meetings, generating structured Markdown study guides, interactive flashcards, practice quizzes, and conversational AI assistants using OpenAI-compatible APIs.

---

## Features

### Audio Processing & Transcription
- **Broad Format Support**: Upload audio in `.mp3`, `.wav`, `.m4a`, `.aac`, `.flac`, or `.ogg`.
- **FFmpeg Standardization**: Automatically converts incoming audio to normalized MP3 format and extracts audio metadata (duration, bitrate, channels, and recorded timestamps).
- **OpenAI-Compatible STT**: Connects to any speech-to-text endpoint (OpenAI Whisper, Groq, local Whisper instances, etc.) with support for English and multilingual transcription.
- **Resilient Pipeline**: Fault-tolerant background processing queue with one-click retry for failed jobs without requiring re-upload.

### Study Aids & Note Generation
- **Structured Markdown**: Generates clean summaries, key takeaways, and detailed explanations.
- **KaTeX & Chemical Formula Support**: Full math formatting via KaTeX inline (`$...$`) and display (`$$...$$`) math, including `mhchem` extension support.
- **Automated Flashcards**: Generates interactive flashcards with front/back flip animations for active recall.
- **Practice Quizzes**: Automatically constructs multiple-choice quizzes with instant feedback, explanations, and optional hints.

### Conversational Q&A
- **Single-Note Chat**: Ask questions and clarify concepts grounded directly in the note's content.
- **Multi-Note Chat**: Query and synthesize knowledge across multiple notes simultaneously.

### Organization & Sharing
- **Course & Subject Classes**: Organize notes into custom user classes and folders.
- **Search & Filter**: Full-text search and sorting by recorded date or upload date.
- **Shift-Click Multi-Select**: Select note ranges for bulk operations.
- **Public Share Links**: Generate standalone public share links for single notes or bulk collections, allowing recipients to view notes and chat with the AI assistant without an account.
- **Apple Shortcut Integration**: Download the pre-built Apple Shortcut (`/public/Upload to notes.shortcut`) and generate user tokens for one-tap voice memo uploads from iOS/macOS.
- **Native iOS App**: Dedicated SwiftUI companion app available in [`terribly-made-notes-app-ios`](https://github.com/koboshchan/terribly-made-notes-app-ios).

---

## Architecture & Processing Pipeline

```
[Audio Input] ──> [FFmpeg Conversion] ──> [Metadata Extraction]
                                                   │
                                                   ▼
[User Notes / Classes] <── [MongoDB] <── [In-Memory Queue]
        │                                          │
        ├──────────────────────┬───────────────────┴───────────────────┐
        ▼                      ▼                                       ▼
  [STT Endpoint]         [LLM Summary]                           [LLM Extraction]
  (Raw Transcript)    (Markdown + KaTeX)                     (Flashcards & Quizzes)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Framework** | [Next.js 16](https://nextjs.org/) (App Router, Server Actions, Route Handlers) |
| **Language** | [TypeScript 5](https://www.typescriptlang.org/) |
| **Authentication** | [Clerk](https://clerk.com/) (`@clerk/nextjs`) |
| **Database** | [MongoDB 7](https://www.mongodb.com/) (Native Node.js Driver) |
| **Audio Processing** | [FFmpeg](https://ffmpeg.org/) via `fluent-ffmpeg` |
| **Math & Markdown** | `marked`, `katex`, `marked-katex-extension` |
| **Networking** | `undici` (Configured for long-running transcription timeouts) |
| **Deployment** | Docker & Docker Compose |

---

## Quick Start (Docker)

The fastest way to deploy the entire stack (Next.js app + MongoDB + FFmpeg runtime):

### 1. Clone the repository
```bash
git clone https://github.com/koboshchan/terribly-made-notes-app.git
cd terribly-made-notes-app
```

### 2. Configure environment variables
Copy the example environment file and fill in your Clerk keys:
```bash
cp .env.local.example .env.local
```

Edit `.env.local`:
```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
MONGODB_URI=mongodb://mongo:27017/notesapp
DATA_DIR=./data
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 3. Launch with Docker Compose
```bash
docker compose up -d
```

> [!NOTE]
> The web application will be accessible at **http://localhost:3000**. MongoDB data is persisted in `./mongodata/` and note artifacts/audio files are stored in `./data/`.

To build the image locally from source:
```bash
docker compose up --build -d
```

---

## Local Development (Node.js)

### Prerequisites
- **Node.js**: v20 or later
- **MongoDB**: Running instance (local or MongoDB Atlas)
- **FFmpeg**: Installed and available in your system `$PATH` (`brew install ffmpeg` on macOS)

### Setup Steps
1. Install dependencies:
   ```bash
   npm install
   ```

2. Configure `.env.local`:
   ```env
   NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
   CLERK_SECRET_KEY=sk_test_...
   MONGODB_URI=mongodb://127.0.0.1:27017/notesapp
   DATA_DIR=./data
   NEXT_PUBLIC_APP_URL=http://localhost:3000
   ```

3. Run the development server:
   ```bash
   npm run dev
   ```

---

## First-Time Configuration

After signing in for the first time:

1. Navigate to **Settings** (`/settings` or `/admin/models`).
2. Configure your AI model providers:
   - **Speech-to-Text (STT)**: Base URL, API Key, Model Name (e.g. `whisper-1`), and Task (`transcribe` / `translate`).
   - **Large Language Model (LLM)**: Base URL, API Key, Model Name (e.g. `gpt-4o`, `claude-3-5-sonnet`, `llama-3`), and Quiz Model.
3. Save your configuration. The settings are stored in MongoDB and applied to all subsequent note processing jobs.

> [!TIP]
> Any OpenAI-compatible endpoint can be used, including OpenAI, Groq, OpenRouter, Ollama, vLLM, and LiteLLM proxies.

---

## Environment Variables

| Variable | Required | Description | Default |
|---|:---:|---|---|
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Yes | Clerk Frontend API key | — |
| `CLERK_SECRET_KEY` | Yes | Clerk Backend Secret key | — |
| `MONGODB_URI` | Yes | MongoDB connection string | `mongodb://mongo:27017/notesapp` |
| `DATA_DIR` | No | Directory for audio and markdown file storage | `./data` |
| `NEXT_PUBLIC_APP_URL` | No | Public domain used when generating share links | `http://localhost:3000` |
| `NEXT_PUBLIC_CLERK_SIGN_IN_URL` | No | Custom sign-in route | `/sign-in` |
| `NEXT_PUBLIC_CLERK_SIGN_UP_URL` | No | Custom sign-up route | `/sign-up` |

---

## Native iOS Companion App

A native SwiftUI companion app for iOS is available in [terribly-made-notes-app-ios](https://github.com/koboshchan/terribly-made-notes-app-ios):
- Native audio recording with live waveform and duration monitoring.
- Interactive 3D flip card animations for flashcards.
- Native practice quizzes with feedback and hints.
- Seamless authentication with ClerkKit & ClerkKitUI.
- Direct background synchronization with the Notes API.

---

## License

This project is licensed under the [MIT License](LICENSE).
