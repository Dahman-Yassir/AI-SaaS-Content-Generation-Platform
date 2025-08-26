# AI SaaS Platform - Developer Guide

## 🎯 Understanding the Application Flow

### **User Journey**
1. **Landing Page** → User discovers the platform
2. **Authentication** → Sign up/login via Clerk
3. **Dashboard** → Access to 5 AI tools
4. **Tool Usage** → Generate content with AI
5. **Upgrade Flow** → Subscribe for unlimited access

## 🔍 Code Architecture Deep Dive

### **Next.js 13 App Router Structure**

The application uses Next.js 13's new app router with route groups:

```
app/
├── (auth)/              # Authentication routes (grouped)
│   ├── sign-in/
│   └── sign-up/
├── (dashboard)/         # Protected dashboard (grouped)
│   ├── layout.tsx       # Dashboard-specific layout
│   └── (routes)/        # Nested route group
│       ├── dashboard/   # Main dashboard
│       ├── conversation/
│       ├── code/
│       ├── image/
│       ├── video/
│       ├── music/
│       └── settings/
├── (landing)/          # Public landing page
├── api/                # API routes
└── layout.tsx          # Root layout
```

### **Component Architecture**

#### **Layout Components**
- **Root Layout** (`app/layout.tsx`): Global providers and metadata
- **Dashboard Layout** (`app/(dashboard)/layout.tsx`): Sidebar + navbar for protected routes
- **Landing Layout**: Simple layout for public pages

#### **Core Business Components**

**Sidebar Navigation** (`components/sidebar.tsx`):
```typescript
const routes = [
  { label: "Dashboard", icon: LayoutDashboard, href: "/dashboard" },
  { label: "Conversation", icon: MessageSquare, href: "/conversation" },
  { label: "Image Generation", icon: ImageIcon, href: "/image" },
  // ... other AI tools
];
```

**Free Counter** (`components/free-counter.tsx`):
- Displays remaining API calls
- Triggers upgrade modal when limit reached
- Real-time usage tracking

**Pro Modal** (`components/pro-modal.tsx`):
- Subscription upgrade interface
- Stripe checkout integration
- Tool showcase with pricing

## 🔌 API Integration Patterns

### **AI Tool API Pattern**
Each AI tool follows the same structure:

```typescript
// Example: /api/conversation/route.ts
export async function POST(req: Request) {
  // 1. Authentication check
  const { userId } = auth();
  if (!userId) return unauthorized();

  // 2. Input validation
  const { messages } = await req.json();
  if (!messages) return badRequest();

  // 3. Usage limit check
  const freeTrial = await checkApiLimit();
  const isPro = await checkSubscription();
  if (!freeTrial && !isPro) return forbidden();

  // 4. AI service call
  const response = await openai.createChatCompletion({
    model: "gpt-3.5-turbo",
    messages,
  });

  // 5. Update usage counter (free users only)
  if (!isPro) await incrementApiLimit();

  return NextResponse.json(response.data);
}
```

### **Database Operations**

**API Limit Tracking** (`lib/api-limit.ts`):
```typescript
export const incrementApiLimit = async () => {
  const { userId } = auth();
  const userApiLimit = await prismadb.userApiLimit.findUnique({
    where: { userId }
  });
  
  if (userApiLimit) {
    // Update existing counter
    await prismadb.userApiLimit.update({
      where: { userId },
      data: { count: userApiLimit.count + 1 }
    });
  } else {
    // Create new counter
    await prismadb.userApiLimit.create({
      data: { userId, count: 1 }
    });
  }
};
```

**Subscription Validation** (`lib/subscription.ts`):
```typescript
export const checkSubscription = async () => {
  const { userId } = auth();
  const userSubscription = await prismadb.userSubscription.findUnique({
    where: { userId },
    select: {
      stripeSubscriptionId: true,
      stripeCurrentPeriodEnd: true,
      stripePriceId: true,
    }
  });

  const isValid = userSubscription?.stripePriceId &&
    userSubscription?.stripeCurrentPeriodEnd?.getTime()! + DAY_IN_MS > Date.now();

  return !!isValid;
};
```

