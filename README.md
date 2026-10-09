# Gym AI Assistant - Backend API

[![Node.js](https://img.shields.io/badge/Node.js-v18%2B-green.svg)](https://nodejs.org/)
[![Express.js](https://img.shields.io/badge/Express.js-4.x-blue.svg)](https://expressjs.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-DB-blue)](https://www.postgresql.org/)
[![OpenRouter API](https://img.shields.io/badge/OpenRouter-API-purple)](https://openrouter.ai/)
[![ES Modules](https://img.shields.io/badge/ES_Modules-Enabled-yellow.svg)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

A high-performance, modular **Express.js backend** for an AI-powered Gym Trainer & Nutrition Coach. This service handles user queries regarding workouts, dietary plans, macro counts, and fitness advice by interfacing with **OpenRouter API** (`openai/gpt-3.5-turbo`) and persisting chat histories in a **PostgreSQL** database.

---

## Table of Contents

- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [System Architecture](#-system-architecture)
- [Database Schema](#-database-schema)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Setup](#environment-setup)
  - [Running the Server](#running-the-server)
- [API Documentation](#-api-documentation)
- [Project Structure](#-project-structure)
- [License](#-license)

---

## Features

-  **AI Gym & Nutrition Specialist**: Custom prompt tailored for fitness advice, workout routines, calorie/protein counts, and motivational coaching.
-  **Real-time AI Chat**: Responds dynamically to user fitness queries via OpenRouter LLM integrations.
-  **Persistent Chat Storage**: Automatically saves user prompts and generated AI answers to PostgreSQL.
-  **Chat History API**: Retrieve complete chat histories sorted by creation timestamp.
-  **CORS & JSON Enabled**: Ready to seamlessly connect with mobile apps (Flutter / React Native) and web frontends.
-  **Clean Modular Architecture**: Separated into controllers, routes, services, and configurations for scalability and maintainability.

---

## Tech Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Runtime** | Node.js (ES Modules) | Asynchronous JavaScript runtime |
| **Framework** | Express.js v4 | Web framework for Node.js |
| **AI Provider** | OpenRouter API (`gpt-3.5-turbo`) | LLM model for domain-specific fitness responses |
| **Database** | PostgreSQL | Relational database for chat logging and history |
| **HTTP Client** | `node-fetch` | Native API fetching |
| **Environment** | `dotenv` | Secure configuration management |

---

## System Architecture

```text
               +-----------------------------------+
               |    Client (Flutter / Web App)     |
               +-----------------------------------+
                                 |
                                 | HTTP REST Requests
                                 v
               +-----------------------------------+
               |        Express.js Server          |
               |           (index.js)              |
               +-----------------------------------+
                                 |
                        /api/chat | /api/history
                                 v
               +-----------------------------------+
               |        Routes & Controllers       |
               +-----------------------------------+
                        /                     \
                       /                       \
                      v                         v
       +----------------------------+  +----------------------------+
       |   AI Service (aiService.js) |  |   Database Pool (db.js)    |
       +----------------------------+  +----------------------------+
                      |                             |
                      | OpenRouter API              | SQL Queries
                      v                             v
       +----------------------------+  +----------------------------+
       |   OpenRouter LLM Engine    |  |    PostgreSQL Database     |
       |  (openai/gpt-3.5-turbo)    |  |        (gym_ai)           |
       +----------------------------+  +----------------------------+
```

---

## Database Schema

Before running the backend, create the `gym_ai` PostgreSQL database and run the following SQL script to set up the `chat_history` table:

```sql
CREATE DATABASE gym_ai;

\c gym_ai;

CREATE TABLE IF NOT EXISTS chat_history (
    id SERIAL PRIMARY KEY,
    message TEXT NOT NULL,
    reply TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Getting Started

### Prerequisites

Ensure you have the following installed on your machine:
- [Node.js](https://nodejs.org/) (v18 or higher)
- [PostgreSQL](https://www.postgresql.org/) (v12 or higher)
- An active [OpenRouter API Key](https://openrouter.ai/)

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/abinjoyal268-glitch/AiChatbot.git
   cd AiChatbot
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

### Environment Setup

Create a `.env` file in the root directory (or copy from `.env.example`):

```bash
cp .env.example .env
```

Configure your environment variables in `.env`:

```env
PORT=3000
OPENROUTER_API_KEY=your_openrouter_api_key_here
```

Ensure PostgreSQL database credentials in `config/db.js` match your local setup:

```javascript
const pool = new Pool({
  user: "postgres",
  host: "localhost",
  database: "gym_ai",
  password: "YOUR_DB_PASSWORD",
  port: 5432,
});
```

### Running the Server

- **Start server:**
  ```bash
  node index.js
  ```
- Server will run at: `http://localhost:3000`

---

## 🔌 API Documentation

### 1. Health Check
Checks if the backend service is active.

- **URL:** `/`
- **Method:** `GET`
- **Response:**
  ```text
  Gym AI Backend Running 
  ```

---

### 2. Send Message to Gym AI
Sends a user message to the AI trainer, receives a customized short response, and records the interaction in PostgreSQL.

- **URL:** `/api/chat`
- **Method:** `POST`
- **Headers:** `Content-Type: application/json`
- **Request Body:**
  ```json
  {
    "message": "Give me a quick chest workout routine"
  }
  ```
- **Success Response (200 OK):**
  ```json
  {
    "reply": "Chest workout → Bench press 4x10, pushups 3x15, incline dumbbell press 3x12 | Keep pushing bro"
  }
  ```
- **Error Response (400 Bad Request):**
  ```json
  {
    "error": "Message required"
  }
  ```

---

### 3. Get Chat History
Retrieves all historical chat interactions saved in the database.

- **URL:** `/api/history`
- **Method:** `GET`
- **Success Response (200 OK):**
  ```json
  [
    {
      "id": 1,
      "message": "Give me a quick chest workout routine",
      "reply": "Chest workout → Bench press 4x10, pushups 3x15, incline dumbbell press 3x12 | Keep pushing bro",
      "created_at": "2026-10-09T16:00:00.000Z"
    }
  ]
  ```

---

## Project Structure

```text
aibackend/
├── config/
│   ├── db.js             # PostgreSQL connection configuration
│   └── openai.js         # OpenRouter API key loader
├── controllers/
│   └── chatController.js # Route handler for incoming chat requests
├── routes/
│   └── aiRoutes.js       # Express routing definitions (/chat, /history)
├── services/
│   └── aiService.js      # OpenRouter API integration & DB persistence logic
├── .env                  # Environment variables (git-ignored)
├── .env.example          # Environment template
├── index.js              # Server entry point & Express middleware
├── package.json          # Node.js dependencies & scripts
└── README.md             # Project documentation
```

---

## Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [Issues page](https://github.com/abinjoyal268-glitch/AiChatbot/issues).

---

## License

This project is licensed under the [ISC License](LICENSE).
