# AI SaaS Platform - Setup & Configuration Guide

## 🚀 Quick Start Guide

### **Prerequisites**
- **Node.js**: Version 18.x.x or higher
- **Database**: MySQL (PlanetScale recommended for production)
- **API Keys**: OpenAI, Replicate, Stripe, Clerk accounts

### **1. Environment Configuration**

Create `.env.local` in the `ai-saas/` directory:

```bash
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...

# Clerk Redirect URLs
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

# AI Service APIs
OPENAI_API_KEY=sk-...
REPLICATE_API_TOKEN=r8_...

# Database
DATABASE_URL="mysql://username:password@host:3306/database_name"

# Stripe Payments
STRIPE_API_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# App Configuration
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

### **2. Installation Steps**

```bash
# Clone the repository
git clone https://github.com/your-username/AI-SaaS-Content-Generation-Platform.git

# Navigate to the project
cd AI-SaaS-Content-Generation-Platform/ai-saas

# Install dependencies
npm install

# Set up the database
npx prisma db push

# Generate Prisma client
npx prisma generate

# Start development server
npm run dev
```

## 🔧 Service Setup Instructions

### **Clerk Authentication Setup**

1. **Create Clerk Account**: Visit [clerk.com](https://clerk.com)
2. **Create New Application**:
   - Choose authentication methods (email, Google, GitHub, etc.)
   - Copy API keys to environment file
3. **Configure Redirect URLs**:
   - Sign-in URL: `/sign-in`
   - Sign-up URL: `/sign-up`
   - After sign-in: `/dashboard`
   - After sign-up: `/dashboard`

### **Database Setup (PlanetScale)**

1. **Create PlanetScale Account**: Visit [planetscale.com](https://planetscale.com)
2. **Create Database**:
   ```bash
   # Install PlanetScale CLI
   brew install planetscale/tap/pscale
   
   # Authenticate
   pscale auth login
   
   # Create database
   pscale database create ai-saas-platform
   
   # Create branch
   pscale branch create ai-saas-platform dev
   
   # Get connection string
   pscale connect ai-saas-platform dev --port 3309
   ```
3. **Update DATABASE_URL** with PlanetScale connection string

### **OpenAI API Setup**

1. **Create OpenAI Account**: Visit [platform.openai.com](https://platform.openai.com)
2. **Generate API Key**:
   - Go to API Keys section
   - Create new secret key
   - Copy to `OPENAI_API_KEY` in environment

### **Replicate AI Setup**

1. **Create Replicate Account**: Visit [replicate.com](https://replicate.com)
2. **Get API Token**:
   - Go to Account settings
   - Generate API token
   - Copy to `REPLICATE_API_TOKEN` in environment

### **Stripe Setup**

1. **Create Stripe Account**: Visit [stripe.com](https://stripe.com)
2. **Get API Keys**:
   - Go to Developers > API keys
   - Copy secret key to `STRIPE_API_KEY`
3. **Set Up Webhooks**:
   - Go to Developers > Webhooks
   - Add endpoint: `https://yourdomain.com/api/webhook`
   - Select events: `checkout.session.completed`, `invoice.payment_succeeded`
   - Copy signing secret to `STRIPE_WEBHOOK_SECRET`

## 📁 Project Structure Explained

