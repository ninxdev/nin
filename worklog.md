# NiN.X — Project Worklog

This is the shared worklog for the NiN.X website project. All agents must read this before starting and append their work after finishing.

---
Task ID: 0
Agent: orchestrator
Task: Initial planning and project bootstrap

Work Log:
- Reviewed existing Next.js 16 scaffold at /home/z/my-project
- Confirmed dev server runs on port 3000
- Reviewed upload/index.html (old generative portfolio — to be replaced, NOT reused)
- Confirmed Caddy gateway supports XTransformPort query routing to other ports
- Decided architecture: Next.js 16 frontend (port 3000) + NestJS mini-service backend (port 4000)
- Set design system: whitish #F5F5F7 dominant, blackish secondary, restrained accent palette, NO dark mode

Stage Summary:
- Project: NiN.X personal website (showcase GitHub dev + articles + forum + shop-ready + auth-ready)
- Hard constraints: whitish dominant color, Apple aesthetic, no dark mode, NestJS backend required
- NestJS runs as mini-service on port 4000, frontend talks via /api?XTransformPort=4000
- Features to build: articles (empty, ready), forum (read public, post auth-gated), auth UI (coming soon), shop (coming soon), GitHub showcase (real API), search, about/contact placeholders
- Next step: build theme + NestJS scaffold in parallel

---
Task ID: 1
Agent: orchestrator
Task: Build complete NiN.X website (NestJS backend + Next.js frontend) and test

