# My Family Clinic Website Makeover - Project Requirements Document

## Executive Summary

This document defines the product scope, experience, architecture, and quality gates for reimagining My Family Clinic's website. It combines a pragmatic, phase-focused approach with a comprehensive long-term vision. The immediate objective (Phase 1) is to deliver a calm, trustworthy, and accessible site that makes core care actions obvious: locate a clinic, understand services, and contact or enquire with minimal friction. This foundation is designed to evolve seamlessly into a full-featured patient-centric digital healthcare ecosystem in future phases.

**Core Journeys (Phase 1):**
- **Locate clinics:** Search by geolocation, postal code, district; filter by services and hours.
- **Explore services:** Clear taxonomy and details; link to clinics offering each service.
- **View doctors:** Filter by specialty, language, and location; easy contact.
- **Healthier SG:** Understand eligibility and steps; find participating clinics.
- **Contact/enquiries:** Simple form with routed handling and confirmation.

**Technology Stack:** Next.js 15 (App Router), shadcn/ui, Tailwind v4, Prisma, Supabase Postgres, NextAuth 5, tRPC 11, TanStack React Query 5.

---

## 1. Design Philosophy & Strategic Direction

### 1.1 Core Design Principles
- **Patient-First Approach**: Every design decision prioritizes patient needs, concerns, and journey clarity.
- **Trust Through Transparency**: Visual design and content that communicate credibility, expertise, and care.
- **Simplicity with Depth**: Intuitive surface navigation with comprehensive information available on demand.
- **Accessibility as Standard**: WCAG 2.2 AA compliance as a baseline, not an afterthought.
- **Mobile-First Experience**: Design primarily for mobile devices with thoughtful desktop adaptation.

### 1.2 Strategic Positioning
This project transforms the website from a static information portal into a dynamic healthcare gateway. Phase 1 focuses on flawless information discovery and trust-building. This foundation will enable future phases that introduce patient self-service, personalized health management, and seamless digital care coordination, positioning My Family Clinic as a technologically advanced yet personally caring neighborhood healthcare provider.

---

## 2. Goals, Non-Goals, and Success Metrics

### 2.1 Goals
- **Findability:** **Locate Clinic** reachable in one click from any page; clear service taxonomy and doctors directory.
- **Accessibility:** WCAG 2.2 AA across core templates; keyboard and screen reader-friendly interactions.
- **Performance:** Core Web Vitals within target thresholds on mobile and desktop; resilient geolocation/map behavior.
- **Trust & Clarity:** Real imagery, concise copy, accepted schemes explained, visible affiliations.
- **Maintainability:** Component-driven UI with design tokens; editorial workflows and content types.

### 2.2 Non-Goals (Phase 1)
- **Online appointment booking:** Placeholder CTAs allowed; no real-time scheduling integration.
- **Payments/e-commerce:** Not in scope.
- **Multilingual content:** Prepared in CMS model but not launched.
- **Patient portal/health records:** Deferred to Phase 2.

### 2.3 Success Metrics
- **Conversion:** **+20%** increase in clicks to Call/Directions/Locate; **+15%** enquiry form completions.
- **Performance:** LCP < 2.5s, CLS < 0.1, INP in "good".
- **Accessibility:** Automated checks pass; manual keyboard and screen reader paths verified.
- **Reliability:** 99.9% monthly uptime for core pages and locator endpoints.

---

## 3. Scope and Functional Requirements

### 3.1 Primary Features

#### 3.1.1 Clinic Locator
- **Inputs:** Geolocation opt-in; postal code; district; filters (services, hours, accessibility).
- **Outputs:** List and map view; distance; open/closed status; actions (Call, Directions, View Details).
- **UX Enhancements:** Skeleton loading states during search; clear "no results" messaging with helpful suggestions; map/list toggle with state persistence in URL.
- **Edge Cases:** Denied geolocation with graceful fallback to search; no results handled with clear guidance; offline mode with minimal fallback functionality.

