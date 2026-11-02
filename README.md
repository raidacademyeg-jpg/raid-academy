# RAID Academy — Official Website

Bilingual (Arabic / English) website for **RAID Academy — Robotics & Arduino Innovation for Development**.

- **Stack:** Next.js 15 (App Router) · TypeScript · Tailwind CSS v4 · Framer Motion · Lucide · Supabase · Vercel
- **Languages:** Arabic (default, RTL, Cairo) and English (LTR, Montserrat), with localized URLs (`/ar/...`, `/en/...`)
- **Pages:** Home · About · Courses · Course details (×6) · Projects & Gallery · Contact & Registration · 404

---

## 1. Run locally

Requires **Node.js 20.9+**.

```bash
npm install
cp .env.example .env.local     # fill in what you have; empty values are hidden
npm run dev                    # http://localhost:3000  → redirects to /ar
```

Before deploying:

```bash
npm run typecheck
npm run build && npm start
```

> **First-build note:** this project was written in a sandbox without npm access. Every page was
> rendered and tested there in both languages and at mobile and desktop sizes, including the form,
> the filters and the API route, but `npm install` and `next build` haven't been run yet. Run
> `npm run typecheck && npm run build` once, or let Vercel do it, and fix anything it reports
> before going live.

---

## 2. Environment variables

| Variable | Required | Purpose |
|---|---|---|
| `NEXT_PUBLIC_SITE_URL` | Yes (prod) | Canonical URL, e.g. `https://raid-academy.com`. Used by sitemap, Open Graph and hreflang. |
| `SUPABASE_URL` | For registrations | Supabase project URL. **Server-only.** |
| `SUPABASE_ANON_KEY` | For registrations | Supabase anon key. **Server-only.** Row Level Security allows insert only. |
| `NEXT_PUBLIC_WHATSAPP_NUMBER` | Optional | Digits only, international, e.g. `2010XXXXXXXX`. Enables every WhatsApp button. |
| `NEXT_PUBLIC_CONTACT_EMAIL` | Optional | Public email. |
| `NEXT_PUBLIC_INSTAGRAM_URL` / `FACEBOOK` / `TIKTOK` / `YOUTUBE` / `LINKEDIN` | Optional | Social links. Only filled ones are shown. |
| `NEXT_PUBLIC_ADDRESS_AR` / `NEXT_PUBLIC_ADDRESS_EN` | Optional | Address shown on Contact. |
| `NEXT_PUBLIC_MAP_EMBED_URL` / `NEXT_PUBLIC_MAP_LINK_URL` | Optional | Google Maps embed `src` and share link. |

Anything left empty is **hidden** or replaced by a clearly labelled "coming soon" message. No placeholder phone numbers or addresses ever appear.
`NEXT_PUBLIC_*` values are baked in at build time, so **redeploy after changing them**.

---

## 3. Registration backend (Supabase)

Until Supabase is configured, the form still validates input, but submitting it shows
*"Online registration is being set up"* and tells visitors to use the other contact channels. Nothing is faked.

