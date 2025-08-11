# Required Website Pages for fitsync.app

## Overview
Your mobile app is fully configured for email authentication. You need these pages on your fitsync.app website to complete the authentication flows.

## 1. Password Reset Page: `/reset-password`

### Purpose
Handle password reset tokens from email links and provide password reset form.

### Implementation
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reset Password - FitSync</title>
    <script src="https://unpkg.com/@supabase/supabase-js@2"></script>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            margin: 0;
            padding: 20px;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .container {
            background: white;
            padding: 40px;
            border-radius: 12px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.1);
            max-width: 400px;
            width: 100%;
            text-align: center;
        }
        .logo {
            font-size: 28px;
            font-weight: bold;
            color: #667eea;
            margin-bottom: 10px;
        }
        .title {
            font-size: 24px;
            font-weight: 600;
            color: #333;
            margin-bottom: 10px;
        }
        .subtitle {
            color: #666;
            margin-bottom: 30px;
        }
        .input {
            width: 100%;
            padding: 12px 16px;
            border: 2px solid #e1e5e9;
            border-radius: 8px;
            font-size: 16px;
            margin-bottom: 16px;
            box-sizing: border-box;
        }
        .input:focus {
            outline: none;
            border-color: #667eea;
        }
        .button {
            width: 100%;
            padding: 14px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            margin-bottom: 20px;
        }
        .button:hover {
            background: #5a6fd8;
        }
        .button:disabled {
            background: #ccc;
            cursor: not-allowed;
        }
        .success {
            color: #28a745;
            font-weight: 500;
        }
        .error {
            color: #dc3545;
            font-weight: 500;
        }
        .link {
            color: #667eea;
            text-decoration: none;
            font-weight: 500;
        }
        .link:hover {
            text-decoration: underline;
        }
        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="logo">FitSync</div>
        
        <!-- Loading State -->
        <div id="loading">
            <div class="title">Validating Reset Link</div>
            <div class="subtitle">Please wait...</div>
        </div>
        
        <!-- Password Reset Form -->
        <div id="reset-form" class="hidden">
            <div class="title">Reset Your Password</div>
            <div class="subtitle">Enter your new password below</div>
            
            <input 
                type="password" 
                id="newPassword" 
                class="input" 
                placeholder="New Password"
                minlength="6"
                required
            />
            
            <input 
                type="password" 
                id="confirmPassword" 
                class="input" 
                placeholder="Confirm New Password"
                minlength="6"
                required
            />
            
            <button id="resetButton" class="button" onclick="resetPassword()">
                Reset Password
            </button>
            
            <div id="message"></div>
            
            <p>
                <a href="fitsync://auth/login" class="link">Back to App</a>
            </p>
        </div>
        
        <!-- Error State -->
        <div id="error-state" class="hidden">
            <div class="title">Invalid Reset Link</div>
            <div class="subtitle error">This password reset link is invalid or has expired.</div>
            <p>
                <a href="fitsync://auth/login" class="link">Open FitSync App</a>
            </p>
        </div>
        
        <!-- Success State -->
        <div id="success-state" class="hidden">
            <div class="title">Password Reset Successfully! ✅</div>
            <div class="subtitle success">Your password has been updated.</div>
            <p>Redirecting to FitSync app...</p>
        </div>
    </div>
    
    <script>
        // Initialize Supabase client
        const supabase = window.supabase.createClient(
            'YOUR_SUPABASE_URL',
            'YOUR_SUPABASE_ANON_KEY'
        );
        
        let isValidSession = false;
        
        // Show different UI states
        function showLoading() {
            document.getElementById('loading').classList.remove('hidden');
            document.getElementById('reset-form').classList.add('hidden');
            document.getElementById('error-state').classList.add('hidden');
            document.getElementById('success-state').classList.add('hidden');
        }
        
        function showResetForm() {
            document.getElementById('loading').classList.add('hidden');
            document.getElementById('reset-form').classList.remove('hidden');
            document.getElementById('error-state').classList.add('hidden');
            document.getElementById('success-state').classList.add('hidden');
        }
        
        function showError() {
            document.getElementById('loading').classList.add('hidden');
            document.getElementById('reset-form').classList.add('hidden');
            document.getElementById('error-state').classList.remove('hidden');
            document.getElementById('success-state').classList.add('hidden');
        }
        
        function showSuccess() {
            document.getElementById('loading').classList.add('hidden');
            document.getElementById('reset-form').classList.add('hidden');
            document.getElementById('error-state').classList.add('hidden');
            document.getElementById('success-state').classList.remove('hidden');
        }
        
        // Handle password reset tokens
        async function handleResetToken() {
            try {
                const hashParams = new URLSearchParams(window.location.hash.substring(1));
                const accessToken = hashParams.get('access_token');
                const refreshToken = hashParams.get('refresh_token');
                const type = hashParams.get('type');
                
                if (type === 'recovery' && accessToken && refreshToken) {
                    const { error } = await supabase.auth.setSession({
                        access_token: accessToken,
                        refresh_token: refreshToken
                    });
                    
                    if (error) {
                        console.error('Session error:', error);
                        showError();
                    } else {
                        isValidSession = true;
                        showResetForm();
                    }
                } else {
                    showError();
                }
            } catch (error) {
                console.error('Error handling reset token:', error);
                showError();
            }
        }
        
        // Reset password function
        async function resetPassword() {
            if (!isValidSession) {
                showMessage('Invalid session. Please use the link from your email.', 'error');
                return;
            }
            
            const newPassword = document.getElementById('newPassword').value;
            const confirmPassword = document.getElementById('confirmPassword').value;
            
            // Validation
            if (!newPassword || !confirmPassword) {
                showMessage('Please fill in both password fields.', 'error');
                return;
            }
            
            if (newPassword !== confirmPassword) {
                showMessage('Passwords do not match.', 'error');
                return;
            }
            
            if (newPassword.length < 6) {
                showMessage('Password must be at least 6 characters long.', 'error');
                return;
            }
            
            // Disable button and show loading
            const button = document.getElementById('resetButton');
            button.disabled = true;
            button.textContent = 'Updating...';
            
            try {
                const { error } = await supabase.auth.updateUser({
                    password: newPassword
                });
                
                if (error) {
                    showMessage(error.message, 'error');
                    button.disabled = false;
                    button.textContent = 'Reset Password';
                } else {
                    showSuccess();
                    // Redirect to app after 3 seconds
                    setTimeout(() => {
                        window.location.href = 'fitsync://auth/password-reset-success';
                    }, 3000);
                }
            } catch (error) {
                console.error('Password reset error:', error);
                showMessage('An unexpected error occurred. Please try again.', 'error');
                button.disabled = false;
                button.textContent = 'Reset Password';
            }
        }
        
        // Show message helper
        function showMessage(text, type) {
            const messageDiv = document.getElementById('message');
            messageDiv.textContent = text;
            messageDiv.className = type;
        }
        
        // Handle Enter key in password fields
        document.addEventListener('keydown', function(event) {
            if (event.key === 'Enter') {
                resetPassword();
            }
        });
        
        // Initialize on page load
        document.addEventListener('DOMContentLoaded', function() {
            showLoading();
            handleResetToken();
        });
        
        // Handle hash changes (for single-page navigation)
        window.addEventListener('hashchange', handleResetToken);
    </script>
