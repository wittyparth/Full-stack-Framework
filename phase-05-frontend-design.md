# Phase 5: Frontend Design

> **Why This Matters:** Frontend design is not decoration—it's architecture that determines whether your app scales to millions of users or crashes under load. A 20-year veteran knows: bad frontend architecture costs **10x more** to fix than bad backend architecture. Why? Because fixing it requires rewriting every component. Get the design right here, before a single component is built.

---

## Table of Contents

1. [UX/UI Design](#1-uxui-design)
   - [Wireframes: Information Architecture First](#11-wireframes-information-architecture-first)
   - [High-Fidelity Mockups: Design System in Action](#12-high-fidelity-mockups-design-system-in-action)
   - [Design System: Single Source of Truth](#13-design-system-single-source-of-truth)
   - [Responsive Design: Breakpoint Strategy](#14-responsive-design-breakpoint-strategy)
   - [Accessibility Audit: WCAG 2.1 AA](#15-accessibility-audit-wcag-21-aa)
2. [Component Architecture](#2-component-architecture)
   - [Component Hierarchy Diagram](#21-component-hierarchy-diagram)
   - [Atomic Design Implementation](#22-atomic-design-implementation)
   - [Component Prop Interface Design](#23-component-prop-interface-design)
   - [Component State Allocation](#24-component-state-allocation)
3. [State Management Design](#3-state-management-design)
   - [Global vs Local State Mapping](#31-global-vs-local-state-mapping)
   - [Server State Caching Strategy](#32-server-state-caching-strategy)
   - [URL State Parameters](#33-url-state-parameters)
   - [Real-Time Data Handling](#34-real-time-data-handling)
4. [Routing & Navigation](#4-routing--navigation)
   - [Route Structure Definition](#41-route-structure-definition)
   - [Protected Routes Strategy](#42-protected-routes-strategy)
   - [Deep Linking](#43-deep-linking)
   - [Navigation Flow Diagrams](#44-navigation-flow-diagrams)
   - [SEO Considerations](#45-seo-considerations)
5. [Performance Planning](#5-performance-planning)
   - [Bundle Splitting Strategy](#51-bundle-splitting-strategy)
   - [Lazy Loading Implementation](#52-lazy-loading-implementation)
   - [Image Optimization Strategy](#53-image-optimization-strategy)
   - [Caching Strategy](#54-caching-strategy)
   - [Core Web Vitals Targets](#55-core-web-vitals-targets)
6. [Design Review Gates](#6-design-review-gates)
   - [Gate 1: Design Review & Approval](#gate-1-design-review--approval)
   - [Gate 2: Accessibility Compliance Check](#gate-2-accessibility-compliance-check)
7. [Summary: What You've Designed](#summary-what-youve-designed)
8. [Critical Decisions for Engineering](#critical-decisions-for-engineering)

---

## 1. UX/UI Design

### 1.1 Wireframes: Information Architecture First

**Before pixels, define structure.** Wireframes are low-fidelity blueprints that show:

- How screens relate to each other
- What information appears where
- What actions are available
- User workflow across screens

> **Never skip wireframes** to jump to mockups. Wireframes catch information architecture problems early. Fixing info arch in Figma costs 2 hours. Fixing it after 10 components are built costs 2 weeks.

#### Example: List Orders Screen

```
┌─────────────────────────────────────────┐
│ Header / Navigation                     │ ← Logo, user menu, search
├─────────────────────────────────────────┤
│                                         │
│ FILTERS                                 │ ← Collapse/expand
│ ├─ Status: [Pending] [Shipped] [...]   │
│ ├─ Date: From [____] To [____]         │
│ ├─ Amount: Min [____] Max [____]       │
│ └─ [Apply] [Reset]                     │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│ ORDERS LIST                             │ ← Sortable table
│ ─────────────────────────────────────── │
│ │ ID     │ Date   │ Total   │ Status  ││
│ ─────────────────────────────────────── │
│ │ ORD123 │ Jan 15 │ $99.99  │ ▼      ││
│ │ ORD122 │ Jan 14 │ $149.99 │ ▼      ││
│ │ ORD121 │ Jan 13 │ $25.00  │ ▼      ││
│ ─────────────────────────────────────── │
│                                         │
│ Pagination: Prev [1] [2] [3] Next      │
│                                         │
└─────────────────────────────────────────┘
```

**Key Wireframe Decisions:**

1. **Filters above list** - User sees constraints before data
2. **Sortable columns** - Click "ID" to sort by order ID
3. **Pagination** - Not infinite scroll (user controls load, easier testing)
4. **Status dropdown inline** - Not separate "view details" modal (faster)
5. **One scrollable list** - Not tabbed sections (simpler UX)

#### Example: Checkout Flow

```
SCREEN 1: Cart Review
┌──────────────────────────┐
│ Items in cart:           │
│ ├─ Laptop  $1299.99 qty:1│
│ ├─ Mouse   $25.00   qty:2│
│ └─ Total:  $1349.99      │
│                          │
│ [Edit Cart] [Proceed]    │
└──────────────────────────┘
         ↓
SCREEN 2: Shipping Address
┌──────────────────────────┐
│ Shipping Address         │
│ Street: [___________]    │
│ City: [____] State: [__] │
│ Zip: [_____]             │
│                          │
│ [Back] [Next]            │
└──────────────────────────┘
         ↓
SCREEN 3: Payment
┌──────────────────────────┐
│ Card Number: [________]  │
│ Exp: [__/__] CVC: [___]  │
│                          │
│ [Back] [Place Order]     │
└──────────────────────────┘
         ↓
SCREEN 4: Confirmation
┌──────────────────────────┐
│ ✓ Order #ORD999          │
│ Confirmation sent to:    │
│ user@example.com         │
│                          │
│ [View Order] [Continue]  │
└──────────────────────────┘
```

#### Information Architecture Rule

**User should answer 3 questions immediately on any screen:**

1. **Where am I?** (page title, breadcrumbs)
2. **What can I do?** (action buttons visible)
3. **What happens next?** (clear navigation path)

---

### 1.2 High-Fidelity Mockups: Design System in Action

After wireframes approve information architecture, create hi-fi mockups showing:

- Exact colors, typography, spacing
- Real content (not Lorem ipsum)
- Micro-interactions (hover states, loading, errors)
- Dark/light modes if applicable
- All screen sizes (mobile, tablet, desktop)

#### Desktop Mockup: Orders List Screen

```
┌─────────────────────────────────────────────────────────┐
│ HEADER                                                  │
│ Logo | Orders | Dashboard | Account ▼ | Logout         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ SIDEBAR (sticky on scroll)                              │
│ ├─ Dashboard                                            │
│ ├─ Orders (active - darker)                             │
│ ├─ Inventory                                            │
│ ├─ Customers                                            │
│ └─ Analytics                                            │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ MAIN CONTENT                                            │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ My Orders                    [Search]  [+]          │ │
│ ├─────────────────────────────────────────────────────┤ │
│ │                                                     │ │
│ │ FILTERS (collapsible, shows 3-4 most recent)       │ │
│ │ ┌──────────────────────────────────────────────┐   │ │
│ │ │ Status: [All ▼] Date: [Last 30 days ▼]      │   │ │
│ │ │ [Apply Filters] [Clear All]                  │   │ │
│ │ └──────────────────────────────────────────────┘   │ │
│ │                                                     │ │
│ │ ORDER LIST (with column headers)                    │ │
│ │ ┌──────────────────────────────────────────────┐   │ │
│ │ │ ORDER # | DATE ▼ | TOTAL ▼ | STATUS | SHIP  │   │ │
│ │ ├──────────────────────────────────────────────┤   │ │
│ │ │ #ORD-999│ Jan 15 │$1,299.99│Shipped ✓│  ►   │   │ │
│ │ │ #ORD-998│ Jan 14 │  $149.99│Processing│  ►   │   │ │
│ │ │ #ORD-997│ Jan 13 │   $25.00│Pending   │  ►   │   │ │
│ │ │ #ORD-996│ Jan 12 │   $89.99│Cancelled │  ►   │   │ │
│ │ ├──────────────────────────────────────────────┤   │ │
│ │ │ Showing 1-4 of 15,847 orders                 │   │ │
│ │ │ Prev [1] [2] [3] ... [100] Next              │   │ │
│ │ └──────────────────────────────────────────────┘   │ │
│ │                                                     │ │
│ └─────────────────────────────────────────────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Mobile Mockup: Responsive (Stack Vertically)

```
┌─────────────────┐
│ ☰ Orders        │ ← Hamburger menu, search
├─────────────────┤
│ Filters  < >    │ ← Slide-in drawer
├─────────────────┤
│                 │
│ ORD-999         │ ← Card-based layout
│ Jan 15 | $1299  │
│ Shipped ►       │ ← Action icon
│                 │
│ ORD-998         │
│ Jan 14 | $149   │
│ Processing ►    │
│                 │
│ ORD-997         │
│ Jan 13 | $25    │
│ Pending ►       │
│                 │
│ ← Prev 1 2 3    │ ← Simplified pagination
│      Next →     │
│                 │
└─────────────────┘
```

**Design Decisions in Mockups:**

1. **Card-based on mobile** - Not tabular (columns don't fit)
2. **Filters drawer** - Not taking space on mobile
3. **Sortable columns** - Only on desktop (mobile: filter instead)
4. **Status indicator color** - Shipped (green), Processing (yellow), Pending (gray)
5. **Whitespace** - 16px margins on mobile, 24px on desktop

---

### 1.3 Design System: Single Source of Truth

**Every color, font, spacing, button style defined once, reused everywhere.**

#### Design System Structure

```
Design System (Figma file)
├─ Colors
│  ├─ Primary: #0284c7 (teal blue)
│  ├─ Success: #22c55e (green)
│  ├─ Warning: #f59e0b (amber)
│  ├─ Danger: #ef4444 (red)
│  ├─ Gray: #64748b (text), #e2e8f0 (background)
│  └─ Dark mode variants (same colors, adjusted opacity)
│
├─ Typography
│  ├─ Heading 1: 32px, 600 weight, line-height 1.2
│  ├─ Heading 2: 24px, 600 weight, line-height 1.3
│  ├─ Body: 16px, 400 weight, line-height 1.5
│  ├─ Small: 14px, 400 weight, line-height 1.4
│  └─ Mono: 12px, 400 weight, font-family: monospace
│
├─ Spacing
│  ├─ xs: 4px
│  ├─ sm: 8px
│  ├─ md: 16px
│  ├─ lg: 24px
│  ├─ xl: 32px
│  └─ 2xl: 48px
│
├─ Components
│  ├─ Button
│  │  ├─ Primary (blue, full width on mobile)
│  │  ├─ Secondary (gray outline)
│  │  ├─ Danger (red, for delete)
│  │  ├─ States: default, hover, active, disabled, loading
│  │  └─ Sizes: sm (12px padding), md (16px), lg (20px)
│  │
│  ├─ Input
│  │  ├─ Text input (16px, light gray border)
│  │  ├─ Label (required star red, optional gray)
│  │  ├─ Error state (red border, red error text)
│  │  ├─ Placeholder (light gray)
│  │  └─ Focus state (blue border, blue outline)
│  │
│  ├─ Card
│  │  ├─ White background, light gray border
│  │  ├─ 8px border-radius
│  │  ├─ 16px padding
│  │  ├─ Subtle shadow
│  │  └─ Hover: shadow increases
│  │
│  ├─ Modal
│  │  ├─ Centered on screen
│  │  ├─ Dark overlay (black, 50% opacity)
│  │  ├─ Max width 600px
│  │  ├─ Escape key closes
│  │  └─ Focus trapped inside (accessibility)
│  │
│  ├─ Dropdown
│  │  ├─ Trigger shows selected value + chevron
│  │  ├─ List appears below (or above if space limited)
│  │  ├─ Keyboard navigation (arrow keys, enter, escape)
│  │  └─ Type to filter options
│  │
│  └─ Badge
│     ├─ Inline status indicator
│     ├─ Colors: green (success), yellow (warning), red (danger), gray (default)
│     └─ Small text, pill-shaped
│
├─ Interactions
│  ├─ Loading: Spinner or skeleton (not blank)
│  ├─ Error: Red border, red error message below
│  ├─ Success: Green checkmark, green toast notification
│  ├─ Hover: Color shift +10% darker, slight shadow
│  └─ Focus: Blue outline, 2px width
│
└─ Accessibility
   ├─ Color contrast: 4.5:1 minimum for text
   ├─ Focus indicators: Always visible (no outline: none)
   ├─ Button sizes: Minimum 44x44px (touch targets)
   ├─ Icons with labels: Every icon has aria-label
   └─ Keyboard navigation: Tab through all interactive elements
```

#### Design System in Code (CSS Variables)

```css
/* Colors */
--color-primary: #0284c7;
--color-success: #22c55e;
--color-warning: #f59e0b;
--color-danger: #ef4444;
--color-text-primary: #1f2937;
--color-text-secondary: #6b7280;
--color-background: #ffffff;
--color-border: #e5e7eb;

/* Typography */
--font-family-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-family-mono: 'Menlo', 'Monaco', monospace;
--font-size-h1: 2rem;
--font-size-h2: 1.5rem;
--font-size-body: 1rem;
--font-size-small: 0.875rem;
--line-height-tight: 1.2;
--line-height-normal: 1.5;

/* Spacing */
--space-xs: 4px;
--space-sm: 8px;
--space-md: 16px;
--space-lg: 24px;
--space-xl: 32px;

/* Components */
--button-padding: var(--space-md) var(--space-lg);
--input-padding: var(--space-sm) var(--space-md);
--border-radius: 6px;
--shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
--shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
```

#### Component Implementation Example

```javascript
// Button.jsx
export function Button({ variant = 'primary', size = 'md', children, ...props }) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      {...props}
    >
      {children}
    </button>
  );
}

/* CSS */
.btn {
  font-family: var(--font-family-base);
  border-radius: var(--border-radius);
  border: none;
  cursor: pointer;
  transition: all 150ms ease;
  font-weight: 500;
}

.btn-primary {
  background: var(--color-primary);
  color: white;
}

.btn-primary:hover {
  background: #0c7daf; /* 10% darker */
  box-shadow: var(--shadow-md);
}

.btn-primary:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.btn-md {
  padding: var(--button-padding);
  font-size: var(--font-size-body);
}
```

> **Figma to Code Synchronization:** Use design tokens in Figma. Export as JSON. Use in CSS. Ensures design + code never drift.

---

### 1.4 Responsive Design: Breakpoint Strategy

**Mobile-first approach:** Design smallest screen first, add complexity for larger.

#### Breakpoint Definitions

```
Mobile:  0px - 639px   (phones)
Tablet:  640px - 1023px (iPad, small laptop)
Desktop: 1024px+        (desktop, large screens)
```

**Specific Breakpoints (if needed):**
- 320px (old iPhones)
- 375px (iPhone)
- 768px (iPad)
- 1024px (laptop)
- 1440px (desktop)
- 1920px (large monitor)

#### Layout Decisions by Breakpoint

**Mobile (320px):**
```
├─ Single column (100% width)
├─ Sidebar hidden (hamburger menu)
├─ Filters collapsed/drawer
├─ Large touch targets (44x44px minimum)
├─ Font size 16px (prevents zoom on iOS)
└─ Full-bleed content (edge-to-edge)
```

**Tablet (768px):**
```
├─ Two columns (sidebar + content)
├─ Sidebar visible but narrower (200px)
├─ Filters visible, less condensed
├─ Moderate touch targets (40x40px)
└─ Content max-width: 900px
```

**Desktop (1024px):**
```
├─ Three columns (sidebar + content + right panel)
├─ Sidebar fixed, always visible
├─ Filters expanded with more options
├─ Content max-width: 1200px
└─ Generous whitespace
```

#### CSS for Responsive Design

```css
/* Mobile-first: Base styles for mobile */
.container {
  width: 100%;
  padding: var(--space-md);
}

.header {
  flex-direction: column; /* Stack vertically */
}

.sidebar {
  display: none; /* Hidden on mobile */
}

/* Tablet and up */
@media (min-width: 640px) {
  .sidebar {
    display: block;
    width: 200px;
  }
  
  .container {
    display: grid;
    grid-template-columns: 200px 1fr;
    gap: var(--space-lg);
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    grid-template-columns: 200px 1fr 300px;
  }
  
  .content {
    max-width: 1200px;
  }
}
```

#### Test All Breakpoints

- iPhone SE (375px)
- iPhone 12 (390px)
- iPad (768px)
- iPad Pro (1024px)
- Laptop (1440px)
- Monitor (1920px)

---

### 1.5 Accessibility Audit: WCAG 2.1 AA

**Accessibility is not a nice-to-have. It's a requirement.** Accessible design benefits everyone.

#### Color Contrast Audit

**Ratio requirement:** 4.5:1 for normal text, 3:1 for large text (18px+, bold)

**Examples:**
- ✓ Black (#000) on white (#fff) = 21:1 (excellent)
- ✓ Teal (#0284c7) on white = 5.5:1 (good)
- ✗ Light gray (#d1d5db) on white = 3.2:1 (fails for normal text)
- ✓ Light gray on white = 3.2:1 (OK for 18px+ text)

**Test with:** WebAIM Contrast Checker, WAVE browser extension

#### Interactive Elements

**Do:**
- ✓ All buttons, links, form inputs have visible focus state
- ✓ Minimum size: 44x44px for touch targets
- ✓ All buttons have accessible labels (aria-label or visible text)
- ✓ Form labels associated with inputs via `<label for="id">`

**Don't:**
- ✗ `outline: none` (removes focus indicator, breaks keyboard navigation)
- ✗ 20px buttons (too small, hard to tap)
- ✗ ▼ (icon with no label)
- ✗ Placeholder alone as label (disappears when typing)

#### Semantic HTML

**Do:**
```html
<button>Click me</button>
<label for="email">Email:</label>
<input id="email" type="email">
<h1>Page Title</h1>
<h2>Section</h2>
<h3>Subsection</h3>
<nav>, <main>, <section>, <article>, <aside>
```

**Don't:**
```html
<div class="button">Click me</div> <!-- Not keyboard accessible -->
<div class="nav">, <div class="main"> <!-- No semantics -->
<h1>Title</h1>
<h3>Another Title</h3> <!-- Skips hierarchy -->
```

#### Keyboard Navigation

- **Tab order = reading order** (left to right, top to bottom)
- ✓ All interactive elements reachable via Tab
- ✓ Escape key closes modals
- ✓ Arrow keys navigate dropdowns
- ✓ Enter key submits forms
- ✗ Click-only interactions (no keyboard alternative)
- ✗ Focus trap in modal (can't Tab out, but can Alt-Tab away)

#### Testing Checklist

- [ ] Zoom to 200% - layout still usable?
- [ ] Remove colors - can you still understand content?
- [ ] Screen reader test - at least one interactive section
- [ ] Keyboard only - navigate entire page, no mouse
- [ ] All images have alt text
- [ ] Form errors clearly shown (color + text, not color alone)
- [ ] Skip link at top of page (jump to main content)

---

## 2. Component Architecture

### 2.1 Component Hierarchy Diagram

**Components are building blocks.** Good hierarchy = reusable, maintainable, scalable.

```
App (root)
├─ Layout
│  ├─ Header
│  │  ├─ Logo
│  │  ├─ Navigation
│  │  │  └─ NavLink (reused 5x)
│  │  └─ UserMenu
│  │     ├─ Avatar
│  │     └─ Dropdown
│  │        └─ DropdownItem (reused 3x)
│  │
│  └─ Sidebar
│     └─ NavLink (reused 4x)
│
├─ Pages
│  ├─ OrdersPage
│  │  ├─ FilterPanel
│  │  │  ├─ Select (reused 3x)
│  │  │  ├─ DateRangeInput
│  │  │  │  └─ Input (reused 2x)
│  │  │  └─ Button
│  │  │
│  │  └─ OrdersTable
│  │     ├─ TableHeader
│  │     │  └─ SortableColumn (reused 4x)
│  │     │
│  │     └─ TableBody
│  │        └─ OrderRow (reused for each order)
│  │           ├─ StatusBadge
│  │           └─ ActionButton
│  │
│  └─ OrderDetailPage
│     ├─ OrderInfo
│     │  ├─ InfoRow (reused 5x)
│     │  └─ Button
│     │
│     ├─ OrderItems
│     │  └─ OrderItemCard (reused for each item)
│     │
│     └─ ShippingInfo
│        └─ Map (third-party)
│
└─ Common Components (reused across pages)
   ├─ Button (primary, secondary, danger)
   ├─ Input (text, email, password, number)
   ├─ Select
   ├─ Modal
   ├─ Toast (notifications)
   ├─ Skeleton (loading placeholder)
   └─ ErrorBoundary
```

#### Reusability Analysis

**Highly reusable (core components):**
```
├─ Button (30+ uses across app)
├─ Input (20+ uses)
├─ Select (10+ uses)
├─ Modal (6+ uses)
└─ Badge (15+ uses)
```

**Medium reusability (feature components):**
```
├─ OrderCard (appears in Orders, Dashboard, recent-orders)
├─ UserAvatar (header, comments, activity)
└─ PriceDisplay (order list, item cards, totals)
```

**Low reusability (page-specific):**
```
├─ OrderDetailHeader (only on order detail page)
├─ CheckoutForm (only on checkout)
└─ AnalyticsChart (only on dashboard)
```

---

### 2.2 Atomic Design Implementation

**Every component classified by size and responsibility:**

#### ATOMS (smallest, no dependencies)
```
├─ Button
├─ Input
├─ Label
├─ Badge
├─ Icon
└─ Avatar
```

#### MOLECULES (combine atoms, do one thing)
```
├─ FormField (Label + Input + Error)
├─ SearchBox (Input + Icon + clear button)
├─ Card (container for content)
├─ LoadingSpinner (Icon + text)
└─ Toast (Icon + Message + close button)
```

#### ORGANISMS (multiple molecules, more complex)
```
├─ Header (Logo + Navigation + UserMenu)
├─ FilterPanel (multiple FormFields + button)
├─ OrderCard (multiple badges, prices, buttons)
├─ Modal (Header + content + footer buttons)
└─ DataTable (header + rows + pagination)
```

#### TEMPLATES (layout + placeholders)
```
├─ PageLayout (Header + Sidebar + main)
├─ FormTemplate (form fields + buttons)
├─ ListTemplate (filters + list + pagination)
└─ DetailTemplate (header + sections + actions)
```

#### PAGES (real content + real data)
```
├─ OrdersPage (OrdersTemplate + real orders)
├─ OrderDetailPage (DetailTemplate + real order)
├─ CheckoutPage (FormTemplate + real cart)
└─ DashboardPage (various organisms + real data)
```

#### Benefits by Level

**Atoms:**
- Reusable everywhere
- No external dependencies
- Tested in isolation
- Style in design system

**Molecules:**
- Combine atoms into patterns
- Single responsibility
- Reusable in forms, pages, modals
- Self-contained logic

**Organisms:**
- Feature-level components
- May contain state + API calls
- Used in multiple pages
- Complex, but well-defined

**Pages:**
- Real app experiences
- Combine organisms into workflows
- Connect to data/state
- Integration tests here

---

### 2.3 Component Prop Interface Design

**Components are contracts.** Props define what data they accept, what they render.

#### Example: Button Component

```typescript
// Button.tsx
interface ButtonProps {
  // Content
  children: React.ReactNode;              // Button text/icon
  
  // Styling
  variant?: 'primary' | 'secondary' | 'danger';  // Color scheme
  size?: 'sm' | 'md' | 'lg';             // Padding/font size
  fullWidth?: boolean;                    // Stretch to 100% width
  
  // Behavior
  onClick?: () => void;                   // Click handler
  disabled?: boolean;                     // Disabled state
  loading?: boolean;                      // Show spinner during action
  type?: 'button' | 'submit' | 'reset';  // Form interaction
  
  // Accessibility
  ariaLabel?: string;                     // Label for screen readers
  ariaDescribedBy?: string;              // Error message ID
  
  // HTML attributes
  className?: string;                     // Custom CSS classes
  title?: string;                        // Hover tooltip
}

export function Button({
  children,
  variant = 'primary',
  size = 'md',
  fullWidth = false,
  onClick,
  disabled = false,
  loading = false,
  type = 'button',
  ariaLabel,
  ...props
}: ButtonProps) {
  return (
    <button
      type={type}
      onClick={onClick}
      disabled={disabled || loading}
      aria-label={ariaLabel}
      className={`btn btn-${variant} btn-${size} ${fullWidth ? 'w-full' : ''}`}
      {...props}
    >
      {loading ? <Spinner /> : children}
    </button>
  );
}

// Usage:
<Button onClick={handleSave}>Save</Button>
<Button variant="danger" loading={isDeleting}>Delete</Button>
<Button variant="primary" size="lg" fullWidth>Checkout</Button>
```

#### Example: Input Component

```typescript
interface InputProps {
  // Content
  value: string;
  placeholder?: string;
  
  // Type
  type?: 'text' | 'email' | 'password' | 'number' | 'date';
  
  // State
  disabled?: boolean;
  readOnly?: boolean;
  error?: string;                        // Error message triggers red border
  
  // Behavior
  onChange: (value: string) => void;
  onBlur?: () => void;
  onFocus?: () => void;
  
  // Form integration
  name?: string;
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: string;
  
  // Accessibility
  label?: string;                        // Associated label text
  ariaLabel?: string;
  ariaDescribedBy?: string;             // Connect to error message
  
  // Styling
  size?: 'sm' | 'md' | 'lg';
  fullWidth?: boolean;
}

export function Input({
  value,
  placeholder,
  type = 'text',
  disabled = false,
  error,
  onChange,
  label,
  required,
  size = 'md',
  ...props
}: InputProps) {
  const inputId = `input-${Math.random()}`;
  const errorId = error ? `${inputId}-error` : undefined;
  
  return (
    <div>
      {label && (
        <label htmlFor={inputId}>
          {label}
          {required && <span className="text-red-500">*</span>}
        </label>
      )}
      
      <input
        id={inputId}
        type={type}
        value={value}
        placeholder={placeholder}
        onChange={(e) => onChange(e.target.value)}
        disabled={disabled}
        aria-describedby={errorId}
        className={`input input-${size} ${error ? 'input-error' : ''}`}
        {...props}
      />
      
      {error && (
        <p id={errorId} className="text-red-500 text-sm mt-1">
          {error}
        </p>
      )}
    </div>
  );
}

// Usage:
<Input
  label="Email"
  type="email"
  value={email}
  onChange={setEmail}
  required
  error={emailError}
/>
```

#### Example: OrderCard Component

```typescript
interface OrderCardProps {
  order: {
    id: string;
    number: string;
    createdAt: string;
    total: number;
    status: 'pending' | 'processing' | 'shipped' | 'delivered' | 'cancelled';
    itemCount: number;
  };
  onClick?: (orderId: string) => void;
  onCancel?: (orderId: string) => void;
  isLoading?: boolean;
}

export function OrderCard({
  order,
  onClick,
  onCancel,
  isLoading
}: OrderCardProps) {
  const statusColor = {
    pending: 'yellow',
    processing: 'blue',
    shipped: 'green',
    delivered: 'green',
    cancelled: 'gray'
  };
  
  return (
    <Card
      onClick={() => onClick?.(order.id)}
      className="cursor-pointer hover:shadow-md"
    >
      <h3 className="text-lg font-bold">{order.number}</h3>
      <p className="text-gray-500">
        {format(new Date(order.createdAt), 'MMM dd, yyyy')}
      </p>
      
      <Badge color={statusColor[order.status]}>
        {capitalize(order.status)}
      </Badge>
      
      <div className="mt-4 flex justify-between items-center">
        <div>
          <p className="text-2xl font-bold">${order.total.toFixed(2)}</p>
          <p className="text-gray-500 text-sm">{order.itemCount} items</p>
        </div>
        
        {order.status === 'pending' && (
          <Button
            size="sm"
            variant="danger"
            onClick={(e) => {
              e.stopPropagation();
              onCancel?.(order.id);
            }}
            loading={isLoading}
          >
            Cancel
          </Button>
        )}
      </div>
    </Card>
  );
}
```

#### Prop Design Rules

**1. Required vs Optional:**
- **Required:** Critical for functionality (value, onChange, title)
- **Optional:** Nice-to-have, sensible defaults (size, variant)
- **Avoid:** Too many required props (refactor as separate components)

**2. Type Safety:**
- Use discriminated unions for variants
- Use enums for choices (not strings)
- Use strict types (not 'any')

**3. Naming:**
- `on*` for callbacks (onClick, onChange, onSubmit)
- `is*` for booleans (isLoading, isDisabled, isOpen)
- `aria*` for accessibility (ariaLabel, ariaDescribedBy)
- **Avoid:** do*, handle* (use on* instead)

**4. Composability:**
- Accept children when possible
- Allow className override for custom styling
- Accept className, but use mergeClasses() (don't overwrite)
- Spread ...rest props to underlying element

**5. Avoid:**
- Props that duplicate browser HTML (use className instead)
- Props that aren't used (cleanup before shipping)
- Props that belong elsewhere (move complex logic to parent)
- Too many optional props (if > 5 optional, redesign)

---

### 2.4 Component State Allocation

**Decide: Should this be local state or global state?**

#### Decision Tree

```
Does multiple components need this state?
├─ NO → Local state (useState in component)
├─ YES → Continue...

Is it frequently accessed?
├─ NO → URL parameter or local state
├─ YES → Continue...

Is it form/input state?
├─ YES → Form state (React Hook Form + local state)
├─ NO → Continue...

Is it server data (user, orders, inventory)?
├─ YES → Server state (TanStack Query)
└─ NO → Global state (Zustand)

Is it UI state (modal open, sidebar collapsed)?
├─ YES → Local state or global (if used by multiple pages)
└─ NO → Re-evaluate
```

#### State Type Examples

**LOCAL STATE (useState):**
```
├─ Form input value (typing, before submit)
├─ Modal open/closed (within single component)
├─ Hover state (button highlight, dropdown open)
├─ Sorting/filtering within a single list
└─ Accordion/tab active section
```

**GLOBAL STATE (Zustand):**
```
├─ Current user (needed in header, footer, dashboard)
├─ App theme (light/dark mode)
├─ Sidebar collapsed state (shared across pages)
└─ UI notifications (toast, alerts)
```

**SERVER STATE (TanStack Query):**
```
├─ Orders (fetched from /api/orders)
├─ User profile (fetched from /api/users/me)
├─ Inventory (fetched from /api/inventory)
└─ Anything that comes from backend
```

**URL PARAMETERS:**
```
├─ Current page in list (?page=2)
├─ Filter values (?status=shipped&date=2024-01)
├─ Selected item in detail view (/:orderId)
└─ Search query (?q=laptop)
```

> **Key Principle:** Every piece of state must be in exactly one place. Choose wisely.

#### State Inventory for Orders App

| State | Type | Location | Shared? |
|-------|------|----------|---------|
| Form inputs (new order) | Local useState | No - single form |
| Modal open/closed | Local useState | No - single modal |
| Loading spinner (POST) | Local useState | No - single request |
| Sorting column | URL params | Yes - preserve on refresh |
| Filter values | URL params | Yes - shareable link |
| Orders list (from API) | TanStack Query | Yes - multiple pages |
| Current user info | Zustand global | Yes - header, sidebar, profile |
| App theme (light/dark) | Zustand global | Yes - entire app |
| Sidebar collapsed state | Zustand global | Yes - all pages |
| Toast notifications | Zustand global | Yes - show from anywhere |
| Payment form state | React Hook Form | No - local, structured validation |
| Cart items | TanStack Query | Yes - add to cart anywhere |

---

## 3. State Management Design

### 3.1 Global vs Local State Mapping

#### Example: Filter Values in URL

```typescript
// OrdersPage.tsx
import { useSearchParams } from 'react-router-dom';

export function OrdersPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  // Read from URL
  const status = searchParams.get('status') || 'all';
  const dateFrom = searchParams.get('from') || '';
  const page = parseInt(searchParams.get('page') || '1');
  
  // Update URL (navigable, shareable)
  function handleFilterChange(newStatus: string) {
    const params = new URLSearchParams(searchParams);
    params.set('status', newStatus);
    params.set('page', '1'); // Reset to page 1 on filter change
    setSearchParams(params);
  }
  
  return (
    <>
      <FilterPanel
        status={status}
        onStatusChange={handleFilterChange}
      />
      <OrdersList status={status} page={page} />
    </>
  );
}

// Result: URL is /orders?status=shipped&page=1
// User bookmarks URL, shares with colleague, works perfectly
// Refresh page: filters persist
```

#### Example: Global State (Zustand)

```typescript
// store/appStore.ts
import { create } from 'zustand';

interface AppState {
  // User
  user: User | null;
  setUser: (user: User | null) => void;
  
  // Theme
  theme: 'light' | 'dark';
  setTheme: (theme: 'light' | 'dark') => void;
  
  // UI
  sidebarCollapsed: boolean;
  toggleSidebar: () => void;
  
  // Notifications
  notifications: Toast[];
  addNotification: (toast: Toast) => void;
  removeNotification: (id: string) => void;
}

export const useAppStore = create<AppState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  
  theme: 'light',
  setTheme: (theme) => set({ theme }),
  
  sidebarCollapsed: false,
  toggleSidebar: () => set((state) => ({ 
    sidebarCollapsed: !state.sidebarCollapsed 
  })),
  
  notifications: [],
  addNotification: (toast) => set((state) => ({
    notifications: [{ ...toast, id: Date.now() }, ...state.notifications]
  })),
  removeNotification: (id) => set((state) => ({
    notifications: state.notifications.filter((n) => n.id !== id)
  }))
}));

// Usage in components:
function Header() {
  const { user } = useAppStore();
  return <div>Hello, {user?.email}</div>;
}

function ThemeToggle() {
  const { theme, setTheme } = useAppStore();
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      {theme === 'light' ? '🌙' : '☀️'}
    </button>
  );
}
```

#### Example: Server State (TanStack Query)

```typescript
// hooks/useOrders.ts
import { useQuery } from '@tanstack/react-query';

interface UseOrdersProps {
  status?: string;
  page?: number;
  limit?: number;
}

export function useOrders({ status, page = 1, limit = 20 }: UseOrdersProps = {}) {
  return useQuery({
    queryKey: ['orders', { status, page, limit }],
    queryFn: async () => {
      const params = new URLSearchParams();
      if (status) params.append('status', status);
      params.append('limit', limit.toString());
      params.append('offset', ((page - 1) * limit).toString());
      
      const response = await fetch(`/api/v1/orders?${params}`);
      if (!response.ok) throw new Error('Failed to fetch orders');
      return response.json();
    },
    staleTime: 5 * 60 * 1000,  // 5 minutes
    gcTime: 10 * 60 * 1000      // 10 minutes (formerly cacheTime)
  });
}

// Usage in component:
function OrdersList({ status }: { status: string }) {
  const { data, isLoading, error } = useOrders({ status });
  
  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <div>
      {data?.data.map(order => (
        <OrderCard key={order.id} order={order} />
      ))}
    </div>
  );
}

// Auto-refetch benefits:
// ├─ User returns to tab → Fresh data
// ├─ Filter changes → New query, old cached, no duplicate requests
// └─ Manual refresh → useQueryClient().invalidateQueries(['orders'])
```

---

### 3.2 Server State Caching Strategy

**Goal:** Minimize API calls, maximize freshness, handle offline.

#### TanStack Query Configuration

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // Data is fresh for 5 minutes
      gcTime: 10 * 60 * 1000,         // Keep in memory for 10 minutes
      retry: 1,                        // Retry failed requests 1 time
      refetchOnWindowFocus: true,      // Refetch when user returns to tab
      refetchOnMount: 'stale'          // Refetch if data is stale
    }
  }
});

// Different caches for different data:

// User profile: Rarely changes, cache 30 minutes
useQuery({
  queryKey: ['user', userId],
  queryFn: fetchUser,
  staleTime: 30 * 60 * 1000
});

// Orders list: Changes frequently, cache 1 minute
useQuery({
  queryKey: ['orders', { status, page }],
  queryFn: fetchOrders,
  staleTime: 1 * 60 * 1000
});

// Inventory: Changes constantly (sales), cache 10 seconds
useQuery({
  queryKey: ['inventory', sku],
  queryFn: fetchInventory,
  staleTime: 10 * 1000
});

// One-time data: Never refetch
useQuery({
  queryKey: ['article', articleId],
  queryFn: fetchArticle,
  staleTime: Infinity  // Article doesn't change
});
```

#### Invalidation: Update Cache Without Re-fetching

```typescript
// When order is created, invalidate orders list to refetch
const { mutate: createOrder } = useMutation({
  mutationFn: postOrder,
  onSuccess: (newOrder) => {
    // Immediately update cache (optimistic)
    queryClient.setQueryData(['orders'], (old: Order[]) => [newOrder, ...old]);
    
    // Also refetch to ensure server matches
    queryClient.invalidateQueries({ queryKey: ['orders'] });
    
    // Show success
    useAppStore.getState().addNotification({
      type: 'success',
      message: `Order ${newOrder.id} created!`
    });
  }
});
```

#### Offline Support with Background Sync

```typescript
// Problem: User creates order, loses internet, what happens?
// Solution: Queue mutation, retry when online

const { mutate: createOrder } = useMutation({
  mutationFn: postOrder,
  networkMode: 'always',  // Attempt even if offline
  onError: (error) => {
    if (!navigator.onLine) {
      // Queue for retry when online
      queryClient.setQueryData(['pendingMutations'], (old = []) => [
        ...old,
        { type: 'createOrder', payload: data }
      ]);
    }
  }
});

// Retry pending mutations when online
window.addEventListener('online', () => {
  queryClient.refetchQueries({
    queryKey: ['orders'],
    type: 'stale'
  });
});
```

---

### 3.3 URL State Parameters

**Filters, sorting, pagination belong in URL, not state.**

#### Benefits

✓ **Shareable:** User filters to shipped orders, copies URL, sends to colleague  
✓ **Bookmarkable:** User bookmarks page with filters, returns later, filters persist  
✓ **Back button:** User navigates back, filters restored (browser history)  
✓ **Refresh:** User refreshes page, filters persist (not lost)  
✓ **Deeplinked:** Share specific filtered view in email, Slack, etc.  
✓ **SEO:** Search engines see different URLs for different filtered views  

#### URL Structure for Filters

```
Base:             /orders
Single filter:    /orders?status=shipped
Multiple filters: /orders?status=shipped&date_from=2024-01-01&date_to=2024-01-31
Sorting:          /orders?status=shipped&sort=-created_at
Pagination:       /orders?status=shipped&page=2&limit=50
Full example:     /orders?status=shipped&total_min=100&total_max=500&sort=-created_at&page=2
```

#### Implementation

```typescript
function OrdersPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const navigate = useNavigate();
  
  // Read all filters from URL
  const filters = {
    status: searchParams.get('status') || undefined,
    dateFrom: searchParams.get('date_from'),
    dateTo: searchParams.get('date_to'),
    page: parseInt(searchParams.get('page') || '1'),
    limit: parseInt(searchParams.get('limit') || '20'),
    sort: searchParams.get('sort') || '-created_at'
  };
  
  // Update filters: Modify URL, data automatically refetches
  function updateFilters(newFilters: Partial<typeof filters>) {
    const params = new URLSearchParams(searchParams);
    
    Object.entries(newFilters).forEach(([key, value]) => {
      if (value) {
        params.set(key, String(value));
      } else {
        params.delete(key);
      }
    });
    
    // Reset to page 1 when filters change
    params.set('page', '1');
    
    // Update URL (no navigation, same page)
    setSearchParams(params, { replace: true });
  }
  
  // Fetch with filters
  const { data, isLoading } = useOrders(filters);
  
  return (
    <>
      <FilterPanel
        filters={filters}
        onFilterChange={updateFilters}
      />
      
      <OrdersList data={data} />
      
      <Pagination
        page={filters.page}
        total={data?.total}
        onPageChange={(page) => updateFilters({ page })}
      />
    </>
  );
}
```

---

### 3.4 Real-Time Data Handling

**When data changes on server, push to client immediately.**

#### Strategy 1: WebSocket (Recommended for Real-Time)

```typescript
// hooks/useOrderUpdates.ts
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';

export function useOrderUpdates(orderId: string) {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    const ws = new WebSocket(`wss://api.company.com/ws/orders/${orderId}`);
    
    ws.onmessage = (event) => {
      const update = JSON.parse(event.data);
      
      // Immediately update cache
      queryClient.setQueryData(['orders', orderId], (old) => ({
        ...old,
        status: update.status,
        updated_at: update.updated_at
      }));
    };
    
    ws.onerror = () => {
      // Fallback to polling
      queryClient.refetchQueries({ queryKey: ['orders', orderId] });
    };
    
    return () => ws.close();
  }, [orderId, queryClient]);
}

// Usage:
function OrderDetailPage({ orderId }: { orderId: string }) {
  useOrderUpdates(orderId);  // Automatically syncs updates
  
  const { data: order } = useQuery({
    queryKey: ['orders', orderId],
    queryFn: () => fetchOrder(orderId)
  });
  
  return <OrderDetail order={order} />;
}
```

#### Strategy 2: Polling (Simpler, Less Scalable)

```typescript
// For less critical data
useQuery({
  queryKey: ['inventory', sku],
  queryFn: fetchInventory,
  refetchInterval: 5000  // Refetch every 5 seconds
});
```

#### Strategy 3: Server-Sent Events (Lightweight)

```typescript
useEffect(() => {
  const eventSource = new EventSource(`/api/v1/orders/${orderId}/updates`);
  
  eventSource.onmessage = (event) => {
    const order = JSON.parse(event.data);
    queryClient.setQueryData(['orders', orderId], order);
  };
  
  return () => eventSource.close();
}, [orderId, queryClient]);
```

---

## 4. Routing & Navigation

### 4.1 Route Structure Definition

**Routes map URLs to pages.** Good structure = intuitive navigation.

#### Route Hierarchy for Orders App

```
/ → Home / Dashboard
├─ /auth
│  ├─ /login → Login page
│  ├─ /register → Registration
│  └─ /forgot-password → Password reset
│
├─ /orders (protected)
│  ├─ / → Orders list (with filters)
│  ├─ /:orderId → Order detail
│  ├─ /:orderId/edit → Edit order (if allowed)
│  ├─ /:orderId/track → Shipping tracking
│  ├─ /:orderId/invoice → Print invoice
│  └─ /new → Create order
│
├─ /checkout (protected)
│  ├─ /cart → Review cart
│  ├─ /shipping → Enter address
│  ├─ /payment → Payment info
│  └─ /confirmation → Order confirmation
│
├─ /account (protected)
│  ├─ /profile → User profile
│  ├─ /addresses → Saved addresses
│  ├─ /payment-methods → Saved cards
│  ├─ /notifications → Notification settings
│  └─ /settings → Account settings
│
├─ /admin (protected, admin-only)
│  ├─ /dashboard → Admin dashboard
│  ├─ /users → Manage users
│  ├─ /inventory → Manage inventory
│  ├─ /reports → Analytics
│  └─ /settings → System settings
│
└─ /* (catch-all)
   └─ → 404 Not Found
```

#### React Router Implementation

```typescript
// router.tsx
import { createBrowserRouter } from 'react-router-dom';
import { Layout } from './pages/Layout';
import { ProtectedRoute } from './components/ProtectedRoute';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <Layout />,
    children: [
      {
        index: true,
        element: <DashboardPage />
      },
      {
        path: 'auth',
        children: [
          { path: 'login', element: <LoginPage /> },
          { path: 'register', element: <RegisterPage /> }
        ]
      },
      {
        path: 'orders',
        element: <ProtectedRoute />,
        children: [
          { index: true, element: <OrdersPage /> },
          {
            path: ':orderId',
            element: <OrderDetailLayout />,
            children: [
              { index: true, element: <OrderDetailPage /> },
              { path: 'track', element: <TrackingPage /> },
              { path: 'invoice', element: <InvoicePage /> }
            ]
          },
          { path: 'new', element: <CreateOrderPage /> }
        ]
      },
      {
        path: 'checkout',
        element: <ProtectedRoute />,
        children: [
          { index: true, element: <CheckoutPage /> }
        ]
      }
    ]
  }
]);
```

---

### 4.2 Protected Routes Strategy

**Not every page is public.** Control access based on authentication + authorization.

```typescript
// components/ProtectedRoute.tsx
import { Navigate } from 'react-router-dom';
import { useAppStore } from '../store/appStore';

interface ProtectedRouteProps {
  requiredRole?: string[];
  children?: React.ReactNode;
}

export function ProtectedRoute({ requiredRole, children }: ProtectedRouteProps) {
  const { user } = useAppStore();
  
  // Not logged in → Redirect to login
  if (!user) {
    return <Navigate to="/auth/login" replace />;
  }
  
  // Logged in but missing role → Redirect to access denied
  if (requiredRole && !requiredRole.includes(user.role)) {
    return <Navigate to="/access-denied" replace />;
  }
  
  // All checks passed → Render page
  return children || <Outlet />;
}

// Usage:
<Route
  path="admin"
  element={<ProtectedRoute requiredRole={['admin']} />}
  children={adminRoutes}
/>
```

#### Redirect to Login with Return URL

```typescript
// On login success
const from = location.state?.from?.pathname || '/orders';
navigate(from, { replace: true });

// Pass return URL when redirecting to login
<Navigate to="/auth/login" state={{ from: location }} replace />
```

---

### 4.3 Deep Linking

**Every page should have a unique URL that can be shared, bookmarked, deep-linked.**

#### Examples of Deep Links

```
/orders?status=shipped&page=2
→ Share filtered view

/orders/ORD-123
→ Link directly to order

/orders/ORD-123/track
→ Link to tracking page

/checkout/shipping?address_id=ADDR-456
→ Resume checkout at shipping step

/account/addresses?edit=ADDR-789
→ Edit specific address

/?utm_source=email&utm_campaign=sale
→ Marketing campaign deep link
```

#### Preserve State in URL

```typescript
// When user sorts/filters, update URL
function OrdersList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const sort = searchParams.get('sort') || '-created_at';
  const status = searchParams.get('status');
  
  function handleSort(field: string) {
    // Determine direction (ascending/descending)
    const isAscending = sort === field;
    const newSort = isAscending ? `-${field}` : field;
    
    setSearchParams({ sort: newSort, status: status || '' }, { replace: true });
  }
  
  return (
    <th onClick={() => handleSort('created_at')}>
      Date {sort === 'created_at' ? '▲' : sort === '-created_at' ? '▼' : ''}
    </th>
  );
}
```

---

### 4.4 Navigation Flow Diagrams

**Map user journeys across pages.** Identify bottlenecks, missing pages, confusing flows.

#### Flow: New Customer Checkout

```
Home Page
    │
    └─→ [Browse Products]
        │
        └─→ Product Detail
            │
            └─→ [Add to Cart] ─→ Cart Count +1
                │
                └─→ Continue Shopping or [Checkout]
                    │
                    ├─→ Cart Review
                    │   ├─ [Edit Quantity] → stays on cart
                    │   ├─ [Remove Item] → cart updates
                    │   └─ [Checkout]
                    │
                    └─→ LOGIN CHECK
                        ├─ Not logged in → /auth/login
                        │   └─ [Login] or [Register]
                        │       └─ → Checkout (resume)
                        │
                        └─ Logged in → Checkout Page
                            │
                            ├─ Shipping Address
                            │   ├─ [Select saved] or [Enter new]
                            │   └─ [Continue]
                            │
                            ├─ Payment
                            │   ├─ [Select saved card] or [Enter new]
                            │   └─ [Place Order]
                            │
                            ├─ Processing... (background job)
                            │
                            └─ Confirmation Page
                                ├─ Order number displayed
                                ├─ [View Order] → /orders/:orderId
                                └─ [Continue Shopping] → Home
```

#### Flow: Returning Customer with Saved Data

```
Home
└─→ [Quick Checkout]
    └─→ Cart Review (auto-filled from last order)
        ├─ Same address? → [Use saved]
        ├─ Same payment? → [Use saved]
        └─ [Place Order] → Confirmation
```

#### Problem Detection in Flows

- ❓ Too many steps (> 5) before order confirmation → Simplify
- ❓ Unclear next action → Add button labels, success messages
- ❓ No undo/back functionality → User feels trapped
- ❓ Form validation errors → Show inline, not after submit
- ❓ Optional fields feel required → Indicate [optional] clearly

---

### 4.5 SEO Considerations

**Frontend routing affects search engine indexing.** Plan for SEO.

#### Meta Tags Per Page

```typescript
// pages/OrderDetailPage.tsx
import { Helmet } from 'react-helmet-async';

export function OrderDetailPage({ orderId }: { orderId: string }) {
  const { data: order } = useOrder(orderId);
  
  return (
    <>
      <Helmet>
        <title>Order #{order?.number}</title>
        <meta
          name="description"
          content={`Order #${order?.number} status: ${order?.status}. Total: $${order?.total}`}
        />
        <link rel="canonical" href={`https://company.com/orders/${orderId}`} />
        
        {/* Open Graph for social sharing */}
        <meta property="og:title" content={`Order #${order?.number}`} />
        <meta property="og:description" content={`Status: ${order?.status}`} />
        <meta property="og:url" content={`https://company.com/orders/${orderId}`} />
      </Helmet>
      
      {/* Content */}
    </>
  );
}
```

#### SSR for Product Pages (If Needed)

**Pages that benefit from SSR:**
- Home page (SEO important)
- Product detail (search users landing here)
- Category pages (SEO)

**Pages that DON'T need SSR:**
- Orders list (protected, logged-in only)
- Checkout (user-specific, not searchable)
- Account settings (not for indexing)

> **Decision:** If SEO critical → Use Next.js for SSR. If SEO not critical (logged-in only) → SPA with Helmet tags is fine.

---

## 5. Performance Planning

### 5.1 Bundle Splitting Strategy

**Goal:** Load only code needed for current page, defer everything else.

#### Webpack/Vite Configuration

```typescript
// Code splitting strategy
const getPageConfig = (pageName: string) => ({
  chunkName: pageName,
  // Lazy load route
  element: lazy(() => import(
    /* webpackChunkName: "[request]" */
    `./pages/${pageName}Page`
  ))
});

export const routes = [
  { path: '/orders', element: getPageConfig('Orders') },
  { path: '/checkout', element: getPageConfig('Checkout') },
  { path: '/admin', element: getPageConfig('Admin') }
];

// Result:
// Initial bundle: 150KB (main, layout, common components)
// Orders page: +45KB (lazy-loaded when user navigates)
// Checkout: +60KB (lazy-loaded)
// Admin: +80KB (lazy-loaded, only if admin user)
// Total: 335KB spread across bundles, not 335KB upfront
```

#### Bundle Size Targets

```
Total JS:          < 500KB (gzip)
Initial bundle:    < 200KB (gzip)
Per-page bundle:   < 150KB (gzip)
HTML:              < 50KB (gzip)
CSS:               < 100KB (gzip)
```

**Real Example:**
```
├─ Initial (JS + CSS): 180KB
├─ Orders page: +45KB
├─ Checkout: +60KB
├─ Admin: +80KB
└─ Total: 365KB (for complete app)
```

**Monitor with:**
- webpack-bundle-analyzer (visualize)
- source-map-explorer (size breakdown)
- Next.js bundle analyzer (if using Next)

---

### 5.2 Lazy Loading Implementation

**Don't load what you don't see.**

#### Route-Level Lazy Loading

```typescript
// router.tsx
import { lazy, Suspense } from 'react';

// Lazy load entire pages
const OrdersPage = lazy(() => import('./pages/OrdersPage'));
const CheckoutPage = lazy(() => import('./pages/CheckoutPage'));
const AdminPage = lazy(() => import('./pages/AdminPage'));

export const routes = [
  {
    path: '/orders',
    element: (
      <Suspense fallback={<PageSkeleton />}>
        <OrdersPage />
      </Suspense>
    )
  },
  {
    path: '/checkout',
    element: (
      <Suspense fallback={<PageSkeleton />}>
        <CheckoutPage />
      </Suspense>
    )
  }
];
```

#### Component-Level Lazy Loading

```typescript
// Lazy load modal content (only when modal opens)
const DeleteConfirmModal = lazy(() => import('./DeleteConfirmModal'));

function OrderRow({ order, onDelete }) {
  const [showConfirm, setShowConfirm] = useState(false);
  
  return (
    <>
      {/* ... */}
      <button onClick={() => setShowConfirm(true)}>Delete</button>
      
      {showConfirm && (
        <Suspense fallback={null}>
          <DeleteConfirmModal
            onConfirm={() => {
              onDelete();
              setShowConfirm(false);
            }}
            onCancel={() => setShowConfirm(false)}
          />
        </Suspense>
      )}
    </>
  );
}
```

#### Image Lazy Loading

```typescript
// Lazy load images below the fold
<img
  src={imageUrl}
  loading="lazy"  // Native lazy loading
  alt="Product"
/>

// Or with IntersectionObserver for older browsers
function LazyImage({ src, alt }: { src: string; alt: string }) {
  const [loaded, setLoaded] = useState(false);
  const ref = useRef<HTMLImageElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setLoaded(true);
        observer.disconnect();
      }
    });
    
    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, []);
  
  return (
    <img
      ref={ref}
      src={loaded ? src : 'placeholder.jpg'}
      alt={alt}
      width={500}
      height={300}
    />
  );
}
```

---

### 5.3 Image Optimization Strategy

**Images are typically 50% of page weight.** Optimize them.

#### Image Optimization Checklist

**Format:**
```
├─ WebP (modern browsers, ~25% smaller than JPEG)
├─ JPEG (fallback for older browsers)
└─ PNG (for screenshots, diagrams only)
```

**Size:**
```
├─ Product thumbnail: 200x200px, 50KB max
├─ Product detail: 600x600px, 150KB max
├─ Hero banner: 1920x600px, 200KB max
└─ Profile avatar: 80x80px, 10KB max
```

**Responsive:**
```
├─ Srcset: 1x (mobile), 2x (retina), 3x (ultra-retina)
├─ Picture element with media queries
└─ Deliver different sizes for different breakpoints
```

**CDN Delivery:**
```
├─ Serve from edge location (CloudFlare, Cloudinary, AWS CloudFront)
├─ Auto-format (WebP for Chrome, JPEG for Safari)
├─ Auto-optimize (quality reduction, compression)
└─ Cache headers: 1 year (immutable)
```

#### Implementation

```html
<picture>
  <source
    srcset="product-600.webp 1x, product-1200.webp 2x"
    type="image/webp"
    media="(min-width: 768px)"
  />
  <source
    srcset="product-400.webp 1x, product-800.webp 2x"
    type="image/webp"
  />
  <img
    src="product-600.jpg"
    srcset="product-600.jpg 1x, product-1200.jpg 2x"
    alt="Product"
    loading="lazy"
  />
</picture>
```

---

### 5.4 Caching Strategy

**Three layers:** Browser, Service Worker, CDN.

#### Browser Cache (HTTP Headers)

**For static assets (never change):**
```
Cache-Control: max-age=31536000, immutable
```

**For HTML (changes often):**
```
Cache-Control: max-age=0, no-cache
```

**For API responses:**
```
Cache-Control: private, max-age=300 (5 min)
```

#### Service Worker (Offline Support)

```typescript
// service-worker.ts
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('app-v1').then((cache) => {
      return cache.addAll([
        '/',
        '/app.js',
        '/styles.css',
        '/manifest.json'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  // Network first for API calls (try network, fallback to cache)
  if (event.request.url.includes('/api')) {
    event.respondWith(
      fetch(event.request)
        .then((response) => {
          caches.open('api-v1').then((cache) => {
            cache.put(event.request, response.clone());
          });
          return response;
        })
        .catch(() => caches.match(event.request))
    );
  }
  // Cache first for static assets
  else {
    event.respondWith(
      caches.match(event.request)
        .then((response) => response || fetch(event.request))
    );
  }
});
```

#### CDN Caching

**Static assets (CSS, JS, images):**
```
├─ Cache for 1 year in CDN
├─ Versioned filenames (app.abc123.js)
└─ Update version on deploy
```

**HTML:**
```
├─ Cache for 5 minutes in CDN
├─ Not versioned (always served fresh)
└─ Add header: Cache-Control: max-age=300, no-cache
```

**API responses (if cacheable):**
```
├─ Cache for 5 minutes in CDN
├─ Only for GET requests
└─ Include X-Cache-Status header
```

---

### 5.5 Core Web Vitals Targets

**Google's performance metrics.** Affects search ranking.

#### Metrics and Targets

**LCP (Largest Contentful Paint):**
```
├─ Target: < 2.5 seconds
├─ Measured: When largest image/text becomes visible
└─ Fix: Lazy load images, compress, split bundles
```

**FID (First Input Delay):**
```
├─ Target: < 100 milliseconds
├─ Measured: Time from click to response
└─ Fix: Offload heavy JS to workers, reduce main thread work
```

**CLS (Cumulative Layout Shift):**
```
├─ Target: < 0.1
├─ Measured: Unexpected layout changes
└─ Fix: Reserve space for images, ads; prevent font swaps
```

**INP (Interaction to Next Paint) - replacing FID:**
```
├─ Target: < 200 milliseconds
├─ Measured: Full interaction time (input → paint)
└─ Fix: Optimize event handlers, reduce animation jank
```

#### Measurement Tools

**Real Users:**
```
├─ Google Analytics (Web Vitals report)
├─ DebugBear (monitor over time)
└─ Sentry (performance monitoring + errors)
```

**Lab:**
```
├─ Google Lighthouse (local testing)
├─ WebPageTest (detailed waterfall)
└─ Chrome DevTools (live inspection)
```

#### Performance Budget (Never Exceed)

```
JavaScript bundle: < 200KB gzip (initial)
CSS bundle: < 80KB gzip
Initial HTML: < 50KB gzip
First Contentful Paint: < 1.5s (fast 3G)
Largest Contentful Paint: < 2.5s (fast 3G)
Cumulative Layout Shift: < 0.1
```

---

## 6. Design Review Gates

### Gate 1: Design Review & Approval

**Before frontend implementation, design approved by product/design team.**

#### Checklist

**Information Architecture:**
- [ ] All necessary pages exist
- [ ] User journeys are clear and intuitive
- [ ] No dead ends or confusing flows
- [ ] Mobile + desktop considered

**Visual Design:**
- [ ] Consistent with design system
- [ ] Colors have sufficient contrast
- [ ] Typography hierarchy clear
- [ ] Whitespace balanced
- [ ] Dark mode reviewed (if applicable)

**Interactions:**
- [ ] Loading states shown
- [ ] Error states shown
- [ ] Success states shown
- [ ] Micro-interactions feel smooth
- [ ] Focus indicators visible

**Responsive:**
- [ ] Mobile layout tested (375px min)
- [ ] Tablet layout tested (768px)
- [ ] Desktop layout tested (1440px)
- [ ] No horizontal scroll at any size
- [ ] Touch targets 44x44px minimum

**Performance:**
- [ ] Bundle size within budget
- [ ] Images optimized
- [ ] Lazy loading plan defined

**Sign-off:**
- [ ] Product lead approval
- [ ] Design lead approval
- [ ] PR comment: "Design approved for implementation"

---

### Gate 2: Accessibility Compliance Check

**Before shipping, accessibility audited and passed.**

#### WCAG 2.1 AA Audit

**Color Contrast:**
- [ ] All text passes 4.5:1 ratio (normal text)
- [ ] Large text passes 3:1 ratio (18px+, bold)
- [ ] Tool: WebAIM Contrast Checker

**Keyboard Navigation:**
- [ ] All interactive elements reachable via Tab
- [ ] Focus order matches reading order
- [ ] No keyboard traps
- [ ] Escape closes modals/dropdowns
- [ ] Enter submits forms
- [ ] Arrow keys navigate multi-select lists
- [ ] Tool: Manual keyboard test

**Screen Reader:**
- [ ] Page structure is semantic (h1, nav, main, etc.)
- [ ] Images have descriptive alt text
- [ ] Icons have aria-labels
- [ ] Form labels associated with inputs
- [ ] Error messages linked to fields (aria-describedby)
- [ ] Skip link to main content
- [ ] Tool: Screen reader test (NVDA, JAWS, VoiceOver)

**ARIA Attributes:**
- [ ] aria-label for icon-only buttons
- [ ] aria-describedby for errors
- [ ] aria-live for dynamic content
- [ ] aria-hidden for decorative elements
- [ ] role properly used (not overridden)

**Zoom & Size:**
- [ ] Page usable at 200% zoom
- [ ] Text resizable without loss of functionality
- [ ] Touch targets 44x44px

**Manual Accessibility Test:**
- [ ] Use keyboard only (no mouse) for 5 minutes
- [ ] Use screen reader for 5 minutes
- [ ] Zoom to 200%, navigate page
- [ ] Remove colors (grayscale), verify clarity
- [ ] Tool: WAVE browser extension

**Sign-off:**
- [ ] Accessibility lead approved
- [ ] Manual testing completed
- [ ] Issues documented in issue tracker
- [ ] Remaining issues assigned to future sprints (if any)

---

## Summary: What You've Designed

By end of Phase 5, you have:

### ✅ UX/UI Design
- Wireframes (information architecture)
- High-fidelity mockups (visual design)
- Design system (reusable components)
- Responsive breakpoints (mobile, tablet, desktop)
- Accessibility audit (WCAG 2.1 AA)

### ✅ Component Architecture
- Component hierarchy (atoms → molecules → organisms → pages)
- Prop interfaces (typed, consistent)
- Reusability analysis (avoid duplication)

### ✅ State Management
- Global state locations (Zustand)
- Server state strategy (TanStack Query)
- Local state patterns (useState)
- URL parameters (shareable filters)

### ✅ Routing & Navigation
- Route structure (organized, intuitive)
- Protected routes (based on authentication/authorization)
- Deep linking (every page shareable)
- Navigation flows (user journeys mapped)

### ✅ Performance Planning
- Bundle splitting strategy (lazy loading)
- Image optimization (WebP, responsive, CDN)
- Caching strategy (browser, service worker, CDN)
- Core Web Vitals targets (LCP, FID, CLS, INP)

### ✅ Design Review Gates
- Design approved by product/design team
- Accessibility compliant (WCAG 2.1 AA)
- Performance budgets set
- Sign-offs documented

---

## Critical Decisions for Engineering

**These design decisions eliminate entire categories of bugs:**

1. **State in URL** → Filter/sort changes are always shareable and bookmarkable
2. **TanStack Query** → No manual refetching, stale data handling, cache invalidation
3. **Component hierarchy** → Easy to find what you need, easy to reuse
4. **Protected routes** → No accidental exposure of user data
5. **Accessibility from start** → Not an afterthought, builds confidence
6. **Bundle splitting** → First page load < 3 seconds even on slow connections
7. **Design system** → No style inconsistencies, easier onboarding

---

## 20-Year Veteran's Take

> I've seen more bugs from "frontend isn't real engineering" attitude than from bad algorithms. Frontend design is harder than backend design. More users see it. More can break. **Do it right here. Save 3 months of rework later.**

---

## Next Phase: Phase 6 (Database Design)

Now that frontend is designed, backend knows exactly what data to store and how to query it.

**Frontend → Backend dependency:** LOW (API contract defined in Phase 4)  
**Backend → Frontend dependency:** HIGH (frontend waits for data structures)  

**Design database after frontend, informed by frontend's query patterns.**

---

