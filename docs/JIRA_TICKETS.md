# Jira Tickets / Backlog

**Project Name:** AI Resume Generator
**Project Key:** RES

---

## Epic 1: Project Initialization & Infrastructure

**Goal:** Set up the Next.js environment, database, and CI/CD.

### RES-1: Initialize Next.js 14 Project

**Type:** Task
**Points:** 1
**Description:**
Initialize a new Next.js 14 project using TypeScript. Configure Tailwind CSS and Shadcn/UI.
**Acceptance Criteria:**

- [ ] `npx create-next-app@latest` ran successfully with TS, Tailwind, App Router.
- [ ] Shadcn/UI initialized (`npx shadcn-ui@latest init`).
- [ ] Folder structure established (`components`, `lib`, `app`, `types`).
- [ ] Repository pushed to GitHub.

### RES-2: Supabase Project Setup

**Type:** Task
**Points:** 2
**Description:**
Create a new Supabase project. Run the initial SQL schema from `docs/DB.md` to set up tables and RLS.
**Acceptance Criteria:**

- [ ] Supabase project created.
- [ ] Tables: `profiles`, `resumes`, `user_credits`, `subscriptions`, `ai_logs` exist.
- [ ] RLS policies enabled and verified.
- [ ] Triggers for user creation functionality verified.
- [ ] Environment variables (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`) added to `.env.local`.

### RES-3: Linting & Precommit Hooks

**Type:** Task
**Points:** 1
**Description:**
Set up ESLint, Prettier, and Husky (optional) to ensure code quality.
**Acceptance Criteria:**

- [ ] `npm run lint` passes with strict rules.
- [ ] Formatting is consistent on save.

---

## Epic 2: Authentication & User Management

**Goal:** Allow users to sign up, log in, and manage their profiles.

### RES-4: Implement Auth UI (Sign Up / Login)

**Type:** Story
**Points:** 3
**Description:**
Create a login page using Supabase Auth helpers. Support Email/Password and Google OAuth.
**Acceptance Criteria:**

- [ ] `/login` route exists.
- [ ] User can sign up with email/password.
- [ ] User can sign in with Google.
- [ ] Successful login redirects to `/dashboard`.
- [ ] Protected routes middleware implemented.

### RES-5: User Dashboard (Resume List)

**Type:** Story
**Points:** 3
**Description:**
Create a dashboard where users can see their list of resumes.
**Acceptance Criteria:**

- [ ] Fetch resumes from `resumes` table filtering by `auth.uid()`.
- [ ] Display resumes in a grid with thumbnails (or placeholders).
- [ ] "Create New Resume" button creates a new row in DB and redirects to editor.
- [ ] Delete resume function working (with confirmation modal).

---

## Epic 3: The Resume Builder (Core)

**Goal:** Data entry and Real-time data management.

### RES-6: MobX Store Implementation

**Type:** Task
**Points:** 5
**Description:**
Create a global MobX RootStore to manage the Resume JSON object. This is critical for performance and fine-grained reactivity.
**Acceptance Criteria:**

- [ ] `ResumeStore` class created.
- [ ] `MobXProvider` or Context setup for accessing stores.
- [ ] Actions: `setResume`, `updateExperience`, `addEducation`, `removeSkill`, etc.
- [ ] TypeScript interfaces for the Resume JSON structure strictly defined.

### RES-7: Interactive Resume Form - Personal & Summary

**Type:** Story
**Points:** 3
**Description:**
Build the form sections for "Personal Info" and "Professional Summary" that update the store.
**Acceptance Criteria:**

- [ ] Input fields for Name, Email, Phone, LinkedIn, etc.
- [ ] Textarea for Summary.
- [ ] Updates reflect in the Store immediately.
- [ ] Debounced auto-save to Supabase (optional for this ticket, but planned).

### RES-8: Interactive Resume Form - Experience & Education

**Type:** Story
**Points:** 5
**Description:**
Build the form sections for Experience and Education, which are arrays.
**Acceptance Criteria:**

- [ ] User can add multiple Experience entries.
- [ ] User can drag-and-drop to reorder (dnd-kit).
- [ ] User can add bullet points for each job.
- [ ] Education section functions similarly.

---

## Epic 4: PDF Generation

**Goal:** Real-time preview and export.

### RES-9: React-PDF Template (Modern)

**Type:** Story
**Points:** 8
**Description:**
Create the initial "Modern" template using `@react-pdf/renderer`.
**Acceptance Criteria:**

- [ ] Template accepts the standard Resume JSON object as props.
- [ ] Renders correctly in the `PDFViewer` component.
- [ ] Layout matches the design mockups.
- [ ] Fonts are embedded and ATS-safe.

### RES-10: Live Preview Split Screen

**Type:** Story
**Points:** 3
**Description:**
Integrated the Form (Left) and PDF Preview (Right).
**Acceptance Criteria:**

- [ ] Layout is responsive (stacks on mobile, side-by-side on desktop).
- [ ] Typing in the form updates the PDF preview within <500ms (or reasonably fast).

---

## Epic 5: AI Integration (Gemini)

**Goal:** Smart content generation.

### RES-11: Gemini API Server Action

**Type:** Task
**Points:** 3
**Description:**
Create a robust Server Action to call Google Gemini API via Vercel AI SDK.
**Acceptance Criteria:**

- [ ] Google API Key configured securely.
- [ ] Function `generateText(prompt)` works.
- [ ] Error handling for API limits.

### RES-12: Feature - Smart Bullet Points

**Type:** Story
**Points:** 5
**Description:**
"Improve this bullet" button.
**Acceptance Criteria:**

- [ ] User clicks "Magic Wand" icon next to a bullet point.
- [ ] Request prompt: "Rewrite this resume bullet to be more professional and result-oriented: '${input}'".
- [ ] The received suggestion replaces or offers a replacement for the input.
- [ ] Decrement user credits in DB.

### RES-13: Feature - Summary Generator

**Type:** Story
**Points:** 5
**Description:**
"Generate Summary" button based on entered experience.
**Acceptance Criteria:**

- [ ] Concatenates job titles and skills from the store.
- [ ] Sends prompt to Gemini to write a 3-sentence summary.
- [ ] Streams response to the UI.

---

## Epic 6: Payments (Razorpay)

**Goal:** Monetization.

### RES-14: Razorpay Integration Setup

**Type:** Task
**Points:** 3
**Description:**
Create a Razorpay account and configure API keys. Create a "Pro" plan in Razorpay dashboard.
**Acceptance Criteria:**

- [ ] `NEXT_PUBLIC_RAZORPAY_KEY_ID` configured.
- [ ] Plan created in Razorpay.

### RES-15: Subscription Checkout Flow

**Type:** Story
**Points:** 8
**Description:**
Implement the subscription flow.
**Acceptance Criteria:**

- [ ] Pricing page displays Free vs Pro.
- [ ] "Upgrade" button opens Razorpay Checkout modal.
- [ ] Webhook handler created to receive `subscription.activated` event.
- [ ] Webhook updates `subscriptions` table and sets `user_credits` to premium.

---

## Epic 7: Launch Polish

**Goal:** Non-functional requirements.

### RES-16: Rate Limiting & Security

**Type:** Task
**Points:** 3
**Description:**
Protect AI routes.
**Acceptance Criteria:**

- [ ] Implement Upstash Rate Limiting on the API route (e.g., 5 requests per minute per user).

### RES-17: SEO & Meta Tags

**Type:** Task
**Points:** 2
**Description:**
Add metadata for SEO.
**Acceptance Criteria:**

- [ ] Title and Description dynamic based on pages.
- [ ] Open Graph images added.
