---
name: pagesforpros
description: Working with PagesForPros MCP - templates, pages, settings, compliance, and known gotchas
---

# PagesForPros MCP Skill

## Overview
PagesForPros is a website builder for local service businesses. Sites are built with templates (service, service_area, services_hub, service_areas_hub, reviews_page) that generate downstream pages using AI content. Direct-content pages (homepage, landing pages, about, contact) are edited directly.

## Critical Rules

### 1. ALWAYS include `templateMeta` when publishing templates
When calling `publish_template`, you MUST include the full `templateMeta` object containing AI prompts and component configurations. Without it, the template appears **empty** in the dashboard editor and AI content generation breaks.

```
publish_template({
  websiteId, pageType,
  draft: {
    data: { ... content blocks ... },
    type: "puck-template",
    version: 1,
    templateMeta: {  // <-- NEVER OMIT THIS
      version: 1,
      pageMeta: { ... },
      components: { ... }  // AI prompts for each component
    }
  }
})
```

### 2. `regenerate_pages` destroys page overrides
Calling `regenerate_pages` deletes ALL pages and creates new ones with **fresh IDs**. All `update_page_overrides` changes are lost. After any regeneration:
- Re-fetch page IDs with `list_pages`
- Re-apply all overrides to new IDs
- Avoid calling `regenerate_pages` unless absolutely necessary

### 3. AI prompts with negative instructions are unreliable
AI content generation sometimes ignores "Do not use the word free" or "Do not mention guarantees" in prompts. The AI may generate the exact content you told it not to. Always:
- Set page-level overrides for critical compliance content (FAQ Q&A, QuoteForm headings)
- Verify rendered output with Firecrawl after publishing
- Don't trust that AI-generated content will follow negative constraints

### 4. Template publish does NOT immediately update live site
Template changes update the database but the Next.js site needs a Vercel deploy to render new content. The flow is:
1. `publish_template` (updates data + queues AI regeneration)
2. AI regeneration runs in background (fills empty fields)
3. Site redeploys on Vercel (renders new HTML)

## Page Types & Content Models

| Content Model | Edit Method | Examples |
|---|---|---|
| `direct` | `update_page` with full content | Homepage, LP, About, Contact, Privacy Policy, Terms |
| `template` | `publish_template` + `update_page_overrides` | Service pages, Service area pages, Hub pages |

## Template Types
- `service` - All service pages (e.g., /services/rat-control)
- `service_area` - All service area pages (e.g., /service-areas/rat-control-knoxville-tn)
- `services_hub` - The /services index page
- `service_areas_hub` - The /service-areas index page
- `reviews_page` - Reviews page

## Key Component IDs (consistent across pages of same template type)

### Service Template
- Hero: `Hero-d63400eb-48f1-43e1-ba0f-30b7833bee0b`
- Our Process: `SectionContent-04688a0c-5a35-4eae-a7b3-6356b0b2f973`
- Why Us: `SectionContent-73db3480-3528-434d-afb2-c8cc5a38fb5a`
- QuoteForm: `QuoteForm-89710ee5-50b9-4739-bcf1-96c8187e66fc`
- FAQ: `FAQ-f791e46c-3a73-4ecf-8e52-6266bba1f4e4`

### Service Area Template
- Hero: `Hero-d53327ac-c435-4c1b-9047-4acd503041f9`
- Why Us: `SectionContent-31876310-1ccb-406a-aeec-af123faabea6`
- Our Process: `SectionContent-6e4147bf-7828-4db1-b0b0-1bc309e90009`
- QuoteForm: `QuoteForm-8eb511d0-f98d-45e2-9ab1-31e6f67066bd`
- FAQ: `FAQ-1800cabd-d302-4d58-a31f-485c7f5db03f`

## Override Format
Override keys follow the pattern: `{componentId}__{propName}`

```
update_page_overrides({
  websiteId, pageId,
  contentOverrides: {
    "FAQ-f791e46c__question2": "What should I look for?",
    "FAQ-f791e46c__answer2": "We recommend...",
    "QuoteForm-89710ee5__buttonText": "Get My Quote",
    "QuoteForm-89710ee5__subheading": "Get a quote in minutes."
  }
})
```

## Website Settings

### Key settings paths
- `hero.ctaText` - Hero CTA button text (site-wide)
- `preHeader.promoText` - Top banner text
- `footer.description` - Footer description text
- `footer.ctaBarText` - Pre-footer CTA bar body text
- `footer.ctaBarButtonText` - Pre-footer CTA bar button text
- `selectedBenefits` - Array of benefit badges
- `company.phone` - Business phone number
- `theme.primaryColor` / `theme.accentColor` - Brand colors

### Settings that DON'T work
- `theme.styles.navLinkColor` - Accepted by API but not rendered by frontend

## QuoteForm Props
- `heading` - Form section heading
- `subheading` - Subtitle below heading
- `buttonText` - Submit button text (overrides default "Get My Free Quote")
- `followupText` - Consent/legal text below form
- `bullet1/bullet2/bullet3` - Benefit badges above form
- `bullets` - Object format: `{bullet1: "", bullet2: "", bullet3: ""}`

## Image Handling

### Template images
Set `imageMode: "static"` to prevent AI from regenerating the image:
```json
{
  "image": "https://blob-url...",
  "imageMode": "static"
}
```

### Generating images
```
generate_image({ websiteId, prompt, width, height, requireLogo: false })
```
Returns `{ imageUrl, storagePath }`. Use the `imageUrl` in template/page content.

### Image override gotcha
SectionContent image overrides don't work when base content `image` is empty - the widget falls back to a static default before checking overrides. Set the image directly in the template content instead.

## Compliance Workflow (Lead Gen Sites)

For home services campaign compliance:

1. **Disclaimer** - Add to LP footer as a Text block
2. **Privacy Policy** - Use HTML (not markdown) in Text blocks - markdown renders as raw syntax
3. **Remove "free"** - Check: hero benefits, QuoteForm heading/subheading/buttonText, FAQ Q&A, pre-header, CTA bar, selectedBenefits, AI prompts
4. **Remove guarantees** - Check: FAQ Q&A, hero benefits, section headings, AI prompts
5. **Generic content** - Site should say "we connect you with..." not "our technicians will..."
6. **No pricing** - Remove dollar amounts from FAQ answers and AI prompts
7. **No fake reviews** - ReviewsSection pulls from real data only

### Scanning for compliance issues
Use Firecrawl to scan rendered pages:
```
firecrawl_scrape({
  url: "https://site.com/page",
  formats: ["json"],
  jsonOptions: {
    prompt: "Find every instance of 'free', 'guarantee', 'refund'...",
    schema: { ... }
  },
  waitFor: 8000
})
```

## Common Pitfalls

1. **Don't call `regenerate_pages` casually** - It destroys all overrides and generates fresh content that may violate compliance
2. **Page IDs change on every regeneration** - Never hardcode page IDs; always re-fetch with `list_pages`
3. **Template publish without templateMeta = broken template** - The dashboard will show empty blocks
4. **AI ignores negative prompts** - Always verify output and apply overrides for critical content
5. **Firecrawl caches aggressively** - Results may be stale; check `cacheState` and `cachedAt` in response
6. **Direct pages vs template pages** - Direct pages use `update_page` with full content; template pages use `update_page_overrides`
7. **Hub pages have different QuoteForm/FAQ component IDs** than service/service_area templates
