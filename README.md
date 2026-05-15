# online-auction-system
A full-stack real-time auction platform built with the MERN stack
Live Demo

Status License PRs Welcome Issues Forks Stars Last Commit

Create auctions · Bid in real-time · Manage everything from an admin panel

Live Demo · Report Bug · Request Feature · Architecture · Learning Guide · Backend Docs · Frontend Docs

Why This Project?
Most auction system tutorials stop at basic CRUD. This project goes much further:

Real-time bidding — Socket.io rooms with atomic MongoDB updates prevent race conditions
Production security — httpOnly cookies, JWT auth, XSS-safe email templates, input sanitization
Smart UX — Hover prefetching, View Transitions API, live countdown timers, auto-winner detection
Deployment-ready — CI/CD pipeline, Vercel serverless support, AWS EC2 with PM2, graceful shutdown
Built as a Major Project for Computer Science Engineering by Avnish Kumar, designed to be a real-world reference for full-stack MERN development.

📖 New here? Read the Architecture Guide to understand how the system works, and the Learning Guide to see what's implemented, why, and what you can build next.

Features
Category	Features
Authentication	JWT with httpOnly secure cookies · Auto-login on refresh · Role-based access (User/Admin) · Password change with validation
Auctions	Signed Cloudinary upload on image select (instant preview + progress) · Create via metadata payload (formId, public_id, secure_url) · Browse with pagination · Category filtering · Live countdown timers · Auto-winner detection on expiry
Real-time Bidding	Socket.io room-based architecture · Atomic bid updates (no race conditions) · Live active user count · Instant bid broadcast to all viewers · Seller cannot bid on own auction
Dashboard	Personal stats (total/active auctions) · Recent auctions grid · Quick navigation to all sections
Admin Panel	System-wide statistics · User management with search, sort, pagination · Recent signups tracking · Role-based route protection
Security	Login tracking (IP, geo-location, device, browser) · Login history per user · bcrypt password hashing · Environment variable validation at startup
Email	Contact form with Resend · Dual email (admin notification + user confirmation) · XSS-safe HTML templates
Performance	React Query caching · Hover-based data prefetching · View Transitions API page animations · gzip compression · Optimized MongoDB indexes
Deployment	GitHub Actions CI/CD → AWS EC2 · Vercel serverless support · PM2 process management · Graceful shutdown handling
Tech Stack
Frontend	Backend	Infrastructure
React 19 + Vite
Tailwind CSS v4
React Router v7
Redux Toolkit
TanStack React Query
Socket.io Client
React Hot Toast

Node.js + Express 5
MongoDB + Mongoose
Socket.io
JWT + bcrypt
Cloudinary (signed direct upload)
Resend (email)
Compression

AWS EC2
Vercel (frontend)
GitHub Actions CI/CD
PM2
Cloudinary CDN

Quick Start
Prerequisites
Node.js 20+ and npm
MongoDB (local or Atlas)
Cloudinary account (free tier)
1. Clone & Install
git clone https://github.com/KLUsujith/online auction system.git
cd online-auction-system

# Install backend
cd server && npm install

# Install frontend
cd ../client && npm install
2. Environment Variables
Server (server/.env):

PORT=3000
ORIGIN=http://localhost:5173
MONGO_URL=mongodb://localhost:27017/auction
JWT_SECRET=your-secret-key-here
JWT_EXPIRES_IN=7d
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
CLOUDINARY_URL=cloudinary://...
RESEND_API_KEY=re_xxxxxxxxxxxx
Client (client/.env):

VITE_API=http://localhost:3000
VITE_AUCTION_API=http://localhost:3000/auction
3. Run
# Terminal 1 — Backend
cd server && npm run dev

# Terminal 2 — Frontend
cd client && npm run dev
Open http://localhost:5173 — you're live!

Project Structure
online-auction-system/
├── client/                      # React frontend (see client/README.md)
│   ├── src/
│   │   ├── components/          # Reusable UI (Navbar, AuctionCard, Footer)
│   │   ├── pages/               # Route pages (Dashboard, ViewAuction, etc.)
│   │   ├── hooks/               # React Query hooks + Socket hook
│   │   ├── services/            # API service layer (Axios)
│   │   ├── store/               # Redux Toolkit (auth state)
│   │   ├── layout/              # Layouts (Main, Admin, Open)
│   │   └── routers/             # Route definitions
│   └── package.json
│
├── server/                      # Express backend (see server/README.md)
│   ├── controllers/             # Route handlers
│   ├── models/                  # Mongoose schemas (User, Product, Login)
│   ├── routes/                  # REST API routes
│   ├── socket/                  # Socket.io initialization + auction handlers
│   ├── middleware/               # Auth middleware
│   ├── services/                # Cloudinary integration
│   ├── utils/                   # JWT, cookies, geo-location
│   ├── config/                  # DB + env configuration
│   ├── app.js                   # Express app setup
│   └── server.js                # HTTP server + Socket.io + graceful shutdown
│
├── .github/workflows/           # CI/CD pipeline
└── README.md
Architecture
Real-time Bidding Flow
┌─────────────────────────────────────────────────────────────────┐
│  Client (ViewAuction)                                           │
│                                                                 │
│  useSocket hook                    REST API                     │
│  ┌──────────────┐                 ┌──────────────┐              │
│  │ Connect      │                 │ POST /bid    │              │
│  │ Join Room    │                 │ Atomic Update│              │
│  │ Listen Bids  │                 │ Return Data  │              │
│  │ Cleanup      │                 └──────┬───────┘              │
│  └──────┬───────┘                        │                      │
│         │                                │                      │
└─────────┼────────────────────────────────┼──────────────────────┘
          │ WebSocket                      │ HTTP
          │                                │
