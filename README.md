# FounderPath v2

> Interactive educational platform for founders with lessons, simulations, and progress tracking.

## Features

- 🎓 **Interactive Lessons**: Structured learning paths for founders
- 🎮 **Simulations**: Practice decision-making in realistic scenarios
- 📊 **Progress Tracking**: Monitor your learning journey
- 🏆 **Achievements & Quests**: Gamified learning experience
- 🎯 **Capstone Projects**: Apply your knowledge
- 💳 **Premium Access**: Stripe-powered subscription system
- 🔐 **Modern Auth**: Privy integration (email + wallet)
- 🤖 **AI Assistant**: Get personalized guidance

## Tech Stack

### Frontend
- **React 18** with TypeScript
- **Vite** for blazing-fast builds
- **Tailwind CSS** + **shadcn/ui** for beautiful UI
- **TanStack Query** for server state management
- **Wouter** for lightweight routing
- **Privy** for authentication

### Backend
- **Express** + TypeScript
- **PostgreSQL** with **Neon** serverless
- **Drizzle ORM** for type-safe database access
- **Stripe** for payments
- **Zod** for validation

## Getting Started

### Prerequisites

- Node.js 20+
- PostgreSQL database (or Neon account)
- Stripe account
- Privy account

### Installation

1. Clone the repository:
```bash
git clone https://github.com/puncar-dev/founderpath-v2.git
cd founderpath-v2
```

2. Install dependencies:
```bash
npm install
```

3. Set up environment variables:
```bash
cp .env.example .env
# Edit .env with your credentials
```

4. Initialize the database:
```bash
npm run db:push
```

5. Start the development server:
```bash
npm run dev
```

The app will be available at `http://localhost:5000`.

## Development

```bash
# Run development server with hot reload
npm run dev

# Type check
npm run check

# Generate database migrations
npm run db:generate

# Apply migrations
npm run db:migrate

# Open Drizzle Studio (database GUI)
npm run db:studio
```

## Production Build

```bash
# Build both client and server
npm run build

# Start production server
npm start
```

## Deployment on Railway

### Quick Deploy

1. **Fork/Clone** this repository to your GitHub account

2. **Create a new project** on [Railway.app](https://railway.app)

3. **Add PostgreSQL database**:
   - Right-click canvas → "Database" → "Add PostgreSQL"
   - Railway auto-provides `DATABASE_URL`

4. **Deploy from GitHub**:
   - Click "New" → "GitHub Repo"
   - Select `founderpath-v2`
   - Railway auto-detects build settings

5. **Add environment variables**:
   ```
   NODE_ENV=production
   PORT=8080
   STRIPE_SECRET_KEY=sk_live_...
   STRIPE_PUBLISHABLE_KEY=pk_live_...
   VITE_PRIVY_APP_ID=...
   ```

6. **Run migrations**:
   - In Railway dashboard, go to your service
   - Settings → Deploy → Custom Start Command:
   ```bash
   npm run db:migrate && npm start
   ```

7. **Generate domain**:
   - Settings → Networking → Generate Domain

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string | ✅ |
| `NODE_ENV` | Environment (production/development) | ✅ |
| `PORT` | Server port (Railway provides this) | ✅ |
| `STRIPE_SECRET_KEY` | Stripe secret key | ✅ |
| `STRIPE_PUBLISHABLE_KEY` | Stripe publishable key | ❌ |
| `STRIPE_WEBHOOK_SECRET` | Stripe webhook secret | ❌ |
| `VITE_PRIVY_APP_ID` | Privy application ID | ✅ |

## Project Structure

```
founderpath-v2/
├── client/               # React frontend
│   ├── src/
│   │   ├── components/  # Reusable UI components
│   │   ├── pages/       # Route pages
│   │   ├── hooks/       # Custom React hooks
│   │   ├── lib/         # Utilities and configs
│   │   └── types/       # TypeScript types
│   └── index.html
├── server/              # Express backend
│   ├── db/             # Database utilities
│   ├── routes/         # API routes
│   └── index.ts        # Server entry point
├── shared/              # Shared types and schemas
│   └── schema.ts       # Drizzle schema
└── migrations/          # Database migrations
```

## Key Improvements (v2)

### 🔒 Production-Ready Database Migrations
- Dedicated migration runner (`server/db/migrate.ts`)
- Safe migrations using generated SQL files
- Automatic migration on startup in production

### ✅ Environment Variable Validation
- Centralized validation in `server/config/env.ts`
- Type-safe environment access
- Fail-fast on missing critical variables

### 🎨 Cleaner Code Architecture
- Payment logic extracted to custom hook
- Better separation of concerns
- Removed Replit-specific files

### 📦 Optimized Build Process
- Separate client/server builds
- Smaller production bundle
- Better caching

### 🚀 Railway-Optimized Deployment
- Single command deployment
- Auto-migration on startup
- Health check endpoint

## API Routes

### Authentication
- `POST /api/auth/login` - Login with Privy
- `POST /api/auth/logout` - Logout
- `GET /api/auth/me` - Get current user

### User Management
- `GET /api/users/:id` - Get user profile
- `PATCH /api/users/:id` - Update user profile

### Progress
- `GET /api/progress` - Get user progress
- `POST /api/progress` - Update lesson progress

### Payments
- `POST /api/stripe/create-checkout-session` - Start payment
- `POST /api/stripe/webhook` - Stripe webhook handler

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT © [puncar-dev](https://github.com/puncar-dev)
