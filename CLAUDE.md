# Archdiocesan Ministry Tools - Developer Guide

## Project Overview

**Archdiocesan Ministry Tools** is an internal management platform for content editing and administrative tasks across archdiocesan ministries and parishes. Built with modern web technologies for secure, authenticated access.

## Tech Stack

- **SvelteKit** - Framework (using Svelte 5 with runes syntax)
- **Tailwind CSS v4** - Styling (via `@tailwindcss/vite`)
- **Supabase** - Authentication & Database (PostgreSQL)
- **Vite** - Build tool

## Authentication

- **Protected Routes**: Entire site requires authentication (except `/auth`)
- **Supabase Auth**: Email/password authentication with cookie-based sessions
- **Server-Side Security**: SSR with proper session validation
- **Privacy**: `robots.txt` prevents search engine indexing

## Key Application Routes

- `/` - Dashboard home with navigation to all tools
- `/editor` - Main content editor with versioning and database storage
- `/scripture` - Bible passage lookup and display tool
- `/dgr` - DGR tools and utilities
- `/auth` - Login/signup page

## Key Files Structure

### Authentication & Layout

- `/src/hooks.server.ts` - Server-side auth handling with Supabase
- `/src/app.d.ts` - TypeScript definitions for auth state
- `/src/routes/+layout.server.ts` - Global auth protection
- `/src/routes/+layout.svelte` - Main layout with navigation
- `/src/lib/components/AppNavigation.svelte` - Common navigation bar

### Content Management

- `/src/routes/editor/+page.svelte` - Main editor UI
- `/src/lib/components/EditBlockModal.svelte` - Block editing modal
- `/src/lib/components/RichTextDisplay.svelte` - Content display with formatting

### Data & API

- `/static/AHWGP_master.json` - Current data storage (migrating to Supabase)
- `/static/test_blocks_evaluation.json` - Block evaluations (stays static)

### Utilities

- `/src/lib/components/Toast.svelte` - Toast notifications
- `/src/lib/stores/toast.js` - Toast state management
- `/TOAST_GUIDE.md` - Toast system documentation

## Svelte 5 Syntax

**Using Modern Svelte 5 with Runes:**

- Use `$state()` instead of writable stores
- Use `$derived()` for computed values
- Use `$effect()` for side effects
- Refer to `svelte5-cheatsheet.md` for syntax reference when needed

## Database Management

### Quick Database Queries

Use the built-in query tool for fast database inspection:

```bash
# Basic usage
npm run query <table_name> [filters] [limit]

# Examples
npm run query blocks                           # Get 10 latest blocks
npm run query blocks "tag='chapter'" 5        # Get 5 chapter blocks
npm run query dgr_schedule "status='pending'" # Get pending DGR entries
npm run query admin_settings                  # View configuration
npm run query dgr_contributors "active=true"  # Get active contributors
```

### Database Schema & Tables

#### **Core Editor Tables**

- **`blocks`** - Versioned content blocks (immutable)
  - `id`, `block_id`, `content`, `tag`, `metadata`, `chapter_id`, `created_at`, `created_by`
- **`books`** - Document structure versions (immutable)
  - `id`, `document_title`, `blocks` (JSONB), `version`, `created_at`, `created_by`, `parent_version_id`
- **`chapters`** - Chapter metadata and organization
  - `id`, `chapter_number`, `title`, `block_id`, `created_at`
- **`editor_logs`** - Complete audit trail
  - `id`, `action_type`, `entity_type`, `entity_id`, `block_id`, `changes`, `user_id`, `created_at`
- **`admin_settings`** - Application configuration (JSONB storage)
  - `id`, `setting_key`, `setting_value`, `description`, `created_at`, `updated_at`

#### **DGR (Daily Gospel Reflection) System**

- **`dgr_contributors`** - Gospel reflection contributors
  - `id`, `email`, `name`, `active`, `preferred_days`, `notes`, `created_at`, `updated_at`
- **`dgr_schedule`** - Daily assignments and submissions
  - `id`, `date`, `gospel_reference`, `gospel_text`, `contributor_id`, `status`, `reflection_content`, `submission_token`, `liturgical_date`
- **`dgr_email_queue`** - Email notification management
  - `id`, `schedule_id`, `recipient_email`, `email_type`, `subject`, `body`, `status`, `error_message`

#### **Utility Functions**

- `get_complete_book(book_id)` - Retrieve complete book with latest block content
- `get_book_by_chapters(book_id)` - Get book organized by chapters
- `assign_contributor_to_date(date)` - Smart contributor assignment
- `generate_submission_token()` - Secure token generation

### Supabase CLI Commands

For advanced database management, see `SUPABASE-CLI.md`:

```bash
# Essential commands
supabase status                     # Check local database status
supabase db reset                   # Fresh local database with migrations
supabase migration new feature_name # Create new migration
supabase gen types typescript --local > src/lib/database.types.ts

# Direct database access
docker exec supabase_db_editor-app psql -U postgres -d postgres
```

### Database Security

- **Row Level Security (RLS)** enabled on all tables
- **Authenticated users** have full read/write access
- **Public users** can only submit DGR reflections via secure tokens
- **Immutable versioning** prevents data loss (blocks/books never updated)
- **Complete audit trail** via editor_logs table

## Data Flow

1. User authenticates via Supabase Auth
2. Load latest book structure from `books` table
3. Load latest content for each block from `blocks` table
4. Combine and display in editor
5. Save changes create new rows (versioning via timestamps)

## Development Guidelines

- **Authentication Required**: All features require user login
- **Svelte 5 Syntax**: Use runes (`$state`, `$derived`, `$effect`) not legacy stores
- **Content Versioning**: Never update existing records, always create new versions
- **Toast Notifications**: Use centralized toast system for user feedback
- **Responsive Design**: Tailwind CSS with mobile-first approach
- **TypeScript**: Full type safety with Supabase client types
