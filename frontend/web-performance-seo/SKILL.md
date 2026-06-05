# Web Performance & SEO

You are a web performance and SEO expert. Apply these optimizations to every Next.js project.

## Performance Rules

### 1. Core Web Vitals Targets
- LCP (Largest Contentful Paint): < 2.5s
- FID (First Input Delay): < 100ms → INP (Interaction to Next Paint): < 200ms
- CLS (Cumulative Layout Shift): < 0.1

### 2. Image Optimization
- Use `next/image` for ALL images. Never use raw `<img>`.
- Always set explicit `width` and `height` to prevent CLS.
- Use `priority` on above-the-fold images (hero, LCP element).
- Use `placeholder="blur"` with `blurDataURL` for lazy-loaded images.
- Format: WebP with AVIF fallback. Next.js handles this automatically.
- Responsive: `<Image sizes="(max-width: 768px) 100vw, 50vw" ... />`.

### 3. Font Optimization
- Use `next/font/google` or `next/font/local`. Zero layout shift guaranteed.
- Subset fonts: only load weights and character sets you use.
- `display: 'swap'` to show fallback text while font loads.

### 4. JavaScript Optimization
- Lazy load heavy components: `dynamic(() => import('./Chart'), { ssr: false })`.
- Code splitting: Next.js splits by route automatically. Don't undermine this.
- Bundle analysis: `ANALYZE=true next build` to find heavy dependencies.
- Avoid barrel exports (`index.ts` re-exporting everything) — hurts tree-shaking.

### 5. Caching
- Static pages: `export const dynamic = 'force-static'`.
- ISR: `export const revalidate = 3600` for semi-dynamic content.
- Route handlers: `export const dynamic = 'force-static'` if response doesn't change.
- CDN cache headers: set `Cache-Control` in route handlers for API responses.

## SEO Rules

### 6. Metadata
```tsx
export const metadata: Metadata = {
  title: { default: 'Site Name', template: '%s | Site Name' },
  description: 'Compelling 150-char description',
  metadataBase: new URL('https://example.com'),
  openGraph: {
    type: 'website',
    images: [{ url: '/og.png', width: 1200, height: 630 }],
  },
  twitter: { card: 'summary_large_image' },
  robots: { index: true, follow: true },
  alternates: { canonical: 'https://example.com/page' },
};
```

### 7. Structured Data (JSON-LD)
```tsx
export default function Page() {
  return (
    <>
      <script type="application/ld+json" dangerouslySetInnerHTML={{
        __html: JSON.stringify({
          '@context': 'https://schema.org',
          '@type': 'Organization',
          name: 'Company Name',
          url: 'https://example.com',
        }),
      }} />
      <Content />
    </>
  );
}
```

Required schema types: Organization (homepage), Article (blog), BreadcrumbList, Product (e-commerce), Event.

### 8. sitemap.xml & robots.txt
```ts
// app/sitemap.ts
export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: 'https://example.com', lastModified: new Date(), changeFrequency: 'weekly', priority: 1 },
    { url: 'https://example.com/blog', lastModified: new Date(), changeFrequency: 'daily', priority: 0.8 },
  ];
}
// app/robots.ts
export default function robots(): MetadataRoute.Robots {
  return { rules: { userAgent: '*', allow: '/' }, sitemap: 'https://example.com/sitemap.xml' };
}
```

### 9. Performance Checklist
- [ ] Lighthouse score ≥ 90 (Performance, Accessibility, Best Practices, SEO)
- [ ] All images use `next/image` with explicit dimensions
- [ ] Fonts loaded via `next/font`
- [ ] No render-blocking resources (check Coverage tab in DevTools)
- [ ] No layout shifts on load (check CLS in Performance tab)
- [ ] `sitemap.xml` and `robots.txt` exist
- [ ] JSON-LD structured data on all pages
- [ ] OpenGraph + Twitter meta tags
- [ ] Canonical URLs set correctly

## Anti-Patterns
- ❌ Raw `<img>` tags
- ❌ Google Fonts loaded via CSS `@import` — use `next/font`
- ❌ Missing `alt` text on images
- ❌ `loading="lazy"` on LCP image — use `priority` instead
- ❌ Client-side data fetching for SEO-critical content
