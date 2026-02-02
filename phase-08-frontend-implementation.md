# PHASE 8: FRONTEND IMPLEMENTATION

**The gap between design and production code is where most frontends fail.**

A perfect design in Figma is worth $0 if the implementation doesn't survive production. This phase transforms pixel-perfect mockups into resilient, accessible, performant systems that scale to millions of users without crumbling under edge cases.

---

## 8.1 PROJECT SETUP & ARCHITECTURE

### 8.1.1 Monorepo Structure (if scaling beyond single app)

**Single app can be in `/frontend`. Multiple related apps? Monorepo.**

```
apps/
├─ web/                          # Main web application
│  ├─ src/
│  │  ├─ app/
│  │  │  ├─ App.tsx
│  │  │  ├─ App.css
│  │  │  └─ App.test.tsx
│  │  │
│  │  ├─ pages/                  # Route-level components
│  │  │  ├─ HomePage.tsx
│  │  │  ├─ OrdersPage.tsx
│  │  │  └─ CheckoutPage.tsx
│  │  │
│  │  ├─ features/               # Feature-scoped modules
│  │  │  ├─ auth/
│  │  │  │  ├─ components/
│  │  │  │  │  ├─ LoginForm.tsx
│  │  │  │  │  └─ RegisterForm.tsx
│  │  │  │  ├─ hooks/
│  │  │  │  │  ├─ useLogin.ts
│  │  │  │  │  └─ useAuth.ts
│  │  │  │  ├─ store/
│  │  │  │  │  └─ authStore.ts
│  │  │  │  ├─ api/
│  │  │  │  │  └─ authApi.ts
│  │  │  │  ├─ types/
│  │  │  │  │  └─ auth.types.ts
│  │  │  │  └─ __tests__/
│  │  │  │     └─ LoginForm.test.tsx
│  │  │  │
│  │  │  ├─ orders/
│  │  │  │  ├─ components/
│  │  │  │  ├─ hooks/
│  │  │  │  ├─ store/
│  │  │  │  ├─ api/
│  │  │  │  ├─ types/
│  │  │  │  └─ __tests__/
│  │  │  │
│  │  │  └─ checkout/
│  │  │     ├─ components/
│  │  │     ├─ hooks/
│  │  │     ├─ store/
│  │  │     ├─ api/
│  │  │     ├─ types/
│  │  │     └─ __tests__/
│  │  │
│  │  ├─ components/             # Shared/reusable components
│  │  │  ├─ Button.tsx
│  │  │  ├─ Input.tsx
│  │  │  ├─ Modal.tsx
│  │  │  ├─ Header.tsx
│  │  │  ├─ Sidebar.tsx
│  │  │  ├─ Layout.tsx
│  │  │  ├─ ErrorBoundary.tsx
│  │  │  ├─ Loading.tsx
│  │  │  └─ __tests__/
│  │  │
│  │  ├─ hooks/                  # Custom hooks (global usage)
│  │  │  ├─ useApi.ts
│  │  │  ├─ useLocalStorage.ts
│  │  │  ├─ useMediaQuery.ts
│  │  │  ├─ useDebounce.ts
│  │  │  ├─ usePrevious.ts
│  │  │  └─ __tests__/
│  │  │
│  │  ├─ stores/                 # Global state (Zustand)
│  │  │  ├─ appStore.ts
│  │  │  ├─ authStore.ts
│  │  │  ├─ notificationStore.ts
│  │  │  └─ __tests__/
│  │  │
│  │  ├─ api/                    # API client & queries
│  │  │  ├─ client.ts            # Axios/fetch instance
│  │  │  ├─ queries/
│  │  │  │  ├─ orders.ts         # TanStack Query hooks
│  │  │  │  ├─ users.ts
│  │  │  │  └─ products.ts
│  │  │  ├─ mutations/
│  │  │  │  ├─ auth.ts
│  │  │  │  ├─ orders.ts
│  │  │  │  └─ users.ts
│  │  │  └─ __tests__/
│  │  │
│  │  ├─ types/                  # TypeScript types & interfaces
│  │  │  ├─ index.ts
│  │  │  ├─ auth.ts
│  │  │  ├─ orders.ts
│  │  │  ├─ api.ts
│  │  │  └─ common.ts
│  │  │
│  │  ├─ utils/                  # Utility functions
│  │  │  ├─ formatting.ts        # Date, currency, etc.
│  │  │  ├─ validation.ts        # Form validation
│  │  │  ├─ localStorage.ts      # Local storage helpers
│  │  │  ├─ errors.ts            # Error handling
│  │  │  ├─ constants.ts
│  │  │  ├─ classnames.ts        # CSS class merging
│  │  │  └─ __tests__/
│  │  │
│  │  ├─ styles/                 # Global styles & design tokens
│  │  │  ├─ variables.css        # CSS variables
│  │  │  ├─ global.css
│  │  │  ├─ reset.css
│  │  │  └─ animations.css
│  │  │
│  │  ├─ config/                 # Configuration
│  │  │  ├─ routes.tsx           # Route definitions
│  │  │  ├─ api.config.ts        # API base URLs, timeouts
│  │  │  └─ env.ts               # Environment variables
│  │  │
│  │  ├─ middleware/             # Axios/Fetch interceptors
│  │  │  ├─ auth.ts
│  │  │  ├─ errorHandler.ts
│  │  │  ├─ requestLogger.ts
│  │  │  └─ responseTransform.ts
│  │  │
│  │  ├─ main.tsx
│  │  └─ index.css
│  │
│  ├─ public/
│  │  ├─ index.html
│  │  ├─ favicon.ico
│  │  └─ manifest.json
│  │
│  ├─ .env.example
│  ├─ .env.development
│  ├─ .env.staging
│  ├─ .env.production
│  ├─ vite.config.ts
│  ├─ tsconfig.json
│  ├─ package.json
│  └─ README.md
│
├─ admin/                        # Admin panel (separate app)
│  └─ src/
│
├─ mobile/                       # React Native app (if applicable)
│  └─ src/
│
└─ shared/                        # Shared across all apps
   ├─ ui/                        # Design system components
   │  ├─ Button.tsx
   │  ├─ Input.tsx
   │  ├─ Modal.tsx
   │  └─ __tests__/
   │
   ├─ types/                     # Shared types
   │  ├─ api.ts
   │  ├─ domain.ts
   │  └─ common.ts
   │
   ├─ utils/                     # Shared utilities
   │  ├─ formatting.ts
   │  ├─ validation.ts
   │  └─ constants.ts
   │
   └─ hooks/                     # Shared hooks
      ├─ useApi.ts
      └─ useLocalStorage.ts

// package.json (monorepo with workspaces)
{
  "name": "orders-app",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "apps/web",
    "apps/admin",
    "shared"
  ],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint"
  },
  "devDependencies": {
    "turbo": "^1.10.0"
  }
}
```

