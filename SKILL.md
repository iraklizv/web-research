---
name: web-research
description: "Use this skill whenever the user wants to research a topic, find sources, back up a claim, or validate data for any purpose — especially SEO content writing. Trigger on: \"research X\", \"find sources on X\", \"what does the data say about X\", \"back this up\", \"find a source for this\", or any time Claude is writing SEO content and needs to include supporting links or citations. Also trigger automatically when working on an article or landing page and a factual claim would benefit from a credible external source. Use this skill even if the user doesn't explicitly say \"research\" — if they're writing content and sources would strengthen it, consult this skill."
---

# Web Research Skill

General-purpose web research: search, evaluate, and synthesize sources for any topic — with specific rules for SEO content use cases.

---

## Output Modes

Pick the mode based on trigger signals below. Lock in and stay there — don't switch modes mid-task or re-evaluate unless the user explicitly changes the request.

**Mode trigger signals:**
- "Back this up" / "is this real" / "is this stat accurate" / "verify this" → Mode 3, always. Single claim, two to three lines, done.
- "Research X" / "find sources on X" / "what does the data say about X" with no article in progress → Mode 2.
- Article or outline is open and a claim needs sourcing → Mode 1. Stay in Mode 1 for the entire article task even if "back this up" language appears — context is the article, not a one-off check.
- User shares a draft and asks for sources to be checked → Mode 4.

### Mode 1: Inline Drop (default for SEO content writing)
Use when: working inside an article, backing a specific claim, or the user just needs linkable stats.

Format per stat:
- **Claim**: The specific finding, with exact number if available
- **Source**: Name + stable URL (not a CDN link or PDF with no permalink -- if a PDF is the only source, note it and flag that it may not be linkable)
- **Year**: Publication or data year
- **Use like this**: One sentence showing how to work it into copy naturally

Example:
> **Claim**: Only 21% of US workers use AI on the job
> **Source**: Pew Research Center — pewresearch.org/short-reads/...
> **Year**: 2025
>
> **Use like this**: "Despite the hype, only 1 in 5 US workers actually uses AI on the job (Pew Research, 2025) — which means most teams are still leaving productivity on the table."

### Mode 2: Full Research Brief
Use when: the user asks for a full research pass on a topic with no article in progress, or the topic is contested and needs proper coverage.

Per source (3–5 by default):
- **Source**: Name + URL
- **Relevance**: One sentence — why this source matters
- **What it says**: 2–4 bullets of actual findings/data

Below the sources:
- **Bottom line**: 2–3 sentence synthesis
- **Conflicts**: Flag if sources disagree — see Conflict Handling

### Mode 3: Quick Check
Use when: the user says "back this up" or "is this stat real" on a single claim.

Return: confirmed/not confirmed, best available source, exact number if different from what was claimed. Two to three lines max.

### Mode 4: Source Audit
Use when: the user shares a full article draft and wants existing sources checked.

Process:
1. Extract every linked URL from the article
2. For each URL, use `web_fetch` to read the actual page
3. Identify the specific claim in the article that cites it
4. Evaluate and return a verdict per source:

Format per source:
- **URL**: the link as it appears in the article
- **Live**: yes / no / paywalled
- **Claim match**: does the article's claim accurately reflect what the source says? The exact figure must be VISIBLE on the linked page — open the page and confirm the number actually appears (in body text, a chart label, or a table). A resolving URL on the right topic is not enough — the figure itself must be on the page. A live URL that does not display the cited figure FAILS this check. Flag exaggerations, misquotes, or stats taken out of context
- **Competitor flag**: yes / no / uncertain -- if yes, note it and suggest finding an upstream source
- **Freshness**: is there newer data on this specific stat? If yes, flag it
- **Verdict**: Pass / Flag / Replace
  - Pass = accurate, live, not a competitor, not outdated
  - Flag = minor issue (slightly outdated, loose paraphrase, unstable URL) -- usable but worth a second look
  - Replace = wrong claim, dead link, competitor, or significantly outdated with better data available

When a source is flagged or marked Replace: don't just flag and stop. Attempt to find the primary source before returning results. If the article links a secondary source (news article, blog post) that itself cites an original report or announcement, fetch that upstream source and include it as the suggested replacement.

**Gated / unverifiable-on-page figures — work the sourcing ladder before cutting.** When a figure can't be confirmed on a free public page (the number lives in a gated report, a downloadable PDF, or behind a form), do NOT default to dropping the stat. Work the ladder in order:
1. Find a public page that visibly displays the exact figure — a press release, a summary blog from the same publisher, a webinar recap, a slide deck. Link that.
2. If no public page shows the number, link the report's own landing page (the gated page) so the citation points at the primary source, and name the report + year in the copy.
3. Only cut the figure if the source does not substantiate it at all — i.e. the number cannot be traced to the cited publisher anywhere. Cutting is the last resort — exhaust steps 1 and 2 first.

