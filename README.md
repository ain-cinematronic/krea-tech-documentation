# KreaTech Documentation

Welcome to the comprehensive documentation for KreaTech, a learning platform built with **Next.js + Payload CMS**. This documentation covers everything from initial setup to advanced features and maintenance.

---

## Documentation Index

### Core Setup & Development

- **[Development Guide](./development.md)**  
  Complete development environment setup, commands, troubleshooting, and workflows

### Data & Content Management

- **[Data Models](./data-models.md)**  
  Collections, globals, relationships, access control, and content management patterns

- **[Theme Guide](./theme-guide.md)**  
  Design system, colors, typography, button variants, and component styling guidelines

### Deployment & Operations

- **[Deployment Guide](./deployment.md)**  
  Production deployment to Payload Cloud, environment configuration, and live preview setup

- **[MongoDB Backup & Restore](./mongodb-backup-restore.md)**  
  Database backup procedures, local development data management, and restore workflows

### Features & Security

- **[Cookie Consent Implementation](./cookie-consent.md)**  
  Cookie banner + optional analytics category logic and GA integration

---

## Architecture Overview

KreaTech follows a modern full-stack architecture:

- **Frontend**: Next.js 14+ with App Router
- **CMS**: Payload CMS for content management
- **Database**: MongoDB for data storage
- **API**: GraphQL for data fetching
- **Styling**: Chakra UI with custom theme system
- **Deployment**: Payload Cloud with Docker support

### System Architecture (High Level)

Written C4 model (concise) instead of an ASCII box drawing.

```text
LEVEL 1 – SYSTEM CONTEXT
-----------------------------------------------------------------------
People:
  - Learner: browses public site, consumes course content, submits ratings.
  - Content Editor: manages courses, modules, lessons, quizzes, media via Admin UI.
  - SystemAdmin: full governance (analytics, theme, geo policy, platform settings).

Primary System:
  - KreaTech Platform (this system) delivering learning content and admin tooling.

External Systems / Services:
  - Google Analytics (GA4) – receives events only after optional analytics consent.
  - Payload Cloud (hosting environment) – runs Payload CMS + Next.js deploy (infrastructure context).
  - Sentry – errors and telemetry 


LEVEL 2 – CONTAINER VIEW
-----------------------------------------------------------------------
Containers:
  1. Browser (Client) – Renders React UI; minimal client logic (progress cookie, quiz state).
  2. Next.js Application – App Router (server components + route handlers), SSR/ISR, internal API proxy endpoints.
  3. Payload CMS – Express server providing GraphQL + REST APIs and Admin UI; enforces access & hooks.
  4. MongoDB – Persistence for collections, globals, media metadata.
  5. Google Analytics – External analytics endpoint triggered after consent.
  6. Sentry – Telemetry

Data / Call Flows:
  Browser → Next.js (HTTP) : page requests (SSR) & minimal API fetches.
  Next.js → Payload CMS (HTTP GraphQL/REST with API key) : fetchDoc/fetchDocs/fetchGlobals utilities.
  Payload CMS → MongoDB (driver) : CRUD + access/hook execution.
  Browser → GA4 : gtag events only when consented (analytics category enabled + user consent).

Key Boundaries:
  - Security: Access rules & hooks live in Payload; Next.js never queries Mongo directly.
  - Privacy: Analytics container decoupled; consent gating before GA script injection.

LEVEL 3 – COMPONENT VIEW (SELECTED CONTAINERS)
-----------------------------------------------------------------------
Next.js Application Components:
  - Route Handlers (internal API proxy endpoints) – Proxy rating + feedback endpoints, preview, revalidation.
  - Server Components – Render pages; call data layer during SSR.
  - Data Fetch Layer – fetchGraphQL.ts + fetchDoc/fetchDocs/fetchGlobals wrappers adding auth header.
  - Utility Layer – learningProgress, rating banner client enhancer.

Payload CMS Components:
  - Collections & Globals – Schema definitions (courses, modules, lessons, quizzes, ratings, etc.).
  - Access Control Layer – Functions restricting CRUD by role/institution.
  - Hooks – beforeChange / beforeValidate / onInit (indexes, ownership, hashing, publish timestamps).
  - Admin UI Extensions – Dynamic theme, ownership guard, fetch optimizers.
  - GraphQL API – Single endpoint consumed by Next.js.

Cross-Cutting Concerns:
  - Ownership & Institution Pattern – Shared utilities to apply consistent access & metadata.
  - Preview Flow – Standard preview route; live preview button disabled.
  - Logging – Verbosity gated logger (sanitized headers) in server utilities.
  - Privacy – Cookie consent logic controls GA script load + consent mode update.

```

### Content Hierarchy

```text
        +-------------------+
        |      Courses      |
        +----------+--------+
                |
              (has)
                v
        +-------------------+
        |      Modules      |
        +----------+--------+
                |
              (has)
                v
        +-------------------+
        |      Lessons      |
        +----------+--------+
                | 
                v
             +-------+
             | Quiz  |
             +-------+

Supporting Collections:
  Tags, Authors, Images, FileResources, News, Ratings, Feedback
Globals drive: NavBar, Footer, CoursesOverview, NewsOverview, FrontEndTheme, AdminPanelTheme, Cookie/Analytics, FeedbackPage
```

### Server-Side Data Fetch Flow (Simplified)

```text
Request (HTTP)
  │
  ▼
Next.js Route / Server Component
  │  uses fetchDoc / fetchDocs / fetchGlobals
  ▼
fetchGraphQL wrapper
  │  (adds auth header w/ API key)
  ▼
Payload GraphQL Endpoint
  │  (resolves access rules, hooks)
  ▼
MongoDB
  │  (documents returned)
  ▼
Payload Response JSON
  │
  ▼
Next.js renders HTML (RSC) / streams
  │
  ▼
Browser displays page; optional client enhancements
```

---


## Key Features

### Content Management

- **Rich Text Editing** with media blocks and accordions
- **Preview System** using the standard preview route.
- **Institution-based Ownership** with role-based access control
- **Quiz System** with multiple question types
- **Learning Progress Tracking** with cookie-based persistence

### Security & Privacy

- **Anonymous Rating System**
- **Cookie Consent Management** with GDPR compliance
- **Geographic Access Control** for admin panel security
- **Role-based Permissions** with three-tier user model

### Developer Experience

- **TypeScript** throughout with auto-generated types
- **GraphQL** with schema generation and coalescing
- **Component Library** with consistent design system

---

## Documentation Standards

Our documentation follows these principles:

- **Comprehensive**: Cover setup, usage, troubleshooting, and maintenance
- **Practical**: Include copy-paste commands and real examples
- **Current**: Reflect the latest codebase and deployment practices
- **Structured**: Logical organization with clear cross-references
- **Accessible**: Written for different skill levels and roles


---

## Contributing to Documentation

When updating documentation:

1. **Keep it current** - Update docs when making code changes
2. **Include examples** - Provide practical, copy-paste ready examples  
3. **Cross-reference** - Link to related sections in other documents
4. **Test instructions** - Verify setup steps work on fresh environments
5. **Consider your audience** - Write for different skill levels and roles

---

## External Resources

- **[Payload CMS Documentation](https://payloadcms.com/docs)**
- **[Next.js Documentation](https://nextjs.org/docs)**
- **[Chakra UI Documentation](https://chakra-ui.com/docs)**
- **[MongoDB Documentation](https://docs.mongodb.com/)**

