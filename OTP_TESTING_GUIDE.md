# OTP Functionality Testing Guide

## ✅ Ready to Test!

Your app now has full OTP functionality enabled with **both test mode and real SMS support**.

---

## 🧪 Test Scenarios

### Test 1: Random OTP with Test Phone Number (Current Mode)
**Status**: ✅ Active (TEST_MODE = true)

```
Steps:
1. Open app: http://localhost:5173
2. Click "Sign in with Phone OTP" button
3. Phone number field shows: +91 (pre-filled)
4. Enter any 10 digits: 
   Example: 9876543210
   Result: +919876543210 in field
5. Click "Send OTP" button
6. Alert appears with:
   ✓ Generated random OTP (e.g., 456789)
   ✓ Phone number shown
   ✓ Message: "(This is test mode - no real SMS sent)"
7. OTP automatically moves to verification step
8. Enter the OTP from alert in the 6 input fields
9. Click "Verify OTP" button
10. ✅ Verification successful!
11. Phone verified message shows
12. Access to messaging app granted
```

**Expected Result**: ✅ OTP verification completes successfully

---

### Test 2: Predefined Test Numbers
**Phone Numbers**: 3 predefined numbers with fixed OTPs

```
Test Number 1:
  Phone: 9876543210 (becomes +919876543210)
  OTP: 123456
  
Test Number 2:
  Phone: 9988776655 (becomes +919988776655)
  OTP: 654321
  
Test Number 3:
  Phone: 9123456789 (becomes +919123456789)
  OTP: 111111

Testing:
1. Enter phone: 9876543210
2. Click "Send OTP"
3. Alert shows OTP: 123456
4. Enter: 123456
5. ✅ Verified
```

**Expected Result**: ✅ Each predefined number gets its fixed OTP

---

### Test 3: Wrong OTP (Error Handling)
**Testing Error Scenarios**

```
Scenario A: Wrong OTP Code
1. Send OTP to: 9876543210
2. Alert shows: 123456
3. Enter wrong OTP: 000000
4. Click "Verify OTP"
5. ❌ Error appears: "Invalid OTP. Please enter the correct code."
6. OTP fields clear
7. User can try again
8. ✅ Error handling works!

Scenario B: Incomplete OTP
1. Enter only 5 digits in OTP field
2. Click "Verify OTP"
3. ❌ Error: "Please enter all 6 digits"
4. ✅ Validation works!

Scenario C: Non-numeric input
1. Try to type letters in OTP field
2. ✅ Only numbers accepted (a, b, c rejected automatically)
```

**Expected Result**: ✅ All error scenarios handled gracefully

---

### Test 4: Resend OTP (60-second Timer)
**Testing Resend Functionality**

```
1. Send OTP to: 9876543210
2. Notice timer showing: "Resend code in 60s"
3. Wait ~30 seconds
4. Timer updates: "Resend code in 30s"
5. After 60 seconds, timer disappears
6. "Resend OTP" button appears (clickable)
7. Click "Resend OTP"
8. New OTP generated (different from first)
9. Alert shows new OTP
10. Enter new OTP
11. ✅ Second verification works!
```

**Expected Result**: ✅ Resend button active after 60 seconds, new OTP generated

---

### Test 5: Auto-Focus in OTP Input
**Testing UX Enhancement**

```
1. Send OTP to any number
2. First OTP input field is focused automatically
3. Type first digit: auto-moves to 2nd field
4. Type second digit: auto-moves to 3rd field
5. Continue with remaining 4 digits
6. After 6 digits entered, "Verify OTP" button available
7. ✅ All fields filled automatically
```

**Expected Result**: ✅ Smooth auto-focus between fields

---

### Test 6: Backspace Navigation
**Testing Backspace Handling**

```
1. Enter OTP: 123456
2. In 4th field, press Backspace
3. ✅ Moves focus to 3rd field
4. In 3rd field, press Backspace
5. ✅ Moves focus to 2nd field
6. Easy navigation backward
```

**Expected Result**: ✅ Backspace navigates to previous field

---

### Test 7: Change Phone Number
**Testing Phone Change Flow**

```
1. Enter phone: 9876543210
2. Click "Send OTP"
3. OTP step shows with this phone number
4. Click back arrow (⬅️) in header
5. ✅ Returns to phone entry step
6. Phone field still shows: +919876543210
7. Clear and enter different phone: 9988776655
8. Click "Send OTP" again
9. ✅ New OTP generated for new phone
```

**Expected Result**: ✅ Can change phone anytime and resend OTP

---

### Test 8: Real SMS (After Firebase Setup)
**Testing Real SMS Delivery** (when TEST_MODE = false)

```
Prerequisites:
- Firebase Phone Authentication: Enabled ✅
- Billing: Active
- reCAPTCHA: Configured
- Domain: Authorized
- TEST_MODE = false in firebase.ts

Steps:
1. Enter your real phone number: +91XXXXXXXXXX
2. Click "Send OTP"
3. 📱 Wait 10-30 seconds
4. SMS arrives on phone with:
   - Company: "Firebase"
   - Message: "Your sign-in code is: XXXXXX"
   - Code: Valid for 5 minutes
5. Enter received code
6. ✅ Account created successfully!

Expected Result: ✅ Real SMS received and verified
```

