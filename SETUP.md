# FitSync Website Setup Guide

## Quick Setup (Recommended)

Since Supabase anon keys are **safe to expose publicly**, the simplest setup is direct configuration:

### 1. Update Supabase Credentials

Edit `reset-password/index.html` and replace the placeholder values:

```javascript
const supabase = window.supabase.createClient(
    'https://your-project.supabase.co',        // Replace with your Supabase URL
    'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' // Replace with your anon key
);
```

### 2. Find Your Supabase Credentials

1. Go to your [Supabase Dashboard](https://supabase.com/dashboard)
2. Select your project
3. Go to **Settings** → **API**
4. Copy:
   - **Project URL** (for SUPABASE_URL)
   - **Project API keys** → `anon` `public` (for SUPABASE_ANON_KEY)

### 3. Deploy

Commit and push to your repository. The website will work immediately.

## Advanced Setup (GitHub Actions)

For automated deployment with environment variables:

### 1. Add GitHub Secrets

In your GitHub repository:
1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Add repository secrets:
   - `SUPABASE_URL`: Your Supabase project URL
   - `SUPABASE_ANON_KEY`: Your Supabase anon key

### 2. Enable GitHub Pages

1. Go to **Settings** → **Pages**
2. Set source to **GitHub Actions**
3. The workflow will automatically deploy on push to main

### 3. Verify Deployment

Check the **Actions** tab for deployment status. Your website will be available at `https://yourusername.github.io/repository-name/`

## Security Notes

✅ **Supabase anon keys are safe to expose** - they're designed for client-side use
✅ **Security is handled by Row Level Security (RLS)** policies in your database
✅ **No sensitive data is exposed** - this follows industry best practices

## File Structure

```
FitSync-Website/
├── .github/workflows/
│   └── deploy.yml           # Automated deployment
├── reset-password/
│   └── index.html          # Password reset page
├── SECURITY.md            # Security documentation
└── SETUP.md              # This file
```

## Testing

1. Create a test password reset in your app
2. Check the email link leads to your website
3. Verify the password reset functionality works
4. Confirm redirect back to your app

## Troubleshooting

**"Invalid Reset Link" error:**
- Check your Supabase URL and anon key are correct
- Verify the reset email link format matches: `https://your-site.com/reset-password/#access_token=...`

**Console errors about Supabase:**
- Ensure you've replaced both placeholder values
- Check browser developer tools for specific error messages

**Reset link doesn't redirect to app:**
- Verify the deep link format: `fitsync://auth/password-reset-success`
- Test the deep link manually on your device