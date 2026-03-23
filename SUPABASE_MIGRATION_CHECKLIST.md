# Supabase Migration Implementation Checklist

## 🎯 Overall Status: Phase 1 Complete ✓

**What's Done:**
- ✅ Supabase client library created (`src/lib/supabase.ts`)
- ✅ Environment variables template updated (`.env.local`)
- ✅ Comprehensive migration guide created (`SUPABASE_MIGRATION_GUIDE.md`)
- ✅ Complete SQL schema designed with RLS policies

**What's Next:** Follow this checklist to complete the migration.

---

## 📋 Pre-Implementation Requirements

### ☐ Core Setup (Do First)
- [ ] Install Supabase package: `npm install @supabase/supabase-js`
- [ ] Create Supabase project (follow guide Step 1)
- [ ] Enable phone authentication (follow guide Step 2)
- [ ] Create database schema (follow guide Step 3)
- [ ] Get and save API keys (follow guide Step 4)
- [ ] Update `.env.local` with Supabase credentials

**Timeline:** ~15-20 minutes

---

## 🔄 Migration Phase: Update Application Code

### Phase 1: Core App Component (src/App.tsx)

#### ☐ Step 1.1: Update Imports
```typescript
// REMOVE:
// import { sendPhoneOTP, initRecaptcha, auth, db } from './lib/firebase';

// ADD:
import { 
  supabase, 
  signInWithPhone, 
  verifyPhoneOTP, 
  createOrUpdateUser,
  onAuthStateChange 
} from './lib/supabase';
```

#### ☐ Step 1.2: Update Auth State Effect (useEffect)
- Replace `auth.onAuthStateChanged` with `supabase.auth.onAuthStateChange`
- Update user object from Firebase format to Supabase format
- Call `createOrUpdateUser` after successful auth

#### ☐ Step 1.3: Update handlePhoneSubmit Function
- Replace `sendPhoneOTP` call with `signInWithPhone`
- Remove `setConfirmationResult` (not needed in Supabase)
- Keep error handling

#### ☐ Step 1.4: Update handleOTPVerify Function
- Replace `confirmationResult.confirm()` with `verifyPhoneOTP()`
- Use `userPhoneNumber` state for verification
- Update error handling

**Timeline:** ~10 minutes

---

### Phase 2: Profile Modal Component (src/components/ProfileModal.tsx)

#### ☐ Step 2.1: Update Imports
```typescript
// REMOVE:
// import { updateDoc, doc, db } from '../lib/firebase';

// ADD:
import { createOrUpdateUser } from '../lib/supabase';
```

#### ☐ Step 2.2: Update Profile Save Function
- Replace `updateDoc(doc(db, 'users', uid), data)` with `createOrUpdateUser(uid, data)`
- Update field names from Firebase camelCase to Supabase snake_case:
  - `displayName` → `display_name`
  - `photoURL` → `photo_url`

#### ☐ Step 2.3: Update Profile Load Function (if exists)
- Replace Firestore query with `getUser()` from supabase.ts
- Update field mapping (camelCase → snake_case)

**Timeline:** ~5 minutes

---

### Phase 3: OTP Modal Component (src/components/OTPVerificationModal.tsx)

#### ☐ Step 3.1: Check Imports
- Should already use `handlePhoneSubmit` and `handleOTPVerify` props from App.tsx
- No direct Firebase imports needed

#### ☐ Step 3.2: Test OTP Workflow
- Phone input → sends OTP via `signInWithPhone`
- OTP input → verifies via `verifyPhoneOTP`
- Success → closes modal

**Timeline:** ~3 minutes (just verification)

---

### Phase 4: Chat Components (if using chats)

#### ☐ Step 4.1: Update Chat Imports
```typescript
// ADD to any chat components:
import { 
  getUserChats, 
  createChat, 
  subscribeToMessages, 
  sendMessage 
} from '../lib/supabase';
```

#### ☐ Step 4.2: Update Chat List Loading
- Replace Firestore `onSnapshot` with `getUserChats()` + `subscribeToChats()`
- Update field names

#### ☐ Step 4.3: Update Message Sending
- Replace Firebase `addDoc` with `sendMessage()`
- Update field mapping

#### ☐ Step 4.4: Update Message Real-time Listener
- Replace Firestore snapshot with `subscribeToMessages()`
- Connect to message stream

**Timeline:** ~15 minutes (if chat features implemented)

---

### Phase 5: Call Components (if using calls)

#### ☐ Step 5.1: Update Call Imports
```typescript
import { 
  createCall, 
  updateCallStatus, 
  addCallerCandidate, 
  addReceiverCandidate,
  subscribeToIncomingCalls,
  subscribeToCallUpdates 
} from '../lib/supabase';
```

#### ☐ Step 5.2: Update Call Initiation
- Replace Firebase `addDoc` with `createCall()`
- Store offer/answer as JSONB

#### ☐ Step 5.3: Update ICE Candidate Storage
- Replace Firebase `addDoc` with `addCallerCandidate()` / `addReceiverCandidate()`

#### ☐ Step 5.4: Update Call Status Updates
- Replace Firebase `updateDoc` with `updateCallStatus()`

#### ☐ Step 5.5: Update Real-time Listeners
- Replace Firestore snapshots with `subscribeToIncomingCalls()` and `subscribeToCallUpdates()`

**Timeline:** ~20 minutes (if call features implemented)

---

## 🧪 Testing Phase

### Testing Sequence

