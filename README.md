# 💳 Roger — RFID Smart Shopping & Wallet System

> **Team:** Roger_Don_Durkheim  
> **Live Dashboard:** [http://157.173.101.159:9203/](http://157.173.101.159:9203/)

---

## 📖 Overview

Roger is an embedded IoT system that enables cashless shopping using RFID cards. Customers tap their registered RFID card to authenticate, add products to a cart (also identified by RFID tags), and pay directly from their digital wallet — all in real time.

The system consists of three layers:

- **ESP8266 Edge Controller** — reads RFID cards/tags and publishes events over MQTT
- **Node.js Backend** — processes MQTT events, manages the database, exposes a REST API and WebSocket stream
- **Web Dashboard** — real-time admin POS interface for managing products, users, top-ups and transactions

---

## 🌐 Live Access

| Resource | URL |
|----------|-----|
| 🖥️ Admin Dashboard | [http://157.173.101.159:9203/](http://157.173.101.159:9203/) |
| 🔌 REST API Base | `http://157.173.101.159:9203/api` |
| 📡 WebSocket | `ws://157.173.101.159:9203` |

**Default Admin Credentials:**
```
Email:    admin@roger.com
Password: admin123
```

---

## 🏗️ System Architecture

```
┌─────────────────┐        MQTT         ┌─────────────────┐       HTTP/WS      ┌──────────────┐
│   ESP8266 +     │ ──────────────────► │   Node.js +     │ ◄────────────────► │  Web Browser │
│   MFRC522 RFID  │ ◄────────────────── │   Express API   │                    │  Dashboard   │
└─────────────────┘                     │   + MongoDB     │                    └──────────────┘
                                        └─────────────────┘
                                               │
                                        MQTT Broker
                                     broker.benax.rw:1883
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|------------|
| Microcontroller | ESP8266 (MicroPython) |
| RFID Reader | MFRC522 |
| Backend | Node.js + Express |
| Database | MongoDB |
| Real-time | MQTT + WebSocket |
| Auth | bcrypt + express-session |

---

## 📁 Project Structure

```
RFID/
├── server.js              # Main backend server (Express + MQTT + WebSocket)
├── seed.js                # MongoDB database seeder (run once)
├── reset-admin.js         # Admin password reset utility
├── dashboard.html         # Frontend admin POS interface
├── package.json
└── README.md
```

---

## ⚡ Getting Started

### Prerequisites

- Node.js v16+
- MongoDB Community Server running locally or on VPS
- MQTT broker access (`broker.benax.rw:1883`)

### 1. Clone & Install

```bash
git clone <your-repo-url>
cd RFID
npm install
```

### 2. Seed the Database

Run this **once** to create collections, indexes, and seed products + the admin user:

```bash
node seed.js
```

### 3. Start the Server

```bash
node server.js
```

Server starts at `http://localhost:3000` (or the configured `PORT`).

### 4. Open the Dashboard

Visit: [http://157.173.101.159:9203/](http://157.173.101.159:9203/)

---

## ⚙️ Environment Variables

Create a `.env` file or set these variables before starting the server:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `3000` | HTTP server port |
| `MONGO_URI` | `mongodb://127.0.0.1:27017` | MongoDB connection string |
| `DB_NAME` | `roger_db` | MongoDB database name |
| `MQTT_BROKER` | `mqtt://broker.benax.rw:1883` | MQTT broker URL |

---

## 📡 MQTT Topics

All topics are namespaced under the team ID:

| Topic | Direction | Description |
|-------|-----------|-------------|
| `rfid/Roger_Don_Durkheim/card/status` | ESP → Backend | Card scan event |
| `rfid/Roger_Don_Durkheim/card/topup` | Backend → ESP | Top-up confirmation |
| `rfid/Roger_Don_Durkheim/card/pay` | Backend → ESP | Payment confirmation |
| `rfid/Roger_Don_Durkheim/card/balance` | ESP → Backend | Balance update |

---

## 🔌 REST API Endpoints

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/register` | Register a new customer card |
| `POST` | `/auth/login` | Admin login |
| `GET` | `/auth/user` | Get current session user |
| `POST` | `/auth/logout` | Logout |

### Products

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/products` | Public | List all products |
| `POST` | `/api/products` | Admin | Create product |
| `PUT` | `/api/products/:id` | Admin | Update product |
| `DELETE` | `/api/products/:id` | Admin | Delete product |

### Wallet & Payments

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/wallet/topup` | Admin | Top up customer wallet |
| `POST` | `/payment/checkout` | Card/Session | Process cart checkout |

### Transactions

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/transactions` | Admin | List recent 50 transactions |

---

## 🗄️ MongoDB Collections

| Collection | Description |
|------------|-------------|
| `users` | Customers and admin accounts |
| `products` | RFID-tagged products with prices and stock |
| `transactions` | Payment records |
| `transaction_items` | Line items per transaction |
| `topups` | Wallet top-up history |

---

## 🔧 Hardware Setup (ESP8266)

**Wiring — MFRC522 to ESP8266:**

| MFRC522 Pin | ESP8266 Pin |
|-------------|-------------|
| SDA (CS) | GPIO2 |
| SCK | GPIO14 |
| MOSI | GPIO13 |
| MISO | GPIO12 |
| RST | GPIO0 |
| 3.3V | 3.3V |
| GND | GND |

**Flash MicroPython firmware**, then upload:
- `main.py` (the ESP8266 edge controller script)
- `mfrc522.py` (RFID library)

Update WiFi credentials in `main.py`:
```python
WIFI_SSID = "YourNetwork"
WIFI_PASS  = "YourPassword"
```

---

## 🔑 Admin Password Reset

If login fails on a new environment, reset the admin password:

```bash
node reset-admin.js
```

This rehashes `admin123` and updates the admin record in MongoDB.

---

## 🛒 How It Works

1. **Customer taps RFID card** → ESP8266 reads UID and publishes to `card/status`
2. **Backend receives scan** → looks up user in MongoDB → broadcasts to dashboard via WebSocket
3. **Admin adds products to cart** by tapping product RFID tags
4. **Checkout** → backend deducts total from customer wallet, records transaction, updates stock
5. **Admin tops up wallet** via dashboard → MongoDB wallet balance updated instantly

---

## 👥 Team

**Roger_Don_Durkheim**

---

## 📄 License

This project was developed for the Embedded Systems course. All rights reserved.
