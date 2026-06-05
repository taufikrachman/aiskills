# Dashboard & Data Visualization

Design clear, actionable dashboards and data displays.

## Rules

### 1. Dashboard Layout
```
┌──────────────────────────────────────────────┐
│  [Spoke Selector ▼]           [👤 Profile]   │  ← Top bar
├──────────────────────────────────────────────┤
│  ┌────────┐ ┌────────┐ ┌────────┐           │  ← KPI cards
│  │  12     │ │  45     │ │  Rp 8M │           │
│  │Booking  │ │ Peserta │ │Revenue │           │
│  └────────┘ └────────┘ └────────┘           │
├──────────────────────┬─────────────────────────┤
│  Booking Minggu Ini  │  Event Mendatang       │  ← Content
│  (table/list)        │  (cards)               │
└──────────────────────┴─────────────────────────┘
```

### 2. KPI Cards
- 3-5 cards max in a row. Each card: label + value + trend indicator.
- Values: short format (Rp 8.5M not Rp 8,500,000). Trend: ↑↓ with color.
- Loading: skeleton matching card dimensions. Empty: 0 with label.

### 3. Chart Rules
- Bar chart: comparing categories. Line chart: trends over time. Pie/donut: part-of-whole (max 5 slices).
- Always label axes. Show unit (Rp, %, unit).
- Color: use brand primary, not random. Muted grid lines, no 3D effects.
- Responsive: charts resize to container width.
- Loading: skeleton rectangle. Empty: "No data for this period."

### 4. Tables
- Header: bold, sticky on scroll. Rows: alternating background (zebra).
- Actions per row: max 2-3 (edit, delete, view). Use icons with tooltips.
- Pagination: bottom, showing "1-50 of 150". Page numbers or "Load More".
- Empty: "No results" with create button. Loading: skeleton rows.

### 5. Filter & Sort
```tsx
<div className="flex gap-4">
  <Select value={status} onChange={setStatus}>
    <option>All Status</option>
    <option>Active</option>
    <option>Inactive</option>
  </Select>
  <DatePicker value={dateRange} onChange={setDateRange} />
  <SearchInput value={search} onChange={setSearch} placeholder="Search..." />
</div>
```
Filters at the top, persistent across navigation (URL query params).