**Why monorepo:**
- ✅ Shared types between frontend & backend (no duplication)
- ✅ Shared UI components across multiple apps (web, admin, mobile)
- ✅ Single deploy pipeline (all apps versioned together)
- ✅ Easier refactoring (change types in one place)

**Monorepo tools:**
- Turbo (build orchestration)
- pnpm workspaces (package management)
- TypeScript path aliases (type-safe imports)

### 8.1.2 Critical Dependencies & Version Pinning

**Choose carefully. Each dependency is a liability.**

```json
{
  "dependencies": {
    // Framework (locked to minor version)
    "react": "18.2.0",
    "react-dom": "18.2.0",
    
    // Routing (TanStack Router is modern, Vite-first)
    "@tanstack/react-router": "1.28.0",
    
    // State management
    "zustand": "4.4.0",
    
    // Server state (required for any real app)
    "@tanstack/react-query": "5.28.0",
    
    // Form handling (don't build your own)
    "react-hook-form": "7.48.0",
    "zod": "3.22.0",  // Validation
    
    // HTTP client
    "axios": "1.6.0",
    
    // UI components (headless, unstyled)
    "@headlessui/react": "1.7.0",
    "@radix-ui/react-dialog": "1.1.0",
    "@radix-ui/react-popover": "1.0.0",
    
    // Icons
    "lucide-react": "0.294.0",
    
    // Utilities
    "clsx": "2.0.0",
    "date-fns": "2.30.0",
    "lodash-es": "4.17.21",
    
    // Analytics & error tracking
    "@sentry/react": "7.82.0",
    "posthog-js": "1.130.0"
  },
  "devDependencies": {
    // Build tool (Vite only, forget Webpack)
    "vite": "5.0.0",
    "@vitejs/plugin-react": "4.2.0",
    
    // TypeScript (strict mode only)
    "typescript": "5.3.0",
    
    // Linting & formatting
    "eslint": "8.54.0",
    "@typescript-eslint/eslint-plugin": "6.13.0",
    "@typescript-eslint/parser": "6.13.0",
    "prettier": "3.1.0",
    
    // Testing
    "vitest": "1.0.0",
    "@testing-library/react": "14.1.0",
    "@testing-library/jest-dom": "6.1.0",
    "msw": "1.3.0",  // Mock Service Worker
    "playwright": "1.40.0",  // E2E testing
    
    // Styling (Tailwind only, no CSS-in-JS)
    "tailwindcss": "3.3.0",
    "autoprefixer": "10.4.0",
    "postcss": "8.4.0"
  }
}
```

