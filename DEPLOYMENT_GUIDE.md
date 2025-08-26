# AI SaaS Platform - Architecture Diagram & Deployment Guide

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (Frontend)                        │
├─────────────────────────────────────────────────────────────────┤
│  Next.js 13 App Router                                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   Landing Page   │  │   Auth Pages    │  │   Dashboard     │  │
│  │   (Public)       │  │   (Clerk)       │  │   (Protected)   │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │              Tailwind CSS + shadcn/ui                      │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  │ HTTPS
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                       SERVER (Backend)                         │
├─────────────────────────────────────────────────────────────────┤
│  Next.js API Routes                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   Auth          │  │   AI Services   │  │   Payments      │  │
│  │   Middleware    │  │   /api/*        │  │   /api/stripe   │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                    │                        │                │
                    ▼                        ▼                ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Clerk Auth    │  │   AI Services   │  │   Stripe API    │
│   - OAuth       │  │   - OpenAI      │  │   - Payments    │
│   - JWT tokens  │  │   - Replicate   │  │   - Webhooks    │
└─────────────────┘  └─────────────────┘  └─────────────────┘
                                                               │
                    ┌─────────────────┐                       │
                    │   Database      │◄──────────────────────┘
                    │   MySQL/Prisma  │
                    │   - Users       │
                    │   - Limits      │
                    │   - Subscriptions│
                    └─────────────────┘
```

## 🔄 Data Flow Diagram

```
User Request → Middleware → API Route → Service Integration → Database → Response

1. User Authentication:
   Browser → Clerk → JWT → Middleware → Dashboard

2. AI Generation:
   Dashboard → Form → API Route → Usage Check → AI Service → Response

3. Subscription Flow:
   Pro Modal → Stripe Checkout → Webhook → Database → Access Granted
```

## 🚀 Deployment Strategies

### **1. Vercel Deployment (Recommended)**

Vercel provides the best integration with Next.js:

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy from root directory
vercel --prod

# Set environment variables in Vercel dashboard
# Configure custom domain
# Set up automatic deployments from GitHub
```

**Vercel Configuration** (`vercel.json`):
```json
{
  "framework": "nextjs",
  "buildCommand": "cd ai-saas && npm run build",
  "outputDirectory": "ai-saas/.next",
  "installCommand": "cd ai-saas && npm install",
  "env": {
    "DATABASE_URL": "@database-url",
    "OPENAI_API_KEY": "@openai-api-key",
    "STRIPE_API_KEY": "@stripe-api-key"
  }
}
```

### **2. Docker Deployment**

Create `ai-saas/Dockerfile`:
```dockerfile
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

**Docker Compose** (`docker-compose.yml`):
```yaml
version: '3.8'
services:
  app:
    build: ./ai-saas
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - STRIPE_API_KEY=${STRIPE_API_KEY}
    depends_on:
      - db
  
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: ai_saas
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

### **3. AWS Deployment**

**Using AWS Amplify:**
```bash
# Install Amplify CLI
npm install -g @aws-amplify/cli

# Initialize project
amplify init

# Add hosting
amplify add hosting

# Deploy
amplify publish
```

**Using AWS ECS with Fargate:**
```bash
# Build and push Docker image to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

docker build -t ai-saas ./ai-saas
docker tag ai-saas:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/ai-saas:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/ai-saas:latest

# Deploy using ECS task definition
```

## 🔧 Production Configuration

### **Environment Variables for Production**

```bash
# Production Environment (.env.production)
NODE_ENV=production
NEXT_PUBLIC_APP_URL=https://yourdomain.com

# Database (Use connection pooling in production)
DATABASE_URL="mysql://user:password@host:3306/db?connection_limit=20&pool_timeout=20&sslmode=require"

# Clerk (Production keys)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_...
CLERK_SECRET_KEY=sk_live_...

# Stripe (Live mode keys)
STRIPE_API_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...

# AI Services (Production keys with higher limits)
OPENAI_API_KEY=sk-...
REPLICATE_API_TOKEN=r8_...
```

### **Performance Optimizations**

**Next.js Configuration** (`next.config.js`):
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    serverComponentsExternalPackages: ["@prisma/client"],
  },
  images: {
    domains: ['images.unsplash.com', 'replicate.delivery'],
  },
  // Enable standalone output for Docker
  output: 'standalone',
  // Optimize for production
  swcMinify: true,
  compress: true,
}

module.exports = nextConfig
```

**Database Optimization:**
```prisma
// Prisma schema optimizations
model UserApiLimit {
  id        String   @id @default(cuid())
  userId    String   @unique
  count     Int      @default(0)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([userId]) // Add index for faster queries
}

model UserSubscription {
  id                     String    @id @default(cuid())
  userId                 String    @unique
  stripeCustomerId       String?   @unique @map(name: "stripe_customer_id")
  stripeSubscriptionId   String?   @unique @map(name: "stripe_subscription_id")
  stripePriceId          String?   @map(name: "stripe_price_id")
  stripeCurrentPeriodEnd DateTime? @map(name: "stripe_current_period_end")
  
  @@index([userId, stripeSubscriptionId]) // Compound index
}
```

## 🔍 Monitoring & Analytics

### **Error Tracking** (Sentry Integration)

```bash
npm install @sentry/nextjs

# Configure Sentry
echo "SENTRY_DSN=your-sentry-dsn" >> .env.production
```

**Sentry Configuration** (`sentry.client.config.js`):
```javascript
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  tracesSampleRate: 1.0,
});
```

### **Analytics** (Google Analytics)

```typescript
// lib/gtag.ts
export const GA_TRACKING_ID = process.env.NEXT_PUBLIC_GA_ID;

export const pageview = (url: string) => {
  window.gtag('config', GA_TRACKING_ID, {
    page_path: url,
  });
};
```

### **Performance Monitoring**

```typescript
// Monitor API response times
export async function POST(req: Request) {
  const start = Date.now();
  
  try {
    // API logic
    const result = await processRequest();
    
    // Log performance
    console.log(`API call took ${Date.now() - start}ms`);
    
    return NextResponse.json(result);
  } catch (error) {
    // Error tracking
    console.error('API Error:', error);
    throw error;
  }
}
```

## 🛡️ Security Checklist

### **Production Security**

✅ **HTTPS Enforcement**: Use HTTPS in production  
✅ **Environment Variables**: Never commit secrets to git  
✅ **API Rate Limiting**: Implement rate limiting for APIs  
✅ **Input Validation**: Validate all user inputs with Zod  
✅ **CORS Configuration**: Restrict CORS to your domain  
✅ **Security Headers**: Implement security headers  
✅ **Database Security**: Use connection pooling and SSL  
✅ **Webhook Verification**: Verify Stripe webhook signatures  

### **Security Headers** (`next.config.js`):

```javascript
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin',
          },
        ],
      },
    ];
  },
};
```

## 🔄 CI/CD Pipeline

### **GitHub Actions** (`.github/workflows/deploy.yml`):

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: ai-saas/package-lock.json
    
    - name: Install dependencies
      run: cd ai-saas && npm ci
    
    - name: Run tests
      run: cd ai-saas && npm test
    
    - name: Build application
      run: cd ai-saas && npm run build
    
    - name: Deploy to Vercel
      uses: amondnet/vercel-action@v20
      with:
        vercel-token: ${{ secrets.VERCEL_TOKEN }}
        vercel-org-id: ${{ secrets.ORG_ID }}
        vercel-project-id: ${{ secrets.PROJECT_ID }}
        working-directory: ./ai-saas
```

This comprehensive deployment guide ensures your AI SaaS platform is production-ready with proper security, monitoring, and scalability considerations.