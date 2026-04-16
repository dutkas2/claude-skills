---
name: link-prospect-audit
description: >
  Vet link building prospects using Ahrefs data and outbound link analysis.
  Identifies PBN networks, guest post farms, spam outbound links (casino, CBD, vape, VPN, poker,
  gambling, kratom, slots, betting), and inflated metrics. Produces a tiered pass/fail report.
  Triggers on: link audit, vet these domains, link prospect, link quality check, are these sites
  spam, outbound link check, PBN check, link building audit, guest post quality.
---

# Link Prospect Audit

## Overview

This skill audits a list of domains to determine whether they are safe to buy backlinks from. It checks Ahrefs metrics for red flags, identifies PBN networks and guest post farms, and pulls actual outbound link profiles to find spam destinations (casino, CBD, vape, VPN, poker, gambling, kratom, slots, betting, porn, weed, hemp).

**Use this when:**
- A link builder sends a list of prospect domains to vet
- You need to decide whether a site is safe to place a guest post on
- You want to check if a domain is part of a PBN network
- You need to audit outbound link profiles for spam

## Step 1: Pull Ahrefs Batch Metrics

**Tool:** `mcp__ahrefs__batch-analysis`

For every domain in the list, pull these columns:
```
select: url, domain_rating, org_traffic, org_keywords, refdomains, linked_domains, outgoing_links, outgoing_links_dofollow
```

**Parameters per target:**
- `mode`: `subdomains`
- `protocol`: `both`

**Batch size:** The API returns max 10 results per call. Split domains into batches of 10 and run them in parallel.

**Important:** The column is `url`, not `target`. The column is `org_traffic`, not `organic_traffic`. The column is `org_keywords`, not `organic_keywords`.

## Step 2: Flag Red Flags from Metrics

Score each domain against these red flag patterns. Any single HARD flag = instant fail. Accumulating 3+ SOFT flags = fail.

### HARD Flags (instant fail)

| Flag | Pattern | Why |
|------|---------|-----|
| Traffic/keyword ratio impossible | org_traffic > 5,000 AND org_keywords < 15 | Traffic is manipulated or from a single viral/scraped page. Real sites have proportional keyword counts. |
| DR massively inflated vs refdomains | domain_rating > 50 AND refdomains < 50 | DR is artificially pumped through PBN interlinking. A DR 50+ site should have hundreds of real referring domains. |
| Gibberish domain name | Domain is random letters/numbers (rcsdassk, lwtc148, etc.) | No legitimate business uses a gibberish domain. |
| Domain impersonation | Domain mimics another site with hyphens or alternate TLDs (decoratoradvice-com.com, betterthisworld-com.org) | Classic PBN/parasite pattern. The hyphenated-com or .org version of a real .com is always spam. |
| Zero outgoing links | outgoing_links = 0, linked_domains = 0 | Parked domain or shell site with no real content. |

### SOFT Flags (accumulate to fail)

| Flag | Pattern | Why |
|------|---------|-----|
| Extreme outgoing link ratio | outgoing_links > 50,000 AND domain_rating < 40 | Low-authority site with massive outgoing = link farm. |
| High outgoing dofollow ratio | outgoing_links_dofollow / outgoing_links > 0.95 AND outgoing_links > 10,000 | Legitimate sites nofollow sponsored content. When 95%+ of tens of thousands of outgoing links are dofollow, the site doesn't moderate what it links to. |
| Low keyword count for DR | domain_rating > 40 AND org_keywords < 30 | If a DR 40+ site only ranks for 30 keywords, the DR is inflated or the site has been penalized. |
| Suspiciously low refdomains for DR | domain_rating > 60 AND refdomains < 100 | Real DR 60+ sites are cited by hundreds of domains. Low refdomains = PBN ring inflating each other. |
| Made-up/portmanteau domain | Domain is a nonsense word combo (heartomenal, enthrallinggumption, drhandybility) | Common pattern for PBN sites — they invent words that sound vaguely topical. |
| Unusual TLD for the niche | .info, .org, .net, .com.co for a home/lifestyle blog | Legitimate home blogs are almost always .com. Unusual TLDs are cheap PBN fodder. |

### Network Detection

Look for clusters of domains that share naming patterns. Common PBN network signatures:

- **Prefix networks:** Multiple domains starting with the same 2-3 letters (kda*, kd*)
- **Thematic clones:** Multiple domains with similar word patterns (decoratoradvice, decoratorsadvice, decoradtech, decoradhouse)
- **Parallel domains:** Same root name across multiple TLDs or with hyphens (.com, .org, .co, -com.com)
- **Paired domains:** Two domains with similar names in the same list (mygardenandpatio.org + robertmygardenandpatio.com)

If you detect a network, flag ALL domains in it as HARD NO regardless of individual metrics.

## Step 3: Scrape Homepage for Surface-Level Spam Signals

**Tool:** `mcp__firecrawl__firecrawl_scrape` with `formats: ["links", "markdown"]`

Spot-check the most suspicious domains (and a sample of the "looks clean" ones). Look for:

| Signal | What to Look For |
|--------|-----------------|
| Fake social counts | Social media widgets with follower counts that all link to `#` instead of real profiles |
| Default/placeholder text | "This may be a good place to introduce yourself", Lorem ipsum, default WordPress taglines |
| Fake team bios | Invented names with no LinkedIn/social presence, fake addresses (city/state mismatches like "Laramie, Colorado" when Laramie is in Wyoming) |
| Pakistani/Indian phone numbers | +92, +91 country codes on English-language "home decor" or "lifestyle" blogs |
| All content by one author | Every post written by a single "admin" or one name — indicates a one-person content mill |
| Cross-network references | Articles mentioning or linking to other domains in the same suspect list |
| Generic guest post content | Articles about local plumbers, dentists, lawyers, roofers in random cities — these are paid placement articles |