**Decision: Don't use:**
- ❌ Redux (Zustand is simpler, sufficient)
- ❌ Apollo Client (Axios + TanStack Query is better)
- ❌ Styled Components (Tailwind is better)
- ❌ Next.js (unless you need SSR for SEO, usually don't)
- ❌ Webpack (Vite is 100x faster)
- ❌ Jest (Vitest is faster, better)
- ❌ Cypress (Playwright is better)
- ❌ Material-UI (Headless UI + Tailwind is better)

### 8.1.3 Vite Configuration (Build & Dev Server)

**Vite is 100x faster than Webpack. Non-negotiable.**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  
  resolve: {
    alias: {
      // Type-safe imports: import Button from '@/components/Button'
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@features': path.resolve(__dirname, './src/features'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@stores': path.resolve(__dirname, './src/stores'),
      '@api': path.resolve(__dirname, './src/api'),
      '@types': path.resolve(__dirname, './src/types'),
      '@utils': path.resolve(__dirname, './src/utils'),
    }
  },
  
  server: {
    port: 3000,
    open: true,  // Auto-open browser
    
    // Proxy API calls to backend in development
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
      '/ws': {
        target: 'ws://localhost:3001',
        ws: true,
      }
    }
  },
  
  build: {
    // Output directory
    outDir: 'dist',
    
    // Source maps in production (for Sentry error tracking)
    sourcemap: true,
    
    // Rollup optimization options
    rollupOptions: {
      output: {
        // Code splitting strategy
        manualChunks: {
          // Vendor packages: once, never changes
          'vendor-react': ['react', 'react-dom'],
          'vendor-query': ['@tanstack/react-query'],
          'vendor-router': ['@tanstack/react-router'],
          'vendor-ui': ['@headlessui/react', '@radix-ui/react-dialog'],
          
          // Feature-level code splitting
          'feature-auth': [
            'src/features/auth/components',
            'src/features/auth/hooks',
            'src/features/auth/store'
          ],
          'feature-orders': [
            'src/features/orders/components',
            'src/features/orders/hooks',
            'src/features/orders/store'
          ]
        }
      }
    },
    
    // Chunk size warnings
    chunkSizeWarningLimit: 500,  // KB
    
    // Minification
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,  // Remove console logs in production
      }
    }
  },
  
  // Environment variables
  define: {
    __DEV__: JSON.stringify(process.env.NODE_ENV === 'development'),
    __API_URL__: JSON.stringify(process.env.VITE_API_URL),
    __VERSION__: JSON.stringify(process.env.npm_package_version),
  }
});
```

```typescript
// tsconfig.json (strict mode, no compromise)
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    
    // Strict type checking (catch bugs at compile time)
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    
    // Module resolution
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    
    // Output
    "declaration": true,
    "sourceMap": true,
    "outDir": "./dist",
    
    // Path aliases
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@features/*": ["src/features/*"],
      "@hooks/*": ["src/hooks/*"],
      "@stores/*": ["src/stores/*"],
      "@api/*": ["src/api/*"],
      "@types/*": ["src/types/*"],
      "@utils/*": ["src/utils/*"]
    },
    
    // React
    "jsx": "react-jsx",
    "jsxImportSource": "react",
    
    // Other
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### 8.1.4 Environment Configuration (Not Hardcoded Secrets)

```typescript
// src/config/env.ts
// ⚠️  NEVER put secrets in frontend. Use backend API for sensitive data.

const validateEnv = () => {
  const requiredEnvVars = [
    'VITE_API_URL',
    'VITE_APP_NAME',
    'VITE_ENVIRONMENT'
  ];
  
  const missing = requiredEnvVars.filter(
    (env) => !import.meta.env[env as keyof typeof import.meta.env]
  );
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
};

validateEnv();

export const env = {
  // API
  apiUrl: import.meta.env.VITE_API_URL as string,
  apiTimeout: parseInt(import.meta.env.VITE_API_TIMEOUT ?? '30000'),
  
  // App
  appName: import.meta.env.VITE_APP_NAME as string,
  environment: import.meta.env.VITE_ENVIRONMENT as 'development' | 'staging' | 'production',
  isDev: import.meta.env.DEV,
  isProd: import.meta.env.PROD,
  
  // Feature flags
  enableAnalytics: import.meta.env.VITE_ENABLE_ANALYTICS === 'true',
  enableErrorTracking: import.meta.env.VITE_ENABLE_ERROR_TRACKING === 'true',
  enableDebugTools: import.meta.env.VITE_ENABLE_DEBUG_TOOLS === 'true',
  
  // Version
  version: import.meta.env.VITE_VERSION as string,
} as const;
```

```
// .env.development
VITE_API_URL=http://localhost:3001/api
VITE_API_TIMEOUT=30000
VITE_APP_NAME=Orders App (Dev)
VITE_ENVIRONMENT=development
VITE_ENABLE_ANALYTICS=false
VITE_ENABLE_ERROR_TRACKING=false
VITE_ENABLE_DEBUG_TOOLS=true
VITE_VERSION=0.1.0-dev

// .env.staging
VITE_API_URL=https://api-staging.company.com/api
VITE_API_TIMEOUT=30000
VITE_APP_NAME=Orders App (Staging)
VITE_ENVIRONMENT=staging
VITE_ENABLE_ANALYTICS=true
VITE_ENABLE_ERROR_TRACKING=true
VITE_ENABLE_DEBUG_TOOLS=false
VITE_VERSION=0.1.0-staging

// .env.production
VITE_API_URL=https://api.company.com/api
VITE_API_TIMEOUT=60000
VITE_APP_NAME=Orders App
VITE_ENVIRONMENT=production
VITE_ENABLE_ANALYTICS=true
VITE_ENABLE_ERROR_TRACKING=true
VITE_ENABLE_DEBUG_TOOLS=false
VITE_VERSION=0.1.0
```

### 8.1.5 Routing Setup (TanStack Router)

**TanStack Router is modern, type-safe, and Vite-first.**

```typescript
// src/config/routes.tsx
import { RootRoute, Route, createRouter, RouterProvider } from '@tanstack/react-router';
import { Layout } from '@/components/Layout';
import { HomePage } from '@/pages/HomePage';
import { LoginPage } from '@/features/auth/pages/LoginPage';
import { OrdersListPage } from '@/features/orders/pages/OrdersListPage';
import { OrderDetailPage } from '@/features/orders/pages/OrderDetailPage';
import { CheckoutPage } from '@/features/checkout/pages/CheckoutPage';
import { NotFoundPage } from '@/pages/NotFoundPage';
import { ProtectedRoute } from '@/components/ProtectedRoute';

// Root route (layout)
const rootRoute = new RootRoute({
  component: Layout,
  beforeLoad: async ({ context }) => {
    // Check if user is authenticated before any route loads
    try {
      const user = await context.authService.getCurrentUser();
      return { user };
    } catch {
      return { user: null };
    }
  },
});

// Public routes
const homeRoute = new Route({
  getParentRoute: () => rootRoute,
  path: '/',
  component: HomePage,
  beforeLoad: ({ context }) => {
    // Optional: redirect to /orders if already logged in
    if (context.user) {
      throw redirect({ to: '/orders' });
    }
  },
});

const loginRoute = new Route({
  getParentRoute: () => rootRoute,
  path: '/auth/login',
  component: LoginPage,
});

// Protected routes
const ordersRoute = new Route({
  getParentRoute: () => rootRoute,
  path: '/orders',
  component: ProtectedRoute,
  beforeLoad: ({ context }) => {
    if (!context.user) {
      throw redirect({ to: '/auth/login' });
    }
  },
  children: [
    new Route({
      getParentRoute: () => ordersRoute,
      path: '/',
      component: OrdersListPage,
    }),
    new Route({
      getParentRoute: () => ordersRoute,
      path: '$orderId',
      component: OrderDetailPage,
      // Parse orderId from URL
      parseParams: ({ orderId }) => ({ orderId: String(orderId) }),
    }),
  ],
});

const checkoutRoute = new Route({
  getParentRoute: () => rootRoute,
  path: '/checkout',
  component: CheckoutPage,
  beforeLoad: ({ context }) => {
    if (!context.user) {
      throw redirect({ to: '/auth/login', search: { from: '/checkout' } });
    }
  },
});

// Catch-all 404 route
const notFoundRoute = new Route({
  getParentRoute: () => rootRoute,
  path: '*',
  component: NotFoundPage,
});

// Route tree
const routeTree = rootRoute.addChildren([
  homeRoute,
  loginRoute,
  ordersRoute,
  checkoutRoute,
  notFoundRoute,
]);

// Create router
export const router = createRouter({
  routeTree,
  defaultPreloadDelay: 50,  // Preload routes on hover
  defaultErrorComponent: ErrorBoundary,
  defaultNotFoundComponent: NotFoundPage,
});

// Router provider component
export function RouterProvider() {
  return <RouterProvider router={router} />;
}
```

### 8.1.6 API Client Setup (Axios with Interceptors)

```typescript
// src/api/client.ts
import axios, { AxiosInstance, InternalAxiosRequestConfig, AxiosResponse, AxiosError } from 'axios';
import { env } from '@/config/env';
import { useAppStore } from '@/stores/appStore';

// Create Axios instance
const apiClient: AxiosInstance = axios.create({
  baseURL: env.apiUrl,
  timeout: env.apiTimeout,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor: Add authentication token
apiClient.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const token = localStorage.getItem('auth_token');
    
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // Log request in development
    if (env.isDev) {
      console.log(`[API] ${config.method?.toUpperCase()} ${config.url}`, {
        params: config.params,
        data: config.data,
      });
    }
    
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor: Handle errors, refresh tokens
apiClient.interceptors.response.use(
  (response: AxiosResponse) => {
    // Log response in development
    if (env.isDev) {
      console.log(`[API] Response ${response.status}`, response.data);
    }
    
    return response;
  },
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & { _retry?: boolean };
    
    // Handle 401 Unauthorized (token expired)
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        // Try to refresh token
        const response = await axios.post(`${env.apiUrl}/auth/refresh`, {
          refreshToken: localStorage.getItem('refresh_token'),
        });
        
        const { token } = response.data;
        localStorage.setItem('auth_token', token);
        
        // Retry original request with new token
        originalRequest.headers.Authorization = `Bearer ${token}`;
        return apiClient(originalRequest);
      } catch {
        // Refresh failed, redirect to login
        useAppStore.getState().logout();
        window.location.href = '/auth/login';
        return Promise.reject(error);
      }
    }
    
    // Handle 403 Forbidden (insufficient permissions)
    if (error.response?.status === 403) {
      useAppStore.getState().addNotification({
        type: 'error',
        message: 'You do not have permission to perform this action',
      });
    }
    
    // Handle 5xx Server errors
    if (error.response?.status && error.response.status >= 500) {
      useAppStore.getState().addNotification({
        type: 'error',
        message: 'Server error. Please try again later.',
      });
    }
    
    // Log error in development
    if (env.isDev) {
      console.error('[API] Error', {
        status: error.response?.status,
        message: error.message,
        data: error.response?.data,
      });
    }
    
    return Promise.reject(error);
  }
);

export default apiClient;
```

---

## 8.2 COMPONENT LIBRARY IMPLEMENTATION

### 8.2.1 Base Components (Button, Input, Modal)

**Every app uses the same primitives. Design once, use everywhere.**

```typescript
// src/components/Button.tsx
import { forwardRef, ButtonHTMLAttributes } from 'react';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/utils/classnames';

// CVA = Class Variance Authority (type-safe style variants)
const buttonVariants = cva(
  // Base styles
  'inline-flex items-center justify-center font-medium rounded-md transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700 focus-visible:ring-blue-500',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300 focus-visible:ring-gray-500',
        danger: 'bg-red-600 text-white hover:bg-red-700 focus-visible:ring-red-500',
        outline: 'border-2 border-gray-300 text-gray-700 hover:bg-gray-50 focus-visible:ring-gray-500',
        ghost: 'text-gray-700 hover:bg-gray-100 focus-visible:ring-gray-500',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4 text-base',
        lg: 'h-12 px-6 text-lg',
      },
      fullWidth: {
        true: 'w-full',
        false: '',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

interface ButtonProps
  extends ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
  icon?: React.ReactNode;
  iconPosition?: 'left' | 'right';
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({
    className,
    variant,
    size,
    fullWidth,
    isLoading,
    icon,
    iconPosition = 'left',
    children,
    disabled,
    ...props
  }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size, fullWidth }), className)}
        disabled={disabled || isLoading}
        {...props}
      >
        {isLoading && (
          <svg className="animate-spin -ml-1 mr-2 h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" />
            <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
          </svg>
        )}
        {icon && iconPosition === 'left' && !isLoading && <span className="mr-2">{icon}</span>}
        {children}
        {icon && iconPosition === 'right' && !isLoading && <span className="ml-2">{icon}</span>}
      </button>
    );
  }
);

Button.displayName = 'Button';

// Usage:
// <Button variant="primary" size="md">Save</Button>
// <Button variant="danger" size="lg" fullWidth>Delete</Button>
// <Button isLoading>Processing...</Button>
```

```typescript
// src/components/Input.tsx
import { forwardRef, InputHTMLAttributes } from 'react';
import { cn } from '@/utils/classnames';

interface InputProps extends InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
  helperText?: string;
  required?: boolean;
  icon?: React.ReactNode;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({
    className,
    label,
    error,
    helperText,
    required,
    icon,
    type,
    ...props
  }, ref) => {
    const id = props.id || `input-${Math.random()}`;
    const errorId = error ? `${id}-error` : undefined;
    
    return (
      <div className="w-full">
        {label && (
          <label
            htmlFor={id}
            className="block text-sm font-medium text-gray-700 mb-2"
          >
            {label}
            {required && <span className="text-red-500 ml-1">*</span>}
          </label>
        )}
        
        <div className="relative">
          <input
            ref={ref}
            id={id}
            type={type || 'text'}
            className={cn(
              'w-full px-4 py-2 border-2 rounded-md transition-colors focus:outline-none focus:border-blue-500',
              error ? 'border-red-500 focus:border-red-500' : 'border-gray-300',
              icon && 'pl-10',
              className
            )}
            aria-describedby={errorId || helperText ? `${id}-help` : undefined}
            {...props}
          />
          
          {icon && (
            <div className="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-500">
              {icon}
            </div>
          )}
        </div>
        
        {error && (
          <span id={errorId} className="text-sm text-red-500 mt-1 block">
            {error}
          </span>
        )}
        
        {helperText && !error && (
          <span id={`${id}-help`} className="text-sm text-gray-500 mt-1 block">
            {helperText}
          </span>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input';
```

```typescript
// src/components/Modal.tsx
import { useEffect, useRef, ReactNode } from 'react';
import { createPortal } from 'react-dom';
import { cn } from '@/utils/classnames';
import { Button } from './Button';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title?: string;
  children: ReactNode;
  actions?: {
    primary?: { label: string; onClick: () => void; loading?: boolean };
    secondary?: { label: string; onClick: () => void };
  };
  size?: 'sm' | 'md' | 'lg';
}

export function Modal({
  isOpen,
  onClose,
  title,
  children,
  actions,
  size = 'md',
}: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (!isOpen) return;
    
    // Trap focus inside modal
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };
    
    document.addEventListener('keydown', handleKeyDown);
    document.body.style.overflow = 'hidden';
    
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      document.body.style.overflow = 'unset';
    };
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  const sizeClasses = {
    sm: 'max-w-sm',
    md: 'max-w-md',
    lg: 'max-w-lg',
  };
  
  return createPortal(
    <div
      className="fixed inset-0 z-50 bg-black bg-opacity-50 flex items-center justify-center p-4"
      onClick={(e) => {
        if (e.target === e.currentTarget) {
          onClose();
        }
      }}
    >
      <div
        ref={modalRef}
        className={cn(
          'bg-white rounded-lg shadow-lg max-h-[90vh] overflow-y-auto',
          sizeClasses[size]
        )}
      >
        {/* Header */}
        {title && (
          <div className="border-b px-6 py-4 flex items-center justify-between">
            <h2 className="text-lg font-semibold">{title}</h2>
            <button
              onClick={onClose}
              className="text-gray-500 hover:text-gray-700"
              aria-label="Close modal"
            >
              ✕
            </button>
          </div>
        )}
        
        {/* Content */}
        <div className="px-6 py-4">{children}</div>
        
        {/* Footer */}
        {actions && (
          <div className="border-t px-6 py-4 flex gap-2 justify-end">
            {actions.secondary && (
              <Button
                variant="secondary"
                onClick={actions.secondary.onClick}
              >
                {actions.secondary.label}
              </Button>
            )}
            {actions.primary && (
              <Button
                variant="primary"
                onClick={actions.primary.onClick}
                isLoading={actions.primary.loading}
              >
                {actions.primary.label}
              </Button>
            )}
          </div>
        )}
      </div>
    </div>,
    document.body
  );
}
```

### 8.2.2 Form Components (Error Display, Validation)

```typescript
// src/components/FormField.tsx
import { ReactNode } from 'react';
import { FieldValues, FieldPath, Controller, ControllerProps } from 'react-hook-form';
import { Input } from './Input';

