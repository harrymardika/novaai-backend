# NOVA AI: Backend

REST API for **NOVA AI** (*Next-gen Orbit of Visionary Artificial Intelligence Technology*), a multimodal chatbot web app powered by Google Gemini. The backend stores each user's chat history in MongoDB, protects every chat endpoint with Clerk authentication, signs image uploads for ImageKit, and serves the built React frontend.

- **Frontend:** [novaai-client](https://github.com/harrymardika/novaai-client)
- **Live demo:** https://novaai-backend-rho.vercel.app

## Features

- **Per-user chat history:** conversations are stored in MongoDB and linked to the signed-in Clerk user; users can only read, update, or delete their own chats.
- **Chat list with titles:** a new chat is titled with the first 40 characters of its first message.
- **Image uploads:** the API returns ImageKit authentication parameters so the browser can upload images directly, and the image URL is saved with the message.
- **Single deployment:** the server also serves the production frontend build from `dist/`, so the app runs as one Vercel deployment.

## Architecture

```
React client (novaai-client)
   │  Clerk session token; Gemini is called from the browser (streaming)
   ▼
Express API (this repo) ──► MongoDB (Mongoose): chats, userchats
   └──► ImageKit: signed upload parameters
```

The Gemini model is called from the client; the backend saves the question, answer, and optional image URL after each turn.

## API Endpoints

All `/api/chats` and `/api/userchats` routes require a valid Clerk session.

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/upload` | ImageKit authentication parameters |
| `POST` | `/api/chats` | Create a chat from the first message `{ text }`, returns its ID |
| `GET` | `/api/userchats` | List the user's chats (ID, title, date) |
| `GET` | `/api/chats/:id` | One chat with full history |
| `PUT` | `/api/chats/:id` | Append a turn `{ question, answer, img }` |
| `DELETE` | `/api/chats/:id` | Delete a chat and remove it from the list |
| `GET` | `*` | Serve the frontend from `dist/` |

## Data Model

- **`chat`**: `userId` and `history[]`, each entry with `role` (`user` or `model`), `parts[].text`, and optional `img` (same format Gemini uses).
- **`userchats`**: `userId` and `chats[]` with `_id`, `title`, and `createdAt`.

## Tech Stack

Bun, Express 4, MongoDB with Mongoose 8, Clerk (`@clerk/clerk-sdk-node`), ImageKit, Vercel (`@vercel/node`).

## Project Structure

```
novaai-backend/
├── server.js                  # Express app, routes, static frontend
├── config/                    # database.js (MongoDB), imagekit.js
├── controllers/chatController.js
├── models/                    # chat.js, userChats.js
├── dist/                      # Production build of novaai-client
└── vercel.json
```

## Getting Started

Requirements: [Bun](https://bun.sh/), a MongoDB database, a [Clerk](https://clerk.com/) app, and an [ImageKit](https://imagekit.io/) account.

```bash
git clone https://github.com/harrymardika/novaai-backend.git
cd novaai-backend
bun install
```

Create a `.env` file:

```env
PORT=3000
CLIENT_URL=http://localhost:5173
MONGO=<MongoDB connection string>
CLERK_PUBLISHABLE_KEY=<Clerk publishable key>
CLERK_SECRET_KEY=<Clerk secret key>
IMAGE_KIT_ENDPOINT=<ImageKit URL endpoint>
IMAGE_KIT_PUBLIC_KEY=<ImageKit public key>
IMAGE_KIT_PRIVATE_KEY=<ImageKit private key>
```

```bash
bun start
```

The API runs at `http://localhost:3000`. Run [novaai-client](https://github.com/harrymardika/novaai-client) with `VITE_API_URL=http://localhost:3000`.

## Author

**Harry Mardika** · [GitHub](https://github.com/harrymardika)
