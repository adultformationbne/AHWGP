# AHWGP Editor - Quick Reference

## Tech Stack
- **SvelteKit** - Framework (using Svelte 5 with `$state` runes)
- **Tailwind CSS v4** - Styling (via `@tailwindcss/vite`)
- **Supabase** - Database (PostgreSQL)
- **Vite** - Build tool

## Key Files

### Editor Interface
- `/src/routes/editor/+page.svelte` - Main editor UI
- `/src/routes/builder/+page.svelte` - Original builder (being replaced)

### API/Data
- `/src/routes/api/builder/+server.js` - Current JSON file API (being replaced)
- `/static/AHWGP_master.json` - Current data storage (migrating to Supabase)
- `/static/test_blocks_evaluation.json` - Block evaluations (stays static)

### Components
- `/src/lib/components/EditBlockModal.svelte` - Block editing modal
- `/src/lib/components/RichTextDisplay.svelte` - Content display with formatting
- `/src/lib/components/Toast.svelte` - Toast notification component
- `/src/lib/components/ToastContainer.svelte` - Toast container component

### Toast Notification System
- **Documentation**: See `/TOAST_GUIDE.md` for comprehensive usage guide
- **Store**: `/src/lib/stores/toast.js` - Centralized toast state management
- **Features**: Success/error/warning/info/loading types, multi-step support, auto-dismiss

### Database Schema (New)
```sql
-- books table: document structure
CREATE TABLE books (
    id UUID PRIMARY KEY,
    blocks JSONB, -- [{id: "block-id", hidden: boolean}, ...]
    created_at TIMESTAMP
);

-- blocks table: content versions
CREATE TABLE blocks (
    id UUID PRIMARY KEY,
    block_id TEXT, -- matches the existing IDs
    content TEXT,
    tag TEXT,
    created_at TIMESTAMP
);
```

## Data Flow
1. Load latest book structure from `books` table
2. Load latest content for each block from `blocks` table
3. Combine and display in editor
4. Save changes create new rows (not updates)

## Important Notes
- Using Svelte 5 syntax (`$state`, not stores)
- Content includes inline HTML formatting (bold, italic, etc.)
- Versioning is automatic via `created_at` timestamps
- No user auth/permissions needed (simple internal tool)