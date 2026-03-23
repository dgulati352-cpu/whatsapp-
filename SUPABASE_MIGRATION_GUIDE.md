# Supabase Database Migration Guide

This guide walks through migrating from Firebase Firestore to Supabase PostgreSQL with phone authentication.

---

## 📋 Table of Contents
1. [Create Supabase Project](#1-create-supabase-project)
2. [Set Up Phone Authentication](#2-set-up-phone-authentication)
3. [Create Database Schema](#3-create-database-schema)
4. [Configure API Keys](#4-configure-api-keys)
5. [Update Application Code](#5-update-application-code)
6. [Test Migration](#6-test-migration)
7. [Deploy to Production](#7-deploy-to-production)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Create Supabase Project

### Step 1.1: Sign Up/Login
- Go to [supabase.com](https://supabase.com)
- Click **"Start your project"**
- Sign in with GitHub or email

### Step 1.2: Create New Project
- Click **"+ New project"** in the dashboard
- **Name**: `whatsapp-applet`
- **Password**: Create a strong database password (save this!)
- **Region**: Choose closest to your target users (e.g., `ap-south-1` for India)
- Click **"Create new project"** and wait 2-3 minutes for initialization

### Step 1.3: Note Project Details
Once the project initializes, you'll see:
- **Project URL**: `https://your-project.supabase.co`
- **API Keys** (visible under Settings > API):
  - `public (anon) key` - Used for frontend
  - `service_role key` - Keep secret (backend only)

---

## 2. Set Up Phone Authentication

### Step 2.1: Enable Phone Auth
1. Navigate to **Authentication > Providers** in Supabase dashboard
2. Click **"Phone"** provider
3. Toggle **"Enable Phone provider"** ON
4. Keep default settings (Twilio integration requires paid account)
5. Click **"Save"**

### Step 2.2: Configure SMS Settings (Test Mode)
For development/testing with fake OTP:
1. Go to **Authentication > SMS Templates**
2. The default template is already configured: `{{ token }}`
3. No additional setup needed for test mode

### Step 2.3: SMS Provider Setup (Optional - Production Only)
To send real SMS in production:
1. Go to **Authentication > Providers > Phone**
2. Choose **Twilio** or **Vonage** as SMS provider
3. Enter provider credentials
4. Test with real phone numbers

**For now, we'll start with test mode (fake OTP).**

---

## 3. Create Database Schema

### Step 3.1: Open SQL Editor
1. In Supabase dashboard, click **"SQL Editor"** (left sidebar)
2. Click **"New Query"**
3. Paste the following SQL schema:

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ============================================
-- USERS TABLE
-- ============================================
CREATE TABLE public.users (
  uid UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  display_name TEXT,
  email TEXT UNIQUE,
  phone_number TEXT UNIQUE,
  phone_verified BOOLEAN DEFAULT FALSE,
  verified_phone_number TEXT,
  verified_at TIMESTAMP WITH TIME ZONE,
  photo_url TEXT,
  status TEXT,
  last_seen TIMESTAMP WITH TIME ZONE,
  last_updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS on users
ALTER TABLE public.users ENABLE ROW LEVEL SECURITY;

-- RLS: Users can only view/edit their own data
CREATE POLICY "Users can view own data"
ON public.users
FOR SELECT
USING (auth.uid() = uid);

CREATE POLICY "Users can update own data"
ON public.users
FOR UPDATE
USING (auth.uid() = uid)
WITH CHECK (auth.uid() = uid);

-- Index
CREATE INDEX idx_users_phone ON public.users(phone_number);
CREATE INDEX idx_users_display_name ON public.users USING GIN (to_tsvector('english', display_name));

-- ============================================
-- CHATS TABLE
-- ============================================
CREATE TABLE public.chats (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  chat_type TEXT NOT NULL CHECK (chat_type IN ('individual', 'group')),
  chat_name TEXT,
  created_by UUID NOT NULL REFERENCES public.users(uid) ON DELETE SET NULL,
  last_message TEXT,
  last_message_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS on chats
ALTER TABLE public.chats ENABLE ROW LEVEL SECURITY;

-- RLS: Users can view chats they're participants in
CREATE POLICY "Users can view their chats"
ON public.chats
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM public.chat_participants cp
    WHERE cp.chat_id = chats.id AND cp.user_id = auth.uid()
  )
);

-- Index
CREATE INDEX idx_chats_created_at ON public.chats(created_at DESC);

-- ============================================
-- CHAT PARTICIPANTS TABLE
-- ============================================
CREATE TABLE public.chat_participants (
  chat_id UUID NOT NULL REFERENCES public.chats(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES public.users(uid) ON DELETE CASCADE,
  joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(chat_id, user_id)
);

-- Enable RLS on chat_participants
ALTER TABLE public.chat_participants ENABLE ROW LEVEL SECURITY;

-- RLS: Users can view their own participations
CREATE POLICY "Users can view their chat participations"
ON public.chat_participants
FOR SELECT
USING (auth.uid() = user_id);

-- Index
CREATE INDEX idx_chat_participants_user ON public.chat_participants(user_id);
CREATE INDEX idx_chat_participants_chat ON public.chat_participants(chat_id);

-- ============================================
-- MESSAGES TABLE
-- ============================================
CREATE TABLE public.messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  chat_id UUID NOT NULL REFERENCES public.chats(id) ON DELETE CASCADE,
  sender_id UUID NOT NULL REFERENCES public.users(uid) ON DELETE SET NULL,
  message_text TEXT,
  message_type TEXT DEFAULT 'text',
  media_url TEXT,
  status TEXT DEFAULT 'sent' CHECK (status IN ('sent', 'delivered', 'read')),
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS on messages
ALTER TABLE public.messages ENABLE ROW LEVEL SECURITY;

-- RLS: Users can view messages in their chats
CREATE POLICY "Users can view messages in their chats"
ON public.messages
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM public.chat_participants cp
    WHERE cp.chat_id = messages.chat_id AND cp.user_id = auth.uid()
  )
);

-- RLS: Users can create messages
CREATE POLICY "Users can create messages"
ON public.messages
FOR INSERT
WITH CHECK (auth.uid() = sender_id);

-- Index
CREATE INDEX idx_messages_chat ON public.messages(chat_id);
CREATE INDEX idx_messages_timestamp ON public.messages(timestamp DESC);
CREATE INDEX idx_messages_sender ON public.messages(sender_id);

-- ============================================
-- CALLS TABLE
-- ============================================
CREATE TABLE public.calls (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  caller_id UUID NOT NULL REFERENCES public.users(uid) ON DELETE CASCADE,
  receiver_id UUID NOT NULL REFERENCES public.users(uid) ON DELETE CASCADE,
  call_type TEXT NOT NULL CHECK (call_type IN ('audio', 'video')),
  status TEXT DEFAULT 'ringing' CHECK (status IN ('ringing', 'accepted', 'rejected', 'ended')),
  offer JSONB,
  answer JSONB,
  start_time TIMESTAMP WITH TIME ZONE,
  end_time TIMESTAMP WITH TIME ZONE,
  duration_seconds INT,
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS on calls
ALTER TABLE public.calls ENABLE ROW LEVEL SECURITY;

-- RLS: Users can view calls they're involved in
CREATE POLICY "Users can view their calls"
ON public.calls
FOR SELECT
USING (auth.uid() = caller_id OR auth.uid() = receiver_id);

-- Index
CREATE INDEX idx_calls_receiver ON public.calls(receiver_id);
CREATE INDEX idx_calls_caller ON public.calls(caller_id);
CREATE INDEX idx_calls_status ON public.calls(status);

-- ============================================
-- CALL ICE CANDIDATES - CALLER
-- ============================================
CREATE TABLE public.call_caller_candidates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  call_id UUID NOT NULL REFERENCES public.calls(id) ON DELETE CASCADE,
  candidate JSONB NOT NULL,
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS on call_caller_candidates
ALTER TABLE public.call_caller_candidates ENABLE ROW LEVEL SECURITY;

-- RLS: Users can view candidates for their calls
CREATE POLICY "Users can view call caller candidates"
ON public.call_caller_candidates
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM public.calls c
    WHERE c.id = call_caller_candidates.call_id 
      AND (c.caller_id = auth.uid() OR c.receiver_id = auth.uid())
  )
);

-- Index
CREATE INDEX idx_call_caller_candidates_call ON public.call_caller_candidates(call_id);

-- ============================================
-- CALL ICE CANDIDATES - RECEIVER
-- ============================================
CREATE TABLE public.call_receiver_candidates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  call_id UUID NOT NULL REFERENCES public.calls(id) ON DELETE CASCADE,
  candidate JSONB NOT NULL,
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS on call_receiver_candidates
ALTER TABLE public.call_receiver_candidates ENABLE ROW LEVEL SECURITY;

-- RLS: Users can view candidates for their calls
CREATE POLICY "Users can view call receiver candidates"
ON public.call_receiver_candidates
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM public.calls c
    WHERE c.id = call_receiver_candidates.call_id 
      AND (c.caller_id = auth.uid() OR c.receiver_id = auth.uid())
  )
);

-- Index
CREATE INDEX idx_call_receiver_candidates_call ON public.call_receiver_candidates(call_id);

-- ============================================
-- GRANT PERMISSIONS
-- ============================================
GRANT USAGE ON SCHEMA public TO authenticated, anon;
GRANT ALL ON ALL TABLES IN SCHEMA public TO authenticated;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO authenticated;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO authenticated;

-- Anon (unauthenticated) can only access limited functions
GRANT EXECUTE ON FUNCTION uuid_generate_v4() TO anon;
```

### Step 3.2: Run SQL
1. Click **"Run"** button (or press `Cmd+Enter`)
2. Wait for confirmation: "Success - Tables created"
3. Check **"Table Editor"** to verify all tables are created:
   - `users`
   - `chats`
   - `chat_participants`
   - `messages`
   - `calls`
   - `call_caller_candidates`
   - `call_receiver_candidates`

---

## 4. Configure API Keys

### Step 4.1: Get API Keys
1. Go to **Settings > API** in Supabase dashboard
2. Copy:
   - **Project URL**: `https://your-project.supabase.co`
   - **Public (anon) Key**: Starts with `eyJ...`

### Step 4.2: Update .env.local
Edit `.env.local` in your project root and replace:

```bash
# SUPABASE CONFIGURATION (PRIMARY - IN USE)
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-supabase-anon-key-here
```

**Example:**
```bash
VITE_SUPABASE_URL=https://abcdefghij.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 5. Update Application Code

### Step 5.1: Install Supabase Package
```bash
npm install @supabase/supabase-js
```

### Step 5.2: Update src/App.tsx
Replace Firebase imports with Supabase:

**REPLACE:**
```typescript
import { sendPhoneOTP, initRecaptcha, auth, db } from './lib/firebase';
```

**WITH:**
```typescript
import { 
  supabase, 
  signInWithPhone, 
  verifyPhoneOTP, 
  createOrUpdateUser,
  onAuthStateChange 
} from './lib/supabase';
```

### Step 5.3: Update Auth State Handling
**REPLACE:**
```typescript
useEffect(() => {
  const unsubscribe = auth.onAuthStateChanged(async (user) => {
    if (user) {
      setCurrentUser(user);
    } else {
      setCurrentUser(null);
      setShowOTPModal(true);
    }
  });
  return unsubscribe;
}, []);
```

**WITH:**
```typescript
useEffect(() => {
  const { data: { subscription } } = supabase.auth.onAuthStateChange(async (event, session) => {
    if (session) {
      setCurrentUser(session.user);
      // Store user profile in database
      await createOrUpdateUser(session.user.id, {
        email: session.user.email,
        phone_number: session.user.phone,
      });
    } else {
      setCurrentUser(null);
      setShowOTPModal(true);
    }
  });
  
  return () => subscription?.unsubscribe();
}, []);
```

### Step 5.4: Update Phone OTP Submission
**REPLACE:**
```typescript
const handlePhoneSubmit = async (phoneNumber: string) => {
  try {
    setLoading(true);
    const confirmationResult = await sendPhoneOTP(phoneNumber);
    setConfirmationResult(confirmationResult);
    setUserPhoneNumber(phoneNumber);
  } catch (error: any) {
    alert('Failed: ' + error.message);
  } finally {
    setLoading(false);
  }
};
```

**WITH:**
```typescript
const handlePhoneSubmit = async (phoneNumber: string) => {
  try {
    setLoading(true);
    const result = await signInWithPhone(phoneNumber);
    setUserPhoneNumber(phoneNumber);
    // confirmationResult not needed in Supabase - it's handled by auth.verifyOtp
  } catch (error: any) {
    alert('Failed: ' + error.message);
  } finally {
    setLoading(false);
  }
};
```

### Step 5.5: Update OTP Verification
**REPLACE:**
```typescript
const handleOTPVerify = async (otp: string) => {
  try {
    setLoading(true);
    const result = await confirmationResult?.confirm(otp);
    if (result?.user) {
      setCurrentUser(result.user);
      setShowOTPModal(false);
    }
  } catch (error: any) {
    alert('Verification failed: ' + error.message);
  } finally {
    setLoading(false);
  }
};
```

**WITH:**
```typescript
const handleOTPVerify = async (otp: string) => {
  try {
    setLoading(true);
    const { data, error } = await verifyPhoneOTP(userPhoneNumber, otp);
    if (error) throw error;
    if (data.user) {
      setShowOTPModal(false);
    }
  } catch (error: any) {
    alert('Verification failed: ' + error.message);
  } finally {
    setLoading(false);
  }
};
```

### Step 5.6: Update ProfileModal.tsx
Replace Firebase Firestore calls with Supabase:

**REPLACE:**
```typescript
import { updateDoc, doc, db } from '../lib/firebase';

// In the update function:
await updateDoc(doc(db, 'users', currentUser.uid), {
  displayName: name,
  photoURL: photoBase64,
});
```

**WITH:**
```typescript
import { createOrUpdateUser } from '../lib/supabase';

// In the update function:
await createOrUpdateUser(currentUser.id, {
  display_name: name,
  photo_url: photoBase64,
});
```

---

## 6. Test Migration

### Test 6.1: Test Mode (Development)
1. Start dev server: `npm run dev`
2. Application loads with OTP modal
3. Enter test phone: `+919876543210`
4. Check browser console for OTP (Supabase test mode shows in dashboard)
5. Enter OTP and verify sign-in works

### Test 6.2: Database Operations
1. After successful login, create a chat
2. Send a message
3. Open Supabase **Table Editor**
4. Verify:
   - `users` table has new user record
   - `chats` table has new chat record
   - `messages` table has new message
   - `chat_participants` has participant entry

### Test 6.3: Real-time Updates
1. Open chat in two browser windows
2. Send message from one window
3. Verify message appears in real-time in other window (using subscription)

### Test 6.4: Call Management
1. Initiate a call
2. Check `calls` table for call record
3. Verify ICE candidates stored in `call_caller_candidates`/`call_receiver_candidates`

---

## 7. Deploy to Production

### Step 7.1: Build for Production
```bash
npm run build
```

Verify:
- No TypeScript errors
- All chunks generated
- Output size reasonable

### Step 7.2: Enable Real SMS (Optional)
1. In Supabase **Authentication > Providers > Phone**
2. Add SMS provider (Twilio/Vonage)
3. Users will receive real SMS codes instead of test codes

### Step 7.3: Deploy to Production Host
Choose one:
- **Vercel** (recommended for Next.js/Vite apps)
- **Netlify**
- **Firebase Hosting**
- **Your own server**

**For Vercel:**
```bash
npm install -g vercel
vercel
```

Add environment variables in Vercel dashboard:
```
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_GEMINI_API_KEY=your-gemini-key
```

---

## 8. Troubleshooting

### Issue: "Failed to send OTP"
**Solution:**
- Verify `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` in `.env.local`
- Check phone number format (must include country code, e.g., `+919876543210`)
- Ensure phone provider is enabled in Supabase dashboard
- Check Supabase logs: **Logs > Auth**

### Issue: "User not found"
**Solution:**
- Phone authentication must complete before user profile can be queried
- Ensure `verifyPhoneOTP` is called successfully before accessing `supabase.auth.user()`

### Issue: "Row Level Security denied"
**Solution:**
- Verify RLS policies are correctly created (check SQL in Table Editor > Policies)
- Ensure `auth.uid()` matches the user's UUID in the users table
- For testing, temporarily disable RLS: `ALTER TABLE public.users DISABLE ROW LEVEL SECURITY;`

### Issue: "Real-time subscriptions not working"
**Solution:**
- Supabase requires PostgreSQL >= 11 (should be included by default)
- Check permissions: RLS policies must allow reads
- Verify subscription is called AFTER RLS policies exist

### Issue: "Can't connect to Supabase"
**Solution:**
- Check internet connection
- Verify Supabase project is not paused (check project settings)
- Ensure API key is not expired
- Check CORS settings: **Settings > API Configuration**

---

## 📚 Additional Resources

- [Supabase JavaScript Client Docs](https://supabase.com/docs/reference/javascript/introduction)
- [Supabase Auth Phone Docs](https://supabase.com/docs/guides/auth/phone-login)
- [Supabase Real-time Docs](https://supabase.com/docs/guides/realtime)
- [Row Level Security Guide](https://supabase.com/docs/guides/auth/row-level-security)

---

## ✅ Migration Checklist

- [ ] Supabase project created
- [ ] Phone authentication enabled
- [ ] Database schema created (SQL executed)
- [ ] API keys added to `.env.local`
- [ ] `@supabase/supabase-js` package installed
- [ ] `src/lib/supabase.ts` created
- [ ] Firebase imports replaced with Supabase
- [ ] Auth state handling updated
- [ ] Phone OTP submission updated
- [ ] OTP verification updated
- [ ] ProfileModal updated to use Supabase
- [ ] Dev server tested with phone auth
- [ ] Database operations tested
- [ ] Real-time subscriptions tested
- [ ] Call management tested
- [ ] Production build successful
- [ ] Deployed to production hosting

---

**Status:** ✓ Migration guide complete. Ready for implementation.
