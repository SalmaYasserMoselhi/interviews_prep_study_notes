# React 19 → Next.js Mastery Roadmap

Sources: [react.dev](https://react.dev/) (v19.2 docs) and [nextjs.org/docs](https://nextjs.org/docs) (App Router, current).
Order matters here — each section assumes the one before it. Don't skip ahead to Next.js before Section 7.

---

## PART 1 — REACT 19

### 1. Prerequisites (before touching React at all)
- Modern JS: `let`/`const`, arrow functions, destructuring, spread/rest, template literals, modules (`import`/`export`)
- Array methods: `map`, `filter`, `reduce`, `find`
- Promises and `async`/`await`
- Basic DOM concepts (the DOM tree, events)
- Reading a `package.json`, npm/pnpm basics

### 2. Setup & Your First Component
- Creating a project (Vite is react.dev's recommended starting point for learning; Next.js comes later)
- JSX rules: must return one root element, must close every tag, `className` not `class`, `{}` for JS expressions
- Components are just functions returning JSX; component names must be capitalized
- Exporting/importing components (`Importing and Exporting Components`)
- Composing components (parent/child), and **never** defining a component inside another component

### 3. Describing the UI
*(react.dev "Describing the UI" section — do these in order)*
1. Your First Component
2. Importing and Exporting Components
3. Writing Markup with JSX
4. JavaScript in JSX with Curly Braces
5. Passing Props to a Component
6. Conditional Rendering
7. Rendering Lists (and **why `key` matters**)
8. Keeping Components Pure (pure functions, no side effects during render)
9. Understanding Your UI as a Tree

### 4. Adding Interactivity
1. Responding to Events (`onClick`, event handlers, passing arguments)
2. State: A Component's Memory (`useState`)
3. Render and Commit (the render lifecycle, mental model)
4. State as a Snapshot (why state updates don't show immediately)
5. Queueing a Series of State Updates (batching, updater functions)
6. Updating Objects in State (immutability)
7. Updating Arrays in State (immutability patterns)

### 5. Managing State
1. Reacting to Input with State (declarative UI thinking)
2. Choosing the State Structure (avoiding redundant/duplicated state)
3. Sharing State Between Components ("lifting state up")
4. Preserving and Resetting State (how state maps to position in the tree, `key` resets)
5. Extracting State Logic into a Reducer (`useReducer`)
6. Passing Data Deeply with Context (`useContext`, `createContext`) — and when *not* to reach for it
7. Scaling Up with Reducer and Context (combining both for medium-complexity state)

### 6. Escape Hatches
1. Referencing Values with Refs (`useRef`)
2. Manipulating the DOM with Refs
3. Synchronizing with Effects (`useEffect`) — what an Effect actually is
4. **You Might Not Need an Effect** — read this carefully; it's the most commonly violated page in the docs
5. Lifecycle of Reactive Effects (dependency arrays, cleanup functions)
6. Separating Events from Effects (`useEffectEvent`)
7. Removing Effect Dependencies
8. Reusing Logic with Custom Hooks (build your own `useX` hooks)

### 7. Hooks Reference (by category, react.dev's own grouping)
At this point go deep on the **Hooks** reference page by category rather than reading them as a flat list:
- **State Hooks**: `useState`, `useReducer`
- **Context Hooks**: `useContext`
- **Ref Hooks**: `useRef`, `useImperativeHandle`
- **Effect Hooks**: `useEffect`, `useLayoutEffect`, `useInsertionEffect`, `useEffectEvent`
- **Performance Hooks**: `useMemo`, `useCallback` (memoization), `useTransition`, `useDeferredValue` (concurrent rendering / responsiveness)
- **Other Hooks**: `useDebugValue`, `useId`, `useSyncExternalStore`
- **React 19 Action Hooks**: `useActionState`, `useOptimistic`
- `use` — the new React 19 API (not technically a "Hook"; can be called conditionally/in loops; reads Promises and Context, integrates with Suspense)

### 8. React 19: Actions, Forms & Concurrent Features (the headline changes)
Learn this as one connected story, in this order:
1. **Async functions in transitions** — `useTransition` now accepts async functions; pending state is handled for you
2. **Actions** — the umbrella concept: functions passed to `<form action={...}>` or `startTransition` that get automatic pending state, error handling, and optimistic update support
3. **`<form>` Actions + `useFormStatus`** (from `react-dom`) — reading submission status (`pending`, `data`, `method`) in a child of a `<form>` without prop drilling
4. **`useActionState`** — wraps an Action and gives you `[state, formAction, isPending]` in one call; replaces manual `isLoading`/`error`/`data` juggling
5. **`useOptimistic`** — instant UI feedback before the server confirms, with automatic rollback on failure
6. **`use(promise)`** and **`use(context)`** — reading resources directly in render, Suspense-integrated, conditional calls allowed
7. **`ref` as a prop** — function components can now accept `ref` directly; `forwardRef` is no longer required (and is on the path to deprecation)
8. **Document Metadata in JSX** — `<title>`, `<meta>`, `<link>` can now be rendered directly inside any component and React hoists them to `<head>`
9. **Stylesheets & async script support**, **`useId` improvements**, **improved hydration error diffs**

### 9. Performance & Compiler
- **React Compiler** — build-time tool that auto-memoizes components/values (reduces manual `useMemo`/`useCallback` need going forward); know its directives and config options conceptually even if you don't configure it day one
- Concurrent rendering mental model: what "interruptible rendering" means and why `useTransition`/`useDeferredValue` exist
- `React.memo`, code-splitting with `lazy` + `Suspense`

### 10. Rules of React (now formalized as docs, not folklore)
- Components and Hooks must be pure
- React calls Components and Hooks (you don't call them like normal functions)
- Rules of Hooks (top-level calls only, same order every render) — and how `use()` is the deliberate exception
- The `eslint-plugin-react-hooks` lint rules — install and actually read the violations it catches

### 11. React DOM specifics
- Built-in HTML/SVG components React supports natively
- `react-dom/client` (`createRoot`, `hydrateRoot`)
- `react-dom/server` (`renderToString`, `renderToPipeableStream`, etc. — context for *why* SSR frameworks exist)
- `react-dom/static` (static HTML generation APIs)
- Directives: `'use client'` and `'use server'` (you'll use these for real in Next.js, but know what they mean at the React level first)

### 12. Testing & Tooling (not on react.dev, but required for "great dev")
- React DevTools (Components + Profiler tabs)
- Testing components: React Testing Library + Vitest/Jest
- TypeScript with React: typing props, typing hooks, generic components

### 12.5 React Performance & Anti-Patterns Checklist
*(This is the section most tutorials skip. Go through it deliberately — it's the difference between "knows React" and "writes React that doesn't fall over at scale.")*

**The #1 source of bad React code: unnecessary `useEffect`.** React's own docs dedicate a full page to this — [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect) — because it's the most commonly violated rule in the entire library, including in AI-generated code. The core test: *if the logic is caused by the user seeing the component on screen, it's an Effect; if it's caused by a specific interaction, it belongs in the event handler.* Concretely, don't reach for an Effect to:
- **Compute derived state.** Don't store `fullName` in state and sync it from `firstName`/`lastName` in an Effect — just calculate `fullName` during render. Storing what can be derived causes an extra wasted render pass and creates bugs where the two pieces of state drift out of sync.
- **Reset all state when a prop changes.** Instead of an Effect that resets state on prop change, pass a `key` to the component so React remounts it cleanly.
- **Adjust state based on a prop change (partial reset).** Calculate the needed value directly during rendering instead of `useEffect` + a second `setState`.
- **Handle a user event.** Code that should run because the user clicked something (e.g., a POST request, an analytics event tied to the click) belongs in the event handler, not an Effect that fires because some state changed.
- **Chain Effects that each adjust state based on other state's changes.** This produces cascading re-renders. Calculate as much as possible in event handlers or during render instead.
- **Run app-initialization logic in a top-level Effect.** It will run twice in development under Strict Mode (by design, to surface bugs) and your components should be resilient to remounting regardless. If something truly must run once per app load, guard it with a module-level flag, not a component Effect.
- **Fetch data without cleanup.** A `useEffect` fetch with no cleanup creates **race conditions** — e.g. user types `query=a`, then `query=ab`, and the slower `a` response can resolve *after* the faster `ab` response and overwrite it with stale data. Use an `ignore` flag (or `AbortController`) in the cleanup function, or better, use a fetching library (SWR/React Query) or framework-level data fetching (Next.js Server Components) that handles this for you.

**Other recurring performance/correctness mistakes:**
- **Missing or wrong `key` props** in lists — using array index as key when the list can reorder/filter causes state to attach to the wrong item and stale UI bugs.
- **Mutating state directly** (`state.push(...)`, `state.x = y`) instead of creating new objects/arrays — breaks React's change detection and causes missed re-renders.
- **Defining a component inside another component's render body** — causes the inner component to be recreated (and remounted) every render, destroying its state and any DOM it owned.
- **Reaching for `useMemo`/`useCallback` everywhere "just in case."** Memoization has its own cost; per the docs, only memoize after *measuring* (e.g. `console.time`) that a calculation is actually expensive (roughly 1ms+) or that it's needed to prevent a child from re-rendering. The new **React Compiler** is meant to make most manual memoization unnecessary going forward — but you should understand *why* it exists before letting the compiler do it for you.
- **Not using the `eslint-plugin-react-hooks` rules**, especially `exhaustive-deps` and `set-state-in-effect` — these catch a large share of the bugs above automatically. Worth installing on day one of any real project.
- **Overusing Context for frequently-changing values** (e.g., mouse position, every keystroke) — every consumer re-renders on every change; Context is for low-frequency, broadly-needed values (theme, auth, locale), not high-frequency state.
- **Prop drilling vs. Context vs. composition** — reaching for Context before trying composition (passing components as children/props) for what's really a "pass this one level down" problem.

---

✅ **Checkpoint before moving to Next.js:** you should be able to build a non-trivial client-only SPA (e.g., a multi-step form with optimistic submission, context-based theme/auth state, and a few custom hooks) using nothing but React 19 + Vite — and you should be able to look at any `useEffect` in your own code and explain *why* it has to be an Effect rather than a render calculation or an event handler.

---

## PART 2 — NEXT.JS (App Router)

> Next.js docs explicitly recommend a React refresher before this section if you're rusty — which you won't be, having done Part 1.

### 13. Installation & Project Structure
- `create-next-app` CLI, TypeScript/ESLint setup, path aliases
- File and folder conventions: `app/`, `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, route groups `(folder)`, private folders `_folder`
- The distinction vs. the legacy **Pages Router** (`pages/`) — know it exists, know you're not using it

### 14. Routing Fundamentals
- Defining routes via folders, `page.tsx` per route
- Layouts (shared UI, nested layouts) vs. Pages
- Linking and navigating: `<Link>`, `useRouter`, `usePathname`
- Dynamic segments (`[id]`), catch-all (`[...slug]`), optional catch-all (`[[...slug]]`)
- Route Groups, Parallel Routes (`@slot`), Intercepting Routes (`(.)folder`)
- Loading UI & streaming (`loading.tsx` + automatic `<Suspense>`)
- Error handling (`error.tsx`, `global-error.tsx`)
- Navigation internals: prefetching, client-side navigation, prerendering behavior

### 15. Server and Client Components (the conceptual core of App Router)
- Why this split exists: server vs. client environment capabilities
- Server Components by default — fetch data, access backend resources directly, zero client JS cost
- `'use client'` directive — when and where to draw the boundary (push it as far down the tree as possible)
- Composition patterns: passing Server Components as `children`/props into Client Components (e.g., a `Modal` client wrapper containing a server-rendered `Cart`)
- What actually gets sent to the browser: HTML + RSC Payload + JS for Client Components only
- Hydration explained properly

### 16. Data Fetching
- `async`/`await` directly in Server Components
- Fetching with the extended `fetch` API, with ORMs/DB clients directly (safe — credentials never reach the client)
- Request memoization (de-duplicated identical `fetch` calls in one render pass)
- React's `cache()` function for non-fetch data access (DB/ORM calls) + the `preload()` pattern with `server-only`
- Client-side fetching (when you need it) — SWR / React Query
- Streaming with `<Suspense>` for slow/uncached data, and how it interacts with `loading.js`

### 17. Caching & Rendering Model
- The current model: static rendering vs. dynamic rendering, what triggers each
- **Cache Components** (`cacheComponents: true`) and the `"use cache"` directive — the newer caching model
- **Partial Prerendering (PPR)** — static shell + streamed dynamic holes, default behavior under Cache Components
- `cacheTag`, `revalidateTag` (including the newer `profile: "max"` stale-while-revalidate behavior), `revalidatePath`
- Previous/legacy caching model for context: `fetchCache`, segment config (`revalidate`, `dynamic`), `unstable_cache`
- Request-time APIs that opt routes into dynamic rendering: `cookies()`, `headers()`, `searchParams`, `connection()`

### 18. Mutating Data: Server Functions & Server Actions
- `'use server'` directive
- Defining Server Functions, calling them from `<form action={fn}>` and from Client Components
- Tying together React 19 Actions (`useActionState`, `useOptimistic`, `useFormStatus`) with real server mutations
- Revalidating after a mutation (`revalidatePath`/`revalidateTag`) so the UI reflects new data
- Error handling and redirects (`redirect()`, `notFound()`, `forbidden()`, `unauthorized()`)

### 19. Rendering Strategies in Practice
- Static Site Generation (SSG) equivalent under App Router
- Server-Side Rendering (SSR) equivalent
- `generateStaticParams` for pre-rendering dynamic routes at build time
- Incremental adoption / mixing strategies per-route

### 20. Styling
- CSS Modules, global CSS, Tailwind CSS integration
- CSS-in-JS options and trade-offs under Server Components (most require client boundaries)
- `next/font` for font optimization

### 21. Optimizing
- `next/image` (automatic optimization, responsive sizing, lazy loading)
- `next/script` (loading strategies for third-party scripts)
- Metadata API: static `metadata` export, `generateMetadata`, file-based metadata (`opengraph-image`, `icon`, `sitemap.ts`, `robots.ts`)
- `next/dynamic` for code-splitting/lazy-loading components
- Bundle analysis, `next.config.ts` performance options

### 22. Configuring
- `next.config.ts` essentials: `images`, `redirects`, `rewrites`, `headers`, `cacheComponents`, env vars (`NEXT_PUBLIC_*` vs server-only)
- TypeScript config specifics for Next.js
- `proxy.ts` (middleware-equivalent: matchers, rewriting/redirecting at the edge) — and excluding metadata files from its matcher
- Absolute imports & module path aliases

### 23. APIs You'll Use Constantly (Reference, but worth memorizing)
- Functions: `cookies()`, `headers()`, `draftMode()`, `fetch` (extended), `generateStaticParams`, `redirect`/`permanentRedirect`, `revalidatePath`/`revalidateTag`, `cacheTag`, `connection()`
- File conventions: `page`, `layout`, `template`, `loading`, `error`, `not-found`, `route.ts` (Route Handlers)
- Route Handlers (`route.ts`) — building actual REST endpoints inside `app/`, when to use this vs. a Server Action

### 24. Guides Worth Reading Once You're Building Real Apps
- Authentication (patterns, not a specific library)
- Testing (Next.js + Vitest/Playwright setup)
- Deploying (Vercel and self-hosting considerations)
- Upgrading between major versions + codemods
- SEO and JSON-LD structured data

### 25. Production Concerns
- Environment variables and secrets management
- Draft Mode / preview content
- Internationalization (i18n routing)
- Self-hosting vs. platform deployment trade-offs

### 25.5 Next.js Performance & Anti-Patterns Checklist
*(The Next.js-specific mistakes, on top of every React mistake above — React anti-patterns don't go away just because you're in a framework now.)*

- **Request waterfalls from nested sequential fetches.** This is *the* most-documented Next.js performance pitfall, including in Next's own learn tutorial. If component A fetches data and renders component B, which then fetches its own independent data, the second fetch doesn't start until A's finishes — even though the two requests have nothing to do with each other:
  ```tsx
  // 🔴 Waterfall: revenue fetch blocks invoices fetch unnecessarily
  const revenue = await fetchRevenue();
  const latestInvoices = await fetchLatestInvoices(); // waits for revenue first
  ```
  Fix with `Promise.all` (or `Promise.allSettled` if partial failure is acceptable) when the requests are genuinely independent:
  ```tsx
  // ✅ Parallel
  const [revenue, latestInvoices] = await Promise.all([
    fetchRevenue(),
    fetchLatestInvoices(),
  ]);
  ```
  Sequential fetching is *correct* when one request truly depends on the previous result (e.g., fetch a user, then fetch that user's posts) — the mistake is doing it by default, not doing it at all.
- **Marking too much as a Client Component.** Putting `'use client'` at the top of a large component (or a layout) drags everything underneath it into the client bundle and loses the "fetch and render on the server, ship zero JS" benefit. Push `'use client'` as far down the tree as possible — onto the one button or input that actually needs interactivity, not its whole parent section.
- **Not wrapping slow/uncached data in `<Suspense>`.** An `await` in a Server Component blocks rendering of everything below it unless it's isolated behind a `<Suspense>` boundary (or covered by `loading.tsx`). One slow, uncached data source can stall an entire page that would otherwise be instant.
- **Fetching the same data in multiple components without relying on memoization/`cache()`.** `fetch` calls are automatically de-duplicated within a render pass, but direct DB/ORM calls are not — wrap those in React's `cache()` or you'll issue redundant queries every time a shared piece of data is needed by multiple components in the tree.
- **Leaking sensitive data to the client.** Returning a full DB row (including internal fields, tokens, etc.) from a Server Component into a Client Component prop serializes all of it into the client bundle. Strip to only the fields the client actually needs, and use the `server-only` package to guarantee a module can never accidentally be imported into client code.
- **Defaulting to client-side fetching (`useEffect` + `fetch`, or even SWR/React Query) for data that could be fetched once on the server.** This re-introduces the loading-spinner-on-every-navigation problem that Server Components exist to solve. Reach for client-side fetching only when the data is genuinely client-only in nature (depends on browser state, needs frequent polling/revalidation independent of navigation, etc.).
- **Not tagging/revalidating caches after mutations.** A Server Action that mutates data but never calls `revalidatePath`/`revalidateTag` (or doesn't use `cacheTag` correctly under Cache Components) leaves the UI showing stale data until the cache naturally expires.
- **Using `next/image` incorrectly (or not at all)** — skipping it for hero/above-the-fold images, not setting `priority` where needed, or using raw `<img>` for content images and losing automatic format/size optimization.
- **Ignoring the Metadata API and shipping bad SEO/social previews** by hand-rolling `<head>` tags instead of `generateMetadata`/file-based metadata conventions.

---

✅ **Final checkpoint:** build one full-stack app that touches every layer — nested layouts, a dynamic route with `generateStaticParams`, a Server Component fetching from a real DB, a mutation through a Server Action with `useActionState` + `useOptimistic`, tagged revalidation, and `next/image` + Metadata API for a polished, SEO-correct result. That's the point where "knows React" becomes "production Next.js engineer." Before you call it done, run through the two checklists above against your own code — that's the actual skill, not just having built something that works once.

---

## Quick reference: official doc entry points
- React Learn: https://react.dev/learn
- React Hooks Reference: https://react.dev/reference/react/hooks
- React 19 release notes: https://react.dev/blog/2024/12/05/react-19
- React Rules: https://react.dev/reference/rules
- Next.js App Router Getting Started: https://nextjs.org/docs/app/getting-started
- Next.js Guides: https://nextjs.org/docs/app/guides
- Next.js API Reference: https://nextjs.org/docs/app/api-reference