</body>
</html>
```

### Key Features:
- ✅ Validates reset tokens from email links
- ✅ Secure password form with validation
- ✅ Calls Supabase API to update password
- ✅ Redirects to `fitsync://auth/password-reset-success` on success
- ✅ Professional UI matching FitSync branding
- ✅ Mobile-responsive design
- ✅ Error handling and loading states

## 2. Email Confirmation Page: `/confirm-email`

### Purpose
Handle email confirmation tokens for new user signups.

### Implementation
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Confirm Email - FitSync</title>
    <script src="https://unpkg.com/@supabase/supabase-js@2"></script>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            margin: 0;
            padding: 20px;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .container {
            background: white;
            padding: 40px;
            border-radius: 12px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.1);
            max-width: 400px;
            width: 100%;
            text-align: center;
        }
        .logo {
            font-size: 28px;
            font-weight: bold;
            color: #667eea;
            margin-bottom: 10px;
        }
        .title {
            font-size: 24px;
            font-weight: 600;
            color: #333;
            margin-bottom: 10px;
        }
        .subtitle {
            color: #666;
            margin-bottom: 30px;
        }
        .success {
            color: #28a745;
            font-weight: 500;
        }
        .error {
            color: #dc3545;
            font-weight: 500;
        }
        .link {
            color: #667eea;
            text-decoration: none;
            font-weight: 500;
        }
        .link:hover {
            text-decoration: underline;
        }
        .hidden {
            display: none;
        }
        .emoji {
            font-size: 48px;
            margin-bottom: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="logo">FitSync</div>
        
        <!-- Loading State -->
        <div id="loading">
            <div class="emoji">⏳</div>
            <div class="title">Confirming Your Email</div>
            <div class="subtitle">Please wait while we verify your account...</div>
        </div>
        
        <!-- Success State -->
        <div id="success-state" class="hidden">
            <div class="emoji">🎉</div>
            <div class="title">Welcome to FitSync!</div>
            <div class="subtitle success">Your email has been confirmed successfully!</div>
            <p>Redirecting to your FitSync app...</p>
        </div>
        
        <!-- Error State -->
        <div id="error-state" class="hidden">
            <div class="emoji">❌</div>
            <div class="title">Confirmation Failed</div>
            <div class="subtitle error">This confirmation link is invalid or has expired.</div>
            <p>
                <a href="fitsync://auth/login" class="link">Open FitSync App</a>
            </p>
        </div>
    </div>
    
    <script>
        // Initialize Supabase client
        const supabase = window.supabase.createClient(
            'YOUR_SUPABASE_URL', 
            'YOUR_SUPABASE_ANON_KEY'
        );
        
        // Show different UI states
        function showLoading() {
            document.getElementById('loading').classList.remove('hidden');
            document.getElementById('success-state').classList.add('hidden');
            document.getElementById('error-state').classList.add('hidden');
        }
        
        function showSuccess() {
            document.getElementById('loading').classList.add('hidden');
            document.getElementById('success-state').classList.remove('hidden');
            document.getElementById('error-state').classList.add('hidden');
        }
        
        function showError() {
            document.getElementById('loading').classList.add('hidden');
            document.getElementById('success-state').classList.add('hidden');
            document.getElementById('error-state').classList.remove('hidden');
        }
        
        // Handle email confirmation
        async function handleConfirmation() {
            try {
                const hashParams = new URLSearchParams(window.location.hash.substring(1));
                const accessToken = hashParams.get('access_token');
                const refreshToken = hashParams.get('refresh_token');
                const type = hashParams.get('type');
                
                if (type === 'signup' && accessToken && refreshToken) {
                    const { error } = await supabase.auth.setSession({
                        access_token: accessToken,
                        refresh_token: refreshToken
                    });
                    
                    if (error) {
                        console.error('Confirmation error:', error);
                        showError();
                    } else {
                        showSuccess();
                        // Redirect to app after 3 seconds
                        setTimeout(() => {
                            window.location.href = 'fitsync://auth/confirmed';
                        }, 3000);
                    }
                } else {
                    showError();
                }
            } catch (error) {
                console.error('Error handling confirmation:', error);
                showError();
            }
        }
        
        // Initialize on page load
        document.addEventListener('DOMContentLoaded', function() {
            showLoading();
            handleConfirmation();
        });
        
        // Handle hash changes
        window.addEventListener('hashchange', handleConfirmation);
    </script>
</body>
</html>
```

## 3. Setup Instructions

### Replace Placeholder Values:
```javascript
// In both HTML files, replace these with your actual values:
const supabase = window.supabase.createClient(
    'https://your-project.supabase.co',  // Your Supabase URL
    'your-anon-key'                      // Your Supabase anon key
);
```

### Deploy to fitsync.app:
1. Save `/reset-password` page as `reset-password.html` or `reset-password/index.html`
2. Save `/confirm-email` page as `confirm-email.html` or `confirm-email/index.html`  
3. Deploy to your fitsync.app hosting (GitHub Pages, Netlify, etc.)

## 4. Expected Flows

### Password Reset Flow:
1. **User taps "Forgot Password?"** in LoginScreen
2. **Email sent from**: `noreply@mail.fitsync.app`
3. **Email link opens**: `https://fitsync.app/reset-password#access_token=...`
4. **Website validates token** → Shows password form
5. **User sets new password** → Website calls Supabase API
6. **Success** → Redirects to `fitsync://auth/password-reset-success`
7. **App opens** → User can login with new password

### Email Confirmation Flow:
1. **User signs up** in mobile app
2. **Email sent from**: `noreply@mail.fitsync.app`  
3. **Email link opens**: `https://fitsync.app/confirm-email#access_token=...`
4. **Website confirms email** → Shows welcome message
5. **Redirects to**: `fitsync://auth/confirmed`
6. **App opens** → User is logged in

Your mobile app is already configured for both flows - just need these website pages!