interface FormFieldProps<
  TFieldValues extends FieldValues = FieldValues,
  TName extends FieldPath<TFieldValues> = FieldPath<TFieldValues>
> extends Omit<ControllerProps<TFieldValues, TName>, 'render'> {
  label?: string;
  placeholder?: string;
  helperText?: string;
  type?: string;
  icon?: ReactNode;
  required?: boolean;
}

export function FormField<
  TFieldValues extends FieldValues = FieldValues,
  TName extends FieldPath<TFieldValues> = FieldPath<TFieldValues>
>({
  label,
  placeholder,
  helperText,
  type,
  icon,
  required,
  ...props
}: FormFieldProps<TFieldValues, TName>) {
  return (
    <Controller
      {...props}
      render={({ field, fieldState: { error } }) => (
        <Input
          {...field}
          label={label}
          placeholder={placeholder}
          helperText={helperText}
          type={type}
          icon={icon}
          required={required}
          error={error?.message}
        />
      )}
    />
  );
}

// Usage:
// <FormField
//   control={control}
//   name="email"
//   label="Email"
//   rules={{ required: 'Email is required', pattern: { value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/, message: 'Invalid email' } }}
// />
```

### 8.2.3 Compound Components (Complex Interactions)

```typescript
// src/components/DataTable.tsx (complex compound component)
import { ReactNode, useState } from 'react';
import { ChevronUp, ChevronDown } from 'lucide-react';

