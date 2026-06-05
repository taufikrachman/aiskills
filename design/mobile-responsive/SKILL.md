# Mobile-First Responsive Design

Design for mobile first, then scale up. NEVER design desktop-first.

## Rules

### 1. Breakpoint Strategy
```css
/* Mobile (default) — 320-639px */
/* Tablet — 640-1023px: sm, md */
/* Desktop — 1024px+: lg, xl, 2xl */

/* Tailwind: mobile-first */
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3"
```

### 2. Touch Targets
- Minimum 44×44px for all interactive elements (Apple HIG).
- Minimum 48×48dp for Android (Material Design).
- Spacing between touch targets: minimum 8px.
- Place primary actions in thumb-friendly zones (bottom half of screen).

### 3. Mobile Layout Patterns
- Single column by default. 2 columns at tablet. 3+ at desktop.
- Stack cards vertically on mobile, grid on larger screens.
- Bottom nav/tabs for primary navigation (3-5 items max).
- Swipe gestures for mobile (swipe to delete, swipe between tabs).
- Collapse sidebar into hamburger menu on mobile.

### 4. Typography on Mobile
- Body: 16px minimum (prevents iOS zoom on input tap).
- Line height: 1.5 for body, 1.3 for headings.
- Max line width: 65 characters for readability.
- Avoid text truncation on mobile — wrap or use expand/collapse.

### 5. Image Responsiveness
- Art direction: different images for mobile vs desktop.
- Use `srcset` and `sizes` for responsive images.
- Mobile: 1x (750px), Desktop: 2x (1440px).
- Lazy load below-fold images.

## Anti-Patterns
- ❌ Desktop-first CSS breaking on mobile
- ❌ Small touch targets (< 44px)
- ❌ Fixed-width containers that overflow on mobile
- ❌ Text too small to read (< 16px body)
