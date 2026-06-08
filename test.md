# 🔮 HAWIYAT-ENIGMA VIBECODING TEST — CHALLENGE v2.0
## Next.js Full-Stack Edition

> **Feed this entire document to the LLM under test. Nothing else. Observe.**

---

## CONTEXT & MISSION

You are a senior full-stack engineer and UI/UX engineer specializing in modern React ecosystems. Your task is to build **"Hawiyat Hub"** — a full-stack personal productivity web application — from scratch, using **Next.js 15 (App Router), TypeScript, shadcn/ui, Tailwind CSS v4, and SQLite via better-sqlite3**.

This is a timed design + engineering challenge. Quality, completeness, and attention to detail matter. Think before you code. Design before you implement.

---

## TECH STACK — MANDATORY

| Layer | Technology | Version |
|---|---|---|
| Framework | Next.js (App Router) | 15.x |
| Language | TypeScript | 5.x |
| UI Components | shadcn/ui | latest |
| Styling | Tailwind CSS | v4 |
| Database | SQLite via `better-sqlite3` | latest |
| ORM / Query | Direct `better-sqlite3` (no Prisma/Drizzle) | — |
| Icons | `lucide-react` | latest |
| Font | Plus Jakarta Sans (Google Fonts / `next/font`) | — |
| Form Validation | `zod` + `react-hook-form` | latest |
| State | React built-ins (`useState`, `useOptimistic`) | — |
| Theme | `next-themes` | latest |

**No other dependencies are permitted** unless explicitly noted below. Do not use Prisma, Drizzle, SWR, TanStack Query, Redux, Jotai, Zustand, or any external state library.

---

## PRODUCT SPECIFICATION

### App Name: **Hawiyat Hub**
### Tagline: *"Your tasks. Your pace. Your space."*

---

## PROJECT STRUCTURE — REQUIRED

The project must follow this structure exactly:

