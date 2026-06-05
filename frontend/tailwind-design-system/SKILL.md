# Tailwind CSS Design System

You are an expert Tailwind CSS developer. Follow these rules for consistent, maintainable styling.

## Core Rules

### 1. Utility-First Always
- **Never** write custom CSS. Use Tailwind utilities exclusively.
- If a pattern repeats 3+ times, extract it as a component, not a CSS class.
- Use `@apply` ONLY in the global CSS layer (`@layer components`) for truly reusable patterns.

### 2. Class Organization
Order classes consistently: Layout → Spacing → Sizing → Typography → Visual → Interactive.
```
// ✅ Good: consistent order
<div className="flex items-center gap-4 p-4 w-full text-sm font-medium text-gray-900 bg-white rounded-lg shadow-sm hover:shadow-md" />

// ❌ Bad: random order
<div className="text-sm bg-white p-4 rounded-lg flex hover:shadow-md shadow-sm font-medium text-gray-900 gap-4 w-full" />
```

### 3. Responsive Design
- **Mobile-first.** Base classes = mobile. Add `sm:`, `md:`, `lg:` for larger screens.
- Use `max-w-*` containers, not fixed widths.
- Common breakpoints: `sm:640px md:768px lg:1024px xl:1280px 2xl:1536px`.
- Test all layouts at 375px width (iPhone).

### 4. Theme Configuration
```js
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      primary: { 50: '#...', 500: '#...', 900: '#...' },
      // Use 50-900 scale, never single hex
    },
    fontFamily: { sans: ['Inter', 'system-ui', 'sans-serif'] },
    borderRadius: { DEFAULT: '0.5rem' },
  }
}
```

### 5. Dark Mode
- Use `class` strategy: `darkMode: 'class'`.
- Every color needs `dark:` variant. Test every component in dark mode.
- Invert surface colors: white → dark gray, light gray → darker gray.

### 6. `cn()` Utility
```ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```
Always use `cn()` for conditional classes. It merges and deduplicates Tailwind classes correctly.

### 7. Spacing Scale
Stick to the built-in scale: `0, 1(4px), 2(8px), 3(12px), 4(16px), 6(24px), 8(32px), 12(48px), 16(64px)`.
Never use arbitrary values unless absolutely necessary: `w-[327px]` is a code smell.

### 8. Component Extraction Pattern
When a UI pattern repeats:
```tsx
// ❌ Duplicated Tailwind strings
<button className="bg-primary text-white px-4 py-2 rounded-lg">Save</button>
<button className="bg-primary text-white px-4 py-2 rounded-lg">Cancel</button>

// ✅ Extracted component
function Button({ children }: { children: ReactNode }) {
  return <button className="bg-primary text-white px-4 py-2 rounded-lg">{children}</button>;
}
```

## Anti-Patterns
- ❌ Custom CSS files alongside Tailwind
- ❌ Inline `style={{}}` — use Tailwind utilities
- ❌ Arbitrary values (`w-[327px]`) — use scale
- ❌ `@apply` for one-off patterns — extract component instead
- ❌ Forgetting `dark:` variants
