Here is the comprehensive **Database Design Document** for your Resume Generator. This schema is designed for **Supabase (PostgreSQL)**, ensuring strict data integrity while allowing the flexibility needed for document-like data.

### 1. Design Strategy

- **Relational Core:** Users and high-level Resume entities are strictly relational.
- **Hybrid Approach for Content:** While we _could_ use strict tables for every bullet point, a Resume is inherently document-based. We will use a **hybrid approach**:
- **Strict Tables** for structured entities that might be queried/filtered independently (e.g., `resumes`, `subscriptions`).
- **JSONB** for complex, nested resume content (Education, Experience, Skills) to avoid doing 10+ joins just to load one resume. This improves read performance significantly for the live preview.

---

### 2. Entity-Relationship Diagram (ERD) Conceptual View

### 3. Database Schema (SQL)

You can run this directly in the Supabase SQL Editor.

#### A. User & Authentication

Managed largely by Supabase `auth.users`, but we need a public profile table.

```sql
-- Public profiles table that links to auth.users
CREATE TABLE public.profiles (
  id UUID REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Secure the table
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

```

#### B. The Resume Core

This is the master table for resumes.

```sql
CREATE TABLE public.resumes (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE NOT NULL,
  title TEXT NOT NULL DEFAULT 'Untitled Resume',
  template_id TEXT NOT NULL DEFAULT 'modern-1', -- References your frontend template IDs

  -- The Core Content (Using JSONB for flexibility and performance)
  -- Structure: {
  --   personalInfo: { phone, linkedin, website, city },
  --   experience: [ { id, company, role, startDate, endDate, bullets: [] } ],
  --   education: [ ... ],
  --   skills: { technical: [], soft: [] }
  -- }
  content JSONB DEFAULT '{}'::jsonb,

  thumbnail_url TEXT, -- Preview image of the resume
  is_public BOOLEAN DEFAULT FALSE, -- For sharing links
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index for faster lookup by user
CREATE INDEX idx_resumes_user ON public.resumes(user_id);

```

#### C. AI Credit System & Usage

To track usage and prevent abuse of your **Gemini** integration.

```sql
CREATE TABLE public.user_credits (
  user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE PRIMARY KEY,
  credits_total INTEGER DEFAULT 5, -- Free tier starts with 5
  credits_used INTEGER DEFAULT 0,
  is_premium BOOLEAN DEFAULT FALSE,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE public.ai_logs (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES public.profiles(id),
  action_type TEXT NOT NULL, -- e.g., 'GENERATE_SUMMARY', 'IMPROVE_BULLET'
  input_token_count INTEGER,
  output_token_count INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

```

#### D. Subscriptions (Payments)

Design assumes **Razorpay** integration.

```sql
CREATE TABLE public.subscriptions (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES public.profiles(id) NOT NULL,
  provider_subscription_id TEXT UNIQUE NOT NULL, -- e.g., sub_123456 (Razorpay Subscription ID)
  provider_customer_id TEXT, -- e.g., cust_123456 (Razorpay Customer ID)
  status TEXT NOT NULL, -- 'active', 'canceled', 'past_due'
  plan_type TEXT NOT NULL, -- 'monthly-pro', 'yearly-pro'
  current_period_end TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

```

---

### 4. Row Level Security (RLS) Policies

This is critical for security in Supabase. This ensures users can only see _their own_ data.

```sql
-- 1. Profiles: Users can read/update their own profile
CREATE POLICY "Users can view own profile" ON public.profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update own profile" ON public.profiles
  FOR UPDATE USING (auth.uid() = id);

-- 2. Resumes: Users can CRUD their own resumes
CREATE POLICY "Users can view own resumes" ON public.resumes
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can create own resumes" ON public.resumes
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own resumes" ON public.resumes
  FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own resumes" ON public.resumes
  FOR DELETE USING (auth.uid() = user_id);

-- 3. Credits: Users can view their own credits (Read Only for Client)
CREATE POLICY "Users can view own credits" ON public.user_credits
  FOR SELECT USING (auth.uid() = user_id);
-- Note: Updates to credits should happen via Server Actions/Edge Functions (Service Role), not client-side.

```

---

### 5. Why JSONB for Resume Content?

You might wonder why I didn't make an `experience` table and an `education` table.

**The "Strict Table" Approach (Bad for this use case):**
If you have normalized tables, loading _one_ resume requires:

1. Fetch Resume
2. Join Experience table
3. Join Education table
4. Join Skills table
5. Join Projects table
   _Result:_ 5+ database queries or one massive join, which is slow.

**The JSONB Approach (Recommended):**

1. `SELECT * FROM resumes WHERE id = 'xyz'`
   _Result:_ 1 fast query. You get the whole object.

Since frontend state (MobX Store) is usually a JSON object structure anyway, this mapping is 1:1 and extremely fast for saving/loading drafts.

### 6. Database Triggers (Automation)

Automatically create a `profile` and `user_credits` entry when a user signs up.

```sql
-- Function to handle new user signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, email, full_name, avatar_url)
  VALUES (new.id, new.email, new.raw_user_meta_data->>'full_name', new.raw_user_meta_data->>'avatar_url');

  INSERT INTO public.user_credits (user_id)
  VALUES (new.id);

  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE PROCEDURE public.handle_new_user();

```

### Next Step

Would you like the **MobX Store implementation** (TypeScript) to manage this JSON structure on the frontend, or the **Next.js Server Action** to safely handle the AI generation and credit deduction?