interface Column<T> {
  key: keyof T;
  label: string;
  sortable?: boolean;
  width?: string;
  render?: (value: T[keyof T], row: T) => ReactNode;
}

interface DataTableProps<T extends { id: string }> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (row: T) => void;
  isLoading?: boolean;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  onRowClick,
  isLoading,
}: DataTableProps<T>) {
  const [sortKey, setSortKey] = useState<keyof T | null>(null);
  const [sortOrder, setSortOrder] = useState<'asc' | 'desc'>('asc');
  
  const handleSort = (key: keyof T) => {
    if (sortKey === key) {
      setSortOrder(sortOrder === 'asc' ? 'desc' : 'asc');
    } else {
      setSortKey(key);
      setSortOrder('asc');
    }
  };
  
  const sortedData = [...data].sort((a, b) => {
    if (!sortKey) return 0;
    
    const aValue = a[sortKey];
    const bValue = b[sortKey];
    
    if (aValue < bValue) return sortOrder === 'asc' ? -1 : 1;
    if (aValue > bValue) return sortOrder === 'asc' ? 1 : -1;
    return 0;
  });
  
  return (
    <div className="overflow-x-auto">
      <table className="w-full border-collapse">
        <thead>
          <tr className="border-b-2 border-gray-200">
            {columns.map((column) => (
              <th
                key={String(column.key)}
                className="px-4 py-3 text-left font-semibold text-gray-700 text-sm"
                style={{ width: column.width }}
              >
                <button
                  onClick={() => column.sortable && handleSort(column.key)}
                  className="flex items-center gap-2 cursor-pointer hover:text-gray-900"
                  disabled={!column.sortable}
                >
                  {column.label}
                  {column.sortable && sortKey === column.key && (
                    sortOrder === 'asc' ? <ChevronUp size={16} /> : <ChevronDown size={16} />
                  )}
                </button>
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {isLoading ? (
            <tr>
              <td colSpan={columns.length} className="px-4 py-8 text-center text-gray-500">
                Loading...
              </td>
            </tr>
          ) : sortedData.length === 0 ? (
            <tr>
              <td colSpan={columns.length} className="px-4 py-8 text-center text-gray-500">
                No data found
              </td>
            </tr>
          ) : (
            sortedData.map((row) => (
              <tr
                key={row.id}
                onClick={() => onRowClick?.(row)}
                className="border-b border-gray-100 hover:bg-gray-50 transition-colors cursor-pointer"
              >
                {columns.map((column) => (
                  <td key={String(column.key)} className="px-4 py-3 text-sm text-gray-700">
                    {column.render
                      ? column.render(row[column.key], row)
                      : String(row[column.key])}
                  </td>
                ))}
              </tr>
            ))
          )}
        </tbody>
      </table>
    </div>
  );
}
```

---

## 8.3 FEATURE IMPLEMENTATION

### 8.3.1 Authentication Flow (Complete)

```typescript
// src/features/auth/types/auth.types.ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: 'user' | 'admin';
  createdAt: string;
}

export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  token: string;
  refreshToken: string;
  user: User;
}

export interface RegisterRequest {
  email: string;
  password: string;
  name: string;
}

export interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  error: string | null;
}

// src/features/auth/store/authStore.ts
import { create } from 'zustand';
import { AuthState, User } from '../types/auth.types';
import { authApi } from '../api/authApi';

export const useAuthStore = create<AuthState & {
  login: (email: string, password: string) => Promise<void>;
  register: (email: string, password: string, name: string) => Promise<void>;
  logout: () => void;
  clearError: () => void;
  setUser: (user: User | null) => void;
}>((set) => ({
  user: null,
  token: localStorage.getItem('auth_token'),
  isLoading: false,
  error: null,
  
  login: async (email, password) => {
    set({ isLoading: true, error: null });
    try {
      const response = await authApi.login(email, password);
      
      localStorage.setItem('auth_token', response.token);
      localStorage.setItem('refresh_token', response.refreshToken);
      
      set({
        user: response.user,
        token: response.token,
        isLoading: false,
      });
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : 'Login failed',
        isLoading: false,
      });
      throw error;
    }
  },
  
  register: async (email, password, name) => {
    set({ isLoading: true, error: null });
    try {
      const response = await authApi.register(email, password, name);
      
      localStorage.setItem('auth_token', response.token);
      localStorage.setItem('refresh_token', response.refreshToken);
      
      set({
        user: response.user,
        token: response.token,
        isLoading: false,
      });
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : 'Registration failed',
        isLoading: false,
      });
      throw error;
    }
  },
  
  logout: () => {
    localStorage.removeItem('auth_token');
    localStorage.removeItem('refresh_token');
    set({
      user: null,
      token: null,
      error: null,
    });
  },
  
  clearError: () => set({ error: null }),
  setUser: (user) => set({ user }),
}));

// src/features/auth/api/authApi.ts
import apiClient from '@/api/client';
import { LoginRequest, LoginResponse, RegisterRequest } from '../types/auth.types';