End with a summary: X passed, X flagged, X need replacing -- plus any patterns worth noting (e.g. "3 of your 5 sources are competitor blogs").

---

**Source link quality**: Before including any URL, verify it's a stable publicly accessible page. Flag CDN-hosted PDFs, paywalled pages, or URLs likely to break. If the best source is a PDF with no web version, say so and let the user decide whether to cite it.

**Redundant sources**: If two sources cover the same ground (same stat, same timeframe, same methodology), pick the more authoritative one and drop the other. Don't pad.

---

## Search Strategy

1. Start with 1–2 broad queries to map the landscape
2. Follow up with specific queries targeting data, studies, or official brand documentation
3. Use `web_fetch` to read full pages — snippets are often too thin to evaluate properly
4. Prioritize going wide first (what's out there), then deep (what actually holds up)

**Default depth**: 3–5 sources. Scale up if:
- The topic is contested or the stakes are high
- Initial sources are weak, thin, or conflicting
- The user explicitly asks for more coverage

---

## Default Context

Unless told otherwise, assume the content is for a B2B audience: sales teams, SDRs, GTM engineers, lead gen agencies. This affects what counts as a "relevant" source — a stat about B2C email open rates is less useful than one about cold outreach reply rates, for example.

---

## Source Quality Rules

**Priority order:**

1. **Tier 1 — named B2B research sources**: LinkedIn, Gartner, Forrester, HubSpot, Salesforce, McKinsey. These are the sources a B2B audience recognizes and trusts.

   **Tier 1b — data-rich sources with a competitor wrinkle**: Apollo, ZoomInfo, Outreach, Salesloft publish useful B2B data. **If a source is on your client's do-not-link (competitor) list, never link it.** Use that data only via the upstream sources its reports cite: fetch the page, find what it cites, link that. Sources not on the list are citable and linkable, subject to the page-level keyword-competitor check (Competitor Source Exclusion Rule below).
2. **Official brand/product pages**: linkedin.com/help, product documentation, official announcements from the brand being discussed

   **LinkedIn URL rule**: linkedin.com/help pages move or 404 frequently. Always run `web_fetch` on any LinkedIn URL before including it — do not assume it's live based on search snippet alone. If the page is dead, search for the current equivalent using the topic + site:linkedin.com/help.

   **LinkedIn feature staleness rule**: If a source is 18+ months old AND the claim covers a LinkedIn product feature, limit, or spec (connection request limits, SSI scoring, automation detection, Premium feature sets, message limits, etc.) — flag it as "verify currency" regardless of whether newer data exists. LinkedIn changes these without announcement. An outdated benchmark here isn't just stale, it may be actively wrong and harmful to publish.
3. **Tier 3 — original studies and reports**: PDFs fine if credible and relevant. University research acceptable when the data is the point (persuasion, behavior, benchmarks) — not a priority but valid.
4. **Tier 4 — industry publications**: named authors, cited data, not just an opinion piece

**Avoid:**
- SEO blog posts with no original data and no cited sources
- Thin listicle content
- Data that's outdated AND newer data exists on the same topic — if nothing newer exists, older data is fine, flag it
- Forums, Reddit, Quora unless the question is specifically about user experience or perception

**Rule of thumb**: If the page can't tell you where its data came from, it's not a valid source.

---

## Contrarian and Surprising Data

Actively look for at least one stat or finding that challenges the obvious take on a topic. Something that makes a reader stop and reconsider. This isn't about being edgy — it's about making the content worth reading.

Examples of what to look for:
- A metric that performs worse than the industry assumes
- A tactic that's widely recommended but has data suggesting it underperforms
- A benchmark that's lower (or higher) than the common talking point

**Where to look specifically:**
- Comment sections of top-ranking posts — practitioners often push back on inflated benchmarks in the comments
- Reddit threads (r/sales, r/linkedin, r/emailmarketing, r/outbound) — search the tactic or metric name + "doesn't work" or "overrated"
- LinkedIn posts from practitioners in the space — search the claim + site:linkedin.com
- Competing studies that contradict the primary narrative — search the stat + "actually" or "myth" or "study"
- YouTube creators covering the topic — titles often signal contrarian takes even if you can't watch the video

If nothing genuinely surprising exists, don't force it. But exhaust these sources before concluding that.

---

## Pricing Page Research

When fetching a competitor or third-party pricing page, pricing data is unreliable unless the billing context is explicit. Always do the following:

1. **State the currency**: Note the currency the page is displaying (USD, EUR, GBP, etc.). If the page auto-detects location and may be showing non-USD pricing, flag it — fetch the page via a search snippet URL rather than assuming the rendered price matches what a US buyer sees.
2. **State the billing cycle**: Note whether the price shown is monthly or annual (billed annually). These are often different by 20-40%. Always report both rates. Since `web_fetch` cannot click JS toggles, the fetched HTML may only reflect one billing state (usually the page default). To get both: fetch the page AND run a targeted search for `"[tool name] pricing monthly annual"` to cross-reference. If only one rate is visible in the fetch and the search doesn't surface the other, flag it as "toggle-dependent — only [monthly/annual] rate confirmed, verify the other manually."
3. **Report pricing as**: `$X/month (billed annually)` or `$X/month (billed monthly)` — never just `$X/month` without the qualifier.
4. **Flag plan tier explicitly**: Name the plan (Starter, Pro, Growth, etc.) — don't just say "starts at."
5. **If pricing requires login or is hidden behind a "contact us"**: Note that and do not guess.

**If the pricing page is blocked by robots (403, timeout, or fetch fails)**: Do not report pricing as unverifiable. Instead:
1. Search `"[tool name] pricing"` and check if the snippet shows a price with billing context
2. Check G2 (g2.com) and Capterra (capterra.com) — these are manually maintained and often list current pricing with billing cycle detail. Search `"[tool name] pricing" site:g2.com` or `site:capterra.com`
3. If a price is found via these sources, report it with the source noted: e.g. "via G2 listing — verify on pricing page directly"
4. Only flag as unverifiable if all three methods (fetch, search snippet, G2/Capterra) return nothing

**Example output format:**
> LinkedHelper 2: $15/month (billed annually) or $20/month (billed monthly) — Standard plan. Currency: USD. Verified [date].

---

## Competitor Source Exclusion Rule

When researching for SEO content:

1. Infer the topic and competitive space from context (article draft, outline, brief, or client info)
2. If a candidate source is a competitor, **do not include it as a citation**. Two kinds count: (a) a competing tool or brand in the client's space, and (b) ANY page — even from a trusted research firm — that itself ranks for the article's target keyword. The check is page-level: Gartner's b2b-buying-journey page is off-limits for an article targeting "b2b buyer journey," while other Gartner pages stay fine. Name the source unlinked, or cite an equivalent non-competing page
3. Instead: fetch that competitor page, identify what *they're* linking to (their primary sources), and use those upstream sources instead
4. If Claude suspects a source might be a competitor and isn't sure, flag it: state the URL, why it might be a conflict, and what Claude recommends

**Example**: Writing a LinkedIn automation article for a LinkedIn automation client. A strong data point appears on a MeetAlfred blog post. Don't link MeetAlfred. Check what MeetAlfred linked to — if it's a LinkedIn official announcement or a study, link that instead.

---

## URL Verification Fallback

**NEVER flag a stat or claim as unverifiable based on memory or assumption. You MUST attempt `web_fetch` on the source URL first, then fall back to `web_search` if that fails. Flagging without attempting both steps is a process violation — no exceptions.**

If `web_fetch` fails on a URL (403, timeout, or any error), do NOT flag the claim as unverifiable. Immediately fall back to `web_search` using the report or page title and the specific stat or claim as keywords. A failed fetch means the page is blocking direct access -- it does not mean the source doesn't exist or the stat is wrong. Only flag a claim as unverifiable if both `web_fetch` AND `web_search` return nothing useful.

If sources disagree on a fact, stat, or claim:

1. Present both sides clearly — what each source says and where it's from
2. Dig deeper: run 1–2 more searches to find additional sources that can break the tie
3. Give a recommendation: state which version Claude thinks is more likely correct and why (recency, source authority, methodology if visible)
4. Flag any version that could be wrong and why — even the one Claude recommends

Don't paper over conflicts. Surface them.

---

## SEO Content Integration

When used inside an article or content writing workflow:

- Identify claims in the draft/outline that need backing
- Research each claim separately, don't bundle unrelated claims into one search
- Return source + one-liner on what it proves, formatted ready to drop into the article
- Flag if a source being considered is a ranking competitor for that topic
- If no strong source exists for a claim, say so — don't force a weak citation

---

## What Claude Should Not Do

- **Never flag a stat as unverifiable without first attempting `web_fetch` on the source URL and falling back to `web_search`. Doing so is a process violation.**
- Don't pad the source list with weak sources just to hit a number
- Don't cite a source without actually reading it (use `web_fetch`)
- Don't summarize what a page title implies — read the content
- Don't link competitors when writing SEO content, even if their content is good
- Don't present conflicting data as settled — flag it