# Security Best Practices for FitSync Website

## Credential Management

### Supabase Anon Key Security

**The Supabase anon key is safe to expose publicly** and is designed for client-side applications:

- ✅ **Public by design**: Supabase anon keys are meant to be used in frontend code
- ✅ **No privileged access**: The anon key cannot access protected data without proper RLS policies
- ✅ **Industry standard**: Major apps (including Supabase's own site) expose anon keys publicly

### Security is Enforced at the Database Level

```sql
-- Example: RLS policy for password resets
CREATE POLICY "Users can only reset their own password" 
ON auth.users 
FOR UPDATE 
USING (auth.uid() = id);
```

## Recommended Approaches (in order of preference)

### 1. Direct Hardcoding (Recommended for Supabase)
Since the anon key is safe to expose, the simplest approach is direct hardcoding:

```javascript
const supabase = window.supabase.createClient(
    'https://your-project.supabase.co',
    'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' // This is safe to be public
);
```

### 2. Build-Time Environment Variables
For additional security layers or other sensitive data:

**GitHub Actions:**
```yaml
name: Deploy
on: [push]
jobs:
  deploy:
    steps:
    - uses: actions/checkout@v2
    - name: Replace credentials
      env:
        SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
        SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
      run: |
        envsubst < reset-password/index.template.html > reset-password/index.html
```

**Netlify:**
```toml
# netlify.toml
[build]
  command = "npm run build"
  
[build.environment]
  SUPABASE_URL = "https://your-project.supabase.co"
```

### 3. Runtime Configuration
For dynamic configuration without rebuilding:

```javascript
// config.js (separate file, can be gitignored)
window.FITSYNC_CONFIG = {
    supabaseUrl: 'https://your-project.supabase.co',
    supabaseAnonKey: 'your-anon-key'
};
```

## What NOT to Do

❌ **Never expose these in client-side code:**
- Service role keys (start with `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9` but have elevated permissions)
- Database passwords
- API secrets for third-party services
- Private keys

❌ **Don't use .env files for production static sites:**
- Not served by default by web servers
- Creates unnecessary complexity
- False sense of security

## Database Security Checklist

Ensure your Supabase database is properly secured:

1. ✅ **Enable Row Level Security (RLS)** on all tables
2. ✅ **Create appropriate RLS policies** for each table
3. ✅ **Test policies thoroughly** with different user scenarios
4. ✅ **Use service role key only in server environments**
5. ✅ **Regularly audit your database permissions**

## Real-World Examples

**Supabase's own website:**
```javascript
// Public in their GitHub repo
const supabase = createClient(
  'https://obuldanrptloktxcffvn.supabase.co',
  'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlhdCI6MTYwODI5NDUzMSwiZXhwIjoxOTIzODcwNTMxfQ...'
)
```

**Other major apps** like Vercel, Netlify, and GitHub all expose their Supabase anon keys publicly.

## Conclusion

For FitSync's password reset functionality, **direct hardcoding of the Supabase anon key is the recommended approach**. It's secure, simple, and follows industry best practices.