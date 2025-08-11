# Password Reset Debugging Guide

## Step 1: Test the Updated Page

1. **Deploy the updated code** with enhanced debugging
2. **Request a password reset** from your FitSync app
3. **Click the email link** and immediately open browser Developer Tools (F12)
4. **Check the Console tab** for detailed logs

## Step 2: Check Supabase Configuration

### 2.1 Verify Email Template Settings

1. Go to your [Supabase Dashboard](https://supabase.com/dashboard)
2. Navigate to **Authentication** → **Settings**
3. Scroll down to **URL Configuration** and check:
   - ✅ **Site URL** should be `https://fitsync.app`
   - ✅ **Redirect URLs** should include:
     ```
     https://fitsync.app/**
     https://fitsync.app/reset-password/**
     https://fitsync.app/confirm-email/**
     ```

### 2.2 Check Email Templates

1. In **Authentication** → **Settings**
2. Go to **Email Templates** → **Reset Password**
3. The template should contain a link like:
   ```html
   <a href="{{ .SiteURL }}/reset-password?token={{ .TokenHash }}">Reset Password</a>
   ```
   OR
   ```html
   <a href="{{ .SiteURL }}/reset-password#access_token={{ .Token }}&type=recovery&expires_at={{ .TokenExpiresAt }}">Reset Password</a>
   ```

## Step 3: Common Issues & Solutions

### Issue 1: Wrong Site URL
**Problem:** Email links go to wrong domain
**Solution:** Update Site URL in Supabase Dashboard → Authentication → Settings

### Issue 2: Email Template Format
**Problem:** Email uses query params instead of hash fragments
**Solution:** Update email template to use hash fragments:
```html
<a href="{{ .SiteURL }}/reset-password#access_token={{ .Token }}&refresh_token={{ .RefreshToken }}&expires_at={{ .TokenExpiresAt }}&type=recovery">Reset Password</a>
```

### Issue 3: Redirect URL Not Allowed
**Problem:** Supabase blocks the redirect
**Solution:** Add your website URL to allowed redirect URLs

## Step 4: What to Look For in Console

When you click the reset link, look for these console messages:

### ✅ Good Signs:
```
🔍 Full URL: https://your-site.com/reset-password/#access_token=...
✅ Valid recovery tokens found, setting session...
✅ Session set successfully: {...}
```

### ❌ Bad Signs:
```
❌ URL contains error: invalid_request
❌ Invalid or missing tokens
❌ Session error: Invalid JWT
```

## Step 5: Manual Test URLs

Try these test patterns to see which format your emails use:

**Format 1 (Hash):**
```
https://your-site.com/reset-password/#access_token=ABC&refresh_token=XYZ&type=recovery
```

**Format 2 (Query):**
```
https://your-site.com/reset-password/?access_token=ABC&refresh_token=XYZ&type=recovery
```

**Format 3 (Token Hash):**
```
https://your-site.com/reset-password/?token=ABC123
```

## Step 6: Share Debug Info

After testing, please share:
1. **Console log output** from the browser
2. **Actual email link format** (sanitized, remove actual tokens)
3. **Supabase Site URL** setting
4. **Any error messages** from the console

## Quick Fix: Alternative URL Parsing

If the issue persists, try this enhanced URL parsing in your reset page:

```javascript
// Enhanced token extraction
function extractTokensFromUrl() {
    const url = new URL(window.location);
    let params = new URLSearchParams();
    
    // Try hash first
    if (url.hash) {
        params = new URLSearchParams(url.hash.substring(1));
    }
    // Try search params
    else if (url.search) {
        params = new URLSearchParams(url.search);
    }
    // Try token-based format
    else if (url.searchParams.get('token')) {
        // Handle legacy token format
        return {
            access_token: url.searchParams.get('token'),
            type: 'recovery'
        };
    }
    
    return {
        access_token: params.get('access_token'),
        refresh_token: params.get('refresh_token'),
        type: params.get('type'),
        error: params.get('error')
    };
}
```

This will help us identify exactly where the issue is occurring!