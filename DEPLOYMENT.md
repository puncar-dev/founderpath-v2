# FounderPath v2 - Deployment & Improvements Guide

## Overview

This guide covers the enhanced architecture, improvements over v1, and step-by-step deployment instructions for Railway.

## Key Improvements from v1

### 1. 🔒 Production-Ready Database Migrations

**Problem in v1:** Using `drizzle-kit push` directly modifies the database schema without migration history, risking data loss in production.

**Solution in v2:**
```typescript
// server/db/migrate.ts
import { migrate } from 'drizzle-orm/neon-http/migrator';

export async function runMigrations() {
  const sql = neon(env.DATABASE_URL);
  const db = drizzle(sql);
  await migrate(db, { migrationsFolder: './migrations' });
}
```

**Benefits:**
- Safe, version-controlled migrations
- Rollback capability
- Migration history tracking
- No accidental schema overwrites

### 2. ✅ Environment Variable Validation

**Problem in v1:** Environment variables accessed directly with no validation, leading to runtime crashes.

**Solution in v2:**
```typescript
// server/config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number),
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().min(1),
});

export const env = envSchema.parse(process.env);
```

**Benefits:**
- Fail-fast on startup if config is missing
- Type-safe environment access
- Clear error messages
- Self-documenting configuration

### 3. 🎨 Cleaner Code Architecture

**Problem in v1:** Payment callback logic mixed into Router component.

**Solution in v2:**
```typescript
// client/src/hooks/usePaymentCallback.ts
export function usePaymentCallback() {
  const { toast } = useToast();
  
  useEffect(() => {
    const params = new URLSearchParams(window.location.search);
    const status = params.get('payment');
    
    if (status) {
      handlePaymentStatus(status, toast);
      cleanUrl();
    }
  }, []);
}

// Usage in App.tsx
function App() {
  usePaymentCallback();
  return <Router />;
}
```

**Benefits:**
- Separation of concerns
- Reusable hooks
- Easier testing
- Cleaner component code

### 4. 📦 Optimized Build Process

**Changes:**
```json
{
  "scripts": {
    "build": "npm run build:client && npm run build:server",
    "build:client": "vite build",
    "build:server": "esbuild server/index.ts --platform=node --packages=external --bundle --format=esm --outdir=dist"
  }
}
```

**Benefits:**
- Separate client/server builds
- Smaller bundle sizes
- Better tree-shaking
- Faster cold starts

### 5. 🚀 Railway-Optimized

**Added files:**
- `railway.json` - Railway configuration
- Health check endpoint at `/api/health`
- Auto-migration on startup
- Proper signal handling (SIGTERM/SIGINT)

## Complete File Structure

```
founderpath-v2/
├── .env.example              # Environment template
├── .gitignore
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.ts
├── drizzle.config.ts
├── components.json          # shadcn/ui config
├── railway.json             # Railway deployment config
├── client/
│   ├── index.html
│   └── src/
│       ├── main.tsx
│       ├── App.tsx
│       ├── index.css
│       ├── components/      # Reusable UI components
│       │   ├── ui/          # shadcn components
│       │   ├── layout/      # Layout components
│       │   └── features/    # Feature-specific components
│       ├── pages/           # Route pages
│       │   ├── landing.tsx
│       │   ├── founder-path.tsx
│       │   ├── profile.tsx
│       │   └── ...
│       ├── hooks/           # Custom React hooks
│       │   ├── use-toast.ts
│       │   ├── use-payment-callback.ts
│       │   └── use-user.ts
│       ├── lib/             # Utilities
│       │   ├── queryClient.ts
│       │   └── utils.ts
│       └── types/           # TypeScript types
├── server/
│   ├── index.ts             # Server entry point
│   ├── config/
│   │   └── env.ts           # Environment validation
│   ├── db/
│   │   ├── index.ts         # Database connection
│   │   └── migrate.ts       # Migration runner
│   ├── routes/
│   │   ├── index.ts         # Route registration
│   │   ├── auth.ts          # Auth routes
│   │   ├── users.ts         # User routes
│   │   ├── progress.ts      # Progress routes
│   │   └── stripe.ts        # Payment routes
│   ├── middleware/
│   │   ├── auth.ts          # Auth middleware
│   │   └── error.ts         # Error handling
│   └── vite.ts              # Vite dev/prod setup
├── shared/
│   └── schema.ts            # Drizzle schema & types
└── migrations/              # Generated SQL migrations
```

## Step-by-Step Deployment on Railway

### Prerequisites

