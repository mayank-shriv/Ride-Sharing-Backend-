# 🚗 RideFlow — Uber-like Ride-Hailing Backend API

> A production-grade, backend-focused REST + WebSocket API built with the MERN stack (Node.js, Express, MongoDB, Socket.IO). Implements real-time driver tracking, geospatial matching, ride state machines, surge pricing, and a BullMQ job queue — the same core architecture powering apps like Uber and Lyft.

---

## 📌 Table of Contents

- [Project Title & Description](#-rideflow--uber-like-ride-hailing-backend-api)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture Overview](#-architecture-overview)
- [Folder Structure](#-folder-structure)
- [API Endpoints](#-api-endpoints)
- [Real-time Events (Socket.IO)](#-real-time-events-socketio)
- [Ride State Machine](#-ride-state-machine)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [Running the Project](#-running-the-project)
- [Running Tests](#-running-tests)
- [Key Concepts](#-key-concepts)
- [Contributing](#-contributing)
- [License](#-license)

---

## ✨ Features

- **JWT Authentication** with refresh tokens, OTP via Twilio, and Google OAuth
- **Role-based access control** — Rider, Driver, Admin
- **Geospatial driver matching** — MongoDB `$near` with `2dsphere` index
- **Ride state machine** — `requested → driver_found → accepted → arrived → ongoing → completed`
- **Real-time location tracking** — Socket.IO rooms per ride
- **Surge pricing engine** — demand/supply ratio-based fare multiplier
- **BullMQ + Redis job queues** — ride timeouts, notifications, payment retries
- **Stripe & Razorpay integration** — payments, refunds, and wallet system
- **Firebase push notifications** + Twilio SMS
- **Google Maps APIs** — Directions, Distance Matrix, Geocoding
- **Rate limiting, request logging, and global error handling**

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js (v18+) |
| Framework | Express.js |
| Database | MongoDB + Mongoose |
| Cache / Queue | Redis (ioredis) + BullMQ |
| Real-time | Socket.IO |
| Authentication | JWT, bcrypt, Passport.js |
| Payments | Stripe / Razorpay |
| Notifications | Firebase FCM, Twilio |
| Maps | Google Maps Platform APIs |
| File Storage | Cloudinary |
| Testing | Jest + Supertest |
| Logging | Winston |

---

## 🏗 Architecture Overview

```
Client (Rider / Driver / Admin)
          │
          │  REST + WebSocket
          ▼
   ┌─────────────────────┐
   │   API Gateway        │  ← Rate limiting, JWT auth, logging
   │   (Express.js)       │
   └─────────┬───────────┘
             │
    ┌────────┴─────────────────────────────────┐
    │              Core Services                │
    │  Auth │ Ride │ Location │ Payment         │
    └────────┬─────────────────────────────────┘
             │
    ┌────────┴─────────────────────────────────┐
    │           Support Services                │
    │  Notification │ Pricing │ Matching │ Queue│
    └────────┬─────────────────────────────────┘
             │
   ┌─────────▼───────────┐
   │  Socket.IO Layer     │  ← Ride rooms, GPS broadcast
   └─────────┬───────────┘
             │
   ┌─────────▼───────────────────────────────────┐
   │              Data Layer                      │
   │  MongoDB  │  Redis  │  External APIs         │
   └─────────────────────────────────────────────┘
```

---

## 📁 Folder Structure

```
uber-backend/
│
├── src/
│   ├── config/
│   │   ├── db.js                  # MongoDB connection
│   │   ├── redis.js               # Redis client (ioredis)
│   │   ├── socket.js              # Socket.IO initialization
│   │   └── env.js                 # Environment variable loader
│   │
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.routes.js
│   │   │   ├── auth.controller.js
│   │   │   ├── auth.service.js    # JWT, OTP, refresh token logic
│   │   │   └── auth.validator.js
│   │   │
│   │   ├── user/
│   │   │   ├── user.routes.js
│   │   │   ├── user.controller.js
│   │   │   ├── user.service.js
│   │   │   └── user.model.js
│   │   │
│   │   ├── driver/
│   │   │   ├── driver.routes.js
│   │   │   ├── driver.controller.js
│   │   │   ├── driver.service.js
│   │   │   └── driver.model.js    # 2dsphere geo index
│   │   │
│   │   ├── ride/
│   │   │   ├── ride.routes.js
│   │   │   ├── ride.controller.js
│   │   │   ├── ride.service.js
│   │   │   ├── ride.model.js
│   │   │   └── ride.statemachine.js   # FSM transitions
│   │   │
│   │   ├── location/
│   │   │   ├── location.routes.js
│   │   │   ├── location.service.js    # $near / $geoWithin queries
│   │   │   └── location.handler.js    # Socket GPS event handlers
│   │   │
│   │   ├── payment/
│   │   │   ├── payment.routes.js
│   │   │   ├── payment.controller.js
│   │   │   ├── payment.service.js     # Stripe, wallet, refunds
│   │   │   └── payment.model.js
│   │   │
│   │   ├── pricing/
│   │   │   ├── pricing.service.js     # Base fare calculation
│   │   │   └── surge.calculator.js    # Surge multiplier logic
│   │   │
│   │   ├── matching/
│   │   │   └── matching.service.js    # Nearest driver algorithm
│   │   │
│   │   └── notification/
│   │       ├── notification.service.js
│   │       ├── fcm.provider.js        # Firebase push notifications
│   │       └── sms.provider.js        # Twilio SMS
│   │
│   ├── middlewares/
│   │   ├── auth.middleware.js         # JWT verification
│   │   ├── role.middleware.js         # RBAC: rider / driver / admin
│   │   ├── rateLimit.middleware.js    # express-rate-limit
│   │   ├── error.middleware.js        # Global error handler
│   │   └── validate.middleware.js     # Joi / Zod schema validation
│   │
│   ├── jobs/
│   │   ├── queues.js                  # BullMQ queue definitions
│   │   ├── ride.worker.js             # Ride timeout / auto-cancel
│   │   ├── notification.worker.js     # Async push/SMS delivery
│   │   └── payment.worker.js          # Payment retry logic
│   │
│   ├── sockets/
│   │   ├── index.js                   # Register all namespaces
│   │   ├── ride.socket.js             # Ride status & ETA events
│   │   └── driver.socket.js           # Live GPS location stream
│   │
│   ├── utils/
│   │   ├── apiResponse.js             # Unified JSON response helper
│   │   ├── asyncHandler.js            # try/catch wrapper
│   │   ├── geoUtils.js                # Haversine, coord helpers
│   │   └── logger.js                  # Winston logger config
│   │
│   └── app.js                         # Express app setup & middleware
│
├── tests/
│   ├── unit/
│   │   ├── pricing.test.js
│   │   └── statemachine.test.js
│   └── integration/
│       ├── auth.test.js
│       └── ride.test.js
│
├── .env
├── .env.example
├── .gitignore
├── server.js                          # Entry point — HTTP + Socket.IO
└── package.json
```

---

## 📡 API Endpoints

### Auth
| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/api/v1/auth/register` | Register rider or driver | Public |
| POST | `/api/v1/auth/login` | Login with phone/email | Public |
| POST | `/api/v1/auth/send-otp` | Send OTP via SMS | Public |
| POST | `/api/v1/auth/verify-otp` | Verify OTP | Public |
| POST | `/api/v1/auth/refresh-token` | Refresh JWT access token | Public |
| POST | `/api/v1/auth/logout` | Invalidate tokens | Auth |

### User (Rider)
| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/v1/users/me` | Get own profile | Rider |
| PUT | `/api/v1/users/me` | Update profile | Rider |
| GET | `/api/v1/users/rides` | Ride history | Rider |

### Driver
| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/v1/drivers/me` | Get driver profile | Driver |
| PUT | `/api/v1/drivers/status` | Toggle online/offline | Driver |
| PUT | `/api/v1/drivers/location` | Update live location | Driver |
| GET | `/api/v1/drivers/earnings` | Earnings summary | Driver |

### Ride
| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/api/v1/rides/request` | Create a ride request | Rider |
| GET | `/api/v1/rides/:id` | Get ride details | Auth |
| POST | `/api/v1/rides/:id/accept` | Driver accepts ride | Driver |
| POST | `/api/v1/rides/:id/start` | Start the trip | Driver |
| POST | `/api/v1/rides/:id/complete` | Complete the trip | Driver |
| POST | `/api/v1/rides/:id/cancel` | Cancel a ride | Auth |
| POST | `/api/v1/rides/:id/rate` | Rate rider or driver | Auth |

### Pricing
| Method | Endpoint | Description | Access |
|---|---|---|---|
| POST | `/api/v1/pricing/estimate` | Get fare estimate | Rider |

### Payment
| Method | Endpoint | Description | Access |
|---|---|---|---|
| GET | `/api/v1/payments/history` | Payment history | Auth |
| POST | `/api/v1/payments/refund/:id` | Request a refund | Rider |
| GET | `/api/v1/payments/wallet` | Wallet balance | Auth |

---

## 🔌 Real-time Events (Socket.IO)

### Rider listens
| Event | Payload | Description |
|---|---|---|
| `driver:found` | `{ driverId, eta, location }` | Driver matched |
| `driver:location` | `{ lat, lng }` | Live driver GPS |
| `ride:status` | `{ status }` | Ride state changes |
| `ride:completed` | `{ fare, duration }` | Trip ended |

### Driver listens
| Event | Payload | Description |
|---|---|---|
| `ride:request` | `{ rideId, pickup, dropoff, fare }` | New ride request |
| `ride:cancelled` | `{ rideId }` | Rider cancelled |

### Driver emits
| Event | Payload | Description |
|---|---|---|
| `location:update` | `{ lat, lng, rideId }` | Broadcast GPS to rider |
| `driver:ready` | `{ driverId }` | Mark driver as online |

---

## 🔄 Ride State Machine

```
                   ┌───────────┐
              ┌───►│ cancelled │◄──────────────────┐
              │    └───────────┘                   │
              │                                    │
┌─────────┐   │   ┌─────────────┐   ┌──────────┐  │  ┌─────────┐   ┌───────────┐
│requested├───┼──►│ driver_found├──►│ accepted ├──┴─►│ ongoing ├──►│ completed │
└─────────┘   │   └─────────────┘   └────┬─────┘     └─────────┘   └───────────┘
              │                          │
              │                    ┌─────▼──────┐
              └────────────────────│  arrived   │
                                   └────────────┘
```

Transitions are validated server-side. Invalid transitions return a `400` error with the allowed next states.

---

## 🚀 Getting Started

### Prerequisites

- Node.js v18+
- MongoDB v6+
- Redis v7+
- Google Cloud project with Maps APIs enabled
- Stripe account
- Twilio account
- Firebase project (for push notifications)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/rideflow-backend.git
cd rideflow-backend

# Install dependencies
npm install

# Copy environment file
cp .env.example .env
# Fill in your credentials in .env

# Start MongoDB and Redis (if running locally)
mongod --fork --logpath /var/log/mongod.log
redis-server --daemonize yes
```

---

## 🔐 Environment Variables

```env
# Server
PORT=5000
NODE_ENV=development

# MongoDB
MONGO_URI=mongodb://localhost:27017/rideflow

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your_jwt_secret_here
JWT_REFRESH_SECRET=your_refresh_secret_here
JWT_ACCESS_EXPIRES=15m
JWT_REFRESH_EXPIRES=7d

# Twilio (OTP / SMS)
TWILIO_ACCOUNT_SID=your_sid
TWILIO_AUTH_TOKEN=your_token
TWILIO_PHONE_NUMBER=+1234567890

# Google Maps
GOOGLE_MAPS_API_KEY=your_key_here

# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Firebase (Push Notifications)
FIREBASE_SERVICE_ACCOUNT_KEY=./firebase-service-account.json

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud
CLOUDINARY_API_KEY=your_key
CLOUDINARY_API_SECRET=your_secret
```

---

## ▶️ Running the Project

```bash
# Development (with hot reload)
npm run dev

# Production
npm start

# Run BullMQ workers (separate process)
npm run workers
```

---

## 🧪 Running Tests

```bash
# All tests
npm test

# Unit tests only
npm run test:unit

# Integration tests only
npm run test:integration

# Coverage report
npm run test:coverage
```

---

## 💡 Key Concepts

### Surge Pricing Formula
```
surge_multiplier = 1.0 + (active_requests / available_drivers)
surge_multiplier = Math.min(surge_multiplier, 3.0)  // Capped at 3x

final_fare = base_rate × distance_km × surge_multiplier
```

### Geospatial Driver Query (MongoDB)
```js
// Find online drivers within 5km, sorted by proximity
Driver.find({
  isOnline: true,
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [lng, lat] },
      $maxDistance: 5000  // meters
    }
  }
}).limit(10)
```

### Ride Room Pattern (Socket.IO)
```js
// Each ride gets its own room — only rider + driver receive events
socket.join(`ride:${rideId}`)
io.to(`ride:${rideId}`).emit('driver:location', { lat, lng })
```

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m 'feat: add your feature'`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

Please follow the [Conventional Commits](https://www.conventionalcommits.org/) spec for commit messages.

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](./LICENSE) file for details.

---

<div align="center">
  Built with ❤️ using Node.js · Express · MongoDB · Socket.IO
</div>