## 🎨 Frontend Patterns

### **Form Handling Pattern**
Each tool page uses React Hook Form + Zod:

```typescript
// Example: Code generation form
const formSchema = z.object({
  prompt: z.string().min(1, { message: "Prompt is required" }),
});

const form = useForm<z.infer<typeof formSchema>>({
  resolver: zodResolver(formSchema),
  defaultValues: { prompt: "" },
});

const onSubmit = async (values: z.infer<typeof formSchema>) => {
  try {
    const response = await axios.post("/api/code", { 
      messages: [...messages, { role: "user", content: values.prompt }] 
    });
    setMessages(current => [...current, userMessage, response.data]);
  } catch (error: any) {
    if (error?.response?.status === 403) {
      proModal.onOpen(); // Show upgrade modal
    }
  }
};
```

### **State Management**
Uses Zustand for global state:

```typescript
// hooks/use-pro-modal.ts
interface useProModalStore {
  isOpen: boolean;
  onOpen: () => void;
  onClose: () => void;
}

export const useProModal = create<useProModalStore>((set) => ({
  isOpen: false,
  onOpen: () => set({ isOpen: true }),
  onClose: () => set({ isOpen: false }),
}));
```

## 💰 Business Logic Implementation

### **Freemium Model**
- **Free Tier**: 5 API calls total (constant in `constants.ts`)
- **Pro Tier**: Unlimited usage for $20/month
- **Usage Tracking**: Database counter per user
- **Upgrade Triggers**: Modal shows when limit exceeded

### **Stripe Integration**
```typescript
// /api/stripe/route.ts - Subscription creation
const stripeSession = await stripe.checkout.sessions.create({
  success_url: settingsUrl,
  cancel_url: settingsUrl,
  payment_method_types: ["card"],
  mode: "subscription",
  line_items: [{
    price_data: {
      currency: "USD",
      product_data: {
        name: "Genius Pro",
        description: "Unlimited AI Generations",
      },
      unit_amount: 2000, // $20.00
      recurring: { interval: "month" },
    },
    quantity: 1,
  }],
  metadata: { userId }, // Track who subscribed
});
```

## 🔐 Security Implementation

### **Authentication Middleware**
```typescript
// middleware.ts
export default authMiddleware({
  publicRoutes: ["/", "/api/webhook"], // Only landing and webhooks are public
});
```

### **API Route Protection**
Every API route checks authentication:
```typescript
const { userId } = auth();
if (!userId) {
  return new NextResponse("Unauthorized", { status: 401 });
}
```

## 🎭 UI/UX Patterns

### **Loading States**
Consistent loading indicators across all tools:
```typescript
const isLoading = form.formState.isSubmitting;

{isLoading && (
  <div className="p-8 rounded-lg w-full flex items-center justify-center bg-muted">
    <Loader />
  </div>
)}
```

### **Error Handling**
Toast notifications for user feedback:
```typescript
catch (error: any) {
  if (error?.response?.status === 403) {
    proModal.onOpen(); // Free limit reached
  } else {
    toast.error("Something went wrong.");
  }
}
```

### **Responsive Design**
Mobile-first with Tailwind:
```typescript
<div className="text-4xl sm:text-5xl md:text-6xl lg:text-7xl">
  {/* Responsive text sizing */}
</div>
```

## 🚀 Development Workflow

### **Environment Setup**
1. Clone repository
2. Install dependencies: `npm install`
3. Set up `.env.local` with required API keys
4. Initialize database: `npx prisma db push`
5. Start development: `npm run dev`

### **Adding New AI Tools**
1. Create API route in `/api/[tool-name]/route.ts`
2. Add dashboard route in `/(dashboard)/(routes)/[tool-name]/`
3. Update sidebar navigation in `components/sidebar.tsx`
4. Add tool to pro modal showcase

### **Database Changes**
1. Update `prisma/schema.prisma`
2. Generate client: `npx prisma generate`
3. Push changes: `npx prisma db push`

This architecture provides a scalable foundation for an AI-powered SaaS platform with clear separation of concerns, robust error handling, and a smooth user experience.