```
ai-saas/
├── app/                          # Next.js 13 App Router
│   ├── (auth)/                   # Authentication pages
│   │   ├── sign-in/[[...sign-in]]/page.tsx
│   │   └── sign-up/[[...sign-up]]/page.tsx
│   ├── (dashboard)/              # Protected dashboard routes
│   │   ├── layout.tsx            # Dashboard layout with sidebar
│   │   └── (routes)/             # Individual tool pages
│   │       ├── dashboard/page.tsx
│   │       ├── conversation/page.tsx
│   │       ├── code/page.tsx
│   │       ├── image/page.tsx
│   │       ├── video/page.tsx
│   │       ├── music/page.tsx
│   │       └── settings/page.tsx
│   ├── (landing)/page.tsx        # Public landing page
│   ├── api/                      # API routes
│   │   ├── conversation/route.ts # OpenAI chat API
│   │   ├── code/route.ts         # OpenAI code generation
│   │   ├── image/route.ts        # OpenAI image generation
│   │   ├── video/route.ts        # Replicate video generation
│   │   ├── music/route.ts        # Replicate music generation
│   │   ├── stripe/route.ts       # Stripe checkout
│   │   └── webhook/route.ts      # Stripe webhooks
│   ├── globals.css               # Global styles
│   └── layout.tsx                # Root layout
├── components/                   # Reusable components
│   ├── ui/                       # shadcn/ui components
│   ├── landing-hero.tsx          # Landing page hero
│   ├── sidebar.tsx               # Dashboard sidebar
│   ├── navbar.tsx                # Dashboard navbar
│   ├── pro-modal.tsx             # Subscription modal
│   ├── free-counter.tsx          # Usage counter
│   └── ...                       # Other components
├── lib/                          # Utility libraries
│   ├── prismadb.ts               # Database client
│   ├── stripe.ts                 # Stripe client
│   ├── api-limit.ts              # Usage tracking
│   ├── subscription.ts           # Subscription validation
│   └── utils.ts                  # General utilities
├── hooks/                        # Custom React hooks
│   └── use-pro-modal.ts          # Pro modal state
├── prisma/                       # Database schema
│   └── schema.prisma             # Prisma schema
├── constants.ts                  # App constants
├── middleware.ts                 # Auth middleware
├── package.json                  # Dependencies
└── tailwind.config.js           # Tailwind configuration
```

## 🔄 Development Workflow

### **Running the Application**

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Linting
npm run lint
```

### **Database Operations**

```bash
# Apply schema changes
npx prisma db push

# Generate Prisma client
npx prisma generate

# View database
npx prisma studio

# Reset database (development only)
npx prisma db push --force-reset
```

### **Common Development Tasks**

**Adding a New AI Tool:**
1. Create API route in `app/api/[tool-name]/route.ts`
2. Create page in `app/(dashboard)/(routes)/[tool-name]/page.tsx`
3. Add route to sidebar navigation
4. Update pro modal tool showcase

**Modifying Database Schema:**
1. Edit `prisma/schema.prisma`
2. Run `npx prisma db push`
3. Run `npx prisma generate`

**Testing Stripe Integration:**
1. Use Stripe test mode keys
2. Test webhook with ngrok for local development
3. Use Stripe CLI for webhook testing

## 🚨 Troubleshooting

### **Common Issues**

**Prisma Connection Issues:**
```bash
# Regenerate client
npx prisma generate

# Check database connection
npx prisma db pull
```

**Clerk Authentication Issues:**
- Verify API keys are correct
- Check redirect URLs match configuration
- Ensure middleware is properly configured

**Stripe Webhook Issues:**
- Verify webhook secret matches Stripe dashboard
- Use ngrok for local development
- Check webhook endpoint is publicly accessible

### **Environment Variables Checklist**

✅ `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` - Clerk public key  
✅ `CLERK_SECRET_KEY` - Clerk secret key  
✅ `OPENAI_API_KEY` - OpenAI API key  
✅ `REPLICATE_API_TOKEN` - Replicate API token  
✅ `DATABASE_URL` - Database connection string  
✅ `STRIPE_API_KEY` - Stripe secret key  
✅ `STRIPE_WEBHOOK_SECRET` - Stripe webhook secret  
✅ `NEXT_PUBLIC_APP_URL` - Application URL  

## 📚 Additional Resources

- **Next.js 13 Documentation**: [nextjs.org/docs](https://nextjs.org/docs)
- **Clerk Documentation**: [clerk.com/docs](https://clerk.com/docs)
- **Prisma Documentation**: [prisma.io/docs](https://prisma.io/docs)
- **Stripe Documentation**: [stripe.com/docs](https://stripe.com/docs)
- **OpenAI Documentation**: [platform.openai.com/docs](https://platform.openai.com/docs)
- **Replicate Documentation**: [replicate.com/docs](https://replicate.com/docs)