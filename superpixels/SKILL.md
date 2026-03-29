---
name: superpixels
description: Design and build remarkable, memorable micro-interactions and design elements (Superpixels) for web projects. Inspired by Glen Allsopp's Superpixels framework. Use when asked to add a "superpixel", "remarkable element", "memorable detail", "delight moment", or when you want to make a section feel special and non-generic. Triggers on "superpixel", "remarkable", "delight", "make this special", "add personality", "make this memorable".
---

# Superpixels: Remarkable Design Elements

Build remarkable, memorable micro-interactions and design details that make people
notice your site is different. Inspired by Glen Allsopp's Superpixels framework:
small, intentional design choices that are so specific and thoughtful that visitors
feel the craft behind them.

A Superpixel is NOT a full page or section redesign. It is a single, focused design
element that makes someone pause and think "that's clever" or "I've never seen that
before." The best ones are subtle. The worst ones are gimmicks.

## What Makes a Good Superpixel

**The test:** Would someone screenshot this and send it to a friend? Would a designer
bookmark it? Would a competitor copy it? If yes, it's a Superpixel.

**Categories of Superpixels (with real examples):**

1. **Adaptive content** - Elements that change based on context
   - CTAs that change based on what type of visitor you are (Gmelius)
   - FAQ answers that adapt to the user's location (UnderLuckyStars)
   - Headers that transform on scroll to show contextual actions (SoundGuys)
   - Related post headlines rewritten to be more compelling than the real title (VeryWellHealth)

2. **Trust amplifiers** - Social proof presented in unexpected ways
   - Expert review panels that feel medical-grade for a food site (EatThis)
   - Social proof on LOGIN pages, not just landing pages (SimilarWeb, Wistia)
   - Open analytics dashboards showing real business metrics (BannerBear)
   - "In the news" framing instead of generic "featured on" logos (InsiderIntelligence)

3. **Interactive reveals** - Hover/click states that surprise
   - Product photos that crossfade to video on hover (Inuru)
   - Clean cards that expand into navigation on hover (uSwitch)
   - Heart buttons with escalating animations on repeated clicks (JoshWComeau)
   - Progress bars under cycling headline words instead of typewriter text (SendGrid)

4. **Personal touches** - Details that show the humans behind the brand
   - Founder sticky-note messages in the footer (ShiftNudge)
   - "Days in business" live counter (BentoNow)
   - Current local time shown next to business hours (CulturedCode)
   - Tools/tech stack disclosure to speak directly to your audience (ShiftNudge)

5. **Smart navigation** - Navigation that does more than navigate
   - Newsletter opt-ins built into the main navbar (CoinDesk)
   - Ticker-style scrolling nav bars with mixed content types (BYRDIE)
   - Table of contents with per-section progress bars (AmericanPressInstitute)
   - Footer that reappears when scrolling back up (NYMag)

6. **Micro-copy personality** - Words in unexpected places
   - Opt-in forms with personality instead of "Subscribe to our newsletter" (Dscout)
   - Footer links with humor and personality (MikeMichalowicz)
   - Cookie warnings with custom illustrations and friendly copy (Thomann)
   - Email typo detection on signup forms (Nomics)

7. **Visual craft** - Tiny details that show obsessive attention
   - Image highlights using the exact same gradient as the brand (BYRDIE)
   - Internal links styled with the brand icon, external links without (Fibery)
   - Custom author avatars matching site design instead of mismatched photos (Puck)
   - File extensions and data formats styled in custom fonts (WPSmackDown)

## Process

### Step 1: Understand the Context

Before proposing any Superpixel, understand:
- **What is the page/section trying to accomplish?** (convert, inform, build trust, retain)
- **Who is the audience?** (their sophistication, expectations, pain points)
- **What is the brand tone?** (warm, technical, playful, premium, clinical)
- **What already exists?** Read the current component code and design system

Read the project's DESIGN.md and any existing Superpixels spec docs. Check
`src/components/superpixels/` for existing patterns and conventions.

### Step 2: Propose 3-5 Superpixel Ideas

