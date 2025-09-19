# Quickstart Guide: Elegant Oahu Hotel Website

**Feature**: Elegant Oahu Hotel Website  
**Date**: September 18, 2025  
**Context**: Complete development setup and testing guide for React + TypeScript + Vite hotel booking application

## Prerequisites

Before starting development, ensure you have:

- **Node.js 18+**: Latest LTS version recommended
- **npm 9+** or **yarn 1.22+**: Package manager
- **Git**: Version control system
- **VS Code**: Recommended editor with extensions:
  - TypeScript and JavaScript Language Features
  - ES7+ React/Redux/React-Native snippets
  - CSS Modules extension
  - Prettier code formatter
  - ESLint integration

## Project Initialization

### 1. Create Vite React TypeScript Project

```bash
# Create new Vite project with React TypeScript template
npm create vite@latest oahu-hotel-website -- --template react-ts

# Navigate to project directory
cd oahu-hotel-website

# Install dependencies
npm install

# Install additional required dependencies
npm install react-router-dom

# Install development dependencies
npm install --save-dev @types/react-router-dom vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event eslint-plugin-jsx-a11y prettier
```

### 2. Configure TypeScript

Create or update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 3. Configure Vite

Update `vite.config.ts`:

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import { resolve } from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  css: {
    modules: {
      localsConvention: 'camelCase'
    }
  },
  build: {
    target: 'es2020',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom']
        }
      }
    }
  }
})
```

### 4. Setup Testing Configuration

Create `vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import { resolve } from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
  },
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  }
})
```

Create `src/test/setup.ts`:

```typescript
import '@testing-library/jest-dom'

// Mock localStorage for testing
const localStorageMock = {
  getItem: vi.fn(),
  setItem: vi.fn(),
  removeItem: vi.fn(),
  clear: vi.fn(),
}

Object.defineProperty(window, 'localStorage', {
  value: localStorageMock
})

// Mock window.matchMedia for responsive design tests
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
})
```

## Project Structure Setup

### 1. Create Directory Structure

```bash
# Create main source directories
mkdir -p src/components/common
mkdir -p src/components/Navigation  
mkdir -p src/components/RoomCard
mkdir -p src/components/PackageCard
mkdir -p src/components/BookingForm
mkdir -p src/components/UserAuth
mkdir -p src/pages/Home
mkdir -p src/pages/Rooms
mkdir -p src/pages/Packages
mkdir -p src/pages/Amenities
mkdir -p src/pages/Booking
mkdir -p src/pages/UserDashboard
mkdir -p src/services/mock
mkdir -p src/services/api
mkdir -p src/services/types
mkdir -p src/hooks
mkdir -p src/context
mkdir -p src/utils
mkdir -p src/types
mkdir -p src/styles
mkdir -p src/test/unit
mkdir -p src/test/integration
```

### 2. Setup Global Styles

Create `src/styles/globals.css`:

```css
/* CSS Custom Properties for Hawaiian Theme */
:root {
  /* Hawaiian Color Palette */
  --color-ocean-blue: #0077be;
  --color-ocean-light: #4da6d9;
  --color-sunset-orange: #ff6b35;
  --color-tropical-green: #2d5f3f;
  --color-sand-beige: #f5f1e8;
  --color-coral-pink: #ff9999;
  
  /* Neutral Colors */
  --color-white: #ffffff;
  --color-black: #1a1a1a;
  --color-gray-50: #fafafa;
  --color-gray-100: #f5f5f5;
  --color-gray-200: #e5e5e5;
  --color-gray-300: #d4d4d4;
  --color-gray-400: #a3a3a3;
  --color-gray-500: #737373;
  --color-gray-600: #525252;
  --color-gray-700: #404040;
  --color-gray-800: #262626;
  --color-gray-900: #171717;
  
  /* Semantic Colors */
  --color-primary: var(--color-ocean-blue);
  --color-secondary: var(--color-sunset-orange);
  --color-success: var(--color-tropical-green);
  --color-warning: var(--color-sunset-orange);
  --color-error: #dc2626;
  
  /* Typography */
  --font-family-sans: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  --font-family-serif: Georgia, 'Times New Roman', serif;
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;
  --font-size-4xl: 2.25rem;
  --font-weight-light: 300;
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  
  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-10: 2.5rem;
  --space-12: 3rem;
  --space-16: 4rem;
  --space-20: 5rem;
  
  /* Border Radius */
  --border-radius-sm: 0.25rem;
  --border-radius-md: 0.375rem;
  --border-radius-lg: 0.5rem;
  --border-radius-xl: 0.75rem;
  --border-radius-2xl: 1rem;
  --border-radius-full: 9999px;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  
  /* Transitions */
  --transition-fast: 150ms ease-in-out;
  --transition-normal: 300ms ease-in-out;
  --transition-slow: 500ms ease-in-out;
  
  /* Z-Index Scale */
  --z-index-dropdown: 1000;
  --z-index-sticky: 1020;
  --z-index-fixed: 1030;
  --z-index-modal-backdrop: 1040;
  --z-index-modal: 1050;
  --z-index-popover: 1060;
  --z-index-tooltip: 1070;
}