#### ☐ Test 1: Basic Setup
```bash
npm run dev
```
- App loads without errors
- No TypeScript compilation errors
- Environment variables loaded correctly (check browser console)

**Status:** ✓ Pass / ✗ Fail: _______

#### ☐ Test 2: Phone Authentication
- Enter test phone: `+919876543210`
- OTP sent message appears
- Check Supabase dashboard **Authentication > Users** for new user
- Enter OTP (check Supabase SQL Editor or logs for OTP)
- Login successful, user appears in app

**Status:** ✓ Pass / ✗ Fail: _______

#### ☐ Test 3: Database User Creation
- After login, check Supabase **Table Editor > users**
- Verify new user record created with:
  - `uid` = user's UUID
  - `email` populated (if available)
  - `phone_number` = entered phone

**Status:** ✓ Pass / ✗ Fail: _______

#### ☐ Test 4: Profile Management (if implemented)
- Navigate to profile page
- Edit name/status/photo
- Save changes
- Refresh page
- Verify changes persisted in Supabase

**Status:** ✓ Pass / ✗ Fail: _______

#### ☐ Test 5: Chat Management (if implemented)
- Create a new chat with another user
- Send a message
- Verify in `chats`, `messages`, `chat_participants` tables
- Check message appears in real-time in recipient's window

**Status:** ✓ Pass / ✗ Fail: _______

#### ☐ Test 6: Call Initiation (if implemented)
- Initiate audio/video call
- Check `calls` table for call record
- Check ICE candidates stored in `call_caller_candidates`
- Accept call (if receiver available)

**Status:** ✓ Pass / ✗ Fail: _______

#### ☐ Test 7: Row Level Security
- Login as User A, verify they see only their data
- In private browser window, login as User B
- Verify User B can't see User A's messages/chats (RLS working)

**Status:** ✓ Pass / ✗ Fail: _______

#### ☐ Test 8: Production Build
```bash
npm run build
```
- Build completes successfully
- No errors or warnings
- Output size reasonable (< 500 KB gzipped)

**Status:** ✓ Pass / ✗ Fail: _______

---

## 🚀 Deployment Phase

### ☐ Pre-Deployment Checklist
- [ ] All tests passing (✓ on all 8 above)
- [ ] TypeScript: `npm run lint` returns 0 errors
- [ ] Build: `npm run build` completes successfully
- [ ] No console errors/warnings in production build

### ☐ Deploy to Production
Choose platform (one of):

**Option A: Vercel (Recommended)**
```bash
npm install -g vercel
vercel
```
- Add env variables to Vercel dashboard
- Deploy and verify

**Option B: Netlify**
- Connect Git repository
- Add env variables in Netlify dashboard
- Deploy automatically on commit

**Option C: Firebase Hosting** (still compatible)
```bash
npm run build
firebase deploy
```

**Option D: Custom Server**
- Build: `npm run build`
- Upload `dist/` folder to server
- Set env variables on server

### ☐ Post-Deployment Verification
- [ ] Production URL loads without errors
- [ ] Phone auth works on production
- [ ] Database operations work
- [ ] Real-time features work (if applicable)
- [ ] Performance acceptable (> 2s load time)

---

## ⏱️ Estimated Timeline

| Phase | Time | Status |
|-------|------|--------|
| Pre-Implementation (Supabase setup) | 15-20 min | ☐ |
| Phase 1: App.tsx | 10 min | ☐ |
| Phase 2: ProfileModal.tsx | 5 min | ☐ |
| Phase 3: OTPModal.tsx | 3 min | ☐ |
| Phase 4: Chat Components | 15 min | ☐ |
| Phase 5: Call Components | 20 min | ☐ |
| Testing Phase | 30-45 min | ☐ |
| Deployment | 10 min | ☐ |
| **TOTAL** | **2-3 hours** | |

---

## 🐛 Common Issues & Solutions

### Issue: "Failed to send OTP"
**Checklist:**
- [ ] Phone provider enabled in Supabase dashboard
- [ ] API keys correct in `.env.local`
- [ ] Phone number includes country code (`+91...`)
- [ ] Network connection working

**Fix:** See SUPABASE_MIGRATION_GUIDE.md > Troubleshooting

### Issue: "Cannot find module '@supabase/supabase-js'"
**Solution:** `npm install @supabase/supabase-js`

### Issue: "RLS policy denied access"
**Solution:** Temporarily disable RLS for testing:
```sql
ALTER TABLE public.users DISABLE ROW LEVEL SECURITY;
```
Then re-enable after verifying logic.

### Issue: "Real-time subscriptions not updating"
**Solution:**
- [ ] Verify RLS policies allow reads
- [ ] Check Supabase project status (not paused)
- [ ] Restart dev server

---

## 📞 Support Resources

- **Supabase Docs:** https://supabase.com/docs
- **Official Guides:**
  - [JavaScript Client](https://supabase.com/docs/reference/javascript/introduction)
  - [Phone Auth](https://supabase.com/docs/guides/auth/phone-login)
  - [Real-time](https://supabase.com/docs/guides/realtime)
  - [RLS](https://supabase.com/docs/guides/auth/row-level-security)

---

## ✅ Final Checkmarks

- [ ] Following SUPABASE_MIGRATION_GUIDE.md Steps 1-4
- [ ] All code updates from Phases 1-5 complete
- [ ] All 8 tests passing
- [ ] Production build successful
- [ ] Deployed to production

**Status:** Migration Complete ✓

---

**Last Updated:** Today  
**Migration Version:** 1.0  
**Target:** Production-ready Supabase integration
