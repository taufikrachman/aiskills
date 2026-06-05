# Component Review Checklist

Run through this before marking any React component as complete.

## Before Commit
- [ ] TypeScript: zero `any` types
- [ ] Props interface exported (if reusable)
- [ ] `className` prop accepted and merged with `cn()`
- [ ] Ref forwarded via `forwardRef` (if wrapping native element)
- [ ] Keyboard accessible (Tab, Enter, Escape)
- [ ] ARIA labels on all interactive elements without visible text
- [ ] Semantic HTML used (button, nav, main, etc.)
- [ ] Color contrast: 4.5:1 min for text, 3:1 for large text

## States
- [ ] Loading state → skeleton or spinner
- [ ] Empty state → friendly message + action
- [ ] Error state → message + retry button
- [ ] Edge cases → 0 items, null data, very long text

## Responsive
- [ ] Mobile layout tested (375px)
- [ ] Tablet layout tested (768px)
- [ ] No horizontal scroll on any breakpoint
- [ ] Touch targets minimum 44×44px

## Performance
- [ ] `useMemo` on expensive computations
- [ ] `React.memo` on pure presentational children
- [ ] No inline objects/arrays in JSX props
- [ ] File under 200 lines (extract if exceeding)

## Dark Mode
- [ ] All colors use `dark:` variants
- [ ] No hardcoded colors (use Tailwind theme)
- [ ] Images/illustrations visible in both modes