For each idea, provide:
- **Name** - A short descriptive name
- **Inspired by** - Which Superpixel category/example it draws from
- **What it does** - One sentence description
- **Why it's remarkable** - What makes someone notice it
- **Effort** - Low (CSS only) / Medium (CSS + minimal JS) / High (significant JS logic)

Present these to the user. Let them pick which ones to build.

### Step 3: Build the Superpixel

**File conventions:**
- Location: `src/components/superpixels/ComponentName.astro`
- Each Superpixel is a standalone Astro component with typed Props interface
- Zero framework runtime - use vanilla JS in `<script>` tags when interactivity is needed
- Always respect `prefers-reduced-motion` - provide a static fallback
- Use the project's design system tokens (colors, typography, spacing from `@theme`)
- Include `reveal` class for scroll-triggered entrance animations where appropriate

**Code patterns (from existing Superpixels):**

```astro
---
interface Props {
  // Typed props with sensible defaults
  label?: string;
  variant?: 'default' | 'inline';
}

const { label = 'default value', variant = 'default' } = Astro.props;
---

<!-- Semantic HTML, design system classes, inline styles only for one-off values -->
<section class="relative bg-navy navy-texture overflow-hidden">
  <div class="relative z-10 max-w-[800px] mx-auto px-6 lg:px-12">
    <!-- Component markup -->
  </div>
</section>

<script>
  // Vanilla JS for interactivity
  // Always check prefers-reduced-motion
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    // Static fallback
    return;
  }
  // Use IntersectionObserver for scroll-triggered effects
  // Use requestAnimationFrame for smooth animations
</script>
```

**Quality checklist:**
- [ ] Works without JavaScript (progressive enhancement)
- [ ] Respects `prefers-reduced-motion`
- [ ] Mobile-friendly (touch targets, no hover-only interactions without tap fallback)
- [ ] Uses design system tokens, not hardcoded colors
- [ ] Accessible (ARIA labels, keyboard navigable, sufficient contrast)
- [ ] Performant (lazy load media, `preload="none"` on video, no layout shift)
- [ ] Standalone - can be dropped into any page variant without dependencies

### Step 4: Integration

Show how to use the Superpixel in a page:

```astro
import SuperpixelName from '../components/superpixels/SuperpixelName.astro';

<!-- Usage in page -->
<SuperpixelName prop1="value" prop2={variable} />
```

### Step 5: Verify

Use the browse tool (if available) to screenshot the Superpixel at mobile and
desktop viewports. Take zoomed screenshots of the specific element, not full-page
shots - detail matters for Superpixels.

## Anti-Patterns (What Superpixels Are NOT)

- **Not gimmicks.** If it doesn't serve the page's goal, skip it.
- **Not generic "nice-to-haves."** Rounded cards, gradient backgrounds, and hover
  shadows are baseline design, not Superpixels.
- **Not complex for complexity's sake.** The best Superpixels are often surprisingly
  simple in implementation. A 30-second CSS change can be more remarkable than a
  500-line JS animation.
- **Not AI slop.** No purple gradients, no glass morphism for its own sake, no
  "floating particles" backgrounds, no generic icon grids in colored circles.
- **Not borrowed wholesale.** Adapt ideas to the brand and context. A crypto site's
  open metrics dashboard concept becomes a counselor vetting pipeline for a
  spiritual care brand.

## Existing Superpixels in This Project

Reference these when building new ones to maintain consistency:

| Component | Inspired By | What It Does |
|-----------|------------|--------------|
| TopicCycler | SendGrid animated underlines | Gold progress bar fills under cycling hero words |
| VideoCard | Inuru hover-to-video | Counselor photo crossfades to video on hover |
| VettingPipeline | EatThis trust focus | Horizontal pipeline showing 4-step vetting process |
| LiveCounter | BentoNow days counter | Animated number count-up on scroll into view |
| MissionStatement | NiftyGateway state mission | Bold centered statement with gold accent word |
| BibleVerse | Custom | Contextual scripture with subtle formatting |
| AdaptiveStickyBar | SoundGuys adaptive headers | CTA bar that appears/changes on scroll |

When adding new Superpixels, update this table in the skill file.