1. **GitHub Account** with access to the repository
2. **Railway Account** ([railway.app](https://railway.app))
3. **Stripe Account** with API keys
4. **Privy Account** with App ID

### Step 1: Create Railway Project

1. Go to [railway.app](https://railway.app/new)
2. Click **"New Project"**
3. Select **"Deploy from GitHub repo"**
4. Choose `puncar-dev/founderpath-v2`
5. Railway will create a new project

### Step 2: Add PostgreSQL Database

1. In your Railway project canvas
2. Click **"+"** or right-click → **"New"**
3. Select **"Database"** → **"Add PostgreSQL"**
4. Railway automatically:
   - Provisions a Postgres instance
   - Creates `DATABASE_URL` variable
   - Links it to your service

### Step 3: Configure Environment Variables

1. Click on your **service** (not database)
2. Go to **"Variables"** tab
3. Add the following:

```bash
# Required
NODE_ENV=production
PORT=8080
DATABASE_URL=${{Postgres.DATABASE_URL}}  # Auto-filled if database linked
STRIPE_SECRET_KEY=sk_live_xxxxx
VITE_PRIVY_APP_ID=clxxxxx

# Optional
STRIPE_WEBHOOK_SECRET=whsec_xxxxx
```

**💡 Tip:** Railway automatically injects `DATABASE_URL` when you link the database.

### Step 4: Configure Build Settings

1. Go to **"Settings"** tab
2. Verify/set the following:

**Build Command:**
```bash
npm run build
```

**Start Command:**
```bash
npm run db:migrate && npm start
```

**Watch Paths** (optional, for faster deploys):
```
/client/**
/server/**
/shared/**
package.json
```

### Step 5: Deploy

1. Railway automatically starts deploying
2. Monitor in **"Deployments"** tab
3. Wait for build to complete (2-3 minutes)

### Step 6: Get Your Public URL

1. Go to **"Settings"** → **"Networking"**
2. Click **"Generate Domain"**
3. Railway provides: `founderpath-v2-production.up.railway.app`
4. Optionally add custom domain

### Step 7: Verify Deployment

1. **Health Check:**
   ```bash
   curl https://your-app.railway.app/api/health
   # Should return: {"status":"ok","timestamp":"..."}
   ```

2. **Frontend:**
   - Visit `https://your-app.railway.app`
   - Should load landing page

3. **Database Connection:**
   - Try signing up/logging in
   - Verify no database errors in logs

### Step 8: Configure Webhooks (Optional)

**For Stripe Webhooks:**

1. Go to Stripe Dashboard → **Developers** → **Webhooks**
2. Click **"Add endpoint"**
3. Set URL: `https://your-app.railway.app/api/stripe/webhook`
4. Select events:
   - `checkout.session.completed`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
5. Copy **Signing Secret**
6. Add to Railway: `STRIPE_WEBHOOK_SECRET=whsec_xxxxx`

**For Privy:**

1. Privy Dashboard → **Settings**
2. Add production URL to **Allowed Origins**:
   ```
   https://your-app.railway.app
   ```

## Database Migration Workflow

### Development

```bash
# Make schema changes in shared/schema.ts
# Generate migration
npm run db:generate

# Review generated SQL in migrations/
# Apply migration
npm run db:migrate
```

### Production

Railway runs migrations automatically on deployment:

```bash
# In package.json start script:
"start": "npm run db:migrate && node dist/index.js"
```

Migrations run **before** server starts, ensuring schema is up-to-date.

## Monitoring & Debugging

### View Logs

```bash
# In Railway Dashboard
1. Click on your service
2. Go to "Deployments" tab
3. Click on latest deployment
4. View real-time logs
```

### Common Issues

**1. "DATABASE_URL is required" error:**
- Ensure PostgreSQL database is linked to service
- Check Variables tab for `DATABASE_URL`

**2. Migration fails:**
```bash
# In Railway console, run:
railway run npm run db:push
# This force-pushes schema (use only in emergencies)
```

**3. Build fails:**
- Check build logs for TypeScript errors
- Verify all dependencies in `package.json`
- Ensure `NODE_ENV` is set

**4. App crashes after deployment:**
- Check runtime logs
- Verify environment variables
- Test health endpoint

### Health Check Endpoint

```typescript
// server/routes/index.ts
app.get('/api/health', (req, res) => {
  res.json({ 
    status: 'ok', 
    timestamp: new Date().toISOString(),
    env: process.env.NODE_ENV 
  });
});
```

## Performance Optimization

### Database

1. **Connection Pooling:**
```typescript
// Already configured with Neon serverless
const sql = neon(env.DATABASE_URL, {
  fetchOptions: {
    cache: 'no-store',
  },
});
```

2. **Indexes:**
```typescript
// Add indexes for frequent queries
export const userProgress = pgTable('user_progress', {
  // ...
}, (table) => ({
  userIdIdx: index('user_id_idx').on(table.userId),
  lessonIdIdx: index('lesson_id_idx').on(table.lessonId),
}));
```

### Caching

Add Redis for session storage:

```bash
# In Railway, add Redis
Redis -> Add Redis

# Update session config
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

const redisClient = createClient({ url: process.env.REDIS_URL });
app.use(session({
  store: new RedisStore({ client: redisClient }),
  // ...
}));
```

## Security Checklist

- ✅ Environment variables validated
- ✅ HTTPS enforced (Railway default)
- ✅ CORS configured properly
- ✅ Helmet.js for security headers
- ✅ Rate limiting on API routes
- ✅ SQL injection prevented (Drizzle ORM)
- ✅ XSS protection (React default)
- ✅ Stripe webhook signature verification

## Rollback Procedure

If deployment fails:

1. **In Railway Dashboard:**
   - Go to **"Deployments"**
   - Find last working deployment
   - Click **"Redeploy"**

2. **Database Migration Rollback:**
```bash
# If migration caused issue, manually rollback:
railway run drizzle-kit drop
# Then redeploy previous version
```

## Cost Estimation (Railway)

- **Hobby Plan (Free):**
  - $5 credit/month
  - Good for testing
  - Sleeps after 500 hours

- **Pro Plan ($20/month):**
  - 1 GB RAM: ~$5/month
  - PostgreSQL (1 GB): ~$5/month
  - Bandwidth: ~$2/month
  - **Total: ~$12/month** for moderate traffic

## Next Steps

1. 👥 **Set up team access** in Railway
2. 📊 **Add monitoring** (Sentry, LogRocket)
3. 📧 **Configure email** (SendGrid, Resend)
4. 🌐 **Add custom domain**
5. 🚨 **Set up alerts** for errors
6. 📋 **Enable analytics** (PostHog, Mixpanel)

## Support

For issues:
1. Check Railway logs
2. Review this deployment guide
3. Open an issue on GitHub
4. Contact Railway support

---

**Happy Deploying! 🚀**
