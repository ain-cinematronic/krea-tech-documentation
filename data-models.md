# Data Models – KreaTech

## AdminPanelTheme

**Purpose**: Customize the visual appearance of the admin panel.

**Access & Security**:

- **SystemAdmin Only**: Only users with `SystemAdmin` role can view or modify theme settings
- **Hidden from other roles**: Admin UI hides this global from Admin/Editor users
- **Fallback**: Non-SystemAdmin users automatically use default theme

**Fields**:

- **Save Button Color**: Choose the color scheme for save buttons
  - Visual Yellow — with black text
  - Accent Blue — with white text  
  - Visual Black — with yellow text
  - Light Grey — with black text
  - Custom Color — Uses the custom hex value below
- **Custom Button Color**: Hex color code field (e.g., `#FF5733`)

**Implementation**:

- Dynamic CSS injection applies styles from database settings
- Changes apply immediately without requiring server restart
- Custom colors validated to prevent CSS injection
- Text colors adjusted for readability

**Behavior**:

- Default theme uses Visual Yellow if no settings exist
- Hover states generated automatically
- Falls back to default if fetch fails or user lacks permissions

**Location in Admin**: System Globals → Admin Panel Theme (SystemAdmin only)

---

### FrontEndTheme

**Purpose**: Configure visual settings for dynamic content hero sections.

**Access & Security**:

- **Admin/SystemAdmin Only**: Can view or modify settings
- **Public Read Access**: Settings are publicly readable
- **Hidden from Editors**

**Fields**:

- **Background Blur Intensity (%)**: Range 0–100% (maps to CSS blur up to 20px)
- **Dynamic Hero Overlay**: Color and opacity settings

**Implementation**:

- Only affects dynamic content pages (Course, Module, Lesson)
- Homepage and overview pages manage styles independently
- Settings fetched server-side and cached briefly

**Behavior**:

- Changes apply within seconds
- Defaults used if settings are missing or fetch fails

**Location in Admin**: System Globals → Front End Theme

---

### Social Media (Footer Global Integration)

Configured inside the Footer global under `socialMediaIcons`.

Allowed types: GitHub, LinkedIn, Twitter, Instagram, Facebook, YouTube, TikTok, Email, Website.

**Fields**:

- `type`: Platform (required)
- `href`: Destination URL or mailto (required)
- `label`: Optional accessible label

**Access**:

- Read: Public (footer visible on every page)
- Update: Admin or SystemAdmin

Notes:

- Array order = render order
- Email should use `mailto:` scheme
- If empty, section collapses gracefully

---

## Overview

Collections = multi-document content types.  
Globals = singletons for configuration and theming.

---

## Content Relationships & UI Integration

### Creating UI Components

General steps:

1. Define data structure in CMS (collection/global)
2. Write a fetch utility
3. Connect the route
4. Build the UI component

### Data Flow

1. Creators add lessons, modules, courses in CMS
2. CMS stores content with references
3. Frontend requests via API (never DB directly)
4. Fetch utilities retrieve required data
5. CMS responds with JSON (with relationships)
6. Components render content dynamically

---

## Collections

### Description Length Policy

- Courses, Modules, Lessons descriptions limited to 800 characters
- Enforced via validation in CMS collections

### Courses

- Title: String
- Description: String (max 800 chars)
- Tags: Array
- Thumbnail: Image
- Modules: Array
- Overview: RichText with Blocks (supports media, accordions)
- Prerequisites: Array
- What you will learn: Array
- Accordions: Array
- Rating: Array
- PublishedAt: Date
- Duration: Number (calculated)

### Modules

- Title, Description, Thumbnail
- Learning Objectives: Array
- Lessons: Array (ordered)
- Exercise: Optional
- Content: RichText with Blocks
- Resources: Array of files or links
- Duration: Number (calculated)
- Parent course relationship not stored — derived via query params

### Lessons

- Title, Description, Duration
- Content: RichText with Blocks
- Resources: Array
- Quiz: Optional (1:1 relationship)

### News

**Purpose**: Platform or product updates

**Fields**:

- Title, Slug (auto-generated, immutable after publish)
- Summary: String
- Content: RichText with Blocks
- PublishedAt: Date

**Access**:

- Read: Public
- Create: Admin/Editor (institution auto-assigned)
- Update/Delete: Admin unrestricted, Editors limited

**Behavior**:

- Slug frozen after publish
- Supports draft/published workflow
- Frontend routes: `/news`, `/news/[slug]`

### Quizzes

Attached to a single Lesson.

**Fields**:

- Title
- Questions (blocks: single choice, multiple choice, slider)
- CreatedBy, Institution, PublishedAt

**Access**:

- Read: Public
- Create/Update: Admin/Editor (within institution)
- Delete: Admin only

**Behavior**:

- One quiz per lesson
- Session-only (not persisted yet)
- Extensible for future types

### Tags

- Title: String
- CreatedBy, Institution auto-assigned
- Access: Public read, Admin unrestricted, Editors only within institution

### Authors

- Name, Avatar, Bio, Background, Contact
- Access mirrors other collections

### FAQs

- Question: String
- Answer: RichText

### Ratings

Anonymous, per-device course ratings.

**Fields**:

- Course
- Device hash (anonymized)
- Rating (1–5, halves allowed)
- IP/UserAgent/Metadata (admin-only)

**Access**:

- Create: Public (with signed identifier)
- Read: Public (sensitive fields hidden)
- Update/Delete: Restricted to SystemAdmin or trusted server

**Endpoints**:

- `POST /ratings` → Upsert
- `GET /ratings/me` → Current device rating
- `GET /course-ratings` → Aggregate

### Feedback

Simple feedback submissions at `/feedback`.

**Fields**:

- Name, Message (required)
- IP hash, UserAgent, Metadata (admin-only)

**Access**:

- Create: Public
- Read: Public (safe fields only)
- Update/Delete: SystemAdmin only

**Flow**:

- User submits form → server validates & stores → admin-only metadata

### Users

- Name, Email, Password
- Role: system-admin | admin | editor
- Institution relationship

**Role Semantics**:

- SystemAdmin: Full control, cannot be removed
- Admin: Manage content & users (not SystemAdmin)
- Editor: Content only

**Access Rules**:

- One SystemAdmin enforced
- Deletion/self-deletion restrictions
- Institution scoping applies

### Blocked Countries

- Stores 2-letter country codes to block admin logins
- Access: SystemAdmin only
- Used to restrict by country; broader perimeter controls recommended

### File Resources

- File + metadata
- CreatedBy & Institution auto-set
- Access: Public read, Admin unrestricted, Editors within institution

### Images

- Image + metadata
- CreatedBy & Institution auto-set
- Access: Public read, Admin unrestricted, Editors within institution

### Ownership & Creator Utilities

Shared helpers ensure:

- Institution & creator auto-assigned
- Editors limited to own institution
- Ownership banners shown in UI
- Orphan docs flagged for admin action

---

## Globals

### Footer

Configures footer layout.

**Fields**:

- Logo, supporting logo
- Texts (captions, copyright)
- Links[] (custom or static page references)
- Social media icons[]

Access: Public read, Admin/SystemAdmin update

### Courses Overview

Configures hero section for Courses overview page.

### News Overview

Configures hero section for News listing page.  
Access: SystemAdmin only

### Front End Theme

SystemAdmin-only global. Public read enabled for rendering.

### Cookie & Analytics Settings

SystemAdmin-only. Public read enabled for runtime gating.

### Feedback Page

Controls `/feedback` static page.  
Fields: enabled toggle, title, intro text.  
Access: Admin/SystemAdmin update, public read.

---
