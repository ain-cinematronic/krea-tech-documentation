# Development Guide

This document covers everything you need to know about developing with KreaTech, from basic setup to advanced features.

## Table of Contents

1. [Quick Start](#quick-start)
2. [Environment Configuration](#environment-configuration)
3. [Core Features](#core-features)
4. [Admin Panel Customization](#admin-panel-customization)
5. [Content Management](#content-management)
6. [Security & Access Control](#security--access-control)
7. [Performance Optimization](#performance-optimization)
8. [Advanced Topics](#advanced-topics)
9. [Ratings System](#ratings-system)
10. [Logger & Diagnostics](#logger--diagnostics)
11. [Database Backup & Restore](#database-backup--restore)

---

## Quick Start

### Prerequisites

- **Node.js** - [Download here](https://nodejs.org/en/download/)
- **Yarn** - [Install guide](https://yarnpkg.com/getting-started/install)
- **Docker** - [Get Docker](https://docs.docker.com/get-docker/)

### Setup Steps

1. **Install dependencies**  
   - [Node.js](https://nodejs.org/en/download/)  
   - [Yarn](https://yarnpkg.com/getting-started/install)  
   - [Docker](https://docs.docker.com/get-docker/)  

2. **Start MongoDB with Docker**

   ```sh
   docker run -d -p 27017:27017 --name krea-tech-mongo mongo
   ```

3. **Configure environment**

   ```sh
   # Unix/macOS
   cp .env.example .env.local
   
   # Windows CMD
   copy .env.example .env.local
   ```

4. **Set environment variables** (see [Environment Configuration](#environment-configuration))

5. **Start development server**

   ```sh
   yarn dev
   ```

6. **Setup admin user**
   - Visit [http://localhost:8000/admin](http://localhost:8000/admin)
   - Register admin user (role: `Admin`)
   - Go to **API Keys** collection
   - Create API key: `Enable: true`, `Name: Next.js API Key`, `Role: Viewer`
   - Copy API key to `NEXT_PAYLOAD_API_KEY` in `.env.local`
   - **Restart server**: `Ctrl+C`, then `yarn dev`

  **Application ready at [http://localhost:8000](http://localhost:8000)**

---

## Environment Configuration

### Required Variables

```bash
# Payload CMS
PAYLOAD_PORT=8000
PAYLOAD_PUBLIC_URL=http://localhost:8000
PAYLOAD_PUBLIC_NEXT_URL=http://localhost:8000  # For live preview
PAYLOAD_SECRET=<SECRET_STRING>
PAYLOAD_PUBLIC_DRAFT_SECRET=<SECRET_STRING>
MONGODB_URI=mongodb://<host>/<db>

# Next.js
NEXT_PUBLIC_URL=http://localhost:8000
NEXT_PAYLOAD_API_KEY=<FROM_ADMIN_PANEL>
```

### URL Variables Explained

| Variable | Purpose | Used For |
|----------|---------|----------|
| `PAYLOAD_PUBLIC_URL` | Canonical Payload server URL | CORS/CSRF, serverURL, GraphQL |
| `PAYLOAD_PUBLIC_NEXT_URL` | **Live preview base URL** | Preview links in admin panel |
| `NEXT_PUBLIC_PAYLOAD_SERVER_URL` | Browser API calls override | Client-side API targeting |
| `NEXT_PUBLIC_URL` | Generic site base (fallback) | Legacy compatibility |

### Live Preview Configuration

**Critical for content editors**: `PAYLOAD_PUBLIC_NEXT_URL` generates preview links when editors click "Preview" in the admin.

**Examples:**

**Single domain (development):**

```bash
PAYLOAD_PUBLIC_URL=http://localhost:8000
PAYLOAD_PUBLIC_NEXT_URL=http://localhost:8000
```

**Split domains (production):**

```bash
PAYLOAD_PUBLIC_URL=https://cms.krea-tech.eu       # Admin/API
PAYLOAD_PUBLIC_NEXT_URL=https://krea-tech.eu      # Public site
```

### URL Resolution Priority

`getPayloadServerURL()` checks in this order:

1. `NEXT_PUBLIC_PAYLOAD_SERVER_URL`
2. `PAYLOAD_PUBLIC_URL`  
3. `NEXT_PUBLIC_URL`
4. `PAYLOAD_SERVER_URL` (server-only)
5. **Fallback**: `http://localhost:8000`

---

## Core Features

### Live Preview System

Content editors can preview changes in real-time on the public website.

**How it works:**

1. Editor clicks "Preview" in Payload admin
2. System generates URL using `PAYLOAD_PUBLIC_NEXT_URL`
3. Draft content accessible via a secret-protected preview route.
4. Changes appear immediately in preview window

**Collections with live preview:**

- **Courses**: `/courses/{id}`
- **Modules**: `/modules/{id}`
- **Lessons**: `/lessons/{id}`
- **News**: `/news/{slug}`

**Globals with live preview:**

- **Homepage**: `/`
- **Courses Overview**: `/courses`
- **News Overview**: `/news`

**UI Configuration:**

- **Live Preview button is permanently disabled** by removing `livePreview` configuration from all collections and globals
- Only the standard "Preview" button (using `/api/preview`) is available to content editors
- **Preview window reuse**: Custom `ReusePreviewWindow` component intercepts preview button clicks to reuse the same browser window/tab instead of opening new ones each time

**Troubleshooting:**

- **Preview links broken?** Check `PAYLOAD_PUBLIC_NEXT_URL` points to frontend
- **Shows published instead of draft?** Verify `PAYLOAD_PUBLIC_DRAFT_SECRET` matches
- **API endpoint missing?** Ensure preview route exists

### Quiz System

Interactive quizzes attached to lessons with client-side rendering.

**Question Types:**

- `singleChoice` - One correct answer
- `multipleChoice` - Multiple correct answers  
- `slider` - Numeric answer within tolerance

**Components:**

- `LessonQuizSection.tsx` - Collapsible wrapper
- `QuizRenderer.tsx` - Question rendering and scoring

**Adding new question types:**

1. Extend blocks union in `Quizzes` collection
2. Run `yarn generate:types`
3. Update GraphQL lesson query
4. Add UI render branch in `QuizRenderer.tsx`

### Learning Progress Tracking

Client-side cookie tracks last visited lesson for "Continue Learning" features.

**Cookie format:** `lessonId|moduleId|courseId` (30 days)

**Utilities:** `src/app/_utils/learningProgress.ts`

- `setLastLesson({ lessonId, moduleId?, courseId? })`
- `getLastLesson()`
- `clearLastLesson()`

**Used by:**

- Course overview "Continue" banners
- Homepage start banner upgrades
- Navbar continue button

---

## Admin Panel Customization

### Dynamic Theming

SystemAdmin users can customize admin panel appearance.

**Access:** SystemAdmin → System Globals → Admin Panel Theme

**Features:**

- Preset colors (Visual Yellow, Accent Blue, Visual Black, Light Grey)
- Custom hex colors with validation
- Real-time updates without restart
- Security: CSS injection prevention

**Files:**

- `src/payload/globals/AdminPanelTheme.ts` - Configuration
- `src/payload/admin/components/DynamicAdminTheme.tsx` - CSS injection
- `src/payload/admin/utils/themeCache.ts` - Fetch caching

### Static CSS Overrides

Custom admin CSS hides Payload's block rename inputs for cleaner UX.

**File:** `src/payload/admin-overrides.css`
**Registered in:** `payload.config.ts` → `admin.css`

**Purpose:** Remove per-block label inputs that clutter the admin interface.

### Performance Optimizations

**ListPreloader:**

- Preloads collection preferences on startup
- Caches in sessionStorage
- Eliminates duplicate API calls

**VersionsDelayer:**

- Delays versions polling by 5 seconds
- Reduces redundant delta calls

**DocumentEditOptimizer:**

- Short-circuits list queries with single ID
- Improves edit view navigation

**GraphQL Coalescing:**

- Deduplicates concurrent identical requests
- Automatic cleanup after completion

---

## Content Management

### Content Editing Constraints

**Description field limits:** 800 characters for Courses, Modules, and Lessons.

**Locations:**

- `src/payload/collections/Courses.ts`
- `src/payload/collections/Modules.ts`
- `src/payload/collections/Lessons.ts`

**Implementation:** Validate callback with friendly error showing current length.

### News Slug Lifecycle

Auto-generated slugs with smart regeneration:

1. **Create:** Slug computed from initial title
2. **Draft:** Title changes regenerate slug if base segment changed
3. **Published:** Slug becomes immutable (prevents broken links)
4. **Collisions:** Auto-append `-2`, `-3`, etc.

### Rich Text Editor Harmonization

All content types now use consistent rich text capabilities:

### Content Rich Text Editors

All major content types now use the same editor:  
`ContentRichTextEditorWithBlocks` — supporting media embedding and accordion blocks.

- **Courses Overview**  
- **Modules**  
- **Lessons**  

**Consistent Features Across All Content:**

- Rich text formatting (bold, italic, links, etc.)
- Media blocks (images, videos)
- Accordion/collapsible sections
- Table support
- Consistent typography using Roboto font system

### Ownership & Institution Tracking

**Pattern for all content:**

- `institution` field auto-assigned on create
- `createdBy` field auto-set once (never overwritten)
- `OwnershipGuard` UI warns on cross-institution edits

**Editor access model:**

- **Read:** All content across institutions
- **Edit/Delete:** Own institution content only
- **Create:** Auto-assigned to editor's institution

---

## Security & Access Control

### Role System

Three-tier model with automatic promotion:

- **SystemAdmin:** Full system access, immutable role
- **Admin:** Institution management, user creation  
- **Editor:** Content creation within institution

**First user:** Automatically promoted to SystemAdmin
**SystemAdmin protection:** Cannot be deleted or demoted

### Geo-IP Login Blocking

Minimal middleware for admin login control:

**Environment variables:**

```bash
GEO_FORCE_BLOCK=1                    # Block all logins
GEO_OVERRIDE_COUNTRY=<country-codes>      # Allowlist countries
```

**Scope:** POST `/api/users/login` only
**Location:** `src/server.ts` (search "geo-policy")

### URL Security

Centralized URL sanitization prevents injection attacks:

**File:** `src/app/_utils/url.ts`
**Functions:**

- `sanitizeUrl()` - Validate and normalize URLs
- `getLinkProps()` - Safe anchor props for external links

**Security features:**

- Protocol whitelist (http, https, mailto only)
- IP address rejection
- IDN/punycode rejection  
- Userinfo credential blocking

### Same-origin Checks (CSRF)

The ratings POST endpoint applies a basic same-origin guard compatible with anonymous usage and zero cooldown:

- Compares `Origin`/`Referer` headers against `Host`; rejects if cross-site.
- Does not require CSRF tokens to keep the flow simple for anonymous devices.
- Applies to the ratings write endpoint.

---

## Performance Optimization

### Admin Interface

- **Request deduplication:** Shared promise caching
- **Preference preloading:** SessionStorage caching
- **Version polling delays:** 5-second startup delay
- **GraphQL coalescing:** In-flight request merging

### Frontend

- **Static generation:** Pre-rendered pages where possible
- **Image optimization:** Next.js automatic optimization
- **Bundle splitting:** Code splitting by route

### Monitoring

Request logging shows optimization effectiveness:

```javascript
[req] GET /api/payload-preferences/news-list -> 200 28.5ms  // First
[req] GET /api/payload-preferences/news-list -> 304         // Cached
```

---

## Advanced Topics

### Quiz Background Effects

Decorative floating shapes on wide screens (≥1500px).

**Component:** `QuizBackground.tsx`
**Behavior:** Disabled below threshold to avoid mobile clutter
**Customization:** Adjust `minWidthPx` prop or `count` for fewer shapes

### Institution Ownership

**Collections with ownership:**

- Media: Tags, Images, FileResources
- Content: Authors, Courses, Modules, Lessons, News

**Access helpers** (in `collectionAccess.ts`):

- `AdminOrSameInstitutionAccess` - Boolean institution check
- `ReadByInstitutionAccess` - Row-scoped reading
- Note: Currently unused, available for future collections

### Adding New Features

**New collections:**

1. Create in `src/payload/collections/`
2. Add to `payload.config.ts`
3. Run `yarn generate:types`
4. Add GraphQL queries in `src/app/_graphql/`

**New admin components:**

1. Create in `src/payload/admin/components/`
2. Register in `payload.config.ts` → `admin.components`
3. Use `useEffect` for client-side logic

**New quiz question types:**

1. Extend blocks union in Quizzes collection
2. Regenerate types
3. Update GraphQL fragments
4. Add render logic to QuizRenderer

### Troubleshooting

**Common issues:**

- **TypeScript errors after schema changes:** Run `yarn generate:types`
- **GraphQL errors:** Check query fragments match schema
- **Preview links broken:** Verify environment URL variables
- **Admin performance slow:** Check request logs for duplicate calls
- **Access denied errors:** Review role and institution assignments
- **Font inconsistencies:** Check for conflicting font imports in `layout.tsx`

---

## Development Commands

```bash
# Development
yarn dev                     # Start dev server
yarn build                   # Production build
yarn start                   # Start production server

# Payload
yarn generate:types          # Update TypeScript types
yarn generate:graphQLSchema  # Update GraphQL schema

```

## File Structure Reference

## Ratings System

The platform supports anonymous, per-device course ratings using a signed identifier cookie and one-way hashing to preserve privacy.

**Endpoints (App Router):**

- `POST /kapi/ratings` → Create or update the current device’s rating; includes a same-origin check.
- `GET /kapi/ratings/me?courseId=` → Returns the rating previously submitted from the current device.
- `GET /kapi/course-ratings?courseId=` → Provides `{ averageRating, totalRatings }` for UI display.

**Server logic:**

- Each device receives a signed identifier cookie, validated with a server secret.
- A one-way hash of the identifier is stored; raw values are never persisted.
- Ratings are upserted by `(course, deviceHash)` → update if found, otherwise insert.
- Database enforces a compound unique index on `(course, deviceHash)`.
- The `deviceHash` is immutable once created.

**Client/UI:**

- The banner retrieves the user’s prior rating via `ratings/me`.
- Aggregated results (average + count) are fetched via `course-ratings`.

src/
├── app/                     # Next.js app router
│   ├── _api/               # Data fetching utilities
│   ├──_components/        # React components  
│   ├── _graphql/           # GraphQL queries
│   ├──_utils/             # Shared utilities
│   └── (pages)/            # Route pages
├── payload/                # Payload CMS config
│   ├── collections/        # Content collections
│   ├── globals/           # Global settings
│   ├── access/            # Access control
│   └── admin/             # Admin customizations
└── @types/                # TypeScript definitions