## Step 4: Check Outbound Link Profiles via Ahrefs

**This is the most important step.** Many sites pass Steps 2-3 but fail here.

**Tool:** `mcp__ahrefs__site-explorer-linked-domains`

**Parameters:**
```
target: {domain}
mode: subdomains
select: domain,links_from_target,dofollow_links
order_by: links_from_target:desc
limit: 50
```

**Filter for spam outbound using this `where` clause:**
```json
{"or":[
  {"field":"domain","is":["isubstring","casino"]},
  {"field":"domain","is":["isubstring","poker"]},
  {"field":"domain","is":["isubstring","gambling"]},
  {"field":"domain","is":["isubstring","cbd"]},
  {"field":"domain","is":["isubstring","hemp"]},
  {"field":"domain","is":["isubstring","porn"]},
  {"field":"domain","is":["isubstring","xxx"]},
  {"field":"domain","is":["isubstring","vpn"]},
  {"field":"domain","is":["isubstring","slot"]},
  {"field":"domain","is":["isubstring","betting"]},
  {"field":"domain","is":["isubstring","weed"]},
  {"field":"domain","is":["isubstring","kratom"]},
  {"field":"domain","is":["isubstring","vape"]}
]}
```

### Interpreting Results

| Result | Verdict |
|--------|---------|
| 0 spam outbound domains | Clean. Passes this check. |
| 1-2 spam domains, all with 1 link each | Borderline. Could be incidental (e.g., "nazweedcontrol.com" is a weed control company, not cannabis). Check if the domain names are false positives. |
| 3-5 spam domains | Contaminated. The site accepts paid posts from spam verticals. |
| 5+ spam domains OR any single spam domain with 3+ dofollow links | Heavily contaminated. The site actively sells links to casino/CBD/vape advertisers. Hard no. |

### Common False Positives

Watch for these in the outbound results — they match the spam keywords but are legitimate:
- `nazweedcontrol.com` — weed control company (lawn care), not cannabis
- `tumbleweedhouses.com` — tiny house company, "tumbleweed" matches "weed"
- `alderandtweed.com` — interior design firm, "tweed" matches "weed"
- `tmopestandweed.com.au` — pest and weed control company
- `susunweed.com` — herbalist (author name is Weed)
- `carolineslotte.com` — person's name, not casino slots
- `tweedandhickory.com` — fashion brand
- `locksmithempire.com` — matches no spam keywords, sometimes appears in results
- `protonvpn.com`, `nordvpn.com`, `expressvpn.com` — legitimate VPN companies. A single link to these in a tech/privacy article is fine. Multiple dofollow links across guest posts = paid placement.
- `steamaticofvapen.com` — carpet cleaning franchise location name

When in doubt, check whether the outbound link is dofollow. A single nofollow link to a borderline domain is usually noise. Multiple dofollow links are signal.

## Step 5: Produce the Report

Tier every domain into one of four categories:

### CLEAN (Buy)
- Zero or near-zero spam outbound
- No HARD flags
- No more than 1 SOFT flag
- Metrics are proportional (DR matches refdomains, traffic matches keywords)

### BORDERLINE (Vet the specific page)
- 1-2 possible spam outbound (could be false positives)
- 1-2 SOFT flags but no HARD flags
- Could work if you verify the exact page your link will appear on

### CONTAMINATED (Do Not Buy)
- 3+ spam outbound domains confirmed
- OR previously looked "safe" on surface metrics but outbound check revealed spam links
- Note: always call out what the spam links are so the link builder understands why

### HARD NO (Spam/PBN)
- Any HARD flag triggered
- Part of an identified PBN network
- Guest post farm (50K+ outgoing links)
- Gibberish/impersonation domain

### Report Format

Present results in this structure:

1. **Summary table** — domain, DR, traffic, keywords, verdict, reason (one line each)
2. **CLEAN domains** — full details, confirm zero spam outbound
3. **CONTAMINATED domains** — list the specific spam outbound domains found with link counts
4. **PBN networks identified** — group the networked domains and explain the connection
5. **HARD NO list** — brief reason per domain
6. **Stats** — X clean / Y total, rejection rate percentage

## Parallel Execution Strategy

To keep the audit fast:

1. **Batch analysis** — Run all domains through `batch-analysis` in parallel batches of 10
2. **Triage** — Apply Step 2 red flag scoring to eliminate obvious spam without further API calls
3. **Outbound check** — Run `site-explorer-linked-domains` spam filter on all non-obvious domains in parallel (6 at a time)
4. **Spot-check scrape** — Only scrape 4-6 domains that are borderline to confirm/deny

This minimizes Ahrefs API usage while catching everything. Typical 75-domain audit uses ~80-100 API calls.

## Anchor Text Guidance

When the audit is complete and the user has clean domains, offer to build an anchor text and target page strategy:

1. **Map the client site** — Use `firecrawl_map` to get all service pages, service area pages, and blog posts
2. **Prioritize pages** — Money pages (homepage, top service pages) get 60% of links, blog posts get 20%, supporting pages get 20%
3. **Build anchor text variations** per target page following this distribution:
   - 25-30% branded
   - 25-30% partial match keyword (never repeat the same one)
   - 15-20% generic/natural
   - 10-15% branded + keyword
   - 5-10% exact match (max 1-2 per 10 links)
   - 3-5% naked URL
4. **Rules:** Never repeat an anchor, vary city modifiers, blog links get informational anchors only
