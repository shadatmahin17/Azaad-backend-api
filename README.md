# Azaad Backend API

Azaad Backend API is an Express-based service for managing songs, admin authentication, and Supabase-backed user profiles.

It powers:
- a lightweight built-in upload/admin UI served from `public/`
- API-driven integrations (including the React admin app in `frontend/`)

## Table of Contents
- [Features](#features)
- [Architecture](#architecture)
- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [API Reference](#api-reference)
- [Storage & Media Behavior](#storage--media-behavior)
- [Security Notes](#security-notes)
- [Development Notes](#development-notes)
- [Deployment](#deployment)
- [License](#license)

## Features

- Song catalog API (list, create, update, delete)
- File upload support for audio and cover images (Multer)
- URL-based media support (`http(s)` and `s3://...`)
- Admin API key protection for song management routes
- Hybrid authentication:
  - local admin login (`ADMIN_USERNAME` / `ADMIN_PASSWORD`)
  - Supabase email/password login
- Supabase profile endpoints:
  - sign up / sign in
  - profile read/update
  - avatar upload to Supabase Storage
- Static hosting for uploaded files and built-in web UI

## Architecture

- **Runtime:** Node.js + Express
- **Data storage:** JSON file (`songs.json`) by default
- **Auth:** API key + optional Supabase Auth
- **Media:** local filesystem uploads, optional S3-style URL normalization

## Requirements

- Node.js 18+
- npm 9+

## Quick Start

1. Install dependencies:

```bash
npm install
```

2. Copy environment variables:

```bash
cp .env.example .env
```

3. Start the API server:

```bash
npm start
```

Server default:

- `http://localhost:5000`

Built-in UI:

- `http://localhost:5000/`

## Configuration

Environment variables are loaded from `.env` (or `.env.example` if `.env` does not exist).

### Core

- `PORT` – API server port (default: `5000`)
- `ADMIN_API_KEY` – required in `x-api-key` for protected song endpoints
- `ADMIN_USERNAME` – local admin username
- `ADMIN_PASSWORD` – local admin password

### Storage / Data Paths

- `DATA_DIR` – optional custom data directory (stores `songs.json` when set)
- `SONGS_FILE` – optional absolute/relative path override for songs JSON file
- `AWS_REGION` / `S3_REGION` – used to normalize `s3://bucket/key` URLs

### Supabase

- `SUPABASE_URL` (or `NEXT_PUBLIC_SUPABASE_URL`)
- `SUPABASE_SERVICE_ROLE_KEY` or `SUPABASE_PUBLISHABLE_KEY` (or `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`)
- `SUPABASE_STORAGE_BUCKET` (default: `avatars`)

> **Important:** Do not commit real credentials or service keys. Rotate any exposed keys before production use.

## Project Structure

```text
.
├── server.js                  # Express API server
├── songs.json                 # Song data (default JSON storage)
├── public/                    # Built-in static UI assets
├── uploads/                   # Runtime audio/cover uploads (auto-created)
├── supabase/
│   └── schema.sql             # Optional Supabase schema
├── frontend/                  # Optional React admin client
└── README.md
```

## API Reference

Base URL: `http://localhost:5000`

### Health / Info

- `GET /api` – API metadata and route summary

### Song Endpoints

- `GET /api/songs` – list songs
- `POST /api/songs` – create song (**requires `x-api-key`**)
- `PUT /api/songs/:id` – update song (**requires `x-api-key`**)
- `DELETE /api/songs/:id` – delete song (**requires `x-api-key`**)

#### `POST /api/songs` input

Accepts multipart form data:

- `title` (required)
- `artist` (required)
- `audio` file **or** `audioUrl` (required)
- `cover` file **or** `coverUrl` (required)
- optional: `category`, `genre`, `singers`, `type`, `vibe`, `featured`, `trending`

### Admin Auth

- `POST /api/login`
  - Supports local username/password auth
  - Also supports Supabase email/password auth when configured
- `GET /api/auth-check` – validates `x-api-key`

### Supabase Auth + Profile

- `POST /api/auth/signup`
- `POST /api/auth/signin`
- `GET /api/profile-view` (**requires bearer token**)
- `PUT /api/profile` (**requires bearer token**)
- `POST /api/profile/avatar` (**requires bearer token**, multipart `avatar`)

### Authentication Headers

API key routes:

```http
x-api-key: <ADMIN_API_KEY>
```

Bearer routes:

```http
Authorization: Bearer <access_token>
```

## Storage & Media Behavior

- Uploaded files are stored under:
  - `uploads/audio/`
  - `uploads/covers/`
- Songs are persisted in `songs.json` by default.
- On delete, local uploaded files referenced by the song are removed automatically.
- `s3://bucket/key` media URLs are normalized to public S3 HTTPS URLs.

## Security Notes

Before production:

- Replace default credentials and API key
- Use HTTPS and a restrictive CORS policy
- Store secrets in environment management (not in Git)
- Add rate limiting, request logging, and centralized monitoring
- Move from JSON storage to a managed database for scale and durability

## Development Notes

Useful scripts:

```bash
npm start
npm run dev
```

Both currently run `server.js`.

Optional frontend dev server:

```bash
cd frontend
npm install
npm run dev
```

## Deployment

Recommended topology:

- **API:** Render, Railway, Fly.io, or AWS
- **Frontend (optional):** Vercel or Netlify
- **Media:** object storage (e.g., S3)
- **Data:** migrate from JSON file to PostgreSQL for production workloads

## License

MIT