1. Create a project at [supabase.com](https://supabase.com). Pick the closest region, e.g. Frankfurt `eu-central-1`.
2. Open **SQL Editor**, paste in `supabase/schema.sql`, and run it.
3. Go to **Project Settings → API** and copy the **Project URL** and the **anon public** key into `SUPABASE_URL` and `SUPABASE_ANON_KEY`, both locally and in Vercel.
4. Redeploy. Submissions now show up under **Table Editor → registrations**.

**Security model:**
- The website inserts through `/api/register`. That route validates every field with the same rules the browser uses (`src/lib/validation.ts`), normalizes Egyptian phone numbers (including Arabic digits) to `+20…`, rejects spam caught by the honeypot field, and rate-limits each IP.
- RLS lets the `anon` role **INSERT only**. It can never read, edit or delete registrations, so even a leaked anon key exposes no student data.
- Staff access goes through Supabase Auth plus the `staff` table: sign the person up, then `insert into staff (user_id, role) values ('<auth-user-id>', 'owner');`.
- Registrations have a `status` pipeline: `new → contacted → registered → paid / waitlist / closed`.

---

## 4. Deploy to Vercel and connect a domain

1. Push this folder to a GitHub repository.
2. In [vercel.com/new](https://vercel.com/new), import the repo. Vercel detects Next.js; keep the default settings.
3. Under **Settings → Environment Variables**, add the variables from section 2 for Production (and Preview if you want).
4. Click **Deploy**.
5. To add your domain, go to **Settings → Domains**, add `raid-academy.com` (example) and `www.raid-academy.com`, then follow Vercel's DNS instructions at your registrar:
   - Apex domain: `A` record → `76.76.21.21`
   - `www`: `CNAME` → `cname.vercel-dns.com`
   (Use the values Vercel shows you if they differ.)
6. Set `NEXT_PUBLIC_SITE_URL` to the final domain and redeploy.
7. Submit `https://<domain>/sitemap.xml` in Google Search Console.

---

## 5. Editing content (no code knowledge needed for most changes)

| What | Where |
|---|---|
| Courses: status, ages, sessions, duration, price, schedule, topics, projects | `src/content/courses.ts` |
| Gallery items and verified achievements | `src/content/projects.ts` |
| Logo, hero photo, site name | `src/content/site.ts` (`logoSrc`, `heroImage`) |
| All interface text in both languages | `src/i18n/dictionaries/ar.ts` and `en.ts` |
| Brand colours, fonts, spacing tokens | `src/app/globals.css` (`@theme`) |

**Course status:** set `status` to `"available"`, `"coming-soon"` or `"unavailable"`. Every course ships as `"coming-soon"` because current availability hasn't been confirmed.
**Unknown details:** leave `sessions`, `sessionHours`, `durationWeeks`, `price` and `schedule` as `null` and the site shows "To be announced". `internalNotes` holds historical figures and is **never** sent to the browser.
**Gallery:** put photos in `public/gallery/`, then add entries with `placeholder: false`, an accurate `credit` (`students` / `class` / `founder` / `event`), and confirmed permission to publish.
**Logo:** put `raid-logo.svg` in `public/brand/` and set `logoSrc: "/brand/raid-logo.svg"`. Also replace the placeholder `src/app/icon.svg` favicon.

Search the code for `Placeholder` and `PLACEHOLDER` to find every remaining content gap.

---

## 6. Project structure

```
src/
  app/
    [locale]/                 layout (html lang/dir, fonts, nav, footer) + all pages
      page.tsx                Home
      about/  courses/  courses/[slug]/  projects/  contact/
      not-found.tsx  [...rest]/
    api/register/route.ts     registration endpoint
    sitemap.ts  robots.ts  icon.svg  globals.css
  components/
    brand/      Logo, HeroVisual (SVG illustration)
    layout/     Navbar, LanguageSwitcher, Footer, PageHero
    home/       Hero, HomeSections, CtaBanner
    courses/    CourseCard, CourseCatalog (filter)
    projects/   ProjectCard, ProjectGallery (filter)
    contact/    RegistrationForm
    ui/         Button, SectionHeader, StatusBadge, FilterTabs, Reveal, Placeholder, Icons
  content/      courses.ts, course-types.ts, projects.ts, site.ts   ← editable data
  i18n/         config, dictionaries (ar, en), get-dictionary
  lib/          validation, seo (metadata + JSON-LD), supabase/server, cn
  middleware.ts locale redirect + remembers the chosen language in the NEXT_LOCALE cookie
supabase/schema.sql
```

**i18n:** `/` redirects to the saved language (cookie) or Arabic. Each page outputs `hreflang` alternates (`ar`, `en`, `x-default`), a canonical URL, and localized Open Graph tags.
**SEO:** per-page titles and descriptions, `EducationalOrganization` and `Course` JSON-LD (verified fields only), a sitemap with alternates, `robots.txt`, and `public/og-image.png`.
**Accessibility:** skip link, semantic landmarks, one `h1` per page, labelled form fields with `aria-invalid` and `aria-describedby`, keyboard-reachable mobile menu (Escape closes it), visible focus rings, and respect for `prefers-reduced-motion`.

---

## 7. Phase 2: admin dashboard (prepared, not built)

`supabase/schema.sql` already creates `courses`, `gallery_items` (with a `consent_confirmed` flag), a `gallery` storage bucket and `staff`, all with RLS. Recommended path:

1. Add `/admin` routes behind Supabase Auth (`@supabase/ssr`) that only `staff` can access.
2. Build a registrations inbox (list, filter by status, update status and notes, export CSV).
3. Build a course editor that writes to `courses`. Then switch `src/content/courses.ts` to read from Supabase with `revalidate`. The data shape already matches.
4. Build a gallery uploader (Storage bucket `gallery` → `gallery_items`).

---

## 8. Information still needed from RAID

1. Official logo files (SVG or transparent PNG) and the confirmed expansion of the name.
2. Which courses are open now, and for each one: ages, sessions, duration, fees, schedule and what's included.
3. WhatsApp number, email, social links, and address or service area.
4. Real photos of classes, projects and events, with permission to publish.
5. The founder's story and founding year for the About page, plus approved instructor credentials.
6. An Arabic tagline, if one has been chosen.
7. Verified achievements, such as competition results with year and evidence.