export const authApi = {
  login: async (email: string, password: string): Promise<LoginResponse> => {
    const response = await apiClient.post('/auth/login', {
      email,
      password,
    } as LoginRequest);
    return response.data;
  },
  
  register: async (email: string, password: string, name: string): Promise<LoginResponse> => {
    const response = await apiClient.post('/auth/register', {
      email,
      password,
      name,
    } as RegisterRequest);
    return response.data;
  },
  
  getCurrentUser: async () => {
    const response = await apiClient.get('/auth/me');
    return response.data;
  },
};

// src/features/auth/hooks/useLogin.ts
import { useCallback } from 'react';
import { useNavigate } from '@tanstack/react-router';
import { useAuthStore } from '../store/authStore';

export function useLogin() {
  const navigate = useNavigate();
  const { login, error, isLoading, clearError } = useAuthStore();
  
  const handleLogin = useCallback(
    async (email: string, password: string) => {
      try {
        await login(email, password);
        navigate({ to: '/orders' });
      } catch (error) {
        // Error is already set in store
        console.error('Login failed:', error);
      }
    },
    [login, navigate]
  );
  
  return {
    login: handleLogin,
    error,
    isLoading,
    clearError,
  };
}

// src/features/auth/pages/LoginPage.tsx
import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/Button';
import { FormField } from '@/components/FormField';
import { useLogin } from '../hooks/useLogin';

const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type LoginFormData = z.infer<typeof loginSchema>;

export function LoginPage() {
  const { control, handleSubmit, setError } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });
  
  const { login, isLoading, error } = useLogin();
  
  const onSubmit = async (data: LoginFormData) => {
    try {
      await login(data.email, data.password);
    } catch (err) {
      setError('email', {
        type: 'manual',
        message: error || 'Login failed. Please try again.',
      });
    }
  };
  
  return (
    <div className="max-w-md mx-auto mt-10">
      <h1 className="text-2xl font-bold mb-6">Login</h1>
      
      <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={control}
          name="email"
          label="Email"
          type="email"
          placeholder="user@example.com"
          required
        />
        
        <FormField
          control={control}
          name="password"
          label="Password"
          type="password"
          placeholder="••••••••"
          required
        />
        
        <Button
          type="submit"
          fullWidth
          isLoading={isLoading}
        >
          Login
        </Button>
      </form>
    </div>
  );
}
```

### 8.3.2 Data Fetching (TanStack Query Integration)

```typescript
// src/features/orders/api/ordersApi.ts
import apiClient from '@/api/client';
import { Order, OrdersListResponse, CreateOrderRequest } from '../types/orders.types';

export const ordersApi = {
  list: async (params: { page?: number; limit?: number; status?: string }): Promise<OrdersListResponse> => {
    const response = await apiClient.get('/orders', { params });
    return response.data;
  },
  
  getById: async (id: string): Promise<Order> => {
    const response = await apiClient.get(`/orders/${id}`);
    return response.data;
  },
  
  create: async (data: CreateOrderRequest): Promise<Order> => {
    const response = await apiClient.post('/orders', data);
    return response.data;
  },
  
  update: async (id: string, data: Partial<Order>): Promise<Order> => {
    const response = await apiClient.patch(`/orders/${id}`, data);
    return response.data;
  },
  
  delete: async (id: string): Promise<void> => {
    await apiClient.delete(`/orders/${id}`);
  },
};

// src/features/orders/hooks/useOrders.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useSearchParams } from '@tanstack/react-router';
import { ordersApi } from '../api/ordersApi';
import { useAppStore } from '@/stores/appStore';

export function useOrders() {
  const [searchParams] = useSearchParams();
  
  return useQuery({
    queryKey: [
      'orders',
      {
        page: searchParams.get('page') || '1',
        limit: searchParams.get('limit') || '20',
        status: searchParams.get('status'),
      },
    ],
    queryFn: () =>
      ordersApi.list({
        page: parseInt(searchParams.get('page') || '1'),
        limit: parseInt(searchParams.get('limit') || '20'),
        status: searchParams.get('status') || undefined,
      }),
    staleTime: 5 * 60 * 1000,  // 5 minutes
    gcTime: 10 * 60 * 1000,     // 10 minutes
  });
}

export function useOrderDetail(orderId: string) {
  return useQuery({
    queryKey: ['orders', orderId],
    queryFn: () => ordersApi.getById(orderId),
    staleTime: 5 * 60 * 1000,
    gcTime: 10 * 60 * 1000,
  });
}

export function useCreateOrder() {
  const queryClient = useQueryClient();
  const { addNotification } = useAppStore();
  
  return useMutation({
    mutationFn: (data: CreateOrderRequest) => ordersApi.create(data),
    onSuccess: (newOrder) => {
      // Update cache optimistically
      queryClient.setQueryData(['orders'], (old: any) => ({
        ...old,
        data: [newOrder, ...old?.data || []],
      }));
      
      addNotification({
        type: 'success',
        message: `Order ${newOrder.id} created successfully`,
      });
    },
    onError: () => {
      addNotification({
        type: 'error',
        message: 'Failed to create order',
      });
    },
  });
}
```

---

## 8.4 TESTING STRATEGY

### 8.4.1 Unit Tests (Components)

```typescript
// src/components/__tests__/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '../Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    
    await userEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledOnce();
  });
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled</Button>);
    expect(screen.getByText('Disabled')).toBeDisabled();
  });
  
  it('shows loading state', () => {
    const { container } = render(<Button isLoading>Loading</Button>);
    expect(container.querySelector('svg')).toBeInTheDocument();
  });
  
  it('applies variant styles', () => {
    const { container } = render(<Button variant="danger">Delete</Button>);
    const button = screen.getByText('Delete');
    expect(button).toHaveClass('bg-red-600');
  });
  
  it('is accessible with keyboard', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    
    const button = screen.getByText('Click');
    button.focus();
    
    await userEvent.keyboard('{Enter}');
    expect(handleClick).toHaveBeenCalled();
  });
});
```

### 8.4.2 Integration Tests (Features)

```typescript
// src/features/auth/__tests__/LoginPage.integration.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { setupServer } from 'msw';
import { http, HttpResponse } from 'msw';
import { LoginPage } from '../pages/LoginPage';