---

## 📊 Test Checklist

Complete all tests to verify full functionality:

### Phone Entry Tests
- [ ] Accepts 10 digits
- [ ] Prepends +91 automatically
- [ ] Rejects non-numeric input
- [ ] Shows validation error if < 10 digits
- [ ] Phone field pre-filled with +91 on load

### OTP Entry Tests
- [ ] All 6 fields present
- [ ] Only accepts numbers
- [ ] Auto-focuses to next field after digit entered
- [ ] Backspace moves to previous field
- [ ] Cannot enter more than 1 digit per field
- [ ] All fields must be filled to submit

### OTP Sending Tests
- [ ] Test Mode: Random OTP generated
- [ ] Predefined numbers: Get fixed OTP
- [ ] OTP shown in alert
- [ ] Phone number displayed in verification step
- [ ] Error handling for missing Firebase setup
- [ ] Smooth transition from phone to OTP step

### Verification Tests
- [ ] Correct OTP: Verification succeeds ✅
- [ ] Wrong OTP: Error message shown ❌
- [ ] Incomplete OTP: Validation error ❌
- [ ] After success: Access to app granted
- [ ] Phone marked as verified in Firestore

### Resend Tests
- [ ] 60-second timer shows
- [ ] Resend button disabled during timer
- [ ] Resend button enabled after timer
- [ ] New OTP generated on resend
- [ ] New OTP works for verification

### UX Tests
- [ ] Smooth animations (enter/exit)
- [ ] Clear error messages
- [ ] Loading states (Sending OTP..., Verifying...)
- [ ] Disabled states for buttons during loading
- [ ] Back button to change phone number
- [ ] Close (X) button to cancel
- [ ] Responsive on mobile

---

## 🎯 Testing Commands

### Browser Console Access (F12)
```javascript
// View all generated OTPs (Test Mode)
OTPUtils.getAllOTPs()

// Get specific OTP
OTPUtils.getOTP('+919876543210')

// View statistics
OTPUtils.stats()

// Clear expired OTPs
OTPUtils.clearExpired()

// Log details
OTPUtils.log()
```

---

## 🔍 Debugging

### View Console Logs
1. Open browser: F12 or Ctrl+Shift+I
2. Go to **Console** tab
3. Look for logs marked with:
   - 🧪 TEST MODE
   - ✅ Success messages
   - ❌ Error messages
   - 📝 OTP details

### Check Network Requests
1. Open browser: F12
2. Go to **Network** tab
3. Send OTP
4. Look for:
   - `signInWithPhoneNumber` (Firebase call)
   - Status: 200 (success) or error code
   - Response shows verification ID

### View Stored Data (Firestore)
1. Go to Firebase Console
2. Firestore Database
3. Users collection
4. Find user by phone number
5. Check fields:
   - `phoneNumber`: +91XXXXXXXXXX
   - `phoneVerified`: true/false
   - `verifiedPhoneNumber`: confirmed phone
   - `verifiedAt`: timestamp

---

## 📈 Performance Tests

### Load Time
- [ ] OTP modal loads in < 500ms
- [ ] "Send OTP" button response < 2 seconds
- [ ] Verification completes < 5 seconds

### Multiple Attempts
- [ ] Can send OTP 5+ times
- [ ] Each attempt generates new OTP (test mode)
- [ ] No memory leaks

---

## 🚀 Results Summary

After completing all tests:

```
Test Coverage: ___/100%

✅ Working Features:
- Phone entry validation
- Random OTP generation
- OTP verification
- Error handling
- Resend functionality
- Firebase integration
- Firestore updates
- Real SMS (if enabled)

⚠️ Issues Found:
(List any issues here)

🟢 Status: READY FOR PRODUCTION
```

---

## 📱 Real-World Scenarios

### Scenario 1: New User Registration
```
1. User opens app
2. Clicks "Sign in with Phone OTP"
3. Enters phone number
4. Receives OTP
5. Enters OTP
6. Account created ✅
7. Access to chat/messaging app
```

### Scenario 2: User Returns (with phone verification)
```
1. User opens app
2. Has saved phone number
3. Click "Verify Phone" in profile
4. Uses same phone
5. Receives OTP
6. Marks as verified
7. Enhanced security ✅
```

### Scenario 3: User Changes Phone
```
1. User in app
2. Goes to Profile → Edit
3. Clicks "Change Phone"
4. OTP workflow starts
5. Verifies with new phone
6. Updates phone in system
```

---

## 📞 Support

**If OTP doesn't work**:

1. Check Firebase Console:
   - ✅ Phone auth enabled
   - ✅ Billing active
   - ✅ Domain authorized

2. Check browser console (F12):
   - Look for error messages
   - Check network requests
   - Verify Firebase SDK loaded

3. Try test numbers first:
   - +919876543210 (OTP: 123456)
   - +919988776655 (OTP: 654321)

4. If real SMS doesn't work:
   - Check carrier SMS blocking
   - Verify phone number format
   - Check Firebase quota

---

**Status**: ✅ Ready to Test  
**Test Mode**: Active  
**Real SMS**: Available (after setup)  
**Date**: March 23, 2026