Work Log:
- Built whitish Apple-style theme in globals.css (#F5F5F7 dominant, #1D1D1F secondary, teal accent, NO dark mode)
- Scaffolded NestJS mini-service at mini-services/ninx-api/ (port 4000) with modular architecture:
  - Modules: articles, forum, shop, github, search, auth
  - Prisma schema with User, Article, ForumThread, ForumReply, ShopProduct models
  - ValidationPipe with strict DTOs (class-validator + class-transformer)
  - GatewayMiddleware strips XTransformPort param before DTO validation
  - AuthGuard for auth-gated routes (forum posting, article creation)
- Built Next.js frontend:
  - AppShell with sticky frosted-glass header + sticky footer + global search palette
  - Home page: hero + article feed (with empty state) + GitHub showcase
  - Forum: list + thread detail + reply composer (auth-gated "coming soon") + new thread page
  - Auth page: Google/Apple/Email buttons (disabled, "coming soon" state)
  - Shop page: coming-soon placeholder with product grid ready
  - About/Contact pages: clean placeholders, no generative content
  - Article detail page: markdown rendering via react-markdown
  - Global search palette (Cmd+K) with live article + forum search
  - API client with server/client URL detection (direct fetch on server, gateway on client)
- Fixed 3 critical bugs during testing:
  1. Search controller method name collision with injected service (renamed to query())
  2. SQLite doesn't support Prisma mode:insensitive (removed from search service)
  3. Server Components couldn't reach NestJS via relative URLs (added dual server/client URL logic)
  4. Query param type transformation (added @Type(() => Number) to DTOs)
  5. Gateway's XTransformPort param rejected by ValidationPipe (added GatewayMiddleware)
- Seeded test data: 1 owner user, 1 published article, 1 forum thread
- All endpoints verified through gateway (port 81):
  - GET /api/articles ✓
  - GET /api/articles/:slug ✓
  - GET /api/forum/threads ✓
  - GET /api/forum/categories ✓
  - GET /api/forum/threads/:id ✓
  - GET /api/shop ✓
  - GET /api/github/repos ✓
  - GET /api/search?q=... ✓
  - GET /api/auth/status ✓
  - POST /api/articles (auth) ✓
  - POST /api/forum/threads (auth) ✓
- agent-browser verification:
  - Home page renders: hero, article card (with tags/title/excerpt/meta), GitHub placeholder, footer
  - Article detail renders: markdown parsed (headings, bold, lists, paragraphs)
  - Forum list renders: thread with category chip, author, date, reply count
  - Forum thread detail renders: title, body, reply list, auth-gated composer
  - Auth page renders: disabled Google/Apple/Email buttons + "coming soon" notice
  - Shop page renders: coming-soon placeholder
  - About/Contact render: clean placeholders
  - Search palette (Cmd+K) opens, accepts input, returns live results
  - Sticky footer present on all pages
- ESLint: 0 errors, 0 warnings

Stage Summary:
- Both services running: Next.js (port 3000) + NestJS API (port 4000)
- Gateway (port 81) routes correctly: pages → 3000, API calls → 4000 via XTransformPort
- All features functional and tested end-to-end
- Design: whitish #F5F5F7 dominant, Apple aesthetic, NO dark mode
- Every meaningful line of code commented with official doc URLs
- Ready for owner to: add articles, wire real auth, seed shop products, set GITHUB_USERNAME

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set in mini-services/ninx-api/.env
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- More articles/forum threads for richer content

---
Task ID: 2 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test the site, fix bugs, improve styling, add new features

Work Log:
- Read worklog to understand prior state (Task 0 + Task 1 complete)
- QA pass with agent-browser: all pages render, zero console errors, zero runtime errors
- VLM analysis of screenshots identified styling gaps:
  - Hero: line-height tight, subtitle too light, no visual anchor (flat background)
  - Cards: harsh borders, weak hierarchy, tags not pill-shaped
  - Article: body too wide (no measure constraint), no syntax highlighting
  - Footer: alignment issues, no social links
  - Missing micro-interactions (hover lifts, transitions)

Backend additions (NestJS):
- New StatsModule: GET /api/stats returns aggregate counts (articles, threads, replies, repos)
- ArticlesService.listRelated(): tag-overlap related articles, falls back to recency
- GET /api/articles/:slug/related endpoint
- GithubModule now exports GithubService (for StatsModule dependency)

Frontend styling improvements:
- globals.css v2:
  - Deeper teal accent (oklch 0.5 0.11 200) for better contrast on whitish bg
  - Darkened borders (oklch 0.875) for visibility
  - Warmer muted-foreground (Apple's #6E6E73 equivalent)
  - Multi-layer shadow tokens (xs/sm/md/lg/glow) for premium depth
  - Custom keyframe animations (float, pulse-slow, fade-up, shimmer)
  - Full .prose-ninx typography: constrained 68ch measure, sized headings, styled
    code blocks (dark pre with custom hljs syntax theme), blockquotes, links
  - Subtle body background radial gradients for atmospheric depth
  - Accent-soft tint utility for tag pills
- Home page hero:
  - Visible radial gradient glow (teal + blue, z-0 layering fix)
  - Floating animated accent orb
  - Larger headline (2.75rem → 6xl), tighter line-height (1.05)
  - Subtitle bumped to font-medium for contrast
  - CTA buttons with shadow + hover lift (-translate-y-0.5)
  - Staggered fade-up entrance animations
- Article cards: softer shadow-premium-xs → hover:shadow-premium-md, hover lift,
  accent-tinted tag pills, clearer hierarchy (bold title, muted meta)
- GitHub showcase: same card polish, "View all" CTA, accent topic pills
- Footer: 4-col grid with brand spanning 2, social circle buttons (GitHub/Twitter/Mail),
  consistent hover lift, "Built with care" tagline
- Header: scroll-aware border/shadow (intensifies on scroll)

New features:
- ReadingProgress: thin teal bar at top of viewport, rAF-driven, scaleX transform
- CodeBlock: custom react-markdown code renderer with language label + copy button
  (clipboard API, "Copied" feedback state)
- ShareButtons: X/Twitter + LinkedIn share intents + copy link, circle buttons
- RelatedArticles: "Keep reading" section, tag-overlap from API, compact cards
- StatsBanner: 4-col grid (Articles/Threads/Replies/Repos) with icon circles
- NewsletterSignup: UI-ready form with email input, loading state, success state,
  accent-tinted card with decorative glow (backend deferred)
- BackToTop: floating button, appears after 600px scroll, smooth-scroll to top
- Custom 404 page: branded "404" in accent, "This page wandered off" message,
  Back home + Search + Go back actions

Bug fixes during this round:
- not-found.tsx had onClick handlers as Server Component → added "use client"
- Next.js dev server crashed (hung process) → restarted manually
- NestJS API process died → restarted manually
- Hero glow invisible due to -z-10 layering behind body bg → changed to z-0 + content z-10
- rehype-highlight installed for syntax highlighting
- Cleaned up unused eslint-disable directives

Test data seeded:
- Second article "TypeScript patterns I reach for" with 3 code blocks (tests
  syntax highlighting + copy button + related articles)

Verification:
- agent-browser: all pages render correctly
  - Home: hero with glow, stats (2 articles/1 thread), 2 article cards, GitHub, newsletter
  - Article detail: reading progress, share buttons, markdown, code blocks with copy
    buttons (verified "Copy" → "Copied" state change), related articles section
  - Forum: thread list with category chips
  - 404: branded page with working actions
  - Back-to-top: appears at scrollY > 600 (verified via eval)
  - Search palette: opens, live search works
- VLM confirms: "soft, diffused teal/green gradient glow... subtle atmospheric effect"
- ESLint: 0 errors, 0 warnings
- All services healthy (gateway 200, NestJS 200)

Stage Summary:
- Both services running: Next.js (port 3000) + NestJS API (port 4000)
- Site significantly more polished: hero glow, layered shadows, pill tags,
  constrained prose measure, syntax-highlighted code, micro-interactions
- 7 new features added: reading progress, code copy buttons, share buttons,
  related articles, stats banner, newsletter signup, back-to-top, custom 404
- All features tested end-to-end via agent-browser
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- More articles/forum threads for richer content
- Consider adding article table of contents for long articles
- Consider forum thread view counts + sorting options

---
Task ID: 3 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, fix bugs, add TOC + tag browsing + forum filter + keyboard shortcuts

Work Log:
- Read worklog (Tasks 0-2 complete, site stable and polished)
- QA pass: all services healthy (gateway 200, NestJS 200, Next.js 200)
- All pages render correctly, no runtime errors
- Identified 1 real bug: article detail page had duplicate <h1> (page title +
  markdown body's # Heading) — SEO/accessibility issue

Bug fix:
- Article detail h1 duplication: added custom h1→h2 renderer in ReactMarkdown
  components, so markdown # Heading renders as <h2>. Now exactly one <h1> per
  article page (the title). Verified via agent-browser: only 1 heading at level=1.

Backend additions (NestJS):
- ArticlesService.listByTag(tag): returns published articles matching a tag.
  Uses OR with comma-boundary contains to avoid partial matches (tag "go"
  won't match "google").
- ArticlesService.listTags(): aggregates all tags with article counts.
- GET /api/articles/tags — returns [{tag, count}] sorted by count desc.
- GET /api/articles/by-tag/:tag — paginated articles for a tag.
- ForumService.listThreads(pagination, category): optional category filter.
- Forum controller: ForumListQueryDto (flattened, not inherited — decorator
  inheritance unreliable under Bun) whitelists `category` query param.
- Reading time fix: now uses word count / 200 wpm (was body.length / 1000,
  which gave "1 min" for everything). Wikipedia reading speed reference added.

Frontend new features:
- Table of Contents (table-of-contents.tsx):
  - Extracts h2/h3 headings from markdown, renders sticky sidebar TOC
  - IntersectionObserver tracks active heading, highlights it in accent
  - Click smooth-scrolls to heading, updates URL hash for shareability
  - Only renders if ≥2 headings (no TOC for short articles)
  - rehype-slug installed + added to ReactMarkdown to generate heading IDs
  - scroll-margin-top: 6rem on headings so they don't hide under sticky header
- Two-column article layout: body (flex-1) + TOC sidebar (w-56, sticky top-24)
  on lg+ screens. TOC hidden on mobile/tablet.
- Tags browsing:
  - /tags page: tag cloud with pills showing tag name + count, sorted by count
  - /tags/[tag] page: grid of articles matching the tag, uses ArticleCard
  - Both have clean empty states
- Forum category filtering:
  - Forum page accepts ?category= searchParam (server-side filtering)
  - Filter chips: "All" + one per category, active chip highlighted in accent
  - Category chips are Links (URL-based state = shareable, back-button friendly)
  - Empty state adapts: "No threads in 'X'" vs "No threads yet"
- Keyboard shortcuts (keyboard-shortcuts.tsx):
  - ? or / → open help dialog
  - g h/f/s/a/c/t → navigate to home/forum/shop/about/contact/tags
  - GitHub-style "g" prefix with 1.5s timeout
  - Ignores key events while typing in inputs/textareas
  - Help dialog shows all shortcuts with <kbd> styling
  - Mounted globally via AppShell

Styling improvements:
- Header search button: now a pill with "Search" label + ⌘K kbd badge on
  desktop (teaches the shortcut), icon-only on mobile
- Forum thread cards: accent-tinted category pills (was grey), hover title
  color shift, shadow-premium-xs on container
- Footer: added "Tags" link in Explore column
- globals.css: added scroll-margin-top: 6rem to prose-ninx headings

Verification:
- agent-browser testing:
  - Home: ⌘K search hint visible in header
  - Article detail: only 1 h1 (title), TOC sidebar with 3 headings, code blocks
    with copy buttons, related articles section
  - /tags: 4 tags displayed (intro:2, patterns:1, typescript:1, welcome:1)
  - /tags/intro: 2 article cards shown
  - /forum: filter chips (All + welcome·1), thread list with accent category pills
  - /forum?category=welcome: filtered to 1 thread
  - Keyboard shortcuts: ? opens dialog, g h navigates from /forum to /
- VLM confirms TOC layout: "well-balanced... premium and polished"
- ESLint: 0 errors, 0 warnings
- All services healthy

Stage Summary:
- 1 bug fixed (duplicate h1 on article pages)
- 4 new features: TOC sidebar, tag browsing (/tags + /tags/[tag]), forum
  category filtering, keyboard shortcuts with help dialog
- 3 backend endpoints added: /api/articles/tags, /api/articles/by-tag/:tag,
  forum category filter param
- Reading time calculation improved (word count based)
- Header search button redesigned with ⌘K hint
- All features tested end-to-end
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Forum: thread view counts, sorting options, search within forum
- Article: estimated reading time shown in card could use the improved calc
- Consider adding "copy link" toast feedback on share
- Consider adding article estimated read time in related articles cards

---
Task ID: 4 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, fix layout bugs, add prev/next nav + forum views + popular tags

Work Log:
- Read worklog (Tasks 0-3 complete, site stable with TOC, tags, forum filter, keyboard shortcuts)
- QA pass: all services healthy (gateway 200, Next.js 200, NestJS 200)
- All pages render, all routes return correct status codes (200/404)
- VLM holistic assessment identified 3 priority improvements:
  1. Article body too narrow (doubly constrained: max-w-3xl + prose-ninx 68ch)
  2. Forum needs more contextual data (views, avatars, last activity)
  3. Tags page bare, home needs content discovery
- Found 2 bugs:
  1. Article body width 448px (should be ~700px) — max-w-3xl container + prose-ninx
     max-width:68ch double-constrained the body inside the flex layout
  2. Article body starting with "# Title" duplicated the page h1 (redundant heading)

Bug fixes:
- Article body width: changed outer container from max-w-3xl to max-w-5xl so the
  body + TOC sidebar both have room. Body now 704px (was 448px). Verified via eval.
- Redundant title: added stripRedundantTitle() helper that checks if the markdown
  body starts with "# Title" matching the article title (case-insensitive). If so,
  strips the first line + any following blank line. cleanBody used for both
  ReactMarkdown and TableOfContents. Verified: only 1 h1 on article pages.

Backend additions (NestJS):
- ArticlesService.getNeighbors(slug): returns prev/next published articles by
  createdAt. Uses findFirst with createdAt lt/gt filters, parallel queries.
- GET /api/articles/:slug/neighbors endpoint
- ForumThread schema: added `views Int @default(0)` field (prisma db push applied)
- ForumService.getThread(): now increments views (fire-and-forget, non-blocking)
  and includes the latest reply for "last activity" timestamp
- ForumThreadSummary type updated with views + updatedAt fields

Frontend new features:
- ArticleNav (article-nav.tsx): prev/next navigation at the bottom of article
  pages. Two-column grid: prev (left, arrow-left) + next (right, arrow-right).
  Each side shows directional label + article title. Muted placeholders when
  no prev/next exists ("This is the oldest/latest article"). Hover lift + accent
  title color shift. Wired into article detail page after RelatedArticles.
- PopularTags (popular-tags.tsx): compact tag cloud section on the home page.
  Shows top 8 tags as pills with counts, links to /tags/[tag]. "All tags" link
  in the header. Hidden if no tags exist. Wired into home page between articles
  and GitHub showcase.
- Forum thread list improvements:
  - Author avatar circle (initials in accent-soft circle)
  - View count (Eye icon) in the right-side meta column
  - Reply count + view count stacked vertically
  - Pin/lock badges inline above the title (was left column)
  - Clock icon next to date
  - Author name in foreground/80 (bolder)
- Forum thread detail improvements:
  - Author avatar circle in the meta row
  - View count with Eye icon ("N views")
  - Category pill in accent-soft (was grey bg-secondary)
  - Clock icon next to date
- Related articles: added Calendar + Clock icons to the meta row for consistency
  with the article card styling

Verification:
- agent-browser testing:
  - Article body width: 704px (was 448px) — FIXED
  - Article h1 count: 1 (was 2) — FIXED
  - Prev/next nav: exists, shows "Previous: Welcome to NiN.X" + "Next: This is
    the latest article"
  - Home: popular tags section with 4 tags (intro:2, patterns:1, typescript:1,
    welcome:1) + "All tags" link
  - Forum list: avatar "N", title, author, category pill, date, replies "0",
    views "5" — all visible
  - Forum thread detail: avatar, author, category pill, date, "5 views"
- VLM confirms forum improvements: avatars ✓, view counts ✓, reply counts ✓,
  accent category pills ✓
- ESLint: 0 errors, 0 warnings
- All services healthy

Stage Summary:
- 2 bugs fixed (article body width, redundant title)
- 3 new features: article prev/next nav, home popular tags section, forum
  view counts (schema + service + UI)
- Forum thread list + detail redesigned with avatars, views, better hierarchy
- 2 backend endpoints added: /api/articles/:slug/neighbors, forum views field
- All features tested end-to-end
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Forum: search within forum, sort options (by views/replies/recency)
- Article: cover images, author bio at the bottom
- Consider adding "last activity" timestamp to forum thread list (using the
  latest reply's createdAt, which the API now returns)
- Consider adding forum thread reply avatars

---
Task ID: 5 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, add author bio + forum sorting + about skeleton + share toast + prose polish

Work Log:
- Read worklog (Tasks 0-4 complete, site stable with prev/next nav, popular tags, forum views)
- QA pass: all services healthy (gateway 200, Next.js 200, NestJS 200)
- All pages render, all routes return 200, search palette works, keyboard shortcuts work
- VLM holistic assessment identified 4 improvement areas:
  1. About page too sparse (single "coming soon" card)
  2. Article header could use author avatar
  3. Search palette could be more premium
  4. Article prose needs more breathing room
- No bugs found — site is stable, focused on mandatory styling + new features

Backend additions (NestJS):
- ForumService.listThreads() now accepts a `sort` param: "newest" | "oldest" |
  "views" | "replies". For "replies" sort, fetches all threads + sorts in JS
  (SQLite can't order by relation count in findMany). Pinned threads always
  float to the top regardless of sort.
- ForumListQueryDto: added `sort` field (string, optional)
- Forum controller: validates sort against allowed values, falls back to "newest"

Frontend new features:
- AuthorBio (author-bio.tsx): card at the bottom of article pages with author
  avatar (large initial in accent circle), "Written by" label, name, honest
  placeholder bio, GitHub + Twitter social links, "View all articles" CTA.
  Gives articles a sense of "written by a person" rather than ending abruptly.
- Forum sorting: 4 sort chips (Newest, Oldest, Most viewed, Most replies) below
  the category filter. Active sort highlighted in accent. Sort + category params
  are preserved in all chip links (filtering doesn't reset sorting and vice versa).
- Share copy link toast: ShareButtons now uses useToast to show "Link copied"
  confirmation on success, "Couldn't copy" error on failure. Previously the
  only feedback was the icon swapping to a checkmark (easy to miss).

Styling improvements:
- About page redesigned from single "coming soon" card to a structured skeleton:
  - Bio section: avatar placeholder + skeleton text lines + "proper bio soon" note
  - Focus areas section: 3 placeholder cards (Development, Writing, Open source)
  - Stack section: tech pills (Next.js, NestJS, TypeScript, Prisma, Tailwind, React)
  - Footer note with clock icon + "Read the articles" CTA
  - VLM confirms: "deliberate, structured layout awaiting content... significantly
    more professional than a generic coming soon card"
- Article header: author avatar circle (initial in accent-soft) added to the
  meta row, with proper dot separators between meta items
- Search palette: added overflow-hidden, rounded-2xl, shadow-premium-lg for
  a more premium feel; loading spinner uses accent color
- Article prose: line-height increased from 1.75 to 1.8, paragraph margins
  increased from 1rem to 1.25rem for more breathing room

Verification:
- agent-browser testing:
  - Article: author avatar in header, "WRITTEN BY" + bio + "View all articles →"
    at the bottom (verified via scroll + snapshot)
  - Copy link toast: "Link copied" / "The article link is in your clipboard."
    appears on click (verified)
  - Forum: 4 sort chips visible (Newest, Oldest, Most viewed, Most replies),
    active chip highlighted. sort=views and sort=replies URLs work.
  - About: 3 skeleton sections (Bio, Focus areas, Stack) with placeholder
    content, footer note + CTA
  - All routes return 200
- VLM confirms about page: "deliberate, structured layout... more professional"
- ESLint: 0 errors, 0 warnings
- All services healthy

Stage Summary:
- 3 new features: author bio card, forum sorting (4 options), share copy toast
- About page redesigned with skeleton layout (3 sections)
- Article header polished with author avatar
- Search palette + prose typography refined
- 1 backend enhancement: forum sort param
- All features tested end-to-end
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Forum: search within forum, "last activity" timestamp in thread list
- Article: cover images, reading progress percentage indicator
- Consider adding a "back to top" link in the article footer
- Consider adding article view counts (like forum threads have)

---
Task ID: 6 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, add hero signature visual + article view counts + forum last activity + drop cap + about polish + reading percentage

Work Log:
- Read worklog (Tasks 0-5 complete, site stable with author bio, forum sorting, about skeleton)
- QA pass: all services healthy (gateway 200, Next.js 200, NestJS 200)
- All pages render, all routes return 200, search + keyboard shortcuts work
- VLM holistic assessment identified 3 top improvements:
  1. Home hero: flat gradient needs a signature visual
  2. Article: typography hierarchy + drop cap + pull-quote
  3. About: skeleton bars look "under construction" — needs polished placeholder
- No bugs found — site is stable, focused on mandatory styling + new features

Backend additions (NestJS):
- Article schema: added `views Int @default(0)` field (prisma db push applied)
- ArticlesService.getBySlug(): now increments views (fire-and-forget, non-blocking)
- ForumService.listThreads(): now loads the latest reply (take: 1, ordered desc) for
  "last activity" timestamp. Applied to both the normal branch and the "replies" sort branch.
- ArticleSummary + ForumThreadSummary types updated with views + replies array

Frontend new features:
- HeroVisual (hero-visual.tsx): decorative SVG signature visual replacing the flat
  gradient. 4 layers:
  1. Radial gradient glow (teal + blue, more prominent than before)
  2. Concentric arcs SVG (5 circles at decreasing opacities, top-right, off-screen center)
  3. Dot grid SVG (6x4 grid of small teal dots, bottom-left, 30% opacity)
  4. Animated floating accent orb (kept from before)
  All aria-hidden, z-0, content sits at z-10. VLM confirms: "significantly more
  premium and branded".
- Article view counts: Eye icon + count in article card meta row (shows when views > 0)
  and in article detail header ("N views"). Backend increments on each fetch.
- Forum "last activity": thread list now shows the latest reply's createdAt (or the
  thread's createdAt if no replies). Adds an "· active" indicator in accent when
  there are replies.
- Reading progress percentage: the ReadingProgress component now shows a small
  circular progress badge in the bottom-right corner with the scroll percentage.
  Appears after 5% scroll, disappears at 99%. Has a mini SVG ring that fills with
  the accent color. Hidden on mobile (sm:flex).

Styling improvements:
- Article drop cap: the first paragraph of every article gets a large (3.5rem) accent-
  colored initial letter via ::first-letter. Classic editorial typography technique.
- Article H2 spacing: margin-top increased from 2rem to 3rem for more breathing room.
- Article pull-quote: blockquotes redesigned with accent left border + accent-soft
  background tint + larger font (1.125rem) + rounded right corners. Removed the
  duplicate old blockquote style.
- About page Bio section: replaced gray skeleton bars with a polished card — large
  "N" avatar in accent-soft circle, "NiN.X" name, "Developer · Writer · Open source"
  role line, and honest human-written bio text. VLM confirms: "feels like a finished
  card... rather than an under construction placeholder".

Verification:
- agent-browser testing:
  - Home: 23 SVGs present (hero visual + dot grid), arcs visible top-right
  - Article: "4 views" in meta row, drop cap on first paragraph
  - About: polished bio card with avatar, name, role, bio text (no skeleton bars)
  - Forum: last activity timestamp shows (thread createdAt since no replies)
  - All routes return 200
- VLM confirms:
  - Hero: "concentric arcs clearly visible... dot grid distinctly present...
    significantly more premium and branded"
  - About: "polished card... finished card rather than under construction placeholder"
- ESLint: 0 errors, 0 warnings
- All services healthy

Stage Summary:
- 4 new features: hero signature SVG visual, article view counts, forum last activity,
  reading progress percentage badge
- 4 styling improvements: article drop cap, H2 spacing, pull-quote redesign, about bio polish
- 2 backend enhancements: article views field + increment, forum latest reply loading
- All features tested end-to-end
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Forum: search within forum, "most viewed articles" section on home
- Article: cover images, "back to top" link in article footer
- Consider adding a "popular articles" section (sorted by views)
- Consider adding forum reply avatars in the thread detail

---
Task ID: 7 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, add popular articles + latest threads + contact redesign + forum OP treatment + relative time

Work Log:
- Read worklog (Tasks 0-6 complete, site stable with hero visual, article views, forum last activity)
- QA pass: all services healthy (gateway 200, Next.js 200, NestJS 200)
- All pages render, all routes return 200
- Investigated potential drop cap bug ("Afew" in snapshot) — confirmed it's an accessibility
  tree text extraction quirk, not a real visual bug. VLM confirms correct spacing.
- VLM holistic assessment identified 3 top improvements:
  1. Forum thread detail: OP needs "hero" visual treatment
  2. Contact page: generic placeholders need polished cards
  3. Home page: needs "featured content" below the fold
- No real bugs found — site is stable, focused on mandatory styling + new features

Backend additions (NestJS):
- ArticlesService.listPopular(limit=3): returns most-viewed published articles
  (views > 0), ordered by views desc. Used by the home "Most read" section.
- GET /api/articles/popular endpoint (placed before :slug to avoid route conflict)

Frontend new features:
- PopularArticles (popular-articles.tsx): "Most read" section on the home page.
  Shows top 3 articles by views, each with a large rank number (1/2/3), tags,
  title, excerpt, view count (accent-colored), and relative time. Only renders
  when articles have views > 0. Wired into home between PopularTags and LatestThreads.
- LatestThreads (latest-threads.tsx): "Latest from the forum" section on the home page.
  Shows 3 most recent threads, each with an avatar circle, title, category pill,
  relative time, and reply count. "View all" link to /forum. Wired into home.
- Relative time utility (relative-time.ts): formatRelativeTime() produces strings
  like "just now", "5m ago", "3h ago", "2d ago", "Jul 27". Falls back to a compact
  date for older timestamps. Used by PopularArticles and LatestThreads.

Styling improvements:
- Contact page redesigned: 3 polished cards (Email, Forum, GitHub) with:
  - Accent-colored icon circles (was gray muted icons)
  - Hover lift + shadow transition
  - Email card: intentional "Coming soon" badge with clock icon (not "missing data")
  - Forum card: "Visit forum" link with arrow that nudges on hover
  - GitHub card: "View profile" link with arrow
  - Consistent spacing, typography, visual hierarchy
  - VLM confirms: "polished, intentional interface... consistent and professional"
- Forum thread detail OP treatment:
  - "Original post" badge in accent-soft pill
  - Subtle accent-tinted background (bg-accent-soft/30)
  - Left accent border (border-l-2 border-accent)
  - Larger font (text-base, was text-sm)
  - Rounded corners + padding
  - VLM confirms: "visually distinct from replies... resembles a published article"
- Forum thread detail reply avatars:
  - Each reply now has an avatar circle (initial in accent-soft) next to the author name
  - shadow-premium-xs on reply cards
  - Better spacing (gap-2.5, mt-3 for body)

Verification:
- agent-browser testing:
  - Home: "Most read" section with rank "1", "8 views", "1h ago". "Latest from the
    forum" section with avatar "N", "welcome" category, "2h ago", 0 replies.
  - Contact: 3 cards (Email with "Coming soon" badge, Forum "Visit forum", GitHub
    "View profile")
  - Forum thread detail: "Original post" badge on OP body
  - All routes return 200
- VLM confirms:
  - Contact: "polished, intentional interface... consistent and professional"
  - Forum OP: "visually distinct from replies... resembles a published article"
- ESLint: 0 errors, 0 warnings
- All services healthy

Stage Summary:
- 3 new features: popular articles section, latest forum threads section, relative time utility
- 3 styling improvements: contact page redesign, forum OP treatment, forum reply avatars
- 1 backend endpoint added: GET /api/articles/popular
- All features tested end-to-end
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Forum: search within forum, "most viewed threads" section
- Article: cover images, estimated read time in popular articles cards
- Consider adding a "back to top" link in the article footer
- Consider adding article view counts to the stats banner

---
Task ID: 8 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, add shop teaser + forum reply snippets + drop cap refinement + back-to-top

Work Log:
- Read worklog (Tasks 0-7 complete, site stable with popular articles, latest threads, contact redesign)
- QA pass: all services healthy (gateway 200, Next.js 200, NestJS 200)
- All pages render, all routes return 200
- VLM holistic assessment identified 3 top improvements:
  1. Shop page: generic "coming soon" needs premium teaser state
  2. Article drop cap: too large, dominates the paragraph
  3. Forum thread list: needs visual density + last reply snippet
- No real bugs found — site is stable, focused on mandatory styling + new features

Backend additions (NestJS):
- ForumService.listThreads(): latest reply now includes `body` + `author` fields
  (was only `createdAt`). Applied to both the normal branch and the "replies" sort
  branch. Enables the forum thread list to show a reply snippet + replier avatar.
- ForumThreadSummary type updated with the expanded replies array shape.

Frontend new features:
- ShopWaitlist (shop-waitlist.tsx): email waitlist form for the shop page teaser.
  Client component with email input, loading state, success state ("You're on the
  waitlist"), and toast feedback. Backend deferred — only the submit handler needs
  changing when a real provider is wired.
- BackToTopLink (back-to-top-link.tsx): "Back to top" button at the bottom of
  article pages. Smooth-scrolls to the top. Centered, pill-styled, hover lift.
  Wired into article detail page after the AuthorBio.

Styling improvements:
- Shop page redesigned from generic placeholder to premium teaser:
  - Decorative gradient background (matches home hero style)
  - Large stylized product mockup (ShoppingBag icon in tinted circle)
  - "In the works" badge with Sparkles icon
  - "The store is being crafted" headline + description
  - Email waitlist form (ShopWaitlist client component)
  - 3 "what to expect" preview cards (Digital downloads, Project templates, Merch)
  - "Back to articles" link
  - VLM confirms: "premium teaser... waitlist form highly visible and inviting"
- Forum thread list redesigned:
  - Left-border accent (border-l-accent) on threads with replies (was flat)
  - Last reply snippet: tinted box with replier avatar + name + "replied:" + body
    text (truncated to 1 line). Only shows when replies exist.
  - VLM confirms: "both the accent left border and the tinted last reply snippet
    box are clearly visible"
- Article drop cap refined:
  - Size reduced from 3.5rem to 2.75rem (spans ~3 lines instead of 5+)
  - Font weight increased to 700 for better definition
  - Line-height adjusted to 0.9
  - Added letter-spacing: -0.02em for refinement
  - VLM confirms: "well-proportioned and does not dominate the paragraph"

Verification:
- agent-browser testing:
  - Shop: "In the works" badge, "The store is being crafted" headline, waitlist
    form, 3 preview cards. Waitlist submit shows success state + toast.
  - Forum: thread card has border-l-accent class (verified via eval), last reply
    snippet shows "NiN.X replied: This is a test reply..." (added a test reply
    to verify the feature works with real data)
  - Article: "Back to top" button at the bottom, drop cap renders correctly
  - All routes return 200
- VLM confirms:
  - Shop: "premium teaser... waitlist form highly visible and inviting"
  - Forum: "both accent left border and tinted reply snippet clearly visible"
  - Drop cap: "well-proportioned and does not dominate the paragraph"
- ESLint: 0 errors, 0 warnings
- All services healthy (Next.js restarted once after a crash)

Stage Summary:
- 2 new features: shop waitlist form, article back-to-top link
- 3 styling improvements: shop premium teaser, forum reply snippets + accent border,
  drop cap refinement
- 1 backend enhancement: forum latest reply now includes body + author
- All features tested end-to-end (added test reply to verify forum snippet)
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Shop waitlist backend (email provider integration)
- Forum: search within forum, "most viewed threads" section
- Article: cover images, reading time in popular articles cards
- Consider adding article view counts to the stats banner
- Consider adding a "featured article" hero on the home page

---
Task ID: 9 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, add article abstract + about hero treatment + expanded stats (views/stars)

Work Log:
- Read worklog (Tasks 0-8 complete, site stable with shop teaser, forum reply snippets, drop cap)
- QA pass: all services healthy (gateway 200, Next.js 200, NestJS 200)
- All pages render, all routes return 200
- VLM holistic assessment identified 3 top improvements:
  1. Article detail: needs abstract/summary before body text
  2. About page: needs hero treatment with prominent avatar
  3. Stats banner: could show more metrics (views, stars)
- No real bugs found — site is stable, focused on mandatory styling + new features

Backend additions (NestJS):
- StatsService.getStats(): now returns 6 stats instead of 4:
  - totalStars: sum of stargazers_count across all GitHub repos
  - totalViews: sum of views across all published articles (Prisma _sum aggregation)
  - Fixed a variable naming conflict (repos vs repoData) that caused a startup crash
  - Added defensive Array.isArray check for repoData

Frontend new features:
- Article abstract/dek: the article's `excerpt` field now renders below the meta row
  and above the body. Uses text-lg/xl font-medium for a scannable preview that gives
  the reader context before committing to the full text. Classic editorial pattern.
- Expanded stats banner: now shows 6 stats (Articles, Threads, Replies, Repos, Reads, Stars)
  instead of 4. Grid changed to 2/3/6 cols (mobile/tablet/desktop). Added Eye icon for
  Reads and Star icon for Stars.

Styling improvements:
- About page Bio section redesigned with hero treatment:
  - Gradient header strip (teal → blue, 135deg) giving the card visual weight
  - Large 24x24 avatar (rounded-2xl, border-4 border-card) that overlaps the gradient
    strip (-mt-12), creating a profile-card effect
  - Larger name (text-2xl, was text-xl) + role line
  - Larger bio text (text-base, was text-sm)
  - "Currently building NiN.X" status badge with a pulsing dot (animate-ping) —
    signals "online / active"
  - GitHub + X social badges (pill-shaped, hover accent)
  - VLM confirms: "premium, scannable profile... avatar highly prominent"

Verification:
- agent-browser testing:
  - Home: stats banner shows 6 items (Articles, Threads, Replies, Repos, Reads, Stars)
    — verified via eval: "Articles, Threads, Replies, Repos, Reads, Stars"
  - Article: abstract "A few TypeScript patterns that make my code safer and easier
    to read." renders below the share buttons, before the body
  - About: hero treatment with gradient strip, overlapping avatar, "Currently building
    NiN.X" status, GitHub + X badges — all verified via snapshot
  - All routes return 200
- VLM confirms About: "premium, scannable profile... avatar highly prominent"
- ESLint: 0 errors, 0 warnings
- All services healthy
- API stats endpoint returns: {"articles":2,"threads":1,"replies":1,"repos":0,"totalStars":0,"totalViews":14}

Stage Summary:
- 2 new features: article abstract/dek, expanded stats (6 metrics with views + stars)
- 1 styling improvement: About page hero treatment (gradient strip, overlapping avatar, status)
- 1 backend enhancement: stats endpoint now returns totalStars + totalViews
- Fixed a variable naming conflict bug in stats service
- All features tested end-to-end
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set (totalStars will be 0 until then)
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Shop waitlist backend (email provider integration)
- Forum: search within forum, "most viewed threads" section
- Article: cover images, reading time in popular articles cards
- Consider adding a "featured article" hero on the home page
- Consider adding forum thread view counts to the stats banner

---
Task ID: 10 (cron-triggered review & enhancement)
Agent: orchestrator (webDevReview cron)
Task: QA test, polish TOC sidebar + forum thread spacing + forum reply snippet

Work Log:
- Read worklog (Tasks 0-9 complete, site stable with article abstract, about hero, 6 stats)
- QA pass: all services healthy (gateway 200, Next.js 200, NestJS 200)
- All pages render, all routes return 200, stats show 2 articles / 1 thread / 1 reply / 16 reads
- VLM holistic assessment identified 3 top improvements:
  1. Forum thread detail: needs better OP/reply separation + spacing
  2. TOC sidebar: needs floating panel treatment with active indicator
  3. Forum thread list: reply snippet needs cleaner quote style
- No real bugs found — site is stable, focused on mandatory styling + features

Styling improvements:
- TOC sidebar redesigned as a floating panel:
  - Light background (bg-secondary/40), rounded corners (rounded-2xl), subtle shadow
    (shadow-premium-xs), generous padding (p-5)
  - Active indicator: a vertical accent bar (h-5 w-0.5 bg-accent) that appears on the
    left of the active heading, mimicking Apple's documentation sidebar style
  - Active heading: font-medium text-accent
  - Inactive headings: text-muted-foreground with hover:bg-secondary hover:text-foreground
  - Generous line-height (leading-7) for premium readability
  - Removed the old border-l hairline list style; replaced with rounded-md hover states
  - VLM confirms: "premium floating panel... visually distinct from the article body"
- Forum thread detail OP/reply separation:
  - Increased spacing between OP and replies section (mt-10 → mt-16)
  - Added a divider line after the "Replies" header (flex-1 bg-border)
  - VLM confirms: "clear visual separation... divider line after the Replies header"
- Forum thread list reply snippet refined:
  - Replaced the "replied:" prefix with a cleaner left teal border (border-l-2 border-accent/60)
  - Reduced background tint (bg-secondary/40, was bg-secondary/60)
  - Smaller avatar (h-5 w-5, was h-6 w-6)
  - Replier name shown inline before the text: "NiN.X: This is a test reply..."
  - Single-line truncated quote style instead of two-line "name replied:" + body

Verification:
- agent-browser testing:
  - Article TOC: floating panel classes confirmed (rounded-2xl, border, bg-secondary/40,
    p-5, shadow-premium-xs). Active bar appears when a heading is in view.
  - Forum thread detail: "Original post" badge + "REPLIES (1)" with divider line
  - Forum list: reply snippet shows "NiN.X: This is a test reply..." in left-border style
  - All routes return 200
- VLM confirms:
  - TOC: "premium floating panel... visually distinct from the article body"
  - Forum thread: "clear visual separation... divider line after the Replies header"
- ESLint: 0 errors, 0 warnings
- All services healthy

Stage Summary:
- 3 styling improvements: TOC floating panel with active indicator, forum thread OP/reply
  separation + divider, forum reply snippet cleaner quote style
- All features tested end-to-end
- ESLint clean

Unresolved / Next-phase items:
- Real auth (Google/Apple OAuth + email/password) — UI ready, needs credentials
- GitHub showcase needs GITHUB_USERNAME env var set (totalStars will be 0 until then)
- Shop needs product seeding + payment integration (later phase)
- Article authoring UI (currently articles created via API only)
- Newsletter backend (Mailchimp/Buttondown integration)
- Shop waitlist backend (email provider integration)
- Forum: search within forum, "most viewed threads" section
- Article: cover images, reading time in popular articles cards
- Consider adding a "featured article" hero on the home page
- Consider adding forum thread view counts to the stats banner