```
hawiyat-hub/
├── app/
│   ├── layout.tsx              # Root layout with font, theme provider
│   ├── page.tsx                # Home → redirects to /board
│   ├── board/
│   │   └── page.tsx            # Main Kanban board page
│   └── api/
│       └── tasks/
│           ├── route.ts         # GET (list), POST (create)
│           └── [id]/
│               └── route.ts     # PATCH (update), DELETE (delete)
├── components/
│   ├── board/
│   │   ├── KanbanBoard.tsx
│   │   ├── KanbanColumn.tsx
│   │   ├── TaskCard.tsx
│   │   └── TaskForm.tsx
│   ├── layout/
│   │   ├── Header.tsx
│   │   └── StatsPanel.tsx
│   └── ui/                     # shadcn/ui components (auto-generated)
├── lib/
│   ├── db.ts                   # SQLite singleton + schema init
│   ├── actions.ts              # Next.js Server Actions
│   └── types.ts                # Shared TypeScript types
├── hooks/
│   └── useTasks.ts             # Client-side task state hook
├── public/
│   └── ...
├── .env.local                  # DB_PATH=./hawiyat.db
├── components.json             # shadcn/ui config
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

---

## CORE FEATURES — YOU MUST BUILD ALL OF THESE

### 1. 📋 Kanban Task Board

A three-column Kanban board with the following columns:
- **Backlog** — Ideas and unstarted tasks
- **In Progress** — Actively being worked on
- **Done** — Completed tasks

**Task Card must contain:**
- Title (required, max 60 chars)
- Description (optional, max 200 chars)
- Priority badge: `LOW` / `MEDIUM` / `HIGH` / `CRITICAL`
- Due date (optional, shows `"Overdue"` in destructive red if past today)
- A unique auto-generated task ID (format: `HW-001`, `HW-002`, ...)
- Created timestamp (shown as relative time, e.g., `"2 hours ago"`)
- A colored left-border that reflects priority:
  - `LOW` → `border-l-4 border-emerald-500`
  - `MEDIUM` → `border-l-4 border-amber-400`
  - `HIGH` → `border-l-4 border-orange-500`
  - `CRITICAL` → `border-l-4 border-destructive`

**Board interactions:**
- Click a **"+ Add Task"** button per column to create a new task in that column
- Tasks can be **moved between columns** via a "Move to →" dropdown on the card (shadcn `DropdownMenu`)
- Tasks can be **deleted** with a confirmation prompt (shadcn `AlertDialog`)
- Tasks can be **edited** (click pencil icon to open edit `Dialog`)
- Double-click a card title to quick-edit inline (contenteditable with blur-save)

---

### 2. ➕ Task Creation / Edit Modal

Use a shadcn `Dialog` with:
- Title input (`Input`, required, validates on submit via `zod`)
- Description textarea (`Textarea`, optional)
- Priority selector (shadcn `Select` with color-coded options)
- Due date picker (shadcn `Popover` + `Calendar`)
- Cancel and Save buttons
- Press `Escape` to close (built into Dialog)
- Clicking the backdrop closes the dialog
- Edit mode pre-fills all fields from the existing task

---

### 3. 🔍 Search & Filter Bar

A persistent bar between the header and the board with:
- **Live search** `Input` — filters task cards in real time by title or description (debounced 150ms, case-insensitive, client-side)
- **Priority filter** — shadcn `Select` (`All`, `Low`, `Medium`, `High`, `Critical`)
- **Column filter** — shadcn `Select` (`All Columns`, `Backlog`, `In Progress`, `Done`)
- A **"Clear Filters"** `Button` (variant: ghost) that resets all three filters
- A live count badge: `"Showing 4 of 9 tasks"` using shadcn `Badge`

---

### 4. 📊 Stats Dashboard (Collapsible Panel)

A collapsible summary panel below the header (shadcn `Collapsible`) showing:
- Total tasks count
- Tasks per column with percentage `Progress` bars
- Overdue tasks count (shown in destructive color if > 0)
- High + Critical tasks count
- A `"Done / Total"` completion rate progress bar
- Panel open/close state persisted in `localStorage`

---

### 5. 🌗 Dark / Light Mode Toggle

- Toggle `Button` in the header (moon/sun `lucide-react` icons)
- Uses `next-themes` for theme management
- Default to OS preference (`prefers-color-scheme`)
- Persist choice in `localStorage`
- Smooth CSS transition between modes via Tailwind `transition-colors duration-200`
- No flash of wrong theme on load (SSR-safe via `suppressHydrationWarning`)

---

### 6. 💾 Data Persistence — SQLite via Server Actions

**Database schema** (initialize on first load in `lib/db.ts`):

```sql
CREATE TABLE IF NOT EXISTS tasks (
  id          INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id     TEXT NOT NULL UNIQUE,        -- "HW-001"
  title       TEXT NOT NULL,
  description TEXT DEFAULT '',
  priority    TEXT NOT NULL DEFAULT 'MEDIUM',
  status      TEXT NOT NULL DEFAULT 'backlog',
  due_date    TEXT,                         -- ISO 8601 date string or NULL
  created_at  TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at  TEXT NOT NULL DEFAULT (datetime('now'))
);
```

**Server Actions** (`lib/actions.ts`):
- `createTask(data: CreateTaskInput): Promise<Task>` — inserts, returns full task
- `updateTask(id: string, data: Partial<Task>): Promise<Task>` — patches task
- `deleteTask(id: string): Promise<void>` — deletes by task_id
- `moveTask(id: string, status: ColumnStatus): Promise<Task>` — updates status
- All actions use `"use server"` directive and revalidate `/board` path
- All actions validate input with `zod` before touching the DB

**Seeding**: On first load, if the `tasks` table is empty, seed with **5 sample tasks** spread across all three columns.

**Export**: An `"Export JSON"` button in the header that calls a `/api/tasks?format=json` route and triggers a browser download.

**Clear All**: A shadcn `AlertDialog` with double-confirmation (type `"DELETE"` to confirm) that truncates the table and reloads.

---

### 7. 🎨 UI / UX Quality Requirements

All design requirements below are **mandatory**. This section defines the visual language of the application.

#### 7.1 Design System

| Token | Value |
|---|---|
| **Style** | Flat Design — 2D, minimalist, bold color accents, no drop shadows on cards, clean lines |
| **Primary Color** | `#0D9488` (Teal 600) |
| **Secondary Color** | `#14B8A6` (Teal 500) |
| **Accent / CTA Color** | `#EA580C` (Orange 600) |
| **Background (light)** | `#F0FDFA` (Teal 50) |
| **Foreground (light)** | `#134E4A` (Teal 950) |
| **Card (light)** | `#FFFFFF` |
| **Muted (light)** | `#E8F1F4` |
| **Border (light)** | `#99F6E4` (Teal 200) |
| **Background (dark)** | `#0F172A` (Slate 900) |
| **Foreground (dark)** | `#F0FDFA` (Teal 50) |
| **Card (dark)** | `#1E293B` (Slate 800) |
| **Muted (dark)** | `#1A2235` |
| **Border (dark)** | `#334155` (Slate 600) |
| **Destructive** | `#DC2626` (Red 600) |