/* Base Styles */
* {
  box-sizing: border-box;
}

html {
  font-family: var(--font-family-sans);
  font-size: var(--font-size-base);
  line-height: 1.5;
  color: var(--color-gray-900);
  background-color: var(--color-white);
}

body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
}

/* Responsive Typography */
@media (min-width: 768px) {
  :root {
    --font-size-base: 1.125rem;
    --font-size-lg: 1.25rem;
    --font-size-xl: 1.375rem;
    --font-size-2xl: 1.625rem;
    --font-size-3xl: 2.125rem;
    --font-size-4xl: 2.625rem;
  }
}

/* Accessibility */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Focus styles for accessibility */
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* Screen reader only content */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### 3. Create Type Definitions

Create `src/types/index.ts` (copy from data-model.md entities):

```typescript
// Export all entity interfaces from data-model.md
export * from './entities/User';
export * from './entities/Room';
export * from './entities/Booking';
export * from './entities/Package';
export * from './entities/Amenity';
export * from './entities/Property';
export * from './entities/Session';

// Common utility types
export interface ServiceResponse<T> {
  success: boolean;
  data?: T;
  error?: ServiceError;
  metadata?: ResponseMetadata;
}

export interface ServiceError {
  code: string;
  message: string;
  details?: Record<string, any>;
  timestamp: Date;
  requestId: string;
}

export interface ResponseMetadata {
  requestId: string;
  timestamp: Date;
  processingTime: number;
  pagination?: PaginationInfo;
}

export interface PaginationInfo {
  page: number;
  limit: number;
  total: number;
  hasNext: boolean;
  hasPrevious: boolean;
}
```

## Core Implementation

### 1. Create Basic App Component

Update `src/App.tsx`:

```typescript
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AuthProvider } from '@/context/AuthContext';
import { BookingProvider } from '@/context/BookingContext';
import { Navigation } from '@/components/Navigation';
import { Home } from '@/pages/Home';
import { Rooms } from '@/pages/Rooms';
import { Packages } from '@/pages/Packages';
import { Amenities } from '@/pages/Amenities';
import { Booking } from '@/pages/Booking';
import { UserDashboard } from '@/pages/UserDashboard';
import './styles/globals.css';
import styles from './App.module.css';

function App() {
  return (
    <AuthProvider>
      <BookingProvider>
        <Router>
          <div className={styles.app}>
            <Navigation />
            <main className={styles.main}>
              <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/rooms" element={<Rooms />} />
                <Route path="/packages" element={<Packages />} />
                <Route path="/amenities" element={<Amenities />} />
                <Route path="/booking" element={<Booking />} />
                <Route path="/dashboard" element={<UserDashboard />} />
              </Routes>
            </main>
          </div>
        </Router>
      </BookingProvider>
    </AuthProvider>
  );
}

export default App;
```

### 2. Create Authentication Context

Create `src/context/AuthContext.tsx`:

```typescript
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import { User, Session, LoginCredentials, UserRegistrationData } from '@/types';
import { AuthenticationService } from '@/services/api/AuthenticationService';
import { MockAuthenticationService } from '@/services/mock/MockAuthenticationService';

interface AuthState {
  user: User | null;
  session: Session | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

type AuthAction =
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null }
  | { type: 'LOGIN_SUCCESS'; payload: { user: User; session: Session } }
  | { type: 'LOGOUT' }
  | { type: 'UPDATE_USER'; payload: User };

const initialState: AuthState = {
  user: null,
  session: null,
  isAuthenticated: false,
  isLoading: true,
  error: null,
};

function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload, isLoading: false };
    case 'LOGIN_SUCCESS':
      return {
        ...state,
        user: action.payload.user,
        session: action.payload.session,
        isAuthenticated: true,
        isLoading: false,
        error: null,
      };
    case 'LOGOUT':
      return {
        ...state,
        user: null,
        session: null,
        isAuthenticated: false,
        isLoading: false,
        error: null,
      };
    case 'UPDATE_USER':
      return {
        ...state,
        user: action.payload,
      };
    default:
      return state;
  }
}

interface AuthContextType {
  state: AuthState;
  login: (credentials: LoginCredentials) => Promise<void>;
  register: (userData: UserRegistrationData) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, initialState);
  const authService: AuthenticationService = new MockAuthenticationService();

  // Initialize authentication state from localStorage
  useEffect(() => {
    const initializeAuth = async () => {
      const sessionToken = localStorage.getItem('sessionToken');
      if (sessionToken) {
        try {
          const response = await authService.validateSession(sessionToken);
          if (response.success && response.data) {
            dispatch({
              type: 'LOGIN_SUCCESS',
              payload: {
                user: response.data.user,
                session: response.data.session,
              },
            });
          } else {
            localStorage.removeItem('sessionToken');
            dispatch({ type: 'SET_LOADING', payload: false });
          }
        } catch (error) {
          localStorage.removeItem('sessionToken');
          dispatch({ type: 'SET_LOADING', payload: false });
        }
      } else {
        dispatch({ type: 'SET_LOADING', payload: false });
      }
    };

    initializeAuth();
  }, []);

  const login = async (credentials: LoginCredentials) => {
    dispatch({ type: 'SET_LOADING', payload: true });
    dispatch({ type: 'SET_ERROR', payload: null });

    try {
      const response = await authService.login(credentials);
      if (response.success && response.data) {
        localStorage.setItem('sessionToken', response.data.accessToken);
        dispatch({
          type: 'LOGIN_SUCCESS',
          payload: {
            user: response.data.user,
            session: response.data.session,
          },
        });
      } else {
        dispatch({
          type: 'SET_ERROR',
          payload: response.error?.message || 'Login failed',
        });
      }
    } catch (error) {
      dispatch({
        type: 'SET_ERROR',
        payload: 'Network error. Please try again.',
      });
    }
  };

  const register = async (userData: UserRegistrationData) => {
    dispatch({ type: 'SET_LOADING', payload: true });
    dispatch({ type: 'SET_ERROR', payload: null });

    try {
      const response = await authService.register(userData);
      if (response.success && response.data) {
        localStorage.setItem('sessionToken', response.data.accessToken);
        dispatch({
          type: 'LOGIN_SUCCESS',
          payload: {
            user: response.data.user,
            session: response.data.session,
          },
        });
      } else {
        dispatch({
          type: 'SET_ERROR',
          payload: response.error?.message || 'Registration failed',
        });
      }
    } catch (error) {
      dispatch({
        type: 'SET_ERROR',
        payload: 'Network error. Please try again.',
      });
    }
  };

  const logout = async () => {
    if (state.session) {
      await authService.logout(state.session.id);
    }
    localStorage.removeItem('sessionToken');
    dispatch({ type: 'LOGOUT' });
  };

  return (
    <AuthContext.Provider value={{ state, login, register, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

## Development Workflow

### 1. Running the Development Server

```bash
# Start development server
npm run dev

# The application will be available at http://localhost:5173
```

### 2. Running Tests

```bash
# Run all tests
npm run test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage

# Run specific test file
npm run test -- RoomCard.test.tsx
```

### 3. Code Quality Commands

```bash
# Run ESLint
npm run lint

# Fix ESLint issues
npm run lint:fix

# Format code with Prettier
npm run format

# Type checking
npm run type-check
```

## Testing Strategy

### 1. Component Testing Example

Create `src/components/RoomCard/RoomCard.test.tsx`:

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { RoomCard } from './RoomCard';
import { Room } from '@/types';

const mockRoom: Room = {
  id: 'room-1',
  number: '301',
  type: 'deluxe',
  name: 'Deluxe Ocean View',
  description: 'Beautiful ocean view room with modern amenities',
  capacity: {
    maxGuests: 2,
    maxAdults: 2,
    maxChildren: 0,
    beds: [{ type: 'king', quantity: 1 }],
    hasRollawayOption: false,
  },
  // ... other required properties
};

describe('RoomCard', () => {
  it('displays room information correctly', () => {
    render(<RoomCard room={mockRoom} />);
    
    expect(screen.getByText('Deluxe Ocean View')).toBeInTheDocument();
    expect(screen.getByText(/Beautiful ocean view room/)).toBeInTheDocument();
  });

  it('calls onSelect when clicked', async () => {
    const mockOnSelect = vi.fn();
    const user = userEvent.setup();
    
    render(<RoomCard room={mockRoom} onSelect={mockOnSelect} />);
    
    await user.click(screen.getByRole('button'));
    
    expect(mockOnSelect).toHaveBeenCalledWith(mockRoom.id);
  });

  it('supports keyboard navigation', async () => {
    const mockOnSelect = vi.fn();
    const user = userEvent.setup();
    
    render(<RoomCard room={mockRoom} onSelect={mockOnSelect} />);
    
    const card = screen.getByRole('button');
    card.focus();
    await user.keyboard('{Enter}');
    
    expect(mockOnSelect).toHaveBeenCalledWith(mockRoom.id);
  });
});
```

### 2. Integration Testing Example

Create `src/test/integration/BookingFlow.test.tsx`:

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { BrowserRouter } from 'react-router-dom';
import { AuthProvider } from '@/context/AuthContext';
import { BookingProvider } from '@/context/BookingContext';
import App from '@/App';

