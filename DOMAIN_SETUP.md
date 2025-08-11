# FitSync Custom Domain Setup

## Domain Configuration for https://fitsync.app

### 1. Supabase Configuration (CRITICAL)

**Go to Supabase Dashboard → Authentication → Settings:**

1. **Site URL:** `https://fitsync.app`
2. **Redirect URLs:** Add these URLs:
   ```
   https://fitsync.app/**
   https://fitsync.app/reset-password/**
   https://fitsync.app/confirm-email/**
   ```

### 2. DNS Configuration

Ensure your domain points to GitHub Pages:

```dns
CNAME: fitsync.app → netreapersoftware.github.io
```

### 3. GitHub Repository Settings

1. **Go to repository Settings → Pages**
2. **Custom domain:** `fitsync.app`
3. **Enforce HTTPS:** ✅ Enabled

### 4. CNAME File

The repository should have a `CNAME` file with:
```
fitsync.app
```

### 5. Test Email Links

After updating Supabase settings:

1. **Request password reset** from your FitSync app
2. **Email link should look like:**
   ```
   https://fitsync.app/reset-password/#access_token=eyJ...&refresh_token=...&type=recovery
   ```
3. **NOT like:**
   ```
   https://netreapersoftware.github.io/fitsync-website/reset-password/...
   ```

### 6. Common Issues

**Issue:** Email links still go to GitHub Pages URL
**Solution:** 
1. Clear Supabase cache (wait 5-10 minutes after changing settings)
2. Test password reset again
3. Check Site URL is exactly `https://fitsync.app` (no trailing slash)

**Issue:** "Redirect URL not allowed" error
**Solution:** 
1. Add `https://fitsync.app/**` to redirect URLs
2. Save settings and wait a few minutes
3. Test again

### 7. Verification Steps

✅ **CNAME file exists** in repository root  
✅ **Domain points to GitHub Pages** (check with `dig fitsync.app`)  
✅ **HTTPS certificate is active** (green lock icon)  
✅ **Supabase Site URL is `https://fitsync.app`**  
✅ **Redirect URLs include `https://fitsync.app/**`**  
✅ **Email links go to fitsync.app domain**  

Once all these are configured correctly, password reset should work perfectly!