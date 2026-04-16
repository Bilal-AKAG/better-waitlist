# Better-Waitlist — Development Plan for AI Coding Agents

## Project Description

**Better-Waitlist** is a modern, fully open-source, self-hosted waitlist builder designed for creators, indie hackers, product makers, and startups.

It enables anyone to quickly create and manage waitlists for their upcoming product launches, newsletters, apps, SaaS tools, communities, or services. Users can collect email signups, track interested people, and eventually turn their waitlist into a powerful pre-launch marketing tool.

### Why This Project Exists
Many popular waitlist tools (like Prefinery, Waitlist, LaunchList, etc.) are closed-source SaaS products with monthly fees and data lock-in.  
Better-Waitlist gives developers and creators full control:
- Run it completely on your own server (self-hosted)
- Own all your data
- Customize everything
- Use it for free forever, or offer a convenient hosted version

The goal is to build one of the best-looking, most developer-friendly, and feature-rich open-source waitlist solutions available.

### Core Value Proposition
- Simple and beautiful interface for managing waitlists
- Powerful viral features (referrals, leaderboards, shareable links) planned for future phases
- Support for custom branding and embeddable widgets
- Multi-tenancy support so one instance can serve many customers (subdomains + custom domains)
- Easy deployment with Docker

---

## Current Development Phase: Minimal Viable Prototype

We are intentionally starting small to create a solid foundation.

**Phase 1 – Prototype Focus (What We Build Now):**

Only the essential core features will be implemented:

1. **Authentication**  
   - User registration and login using Better Auth (email + password)

2. **Protected Admin Dashboard**  
   - Secure `/admin` area accessible only to logged-in users

3. **Waitlist Management**  
   - Create new waitlists (with name and unique slug)
   - List all my waitlists
   - View details of a specific waitlist
   - Delete a waitlist

4. **Signup Entries Management**  
   - View all email signups for a particular waitlist in a clean table
   - Show basic information: email, name (optional), and signup date

**Explicitly Out of Scope for This Phase:**
- Public-facing waitlist page (the page visitors see)
- Referral system and viral mechanics
- Email sending and double opt-in
- Templates and custom branding
- Multi-tenancy / subdomain / custom domain support
- Analytics, CSV export, or advanced UI polish

Once this prototype is stable and working, we will gradually add the more exciting features in future phases.

---

## Tech Stack

- **Monorepo**: Turborepo
- **Framework**: TanStack Start (React 19 + TanStack Router + Server Functions)
- **Authentication**: Better Auth (with Drizzle adapter)
- **Database**: Drizzle ORM + PostgreSQL (Neon)
- **Styling**: Tailwind CSS + shadcn/ui
- **Package Manager**: Bun
- **Primary App Folder**: `apps/web/`

---

## Database Schema (Minimal for Prototype)

**File Location:** `packages/db/schema.ts` (or `apps/web/db/schema.ts`)

```ts
import { pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const waitlists = pgTable('waitlists', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').notNull(),           // Owner (links to Better Auth user)
  name: text('name').notNull(),
  slug: text('slug').unique().notNull(),       // Used later for URLs like slug.yourdomain.com
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const waitlistEntries = pgTable('waitlist_entries', {
  id: uuid('id').primaryKey().defaultRandom(),
  waitlistId: uuid('waitlist_id')
    .references(() => waitlists.id, { onDelete: 'cascade' })
    .notNull(),
  email: text('email').notNull(),
  name: text('name'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// Relations
export const waitlistRelations = relations(waitlists, ({ many }) => ({
  entries: many(waitlistEntries),
}));

export const entryRelations = relations(waitlistEntries, ({ one }) => ({
  waitlist: one(waitlists, {
    fields: [waitlistEntries.waitlistId],
    references: [waitlists.id],
  }),
}));