All colors must be defined as CSS custom properties in `globals.css` using the shadcn/ui variable convention (`--background`, `--foreground`, `--primary`, etc.), and consumed via Tailwind utilities (`bg-background`, `text-foreground`, `text-primary`, etc.). No raw hex values in component files.

#### 7.2 Typography

| Use | Font | Weight | Size |
|---|---|---|---|
| Page title / hero | Plus Jakarta Sans | ExtraBold 800 | `text-2xl` |
| Section headings | Plus Jakarta Sans | Bold 700 | `text-xl` |
| Card titles | Plus Jakarta Sans | SemiBold 600 | `text-sm` |
| Body / description | Plus Jakarta Sans | Regular 400 | `text-sm` |
| Labels / badges | Plus Jakarta Sans | Medium 500 | `text-xs` |
| Task ID | Plus Jakarta Sans | Regular 400 | `text-xs` (muted) |

Font must be loaded via `next/font/google` — never via a `<link>` tag. Set as the default sans-serif in `tailwind.config.ts`.

#### 7.3 Spacing System

Use Tailwind's 4pt/8dp spacing scale exclusively:
- Component internal padding: `p-3` or `p-4`
- Card gap: `gap-3`
- Column gap: `gap-4` or `gap-6`
- Section vertical rhythm: `space-y-4`, `space-y-6`, `space-y-8`
- No arbitrary spacing values (no `p-[13px]`, `mt-[7px]`, etc.)

#### 7.4 Animation & Motion

| Interaction | Behavior |
|---|---|
| Card hover | `transition-all duration-150` — subtle `scale-[1.01]` and border color shift |
| Modal open/close | shadcn Dialog built-in fade + scale (do not override) |
| Column drop zone | Background color shift `transition-colors duration-150` |
| Stats panel collapse | shadcn Collapsible built-in animation |
| Toast | shadcn Sonner slide-in from bottom-right |
| Theme toggle | `transition-colors duration-200` on `html` element |
| Filter clear | Immediate; no animation needed |

Rules:
- All durations: 150–300ms. Never exceed 400ms for micro-interactions.
- Use `transform` and `opacity` only. Never animate `height`, `width`, `top`, or `left` directly.
- Always implement `@media (prefers-reduced-motion: reduce) { * { transition: none !important; animation: none !important; } }`

#### 7.5 Responsive Layout

| Breakpoint | Board Layout |
|---|---|
| Mobile (`< 768px`) | Single column, vertical stack. Column switcher tabs at top. |
| Tablet (`768px – 1023px`) | Two-column grid (Backlog + In Progress visible; Done behind tab) |
| Desktop (`≥ 1024px`) | Three-column equal-width grid (`grid-cols-3`) |

- Use `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8` for page container
- Column min-height: `min-h-[500px]` on desktop
- Cards must not overflow their column — use `overflow-hidden` and `text-ellipsis` for long titles
- No horizontal scroll at any viewport width

#### 7.6 Empty States

When a Kanban column has no tasks matching the current filters, render a centered placeholder inside the column:

```
[ghost icon — Inbox from lucide-react]
"No tasks here yet."
"Click '+ Add Task' to get started."
[+ Add Task button — outline variant]
```

Text color: `text-muted-foreground`. Border: `border-2 border-dashed border-border`. Rounded: `rounded-xl`. Padding: `p-8`.

#### 7.7 Priority Badges

Use shadcn `Badge` with custom variants:

| Priority | Variant Style |
|---|---|
| LOW | `bg-emerald-100 text-emerald-800 dark:bg-emerald-900/30 dark:text-emerald-400` |
| MEDIUM | `bg-amber-100 text-amber-800 dark:bg-amber-900/30 dark:text-amber-400` |
| HIGH | `bg-orange-100 text-orange-800 dark:bg-orange-900/30 dark:text-orange-400` |
| CRITICAL | `bg-red-100 text-red-800 dark:bg-red-900/30 dark:text-red-400` |

