Here is the comprehensive **Product Scope Document (PSD)** for your AI-Powered Resume Generator. This document is designed to serve as the "source of truth" for developers, designers, and stakeholders.

---

# Product Scope Document: AI Resume Generator (SaaS)

**Version:** 1.0
**Date:** January 8, 2026
**Target Platform:** Web (Responsive)

## 1. Executive Summary

Development of a production-grade, SaaS-based Resume Generator that utilizes Generative AI to assist users in crafting professional, ATS (Applicant Tracking System) compliant resumes. The system will feature real-time PDF previews, AI-driven content enhancement (text polishing, job description matching), and a subscription-based monetization model.

## 2. Project Goals

1. **ATS Compliance:** Ensure generated PDFs are 100% parsable by standard ATS software (e.g., Greenhouse, Lever) using text-layer PDFs, not images.
2. **AI Utility:** Move beyond generic text generation; provide specific, quantifiable improvements using the "XYZ Formula" (Accomplished [X] as measured by [Y], by doing [Z]).
3. **Performance:** Achieve a Lighthouse Performance score of 90+ and PDF generation time under 2 seconds.

## 3. User Personas

- **The Job Seeker (Free):** Wants a quick, clean resume. Limited templates, manual editing, 1 free download.
- **The Power User (Pro):** Wants AI-optimized content, multiple versions for different job applications, cover letter generation, and unlimited downloads.
- **The Admin:** Manages user data, subscriptions, and template configurations.

---

## 4. Functional Requirements

### 4.1. Authentication & User Management

- **Sign Up/Login:** Email/Password and Social Login (Google/GitHub) via **Supabase Auth**.
- **Onboarding:** A 3-step wizard to collect basic info (Current Role, Experience Level) to seed the AI context.
- **Dashboard:** View, duplicate, rename, and delete resumes.

### 4.2. The Resume Editor (Core Feature)

- **Split-Screen Interface:** Left side for data entry (forms), Right side for **Real-Time PDF Preview**.
- **Form Sections:**
- Header (Contact Info).
- Professional Summary.
- Experience (Work History).
- Education.
- Skills (Categorized: Tech, Soft, Languages).
- Projects.
- Custom Sections (e.g., Certifications, Awards).

- **Drag-and-Drop:** Ability to reorder sections and items within sections.

### 4.3. AI Modules (The "Brain")

- **Smart Bullet Point Generator:**
- Input: "I did sales for the company."
- AI Output: "Drove 20% revenue growth by executing targeted sales strategies across key regional markets."

- **Job Description (JD) Matcher:**
- Input: Paste a Job Description text.
- Output: A "Match Score" (0-100%) and a list of missing keywords found in the JD but missing from the resume.

- **Summary Writer:** Generates a 3-sentence professional summary based on the user's added experience and target job title.
- **Grammar & Tone Check:** One-click polish to ensure professional tone.

### 4.4. Template Engine & Export

- **ATS Templates:** Three initial templates (Modern, Minimalist, Executive) built strictly with `@react-pdf/renderer`.
- **Customization:**
- Accent Color Picker.
- Font Selection (Limited to ATS-safe fonts: Roboto, Open Sans, Lato, Merriweather).

- **Export:** Download as PDF. (Future: Export to DOCX).

### 4.5. Monetization

- **Gateway:** Integration with **Razorpay**.
- **Plans:**
- **Free:** 1 Resume, Watermark, No AI.
- **Pro:** Unlimited Resumes, No Watermark, Full AI Access.

---

## 5. Technical Architecture (Non-Functional Requirements)

### 5.1. Tech Stack

- **Frontend Framework:** Next.js 14 (App Router).
- **Language:** TypeScript (Strict mode).
- **Styling:** Tailwind CSS + Shadcn/UI + Magic UI (for accessibility, speed, and visual effects).
- **State Management:** MobX (react-observed based state management for complex JSON resumes).
- **Database:** Supabase (PostgreSQL).
- **PDF Engine:** `@react-pdf/renderer` (Client-side generation for speed and privacy).
- **AI Provider:** **Google Gemini 1.5 Pro** (via Vercel AI SDK).

### 5.2. Performance & Security

- **Rate Limiting:** Implement Upstash/Redis ratelimiting on AI API routes to prevent abuse.
- **Data Validation:** Use `Zod` schemas for all form inputs to ensure data integrity before saving to DB.
- **Edge Caching:** Cache static assets (fonts, icons) aggressively.

---

## 6. Database Schema (High-Level Entities)

1. **`users`**: Extends Supabase auth (subscription_status, credits).
2. **`resumes`**: The parent container (id, user_id, template_id, title).
3. **`resume_sections`**: (id, resume_id, type [experience/education], content JSONB).

- _Note: Using JSONB for section content is often more flexible for resumes than strict relational tables for every bullet point, allowing for faster schema iteration._

4. **`subscriptions`**: (user_id, stripe_customer_id, plan_type, current_period_end).

---

## 7. Project Phases & Timeline (Estimated)

### Phase 1: MVP Core (Weeks 1-3)

- Setup Next.js & Supabase.
- Build the Resume Builder Form (No AI).
- Implement Real-time PDF Preview with `@react-pdf/renderer`.
- Create 1 standard ATS template.
- Auth implementation.

### Phase 2: AI Integration (Weeks 4-5)

- Connect Vercel AI SDK.
- Build the "Bullet Point Enhancer" feature.
- Build the "Summary Generator."
- Implement Rate Limiting.

### Phase 3: Polish & Payments (Weeks 6-7)

- Integrate Razorpay/Stripe.
- Build the "Job Description Matcher."
- Add 2 more templates.
- Mobile responsiveness check.

### Phase 4: Launch & Marketing (Week 8)

- SEO implementation (Meta tags, Sitemap).
- Landing page creation.
- Deployment to Vercel.

---

## 8. Exclusions (Out of Scope for MVP)

- **Cover Letter Generator:** To be added in V2.
- **LinkedIn PDF Import:** Parsing existing PDFs is highly error-prone and resource-intensive. V1 will require manual data entry or simple JSON import.
- **Recruiter Portal:** No interface for employers to search resumes in V1.

### Next Step

Would you like me to create the **Database Schema (SQL/DBML)** or the **Folder Structure** for the Next.js project to get you started?
