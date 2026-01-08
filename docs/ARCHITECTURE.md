# Architecture & System Design

**Project:** AI Resume Generator
**Version:** 1.0

---

## 1. High-Level Architecture

The application follows a modern **Serverless** architecture utilizing the **T3 Stack** philosophy (Next.js, TypeScript, Tailwind) + Supabase.

```mermaid
graph TD
    User[Clients (Browser)]
    CDN[Edge Network (Vercel)]

    subgraph "Next.js App (Vercel)"
        UI[React UI Components]
        Store[MobX State (Resume Data)]
        PDF[React-PDF Engine]
        API[Server Actions / API Routes]
    end

    subgraph "External Services"
        Auth[Supabase Auth]
        DB[(Supabase Postgres)]
        AI[Google Gemini 1.5 Pro]
        Pay[Razorpay Gateway]
    end

    User --> CDN
    CDN --> UI
    UI <--> Store
    UI -- "Live Render" --> PDF
    UI -- "Save/Load" --> API
    API -- "CRUD" --> DB
    API -- "Generate" --> AI
    UI -- "Checkout" --> Pay
    Pay -- "Webhook" --> API
```

---

## 2. Core Components

### A. Frontend (Client-Side)

- **State Management (MobX):**
  - The "Single Source of Truth" for the Resume editor is a large JSON object managed by a MobX Store class.
  - This avoids prop-drilling and enables fine-grained reactivity for the live preview.
  - _Why?_ Resume data is deeply nested. MobX observers automatically track dependencies and only re-render components when specific data they touch changes, often outperforming manual selector optimization.
- **PDF Generation (`@react-pdf/renderer`):**
  - **Client-Side Rendering:** The PDF is generated directly in the browser's main thread (or a worker).
  - _Benefit:_ Privacy (data doesn't leave the browser for preview), Speed (no network latency), Cost (no server CPU usage for rendering).

### B. Backend (Server Actions)

Next.js Server Actions will handle all sensitive logic.

- **`saveResume(json)`:** Authenticated write to Supabase `resumes` table.
- **`generateExistingBullets(text)`:**
  1.  Check User Credits (VS Supabase `user_credits`).
  2.  If > 0, call Google Gemini API with system prompt.
  3.  deduct credit.
  4.  Return result.
- **`createSubscriptionOrder()`:** Communicates with Razorpay to initialize a payment intent.

### C. Database (Supabase)

- **PostgreSQL:** Relational integrity for Users/Subscriptions.
- **JSONB:** The `content` column in `resumes` table stores the entire unstructured resume data.
- **Row Level Security (RLS):** Policies enforced at the database level ensure a user with ID `X` can never read user `Y`'s resume, even if the API layer fails.

---

## 3. Data Flow

### 3.1. The AI Generation Flow

1.  **User** clicks "Enhance" on a bullet point.
2.  **Client** calls Server Action `enhanceText(currentText)`.
3.  **Server** verifies `auth.uid()`.
4.  **Server** queries `user_credits` table.
    - _If 0:_ Throw Error "Upgrade Plan".
    - _If > 0:_ Continue.
5.  **Server** calls **Google Gemini API**.
6.  **Server** performs Transaction:
    - Decrement `credits_used` in `user_credits`.
    - Log entry in `ai_logs`.
7.  **Server** returns generated text text to Client.
8.  **Client** updates MobX Store.

### 3.2. Payment Flow (Razorpay)

1.  **User** clicks "Upgrade to Pro".
2.  **Client** SDK opens Razorpay Modal.
3.  **User** completes payment.
4.  **Razorpay** sends `subscription.charged` webhook to `/api/webhooks/razorpay`.
5.  **Next.js API Route**:
    - Verifies webhook signature (security).
    - Extracts `payload.payment.entity.notes.user_id`.
    - Updates `subscriptions` table status to `active`.
    - Updates `user_credits.is_premium` to `true`.

---

## 4. Security Considerations

- **Authentication:** Managed entirely by generic Supabase Auth (JWTs).
- **Input Validation:** All AI inputs are validated with `Zod` to prevent large payloads (Token prevention) or prompt injection attacks (sanitization).
- **API Keys:**
  - `GEMINI_API_KEY`: Stored in Vercel Environment Variables (Server-side only).
  - `RAZORPAY_SECRET`: Stored in Vercel Environment Variables (Server-side only).
- **Ratelimiting:**
  - AI Endpoints: Rate limited by IP or User ID using `@upstash/ratelimit` to prevent API billing spikes.

## 5. Deployment Strategy

- **Platform:** Vercel (ideal for Next.js).
- **Environment Variables:**
  - `NEXT_PUBLIC_SUPABASE_URL`
  - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
  - `Are_You_Sure_Need_Service_Role?` -> Only if bypassing RLS is needed (Admin functions).
  - `GOOGLE_GENERATIVE_AI_API_KEY`
  - `RAZORPAY_KEY_ID`
  - `RAZORPAY_KEY_SECRET`
  - `RAZORPAY_WEBHOOK_SECRET`