#### 3.1.2 Services Directory
- **Taxonomy:** GP consults; acute care; chronic care; health screening; vaccination/immunisation; corporate healthcare.
- **Details:** Indications; preparation; duration; eligibility; accepted schemes; FAQs; linked clinics.
- **UX Enhancements:** Progressive disclosure for detailed information; tooltips for technical medical terms; clear visual hierarchy for scannability.

#### 3.1.3 Doctors Directory
- **Filters:** Specialty; clinic location; languages; availability.
- **Profiles:** Credentials; specialties; languages; clinics; bio; actions (Call clinic, Locate).
- **UX Enhancements:** Professional headshots with consistent styling; easy-to-scan profile cards with key information highlighted.

#### 3.1.4 Healthier SG Program
- **Content:** Eligibility; benefits; enrollment steps; participating clinics.
- **Action:** Filtered locator view for participating clinics.
- **UX Enhancements:** Step-by-step visual guide for enrollment; clear callouts for benefits.

#### 3.1.5 Contact and Enquiries
- **Form:** Name; email; phone; topic; message; consent checkbox; spam protection.
- **Routing:** Topic-based routing to clinic or corporate inbox; confirmation and reference ID.
- **UX Enhancements:** Inline validation with helpful error messages; clear success state with confirmation details and next steps; feedback loop via email receipt.

### 3.2 Secondary Features
- **News/updates:** Article list and detail pages with author metadata and publication dates.
- **Accreditation & schemes:** Badges with tooltips; dedicated explainer page.
- **Search:** Site-wide search across services, clinics, doctors, news (Phase 2 optional).

---

## 4. Information Architecture and UX Design

### 4.1 Navigation Model
- **Care:** Services, Doctors, Clinics, Healthier SG.
- **Company:** About, News, Careers.
- **Support:** Contact Us, Enquiries, Locate Us.
- **Persistent CTA:** **Locate Clinic** button in header (primary color); **Contact** as utility link.

### 4.2 Page Templates
- **Homepage:** Hero with promise + twin CTAs; benefits band; quick pathways (locator, services, doctors); Healthier SG highlight.
- **Services overview:** Card grid with taxonomy; filters; schemes badges.
- **Service detail:** Structured sections; CTA cluster (Locate, Call, Contact).
- **Clinics list:** Search; filters; list/map toggle.
- **Clinic detail:** Address; hours; services; actions; map.
- **Doctors list & profile:** Filters; profile detail; linked clinics.
- **Healthier SG:** Steps; FAQs; participating clinics link.
- **Contact & Enquiries:** Simple form; privacy note; confirmation.
- **News:** List; article detail with metadata.
- **About/Accreditation:** Trust signals; affiliations; years of service.

### 4.3 User Interaction Principles
- **Micro-interactions:** Subtle animations on buttons, cards, and form elements to provide feedback and delight.
- **Error Prevention:** Design that prevents errors before they happen (e.g., disabling invalid form submissions).
- **Recovery Paths:** Clear ways to recover from errors or dead ends (e.g., 404 page with helpful navigation).
- **Progressive Disclosure:** Show essential information first with options to explore deeper.

---

## 5. Technical Architecture and Stack

### 5.1 Frontend
- **Framework:** Next.js App Router, React 19, server components default; client components only where needed (forms, maps, geolocation).
- **UI:** shadcn/ui with Radix primitives; Tailwind v4; lucide-react icons; class-variance-authority; tailwind-merge.
- **State/data:** TanStack React Query for client-side async state management, caching, and synchronization; tRPC 11 for type-safe API calls with superjson.
- **Auth:** NextAuth 5 with @auth/prisma-adapter (admin-only CMS functions).
- **Maps & geo:** Map provider via iframe or lightweight JS; browser Geolocation API with graceful fallback.

### 5.2 Backend
- **Database:** Supabase Postgres; Prisma as ORM.
- **APIs:** tRPC routers (app/api/trpc) for services, clinics, doctors, enquiries, healthier-sg.
- **Edge/runtime:** Next.js server actions for simple mutations; route handlers for file upload if needed.
- **Caching:** Static generation for read-most pages; ISR for directories; per-query caching via React Query; CDN caching of static assets.

