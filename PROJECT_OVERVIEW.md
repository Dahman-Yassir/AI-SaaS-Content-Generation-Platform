# AI SaaS Content Generation Platform - Project Overview

## 🚀 Project Summary

This is a comprehensive **AI-powered SaaS platform** built with modern web technologies that provides multiple AI content generation tools through a subscription-based model. The platform integrates various AI services to offer text, code, image, video, and music generation capabilities.

## 🏗️ Architecture Overview

### **Technology Stack**
- **Frontend Framework**: Next.js 13 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Database**: MySQL with Prisma ORM
- **Authentication**: Clerk (supports 9+ social logins)
- **Payments**: Stripe (subscription management)
- **AI Services**: 
  - OpenAI API (text/code generation)
  - Replicate AI (video/music generation)
- **State Management**: Zustand
- **Form Handling**: React Hook Form with Zod validation
- **UI Components**: Radix UI primitives
- **Customer Support**: Crisp Chat integration

### **Project Structure**
```
ai-saas/
├── app/                    # Next.js 13 App Router
│   ├── (auth)/            # Authentication pages
│   ├── (dashboard)/       # Protected dashboard routes
│   │   └── (routes)/      # Individual tool pages
│   ├── (landing)/         # Public landing page
│   ├── api/               # API routes
│   └── layout.tsx         # Root layout
├── components/            # Reusable React components
├── lib/                   # Utility libraries
├── prisma/               # Database schema
├── hooks/                # Custom React hooks
└── public/               # Static assets
```

## 🎯 Core Features

### **AI Generation Tools**
1. **Conversation Tool** - GPT-powered chat interface
2. **Code Generation** - AI-assisted code writing
3. **Image Generation** - AI-created images
4. **Video Generation** - AI-generated videos
5. **Music Generation** - AI-composed music

### **Business Features**
- **Freemium Model**: Free tier with usage limits (5 API calls)
- **Pro Subscription**: Unlimited usage via Stripe
- **User Authentication**: Secure auth with Clerk
- **Usage Tracking**: API limit monitoring
- **Responsive Design**: Mobile-first approach

## 🔧 Technical Implementation

### **Database Schema (Prisma)**
```prisma
model UserApiLimit {
  id        String   @id @default(cuid())
  userId    String   @unique
  count     Int      @default(0)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model UserSubscription {
  id                     String    @id @default(cuid())
  userId                 String    @unique
  stripeCustomerId       String?   @unique
  stripeSubscriptionId   String?   @unique
  stripePriceId          String?
  stripeCurrentPeriodEnd DateTime?
}
```

### **API Routes Structure**
- `/api/conversation` - OpenAI chat completions
- `/api/code` - Code generation with OpenAI
- `/api/image` - Image generation via OpenAI DALL-E
- `/api/video` - Video generation via Replicate
- `/api/music` - Music generation via Replicate
- `/api/stripe` - Subscription management
- `/api/webhook` - Stripe webhook handling

### **Authentication Flow**
1. Users sign up/login via Clerk
2. Middleware protects dashboard routes
3. Public routes: landing page, webhook endpoint
4. JWT tokens manage session state

### **Subscription Management**
1. **Free Tier**: 5 API calls maximum
2. **Pro Tier**: $20/month for unlimited usage
3. Stripe handles payment processing
4. Subscription status tracked in database

## 🎨 User Interface

### **Dashboard Layout**
- **Sidebar Navigation**: Quick access to all AI tools
- **Mobile Responsive**: Collapsible sidebar for mobile
- **Usage Counter**: Shows remaining free tier credits
- **Upgrade Prompts**: Modal for subscription upgrade

### **Tool Pages Structure**
Each AI tool follows a consistent pattern:
- Form input for prompts
- Loading states during generation
- Results display with proper formatting
- Error handling with toast notifications

## 📊 Key Components

### **Core Business Logic**
- **API Limiting** (`lib/api-limit.ts`): Tracks and enforces usage limits
- **Subscription Management** (`lib/subscription.ts`): Validates Pro status
- **Database Connection** (`lib/prismadb.ts`): Prisma client configuration

### **UI Components**
- **Pro Modal**: Subscription upgrade interface
- **Free Counter**: Usage tracking display
- **Loading States**: Consistent loading indicators
- **Toast Notifications**: Error and success messages

## 🚀 Getting Started

### **Prerequisites**
- Node.js 18.x.x
- MySQL database (PlanetScale recommended)
- API keys for external services

### **Environment Variables**
```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=

# AI Service APIs
OPENAI_API_KEY=
REPLICATE_API_TOKEN=

# Database
DATABASE_URL=

# Stripe Payments
STRIPE_API_KEY=
STRIPE_WEBHOOK_SECRET=

# App Configuration
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

### **Installation Steps**
1. Clone the repository
2. Install dependencies: `npm install`
3. Set up environment variables
4. Initialize database: `npx prisma db push`
5. Start development server: `npm run dev`

## 🔒 Security Features

- **Route Protection**: Middleware-based authentication
- **API Rate Limiting**: Usage tracking per user
- **Secure Webhooks**: Stripe webhook verification
- **Environment Variables**: Sensitive data protection

## 📈 Business Model

- **Free Tier**: 5 AI generations to drive adoption
- **Pro Subscription**: $20/month for unlimited access
- **Revenue Streams**: Monthly subscriptions via Stripe
- **Customer Support**: Integrated Crisp chat

## 🛠️ Development Workflow

- **Type Safety**: Full TypeScript implementation
- **Form Validation**: Zod schemas with React Hook Form
- **Error Handling**: Comprehensive error boundaries
- **Code Quality**: ESLint configuration
- **Build Process**: Next.js optimized builds

This platform demonstrates a complete SaaS application with modern web development practices, AI integration, and robust business logic for subscription management.