#### 7.8 Accessibility (Non-Negotiable)

- All interactive elements must have visible `:focus-visible` ring: `focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2`
- All modals must set `aria-labelledby` and `aria-describedby`
- All icon-only buttons must have `aria-label` and `sr-only` text
- Column headers must use `role="region"` with `aria-label`
- Drag-and-drop must have keyboard alternative (Move dropdown)
- Color is never the only indicator of status (priority shown as badge text + color + left border)
- Minimum text contrast: 4.5:1 for body text, 3:1 for large text (WCAG AA)
- Tab order must follow visual order left-to-right, top-to-bottom

---

## NON-FUNCTIONAL REQUIREMENTS

| Requirement | Expectation |
|---|---|
| **Type Safety** | No `any` types. All database results typed via `lib/types.ts`. All Server Action inputs validated by `zod` schema. |
| **Security** | Never use `dangerouslySetInnerHTML` with user data. Parameterized SQL queries only — no string interpolation. Input sanitized via `zod` before DB write. |
| **Performance** | Initial page load uses React Server Components (RSC) to fetch and render tasks server-side. Client components hydrate without layout shift. No unnecessary `"use client"` on server-renderable components. |
| **Error Handling** | All Server Actions wrapped in `try/catch`. API routes return typed error JSON `{ error: string }`. Toast notification on action failure. DB connection errors handled gracefully with fallback message. |
| **No Console Errors** | The app must run without any JS errors, TypeScript errors, or unhandled promise rejections. |
| **Loading States** | `useTransition` or `useOptimistic` used for task mutations. Buttons show spinner during pending state. Cards show optimistic update immediately. |
| **Code Quality** | Components under 200 lines. No prop drilling past 2 levels. Business logic in `lib/`, not in components. Functions named clearly in imperative verb form. |

---

## DATA MODEL — TypeScript Types (`lib/types.ts`)

```typescript
export type Priority = 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
export type ColumnStatus = 'backlog' | 'in_progress' | 'done';

export interface Task {
  id: number;
  taskId: string;         // "HW-001"
  title: string;
  description: string;
  priority: Priority;
  status: ColumnStatus;
  dueDate: string | null; // ISO 8601 date string
  createdAt: string;      // ISO 8601 datetime
  updatedAt: string;      // ISO 8601 datetime
}

export interface CreateTaskInput {
  title: string;
  description?: string;
  priority: Priority;
  status: ColumnStatus;
  dueDate?: string | null;
}

export interface UpdateTaskInput extends Partial<CreateTaskInput> {
  taskId: string;
}

export interface BoardColumn {
  id: ColumnStatus;
  label: string;
  tasks: Task[];
}
```

---

## API ROUTES

### `GET /api/tasks`
Returns all tasks ordered by `created_at DESC`.

**Query params:**
- `?format=json` — triggers a JSON file download (sets `Content-Disposition: attachment; filename="hawiyat-hub-export-<date>.json"`)

**Response:**
```json
{ "tasks": Task[] }
```

### `POST /api/tasks`
Creates a new task. Body: `CreateTaskInput`.

**Response:** `201` + `{ "task": Task }`

### `PATCH /api/tasks/[id]`
Updates task by `taskId`. Body: `Partial<UpdateTaskInput>`.

**Response:** `200` + `{ "task": Task }`

### `DELETE /api/tasks/[id]`
Deletes task by `taskId`.

**Response:** `204` (no body)

---

## BONUS FEATURES (Optional — extra score if implemented)

These are NOT required but significantly increase evaluation score:

- [ ] **Drag & Drop** — `@dnd-kit/core` (permitted additional dependency) for column-to-column drag. Drag handle icon on each card. Column highlights on dragover. Optimistic update on drop.
- [ ] **Keyboard Shortcut** — `Ctrl+K` or `N` opens "New Task" modal from anywhere on the page using a `useEffect` key listener
- [ ] **Undo Delete** — 5-second toast (shadcn Sonner) with "Undo" button after deleting a task; task restored if undo clicked before timeout
- [ ] **Tag System** — Up to 3 custom freeform tags per task stored as JSON in a `tags TEXT` column. Tags rendered as small badges. Tag filter in the filter bar.
- [ ] **Task Count Badge** — Column headers show live count badge (shadcn `Badge`) that updates as filters change
- [ ] **Import JSON** — File input (accept `.json`) in the header that parses exported task data and bulk-inserts via Server Action
- [ ] **Optimistic UI** — All mutations use `useOptimistic` so the UI updates instantly without waiting for the server round-trip