function TestWrapper({ children }: { children: React.ReactNode }) {
  return (
    <AuthProvider>
      <BookingProvider>
        <BrowserRouter>
          {children}
        </BrowserRouter>
      </BookingProvider>
    </AuthProvider>
  );
}

describe('Booking Flow Integration', () => {
  it('allows guest to complete booking without registration', async () => {
    const user = userEvent.setup();
    render(<App />, { wrapper: TestWrapper });
    
    // Navigate to rooms page
    await user.click(screen.getByRole('link', { name: /rooms/i }));
    
    // Select a room
    await user.click(screen.getByRole('button', { name: /deluxe ocean view/i }));
    
    // Fill booking form
    await user.type(screen.getByLabelText(/first name/i), 'John');
    await user.type(screen.getByLabelText(/last name/i), 'Doe');
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    
    // Complete booking
    await user.click(screen.getByRole('button', { name: /complete booking/i }));
    
    // Verify confirmation
    expect(screen.getByText(/booking confirmed/i)).toBeInTheDocument();
  });
});
```

## Performance Monitoring

### 1. Bundle Analysis

Add to `package.json`:

```json
{
  "scripts": {
    "analyze": "vite build && npx vite-bundle-analyzer dist/stats.html"
  }
}
```

### 2. Performance Testing

Create `src/test/performance/LoadTimes.test.ts`:

```typescript
import { performance } from 'perf_hooks';

describe('Performance Tests', () => {
  it('Home page renders within performance budget', async () => {
    const startTime = performance.now();
    
    // Simulate page load
    const response = await fetch('http://localhost:5173');
    const html = await response.text();
    
    const endTime = performance.now();
    const loadTime = endTime - startTime;
    
    // Should load within 3 seconds on 3G connection
    expect(loadTime).toBeLessThan(3000);
  });
});
```

## Accessibility Testing

### 1. Add Accessibility Testing

```bash
npm install --save-dev @axe-core/playwright
```

Create `src/test/accessibility/a11y.test.ts`:

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('Homepage should be accessible', async ({ page }) => {
  await page.goto('http://localhost:5173');
  
  const accessibilityScanResults = await new AxeBuilder({ page })
    .analyze();
    
  expect(accessibilityScanResults.violations).toEqual([]);
});
```

## Deployment Preparation

### 1. Build Configuration

Add to `package.json`:

```json
{
  "scripts": {
    "build": "tsc && vite build",
    "preview": "vite preview",
    "build:analyze": "vite build --mode analyze"
  }
}
```

### 2. Environment Configuration

Create `.env.example`:

```env
VITE_API_BASE_URL=http://localhost:3001
VITE_ENABLE_MOCK_DATA=true
VITE_HOTEL_NAME="Oahu Paradise Resort"
VITE_GOOGLE_ANALYTICS_ID=GA_MEASUREMENT_ID
```

## Success Validation

### 1. Feature Completeness Checklist

- [ ] Home page displays Hawaiian-inspired design
- [ ] Room browsing with date selection works
- [ ] Package selection and compatibility checking
- [ ] Amenity information display
- [ ] User registration and login flow
- [ ] Booking creation for both guests and users
- [ ] User profile management
- [ ] Booking history display
- [ ] Responsive design on mobile and desktop
- [ ] Accessibility compliance (WCAG 2.1 AA)

### 2. Performance Validation

```bash
# Lighthouse CI for performance testing
npm install -g @lhci/cli

# Run Lighthouse audit
lhci autorun --upload.target=temporary-public-storage
```

### 3. Constitutional Compliance Check

- [ ] Bundle size under 250KB (JavaScript) + 100KB (CSS)
- [ ] No external UI library dependencies
- [ ] Component-first architecture implemented
- [ ] Static-first design with progressive enhancement
- [ ] Accessibility compliance verified
- [ ] TypeScript strict mode enabled
- [ ] Mock data services implemented

## Next Steps

After completing this quickstart setup:

1. **Implement Core Components**: Start with navigation, room cards, and basic pages
2. **Add Mock Services**: Implement the service contracts with realistic Hawaiian hotel data
3. **Build Booking Flow**: Create the multi-step booking process with state management
4. **Add Authentication**: Implement user registration, login, and session management
5. **Style with Hawaiian Theme**: Apply the elegant Hawaiian-inspired design system
6. **Test Thoroughly**: Write comprehensive tests for all components and user flows
7. **Optimize Performance**: Implement code splitting, image optimization, and caching
8. **Validate Accessibility**: Run accessibility audits and fix any compliance issues

This quickstart guide provides a solid foundation for building the elegant Oahu hotel website with all constitutional requirements met and modern development practices followed.

**Status**: Phase 1 Quickstart Guide Complete - Ready for Agent Instructions Update