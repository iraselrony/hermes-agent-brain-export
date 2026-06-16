---
name: headless-web-rebuild
description: Professional workflow for migrating websites from closed platforms (Webflow, Wix, Squarespace) to modern headless stacks (Next.js, Payload CMS, Supabase).
---

# Headless Web Rebuild

Professional workflow for migrating websites from closed platforms (Webflow, Wix, Squarespace) to modern headless stacks (Next.js, Payload CMS, Supabase).

## Trigger
- "Clone this Webflow site to Vercel"
- "Rebuild this Wix site with Next.js"
- "Migrate a closed-platform site to a custom stack"

## Workflow

### 1. Deep Audit & Discovery
- **Sitemap Mapping:** Do not rely on surface-level crawls. Always attempt to find `sitemap.xml` to ensure 100% URL coverage.
- **SEO Preservation:** Extract all `<title>`, `<meta description>`, and `<meta og:>` tags. Map them to a migration list.
- **Content Inventory:** Scrape all text, images, and downloadable assets (PDFs).
- **Design Token Extraction:** Analyze CSS to extract exact Hex colors, Font families/weights, and spacing scales (padding/margins).

### 2. Architecture & Schema Design
- **Stack Selection:** Prefer `Next.js` (Frontend) + `Payload CMS` (Admin/Backend) + `Supabase` (Database/Storage/Auth) + `Vercel` (Deployment).
- **CMS Modeling:** Convert existing site "Collections" into Payload CMS Collections. Map every content field (text, image, relation) to a specific Payload field type.

### 3. Scaffolding & Implementation
- **Design System:** Initialize Tailwind CSS using the extracted design tokens.
- **Iterative Build:** Follow the sequence: `Global Layout (Navbar/Footer)` $\rightarrow$ `Home Page` $\rightarrow$ `Dynamic Templates` $\rightarrow$ `Sub-pages`.
- **Media Handling:** Use S3-compatible storage (e.g., Supabase Storage) for high-res assets.

### 4. Verification & Launch
- **SEO Audit:** Verify that all old URLs now resolve to the new URLs with identical metadata.
- **Deployment:** Deploy the Next.js app to Vercel.

## Pitfalls
- **Broken Links:** Failing to map slugs 1:1 will destroy SEO.
- **Missing Assets:** Relying on simple web-scrapers often misses deep-linked files like PDFs. Always check for a sitemap.
- **Content Loss:** Forgetting to map specific "specialty" fields (like pricing or custom metadata) during CMS schema design.

## Related Files
- [Architecture Blueprints](references/architecture-blueprints.md)