const server = setupServer(
  http.post('/api/auth/login', ({ request }) => {
    const body = request.json();
    
    if (body.email === 'user@example.com' && body.password === 'password123') {
      return HttpResponse.json({
        token: 'mock-token',
        refreshToken: 'mock-refresh',
        user: {
          id: '1',
          email: 'user@example.com',
          name: 'Test User',
          role: 'user',
          createdAt: new Date().toISOString(),
        },
      });
    }
    
    return HttpResponse.json(
      { error: 'Invalid credentials' },
      { status: 401 }
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('LoginPage', () => {
  const renderWithProviders = (component: React.ReactElement) => {
    const queryClient = new QueryClient({
      defaultOptions: {
        queries: { retry: false },
        mutations: { retry: false },
      },
    });
    
    return render(
      <QueryClientProvider client={queryClient}>
        {component}
      </QueryClientProvider>
    );
  };
  
  it('renders login form', () => {
    renderWithProviders(<LoginPage />);
    
    expect(screen.getByLabelText('Email')).toBeInTheDocument();
    expect(screen.getByLabelText('Password')).toBeInTheDocument();
    expect(screen.getByText('Login')).toBeInTheDocument();
  });
  
  it('submits form with valid credentials', async () => {
    renderWithProviders(<LoginPage />);
    
    const emailInput = screen.getByLabelText('Email') as HTMLInputElement;
    const passwordInput = screen.getByLabelText('Password') as HTMLInputElement;
    const submitButton = screen.getByText('Login');
    
    await userEvent.type(emailInput, 'user@example.com');
    await userEvent.type(passwordInput, 'password123');
    await userEvent.click(submitButton);
    
    await waitFor(() => {
      expect(localStorage.getItem('auth_token')).toBe('mock-token');
    });
  });
  
  it('displays error message on invalid credentials', async () => {
    renderWithProviders(<LoginPage />);
    
    const emailInput = screen.getByLabelText('Email');
    const passwordInput = screen.getByLabelText('Password');
    const submitButton = screen.getByText('Login');
    
    await userEvent.type(emailInput, 'user@example.com');
    await userEvent.type(passwordInput, 'wrongpassword');
    await userEvent.click(submitButton);
    
    await waitFor(() => {
      expect(screen.getByText(/login failed/i)).toBeInTheDocument();
    });
  });
});
```

### 8.4.3 E2E Tests (Critical Flows)

```typescript
// e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('http://localhost:3000/auth/login');
    await page.fill('input[type="email"]', 'test@example.com');
    await page.fill('input[type="password"]', 'password123');
    await page.click('button:has-text("Login")');
    await page.waitForURL('/orders');
  });
  
  test('user can complete checkout', async ({ page }) => {
    // Navigate to orders
    await page.goto('http://localhost:3000/orders');
    
    // Click first order
    await page.click('[data-testid="order-card"]:first-child');
    await page.waitForURL(/\/orders\/\w+/);
    
    // Verify order details
    const orderNumber = await page.locator('[data-testid="order-number"]').textContent();
    expect(orderNumber).toBeTruthy();
    
    // Click checkout button
    await page.click('button:has-text("Proceed to Checkout")');
    await page.waitForURL('/checkout');
    
    // Fill shipping address
    await page.fill('input[name="street"]', '123 Main St');
    await page.fill('input[name="city"]', 'New York');
    await page.fill('input[name="state"]', 'NY');
    await page.fill('input[name="zip"]', '10001');
    
    // Continue to payment
    await page.click('button:has-text("Continue")');
    
    // Fill payment (use test card)
    await page.fill('input[name="cardNumber"]', '4242 4242 4242 4242');
    await page.fill('input[name="expiry"]', '12/25');
    await page.fill('input[name="cvc"]', '123');
    
    // Place order
    await page.click('button:has-text("Place Order")');
    
    // Verify confirmation
    await page.waitForURL('/checkout/confirmation');
    const confirmation = await page.locator('[data-testid="confirmation-message"]');
    await expect(confirmation).toContainText('Order placed successfully');
  });
});
```

---

## 8.5 PERFORMANCE OPTIMIZATION

### 8.5.1 Code Splitting & Lazy Loading

```typescript
// src/config/routes.tsx (already shown above with lazy loading)
// Key: Each page is lazy-loaded, not in initial bundle

// Verify with:
// npm run build
// ls -lh dist/
// Should see: chunk-orders.js, chunk-checkout.js, etc.

// Monitor bundle size:
// npm install -D @vite-plugin/visualizer
```

```typescript
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true })  // Opens bundle analysis on build
  ],
});
```

### 8.5.2 Image Optimization

```typescript
// src/components/OptimizedImage.tsx
import { useState, useEffect, useRef } from 'react';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  priority?: boolean;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
}: OptimizedImageProps) {
  const [loaded, setLoaded] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);
  
  useEffect(() => {
    if (priority) {
      setLoaded(true);
      return;
    }
    
    // Lazy load images below the fold
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setLoaded(true);
        observer.disconnect();
      }
    });
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [priority]);
  
  return (
    <picture>
      {/* WebP format for modern browsers */}
      <source
        srcSet={`${src}?q=75&w=${width || 500}&f=webp`}
        type="image/webp"
      />
      
      {/* Fallback to JPEG */}
      <img
        ref={imgRef}
        src={loaded ? `${src}?q=75&w=${width || 500}` : undefined}
        alt={alt}
        width={width}
        height={height}
        loading={priority ? 'eager' : 'lazy'}
        onLoad={() => setLoaded(true)}
        className="max-w-full h-auto"
      />
    </picture>
  );
}
```

---

## 8.6 ERROR HANDLING & MONITORING

### 8.6.1 Error Boundary

```typescript
// src/components/ErrorBoundary.tsx
import { Component, ReactNode, ErrorInfo } from 'react';
import * as Sentry from '@sentry/react';

interface ErrorBoundaryProps {
  children: ReactNode;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundaryComponent extends Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log to Sentry
    Sentry.captureException(error, { contexts: { react: errorInfo } });
    
    // Log to console in development
    if (import.meta.env.DEV) {
      console.error('Error caught by boundary:', error, errorInfo);
    }
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="min-h-screen flex items-center justify-center bg-gray-50">
          <div className="max-w-md w-full">
            <h1 className="text-2xl font-bold text-red-600 mb-4">
              Something went wrong
            </h1>
            <p className="text-gray-600 mb-6">
              We're sorry for the inconvenience. Please try refreshing the page.
            </p>
            {import.meta.env.DEV && this.state.error && (
              <pre className="bg-gray-100 p-4 rounded mb-4 text-xs overflow-auto">
                {this.state.error.message}
              </pre>
            )}
            <button
              onClick={() => window.location.reload()}
              className="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700"
            >
              Refresh Page
            </button>
          </div>
        </div>
      );
    }
    
    return this.props.children;
  }
}

export const ErrorBoundary = Sentry.withErrorBoundary(
  ErrorBoundaryComponent,
  {
    fallback: <div>An error has occurred</div>,
    showDialog: false,
  }
);
```

### 8.6.2 Sentry Integration (Error Tracking)

```typescript
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import * as Sentry from '@sentry/react';
import { env } from '@/config/env';
import App from './App';