### 5.3 Rationale for Technology Choices
This stack provides an optimal balance of developer experience, performance, and scalability. Next.js 15's App Router enables efficient server-rendering for optimal SEO and performance. tRPC ensures end-to-end type safety, reducing bugs. Supabase offers a robust backend with real-time capabilities, perfect for future Phase 2 features. This architecture is built for the present needs while being primed for future expansion.

---

## 6. Data Model and Schema (Supabase + Prisma)

### 6.1 Entities and Relationships
*(Adopting the user's PRD schema as it is perfectly scoped for Phase 1)*
- **Clinic**: id, name, slug, address, postal_code, district, phone, email, hours_json, services_json, geolocation (point), is_healthier_sg_participant, accessibility_tags[], updated_at.
- **Service**: id, name, slug, category, short_description, content_md, preparation_md, eligibility_md, accepted_schemes[], faq_json, updated_at.
- **Doctor**: id, full_name, slug, credentials, specialties[], languages[], bio_md, photo_url, updated_at.
- **News**: id, title, slug, summary, content_md, published_at, author, tags[], updated_at.
- **Enquiry**: id, topic_enum, name, email, phone, message_md, consent_bool, status_enum, routed_to, created_at.
- **Accreditation & Scheme**: Supporting tables for badges and insurance information.
- **AdminUser**: For CMS access, linked via NextAuth.

### 6.2 Schema Extensibility
The current schema is designed for future expansion. For example, the `Clinic` and `Doctor` models can be easily linked to a future `Appointment` model. The `Enquiry` model provides a foundation for a future patient messaging system. This foresight minimizes disruptive database migrations in Phase 2.

---

## 7. API Surface and Contracts (tRPC)

### 7.1 Routers
*(Adopting the user's PRD's clear API contracts)*
- **clinics**: `list` (with filters), `detail` (by slug/id), `nearby` (by lat/lng), `create/update` (admin).
- **services**: `list` (category filter), `detail` (by slug), `linkClinics`, `create/update` (admin).
- **doctors**: `list` (filters), `detail` (by slug), `create/update` (admin).
- **enquiries**: `create` (public), `list/update` (admin).
- **healthierSg**: `participants`, `faq`.

### 7.2 Error Handling & Client-Side State
- **Validation:** Zod schemas for strict input validation and defensive parsing.
- **Error Handling:** Consistent tRPC error codes with user-friendly messages. TanStack Query will be configured to handle these errors gracefully, displaying toast notifications or inline error messages.
- **Resilience:** TanStack Query's retry and caching mechanisms will be leveraged to create a resilient user experience, especially for the clinic locator and API-driven data.

---

## 8. UI Components and Pages (shadcn/ui)

### 8.1 Components
*(Adopting the user's PRD's component list and enhancing with interaction details)*
- **Header/NavBar:** Sticky; grouped nav; primary CTA button with subtle hover animation.
- **Footer:** Three-column taxonomy; legal links; affiliation badge.
- **Hero:** Headline, subheading, twin CTAs with clear visual hierarchy.
- **Card:** Variants for services, clinics, doctors with consistent hover states.
- **SearchBar:** With icon, debounce (300ms), clear button; accessible labels.
- **FiltersPanel:** Sheet/drawer on mobile with smooth animation; checkboxes, selects (Radix Select); filter state persisted in URL.
- **MapToggle:** List/map switch with ARIA labels and clear visual indication of active view.
- **Form:** Contact/enquiry with real-time field validation; consent checkbox; success toast with clear message.
- **Badge/Chip:** Schemes, specialties, languages with tooltips for more information.
- **Toast/Alert:** Feedback states; error boundaries.
- **Skeleton:** Loading placeholders for locator/results to reduce perceived load time.
- **Tooltip:** For schemes and accreditation badges.
- **Pagination:** Accessible pagination controls at the bottom of list pages.

### 8.2 Pages
*(Page structure from user's PRD is adopted)*
- **/** Homepage
- **/services** Services overview
- **/services/[slug]** Service detail
- **/clinics** Clinics list/map
- **/clinics/[slug]** Clinic detail
- **/doctors** Doctors list
- **/doctors/[slug]** Doctor profile
- **/healthier-sg** Program page
- **/contact** Contact info
- **/enquiries** Enquiry form
- **/news** News list
- **/news/[slug]** News article
- **/admin/** Admin dashboard (role-protected)

---

## 9. Security, Privacy, and Compliance

- **Auth & Roles:** NextAuth with Prisma adapter; role-based authorization for admin/editor actions.
- **PII Handling:** Minimal storage for enquiries; explicit consent required; defined retention policy (e.g., 12 months).
- **Location:** Explicit opt-in for geolocation; clear purpose explained; allow for easy revocation.
- **Data Protection:** Server-side rendering for public data prevents accidental exposure; admin mutations server-only; parameterized queries via Prisma prevent SQL injection.
- **Future-Proofing:** The current auth and data handling model is designed to scale to the more stringent requirements of a full patient portal (e.g., HIPAA-level compliance) in Phase 2.

---

## 10. Performance, SEO, and Analytics

### 10.1 Rendering Strategy
- **Static/ISR:** Homepage, services, doctors, news lists and details.
- **Server-side:** Locator pages where filters depend on request context.
- **Client:** Only for interactive components (forms, map, filters).

### 10.2 Optimization
- Image optimization with Next.js `<Image>` component.
- Prefetch critical routes on hover/touch.
- Code splitting and lazy-loading of non-critical components, especially the map.
- Reserve layout space to prevent Cumulative Layout Shift (CLS).

### 10.3 SEO
- Metadata and OG tags per page.
- Canonical URLs.
- Structured data (Schema.org) for Organization, MedicalClinic, Physician, Service.

### 10.4 Analytics
- **Phase 1 Events:** Track CTA clicks (Locate, Call, Directions), form submissions, filter usage, and page views.
- **Future Framework:** The analytics implementation will be structured to easily accommodate future Phase 2 events like appointment booking starts/completions, patient portal logins, and prescription requests.

---

## 11. Testing, QA, and Acceptance Criteria

### 11.1 Test Plan
- **Unit:** Component behavior, Zod validations, utility functions.
- **Integration:** tRPC endpoints, Prisma queries, auth guards.
- **E2E (Playwright):** Core journeys across mobile/desktop; geolocation deny/allow paths; no-results and error states.
- **Accessibility:** axe-core automated checks; manual keyboard navigation and screen reader audits.
- **Performance:** Lighthouse CI integration; synthetic tests for Core Web Vitals.
- **Component-Specific Testing:** Test micro-interactions, state transitions, and accessibility features of custom components.

### 11.2 Acceptance Criteria
- **Findability:** **Locate Clinic** accessible in ≤1 click from all pages; search and filters work with clear results and edge-case messaging.
- **Accessibility:** All interactive elements have visible focus, labels, and roles; pages pass automated checks and manual spot tests.
- **Usability:** Services/doctor/clinic pages load in under 2.5s LCP; content is scannable; CTAs are consistently placed.
- **Reliability:** Locator works with geolocation allowed and denied; graceful fallbacks; error surfaces are clear.
- **Content Integrity:** Slugs are unique; 404 page for unknown slugs with helpful navigation; metadata is present.

---

## 12. Project Plan, Milestones, and Deliverables

### 12.1 Milestones
1.  **Foundation (Week 1–2):**
    -   Setup: Next.js, Tailwind, shadcn/ui, Prisma, Supabase connection.
    -   Design tokens: Colors, type, spacing, elevation.
    -   Global UI: Header, footer, grid, hero, core cards.
    -   Auth: NextAuth with roles; admin scaffolding.

2.  **Core Directories (Week 3–4):**
    -   Services: Overview and detail pages; CMS CRUD; schemes linking.
    -   Clinics: Locator list/map; geolocation; filters; clinic detail.
    -   Doctors: List and profile pages; filters.

3.  **Support Pages (Week 5):**
    -   Healthier SG: Program page; participants filter.
    -   Contact/Enquiries: Form with routing and confirmation; News module.

4.  **Quality & Launch (Week 6):**
    -   Accessibility and performance: Fixes; analytics events; SEO schema.
    -   Testing: E2E and accessibility passes; Lighthouse targets met.
    -   Docs: Runbooks, editorial guide, admin onboarding.
    -   Release: Preview → Prod with ISR caches warmed.

### 12.2 Deliverables
- Component inventory and catalog (MDX-based).
- CMS/admin guide and content model documentation.
- Ops runbook: Deployment, ENV configuration, backups, migrations, monitoring.
- QA checklist: Accessibility, performance, and E2E test records.

---

## 13. Phase 2 Vision & Future Roadmap

This section outlines the strategic vision for future development, building upon the solid foundation of Phase 1.

### 13.1 Online Appointment Booking System
- **Real-time Availability:** Integration with clinic management systems for live calendar views.
- **Smart Booking Flow:** Guided process for selecting clinic, doctor, service, date, and time.
- **Appointment Management:** Patient portal for viewing, rescheduling, and canceling appointments.
- **Automated Reminders:** SMS and email reminders with pre-visit instructions.

### 13.2 Patient Portal & Health Records
- **Unified Dashboard:** Personalized view of appointments, health records, and prescriptions.
- **Secure Health Records:** Access to test results, visit summaries, and immunization history.
- **Prescription Management:** Request refills and view medication history with instructions.
- **Secure Messaging:** Direct communication with clinic staff for non-urgent queries.

### 13.3 Interactive Health Tools
- **Symptom Checker:** Guided assessment tool with appropriate disclaimers.
- **Cost Estimator:** Tool to estimate costs for common services based on insurance.
- **Preventive Care Reminders:** Personalized recommendations based on age and risk factors.

### 13.4 Corporate Healthcare Portal
- **Company Dashboard:** For corporate clients to manage employee health programs.
- **Employee Wellness:** Track participation in health screenings and wellness programs.
- **Reporting & Analytics:** Anonymized aggregate data on employee health metrics.

---

## 14. Risks and Mitigations

- **Supabase + Prisma alignment:** Mitigation: Use Supabase Postgres connection string with Prisma; avoid Supabase RLS for server-side API; keep public reads server-rendered.
- **Geolocation permissions variability:** Mitigation: Strong fallback search; helpful copy; default to district/postal code.
- **Map provider performance and quotas:** Mitigation: Lazy-load map; static map images for previews; list-first default on mobile.
- **Content accuracy across clinics/services:** Mitigation: Editorial workflows; required fields; validation; scheduled review cadence.
- **Core Web Vitals regressions:** Mitigation: Performance budgets; code splitting; audits in CI.
- **Feature Creep from Phase 2 Vision:** Mitigation: Strict adherence to Phase 1 scope; clear communication of what is deferred; use Phase 2 vision as inspiration, not distraction.

---

## 15. Operational Details & Quality Assurance

### 15.1 Environment Variables
- `DATABASE_URL` (Supabase)
- `NEXTAUTH_URL`, `NEXTAUTH_SECRET`
- `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY` (server-only)
- `NEXT_PUBLIC_ANALYTICS_WRITE_KEY`
- `NEXT_PUBLIC_MAP_API_KEY` (if applicable)

### 15.2 Directory Structure
- `src/app/` (App Router routes)
- `src/components/` (shadcn/ui and app components)
- `src/server/` (tRPC, Prisma, services)
- `src/lib/` (utils, constants, Zod schemas)
- `tests/` (E2E and integration tests)

### 15.3 Final Quality Assurance Checklist
- **Requirements Coverage:** All Phase 1 core journeys implemented; edge cases handled.
- **Security:** Admin-only mutations; auth and roles enforced; consent required for PII.
- **Accessibility:** WCAG 2.2 AA baseline; manual keyboard and screen reader paths verified.
- **Performance:** Lighthouse targets met; lazy-loading implemented; CLS controlled.
- **Documentation:** Runbooks complete; ENV and migration steps documented; rollback strategy defined.
- **Future Readiness:** Code is modular and well-documented to ease the transition to Phase 2 development.

---

https://chat.z.ai/s/7f419ab3-2708-4ac7-83c5-a70db76ee42d
