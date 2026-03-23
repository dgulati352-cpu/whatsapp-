# Firebase Phone Authentication Setup Guide

## ✅ Current Status

The app now includes **TEST MODE** for development. You can test OTP functionality immediately!

### 🧪 Test Mode (Currently Enabled)
- **Status**: Active and working
- **Test Phone Numbers** (use in login):
  - `+919876543210` → OTP: `123456`
  - `+919988776655` → OTP: `654321`
  - `+919123456789` → OTP: `111111`
- **Benefits**: No Firebase Phone Auth setup needed to test
- **Note**: Real SMS will NOT be sent in test mode

---

## 🚀 For Production (Enable Real SMS OTP)

Switch from test mode to production by disabling `TEST_MODE` in `src/lib/firebase.ts`:

```typescript
// src/lib/firebase.ts - Line 8
const TEST_MODE = false; // Change from true to false
```

Then follow these steps:

### Step 1️⃣: Enable Phone Authentication in Firebase Console

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project
3. Navigate to **Authentication** → **Sign-in method**
4. Click **Phone** and toggle it **ON**

✅ **Result**: Phone authentication is enabled

---

### Step 2️⃣: Configure reCAPTCHA

1. In Firebase Console, go to **Authentication** → **Settings** (gear icon)
2. Scroll to **reCAPTCHA Enterprise Keys** or **reCAPTCHA Configuration**
3. Create a new reCAPTCHA key:
   - **Display Name**: Enter any name (e.g., "WhatsApp Replica")
   - **Domain**: Add your domain (e.g., `localhost`, `yourdomain.com`)
   - **Type**: Select **Invisible reCAPTCHA**
4. Copy the **Site Key** and **Secret Key**

✅ **Result**: reCAPTCHA is configured

---

### Step 3️⃣: Set up test phone numbers (Optional but Recommended)

1. In Firebase Console, **Authentication** → **Phone** → **Phone numbers for testing**
2. Add phone numbers with expected OTP codes:
   - **Phone Number**: `+919876543210`
   - **OTP Code**: `123456`
   - **Expires**: Keep default
3. Click **Add**

✅ **Result**: Test numbers won't consume SMS quota

---

### Step 4️⃣: Enable Billing

⚠️ **Important**: Firebase SMS requires billing to be enabled

1. Go to Firebase Project Settings
2. Navigate to **Billing**
3. Click **Enable Billing**
4. Add a payment method

✅ **Result**: SMS can now be sent from your app

---

### Step 5️⃣: Verify .env.local Variables

Make sure your `.env.local` has all Firebase variables:

```env
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_FIREBASE_MEASUREMENT_ID=your_measurement_id
VITE_GEMINI_API_KEY=your_gemini_api_key
```

---

## 🧪 Testing Workflow

### Phase 1: Test Mode Development
```
1. Keep TEST_MODE = true
2. Use test phone numbers from TEST_PHONE_NUMBERS
3. Test UI, validation, and flows
4. No Firebase Phone Auth setup needed
```

### Phase 2: Production Testing
```
1. Set TEST_MODE = false
2. Test with whitelisted test phone numbers
3. Verify SMS arrives
4. No SMS quota used
```

### Phase 3: Production Live
```
1. TEST_MODE = false
2. All users can sign up with real phone numbers
3. Real SMS OTP codes sent
4. SMS quota consumed
```

---

## Troubleshooting

### ❌ "Phone authentication is not enabled"
- ✅ **Fix**: Go to Firebase Console → Authentication → Enable Phone provider

### ❌ "reCAPTCHA verification failed"
- ✅ **Fix**: Go to Firebase Console → Authentication → Settings → Configure reCAPTCHA keys

### ❌ "SMS quota exceeded" or "Billing required"
- ✅ **Fix**: Enable Billing in Firebase Console

### ❌ "Too many requests"
- ✅ **Fix**: Wait a few minutes or use a different test phone number

### ❌ "Invalid phone number"
- ✅ **Fix**: Ensure format is `+91` followed by exactly 10 digits (e.g., `+919876543210`)

### ✅ OTP not arriving (Real Phone)
1. Check phone carrier SMS is not blocked
2. Verify billing is enabled
3. Check Firebase SMS quota in Console
4. Try a different test number

---

## 📋 Checklist

Complete these before going to production:

- [ ] Firebase project created
- [ ] Phone authentication enabled
- [ ] reCAPTCHA configured
- [ ] Test phone numbers added (optional)
- [ ] Billing enabled
- [ ] .env.local variables set correctly
- [ ] TEST_MODE = false in firebase.ts
- [ ] App tested with real phone number
- [ ] OTP SMS received successfully

---

## 🔑 Key Files

- **`src/lib/firebase.ts`** - Main Firebase configuration and OTP logic
- **`.env.local`** - Firebase credentials (never commit!)
- **`src/components/OTPVerificationModal.tsx`** - OTP UI component
- **`src/App.tsx`** - OTP handlers

---

## 📞 Support

**For Firebase Issues**:
- [Firebase Documentation](https://firebase.google.com/docs/auth/phone-auth)
- [Firebase Console Troubleshooting](https://firebase.google.com/support)

**For App Issues**:
- Check browser console (F12 → Console tab)
- Look for console logs marked with 🧪 (test mode) or 🔐 (production mode)

---

**Last Updated**: March 23, 2026
**Test Mode Enabled**: Yes ✅
**Production Ready**: Follow steps above