// Initialize Sentry
if (env.enableErrorTracking) {
  Sentry.init({
    dsn: import.meta.env.VITE_SENTRY_DSN,
    environment: env.environment,
    tracesSampleRate: env.isProd ? 0.1 : 1.0,  // 10% in prod, 100% in dev
    release: env.version,
    integrations: [
      new Sentry.Replay({
        maskAllText: true,  // Privacy
        blockAllMedia: true,
      }),
      new Sentry.Feedback(),  // Allow users to report issues
    ],
  });
}

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <Sentry.ErrorBoundary fallback={<div>An error occurred</div>}>
      <App />
    </Sentry.ErrorBoundary>
  </React.StrictMode>
);
```

---

## 8.7 DELIVERABLES & QUALITY GATES

### 8.7.1 Pre-Deployment Checklist

**Code Quality:**
- [ ] ESLint: 0 errors, 0 warnings (`npm run lint`)
- [ ] TypeScript: Strict mode passing (`npm run type-check`)
- [ ] Prettier: All files formatted (`npm run format`)
- [ ] No console.log statements (use logger)
- [ ] No commented code (delete or use version control)

**Testing:**
- [ ] Unit tests: >= 70% coverage (`npm run test -- --coverage`)
- [ ] Integration tests: All critical features tested
- [ ] E2E tests: Critical user flows passing (`npm run test:e2e`)
- [ ] All tests passing locally: `npm run test`
- [ ] No test warnings or skipped tests

**Performance:**
- [ ] Bundle size < 200KB gzip (run `npm run build && npm run analyze`)
- [ ] Lighthouse score >= 90 (run locally or use CI)
- [ ] LCP < 2.5s, FID < 100ms, CLS < 0.1 (check with DevTools)
- [ ] No console errors or warnings on page load
- [ ] Images optimized (WebP, responsive, lazy-loaded)

**Accessibility:**
- [ ] WCAG 2.1 AA compliant (use WAVE extension)
- [ ] Color contrast >= 4.5:1 (use WebAIM checker)
- [ ] All interactive elements keyboard navigable
- [ ] Focus indicators visible
- [ ] Screen reader tested (NVDA, JAWS, or VoiceOver)
- [ ] No semantic HTML errors

**Security:**
- [ ] No hardcoded secrets or API keys
- [ ] Dependencies audited: `npm audit` (no critical)
- [ ] CORS configured correctly
- [ ] HTTPS enforced in production
- [ ] CSP headers in place
- [ ] Input sanitized (especially forms)

**Documentation:**
- [ ] Storybook stories for all components (`npm run storybook`)
- [ ] README with setup instructions
- [ ] API documentation (Swagger/OpenAPI)
- [ ] Environment variables documented (.env.example)
- [ ] Architecture decision records (ADR) documented

**Deployment:**
- [ ] Build succeeds: `npm run build`
- [ ] No build warnings
- [ ] Environment-specific configs tested (dev/staging/prod)
- [ ] Deployment pipeline tested (CI/CD)
- [ ] Rollback plan documented
- [ ] Health check endpoint working

### 8.7.2 Code Review Approval Gate

**PR Template:**
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issue
Closes #123

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed
- [ ] Test coverage >= 70%

## Performance Impact
- [ ] No bundle size increase
- [ ] No runtime performance regression
- [ ] Images optimized

## Accessibility
- [ ] WCAG 2.1 AA compliant
- [ ] Keyboard navigable
- [ ] Screen reader tested

## Screenshots (if UI change)
[Include before/after screenshots]

## Checklist
- [ ] Code follows project style guide
- [ ] No console warnings or errors
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
```

**Review Criteria:**
1. ✅ Code quality (no console logs, proper error handling)
2. ✅ Test coverage (>= 70%)
3. ✅ No performance regression
4. ✅ Accessibility compliance
5. ✅ No security issues
6. ✅ Documentation complete

**Required Approvals:**
- 1x Code review (senior dev)
- 1x Design QA (if UI changes)
- 1x Performance review (if significant changes)

---

## PRODUCTION DEPLOYMENT CHECKLIST

### Pre-Deployment

**24 hours before:**
- [ ] All tests passing
- [ ] Code reviewed and approved
- [ ] Staging environment tested thoroughly
- [ ] Rollback procedure tested
- [ ] On-call engineer briefed

**1 hour before:**
- [ ] Database backups taken
- [ ] API health check verified
- [ ] Monitoring alerts configured
- [ ] Communication channel open (Slack)

### During Deployment

**Execution:**
- [ ] Deploy to production (automated or manual)
- [ ] Verify health checks passing
- [ ] Monitor error rates (Sentry)
- [ ] Monitor performance (Datadog)
- [ ] Check user-facing functionality

**Rollback triggers:**
- Error rate > 5% for 5 minutes
- 503 Service Unavailable
- Critical functionality broken
- Database connection pool exhausted

### Post-Deployment

**First 30 minutes:**
- [ ] Monitor error rates
- [ ] Monitor performance
- [ ] Check user feedback (Twitter, support)
- [ ] Verify analytics tracking

**First 24 hours:**
- [ ] Monitor for any anomalies
- [ ] Collect user feedback
- [ ] Update status page
- [ ] Document any issues

---

## SUCCESS METRICS

**By end of Phase 8, you should have:**

✅ **Fully functional frontend**
- User authentication (login, register, logout)
- Main feature screens (orders list, detail, create)
- Responsive design (mobile, tablet, desktop)
- Accessible (WCAG 2.1 AA)

✅ **Robust error handling**
- Graceful error messages
- Automatic error tracking (Sentry)
- User-friendly error recovery

✅ **High performance**
- Bundle < 200KB gzip
- LCP < 2.5s
- Core Web Vitals passing

✅ **Comprehensive tests**
- 70%+ code coverage
- All critical flows tested (E2E)
- Accessibility tested

✅ **Production-ready**
- CI/CD pipeline passing
- Security audit passed
- Monitoring & alerting configured
- Deployment tested

---

## CRITICAL DECISIONS

**These are non-negotiable:**

1. **Type safety everywhere** - TypeScript strict mode, no `any`
2. **Tests before shipping** - 70%+ coverage minimum
3. **No hardcoded secrets** - Use environment variables, backend API
4. **Accessibility from start** - WCAG 2.1 AA, not an afterthought
5. **Monitoring in place** - Sentry, analytics, performance metrics
6. **Error boundaries** - Every major section wrapped
7. **Loading states** - Never show blank screens
8. **No N+1 queries** - Batch data fetching with TanStack Query
9. **Bundle analysis** - Know where bytes go
10. **Performance budgets** - Enforce limits in CI/CD

---

**PHASE 8 COMPLETE: Frontend is production-ready, fully tested, accessible, performant, and monitored.**

Next phase: Deployment, scaling, and operations.