┌─────────┼────────────────────────────────┼──────────────────────┐
│  Server │                                │                      │
│         ▼                                ▼                      │
│  ┌──────────────┐                 ┌──────────────┐              │
│  │ Socket.io    │                 │ Express API  │              │
│  │ Auth via JWT │                 │ secureRoute  │              │
│  │ Room: {id}   │◄────Broadcast───│ placeBid()   │              │
│  │ Track Users  │                 │ Atomic update│              │
│  └──────────────┘                 └──────────────┘              │
│                                          │                      │
│                                   ┌──────▼───────┐              │
│                                   │   MongoDB    │              │
│                                   │ findOneAndUp │              │
│                                   │ date + price │              │
│                                   │  condition   │              │
│                                   └──────────────┘              │
└─────────────────────────────────────────────────────────────────┘
Race condition prevention: Bids use findOneAndUpdate with a price condition — if two users bid simultaneously, only the first succeeds; the second gets a retry prompt.

Authentication Flow
Login/Signup → Server sets httpOnly cookie (auth_token)
     │
Page Refresh → InitAuth dispatches checkAuth()
     │              │
     │         GET /user (cookie sent automatically)
     │              │
     │         Returns { user } or 401
     │              │
     ▼         Redux updates auth state
App renders (protected routes check auth.user)
API Reference
Full backend documentation with request/response examples: server/README.md

Authentication
Method	Endpoint	Description
POST	/auth/signup	Register new user
POST	/auth/login	Login (sets httpOnly cookie)
POST	/auth/logout	Logout (clears cookie)
User
Method	Endpoint	Description	Auth
GET	/user	Get current user profile	Required
PATCH	/user	Change password	Required
GET	/user/logins	Login history (last 10)	Required
Auctions
Method	Endpoint	Description	Auth
GET	/auction	List auctions (paginated)	Required
POST	/auction	Create auction (JSON + uploaded image metadata)	Required
GET	/auction/stats	Dashboard statistics	Required
GET	/auction/myauction	User's own auctions	Required
GET	/auction/mybids	Auctions user has bid on	Required
GET	/auction/:id	Single auction detail	Required
POST	/auction/:id/bid	Place a bid	Required
Admin
Method	Endpoint	Description	Auth
GET	/admin/dashboard	Admin statistics	Admin
GET	/admin/users	List users (paginated, searchable)	Admin
Upload
Method	Endpoint	Description	Auth
GET	/upload/signature	Generate signed Cloudinary upload params	Required
Contact
Method	Endpoint	Description	Auth
POST	/contact	Submit contact form	Public
Socket.io Events
Event	Direction	Payload
auction:join	Client → Server	{ auctionId }
auction:leave	Client → Server	{ auctionId }
auction:bid	Client → Server	{ auctionId, bidAmount }
auction:userJoined	Server → Room	{ userName, userId, activeUsers[] }
auction:userLeft	Server → Room	{ userName, userId, activeUsers[] }
auction:bidPlaced	Server → Room	{ auction, bidderName, bidAmount }
auction:error	Server → Client	{ message }
Socket connections are authenticated via JWT from cookies. Users are tracked per room with automatic cleanup on disconnect.

Deployment
Frontend → Vercel
cd client && npm run build
# Deploy via Vercel CLI or GitHub integration
Backend → AWS EC2 (Automated)
The included GitHub Actions workflow (.github/workflows/deploy.yml) auto-deploys on push to main:

Add GitHub Secrets: EC2_HOST, EC2_USERNAME, EC2_SSH_KEY, EC2_SSH_PORT, EC2_PROJECT_PATH, and all .env variables
EC2 Setup: Node.js 20+, PM2 (npm i -g pm2), Git, SSH keys
Push to main → workflow SSHs into EC2, pulls code, installs deps, writes .env, restarts PM2
Full list of required GitHub Secrets
Contributing
Contributions are what make the open source community amazing. Any contributions you make are greatly appreciated.

Fork the repository
Create a feature branch (git checkout -b feature/amazing-feature)
Install dependencies (cd server && npm i && cd ../client && npm i)
Make your changes following existing code style
Commit using conventional commits (git commit -m "feat: add amazing feature")
Push to your branch (git push origin feature/amazing-feature)
Open a Pull Request
Ideas for contribution
Payment integration — Stripe/Razorpay for winning bids
Push notifications — Real-time bid alerts via WebPush
Advanced search — Full-text search with filters
User ratings — Buyer/seller reputation system
Email notifications — Automated auction activity emails
Testing — Unit and integration test coverage
Accessibility — WCAG compliance improvements
