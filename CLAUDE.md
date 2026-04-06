# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start Vite dev server
npm run build     # Production build
npm run lint      # ESLint
npm run preview   # Preview production build locally
```

No test runner is configured.

### Local Supabase

```bash
supabase start          # Start local Supabase (API at http://127.0.0.1:54321, Studio at http://localhost:54323)
supabase db reset       # Re-apply all migrations from scratch
supabase migration new <name>   # Create a new migration file
supabase db push        # Push migrations to remote project
```

The Supabase client in `src/integrations/supabase/client.ts` automatically switches between local (`127.0.0.1:54321`) and production URLs based on `window.location.hostname`.

## Architecture

**Stack:** React 18 + TypeScript + Vite SPA. No SSR. All routing is client-side via React Router v6.

**Backend:** Supabase (Postgres + Auth + Storage + Realtime). The generated type definitions live in `src/integrations/supabase/types.ts` — do not edit manually; regenerate with `supabase gen types typescript`.

### Database schema (key tables)
- `products` — listings with fields: `product_name`, `price`, `description`, `color`, `leather`, `stamp`, `year_purchased`, `location`, `user_id`
- `product_images` — one-to-many images per product (`image_url`, `product_id`)
- `profiles` — user profiles keyed by `user_id`
- `conversations` — tied to a `product_id`; participants tracked in `participants` table
- `messages` — belong to a `conversation_id`, with read receipts in `message_status`
- `saved_products` — user-saved/bookmarked listings

### Data fetching
All server state uses **TanStack Query** (`@tanstack/react-query`). Custom hooks in `src/hooks/` wrap Supabase queries:
- `useAuth` — session/user state via `supabase.auth.onAuthStateChange`
- `usePublicProducts`, `useUserProducts`, `useUserSavedProducts`, `useSavedProducts` — product queries
- `useUnreadMessagesCount` — unread message badge count

### Realtime
`PresenceProvider` (`src/contexts/PresenceProvider.tsx`) wraps the app and tracks online users via Supabase Realtime presence on the `global-presence` channel. Consume with `usePresence()`.

### UI
shadcn/ui components (in `src/components/ui/`) built on Radix UI primitives + Tailwind CSS. Path alias `@/` maps to `src/`.

### Routing (App.tsx)
| Path | Page |
|---|---|
| `/` | Categories |
| `/categories` | Categories |
| `/product/:id` | ProductDetail |
| `/create-listing` | CreateListing (pick type) |
| `/create-listing/new` | CreateListingForm |
| `/create-listing/edit/:id` | CreateListingForm (edit mode) |
| `/messages` | ConversationList |
| `/conversation/:conversationId` | ConversationDetail |
| `/saved-items` | SavedItems |
| `/profile` | Profile |
| `/auth` | Auth (sign in/up/reset) |