---

## IMPLEMENTATION REQUIREMENTS

### Database Initialization (`lib/db.ts`)

```typescript
// Must be a singleton — no multiple connections
// Must initialize schema on import
// Must handle DB_PATH from process.env.DB_PATH
// Must export a typed `db` instance used by all Server Actions
```

### Server Actions Pattern (`lib/actions.ts`)

```typescript
'use server';
// Each action must:
// 1. Parse + validate input with zod
// 2. Execute synchronous better-sqlite3 query
// 3. Revalidate the /board path via revalidatePath('/board')
// 4. Return typed result or throw formatted error
```

### Client Component Pattern

```typescript
'use client';
// Board page hydrates server-fetched tasks into local state
// Mutations call Server Actions + update local state optimistically
// No direct fetch() calls to /api/* from client components
// (API routes are for external consumers / export only)
```

---

## SEED DATA (Pre-populate on empty DB)

| # | Title | Status | Priority | Due Date |
|---|---|---|---|---|
| HW-001 | Set up project README | `done` | `LOW` | — |
| HW-002 | Design color system | `done` | `MEDIUM` | — |
| HW-003 | Build Kanban board layout | `in_progress` | `HIGH` | Today + 2 days |
| HW-004 | Implement drag-and-drop | `in_progress` | `HIGH` | Today + 1 day |
| HW-005 | Write unit tests for API | `backlog` | `MEDIUM` | Today + 7 days |

---

## DELIVERABLE

A complete Next.js project that:

1. Runs with `npm install && npm run dev` — no additional setup steps
2. Initializes the SQLite DB automatically on first start
3. Seeds sample data if no tasks exist
4. Passes `npx tsc --noEmit` with zero TypeScript errors
5. Has no `eslint` warnings in the default Next.js config
6. Renders correctly at 375px, 768px, 1024px, and 1440px viewport widths
7. Works in dark mode and light mode without visual glitches
8. Contains a comment block at the top of `lib/db.ts` with: App name, author, version `2.0.0`, and date

---

## MANDATORY TOOLING SETUP — BEFORE ANY CODE

Before writing a single line of application code, you **must** complete the following setup steps in order. A model that skips this section and proceeds directly to implementation receives **0 points for UI/UX Polish and Innovation**.

### Step 1 — Install Superpowers

Superpowers is a Claude Code plugin that provides workflow skills (brainstorming, planning, TDD, debugging, code review, etc.).

Run the following in your terminal:

```bash
claude mcp add superpowers -- npx -y @claude-plugins/superpowers@latest
```

Verify installation by checking that the skill `superpowers:brainstorming` is listed in your available skills. If it is not listed, do not proceed — fix the installation first.

### Step 2 — Install UI/UX Pro Max

UI/UX Pro Max is a Claude Code plugin that provides a searchable design intelligence database: 50+ styles, 161 color palettes, 57 font pairings, 99 UX guidelines, and 25 chart types.

Run the following:

```bash
claude mcp add ui-ux-pro-max -- npx -y @claude-plugins/ui-ux-pro-max@latest
```

Verify installation by checking that the skill `ui-ux-pro-max:ui-ux-pro-max` is listed in your available skills.

### Step 3 — Generate the Design System

Once both plugins are installed, invoke the `ui-ux-pro-max:ui-ux-pro-max` skill via Claude Code (use the Skill tool) with the following intent:

> "Generate and persist a full design system for Hawiyat Hub — a productivity kanban task manager, professional SaaS style. Use `--design-system --persist -p 'Hawiyat Hub' -f markdown`."

The skill will run its internal search script and write the result to `design-system/MASTER.md`. All component styling decisions must reference this file. **Do not invent a color palette** — use the one generated by the skill.

### Step 4 — Run the Brainstorming Skill

Before writing any code, invoke the `superpowers:brainstorming` skill. Use it to:
- Confirm your understanding of the product spec
- Identify the RSC vs client component boundaries
- Define the data flow: Server Actions → optimistic state → UI
- Write a brief implementation plan

Only after completing all four steps above should you begin writing application code.

**Evaluation note:** Reviewers will check for the presence of `design-system/MASTER.md` in the submitted project. Its absence is a 15-point deduction. Evidence of brainstorming (in commit history, comments, or a `docs/plan.md` file) adds 5 innovation points.

---

## EVALUATION CRITERIA

Your output will be evaluated on:

| Category | Weight | What We Look For |
|---|---|---|
| **Functional Completeness** | 25% | Every required feature works end-to-end (create, read, update, delete, move, filter, search, export) |
| **Type Safety & Code Quality** | 20% | No `any`, no console errors, clean separation of concerns, readable naming |
| **UI/UX Polish** | 20% | Design system respected, responsive, accessible, empty states, loading states, dark mode |
| **Database & Server Actions** | 15% | Correct schema, parameterized queries, proper revalidation, seed data |
| **Architecture** | 10% | RSC vs client boundary correct, file structure matches spec, no unnecessary client components |
| **Bonus Features** | 5% | Drag-and-drop, undo delete, optimistic UI, tag system |
| **Innovation** | 5% | See Innovation Scoring section below |

---

## INNOVATION SCORING

Innovation points are awarded for going meaningfully beyond the spec in ways that demonstrate creative engineering judgment. Each qualifying innovation earns **+1 to +3 points** (max 15 innovation points total, capped at 100 overall).

### What Counts as Innovation

| Innovation | Max Points | Criteria |
|---|---|---|
| **Micro-animation system** | +3 | A documented, reusable set of Tailwind/CSS animation primitives (e.g., `data-[state=open]:animate-in`, spring curves, stagger utilities) used consistently across ≥ 5 components |
| **Keyboard-first UX** | +2 | Full keyboard navigation: `Ctrl+K` command palette, `N` for new task, `E` for edit, `D` for delete, `←/→` to move between columns — all announced via `aria-live` |
| **Optimistic UI everywhere** | +2 | Every mutation (create, update, delete, move) uses `useOptimistic` with instant visual feedback and rollback on error |
| **AI-powered task suggestions** | +3 | A "Suggest next task" button that calls the Claude API (or any LLM) to generate a task based on existing board content |
| **Real-time collaboration readiness** | +2 | Architecture comment explaining how the app would extend to multi-user real-time (e.g., Server-Sent Events or WebSocket design sketch in `docs/realtime.md`) |
| **Custom shadcn theme** | +1 | A `components.json` + CSS variables file that goes beyond the default shadcn palette and matches the Hawiyat Hub teal/orange brand exactly, with named semantic tokens |
| **Board analytics view** | +2 | A `/analytics` page showing task velocity (tasks completed per day chart), priority distribution (donut chart), and average time-in-column — built with a lightweight charting lib (e.g., Recharts — permitted as bonus dependency) |
| **Pomodoro timer** | +1 | A 25-min countdown timer in the header, linkable to a task card, with start/pause/reset and a toast notification when time is up |
| **Design system provenance** | +1 | A `design-system/MASTER.md` generated by UI/UX Pro Max, referenced in code comments, with evidence of adherence (palette matches, font matches, spacing matches) |
| **Brainstorming evidence** | +1 | A `docs/plan.md` or commit message trail showing structured thinking before implementation (architecture decisions, trade-offs considered) |

### What Does NOT Count as Innovation

- Adding more dependencies than needed
- Rebuilding features that shadcn already provides
- Decorative animations with no UX purpose
- Features that break core functionality
- AI-generated boilerplate without meaningful adaptation

Innovation is judged on **intentionality, elegance, and user impact** — not line count.

---

## SCORING RUBRIC

| Score | Meaning |
|---|---|
| 0–40 | Incomplete or non-running app |
| 41–60 | Core board works but missing features (no filter, no persistence, or broken dark mode) |
| 61–75 | All core features work; minor UI inconsistencies or missing accessibility |
| 76–90 | Polished, fully functional, design system respected, accessible |
| 91–100 | All of the above + 2+ bonus features + optimistic UI + zero TS errors + innovation points |

---

**Begin. Install the tools. Generate the design system. Plan your implementation. Then build the complete Hawiyat Hub application.**
