# Termly Cross-Domain Consent Sync v3.0

Synchronizes Termly cookie consent preferences between WordPress and HubSpot (or any subdomains under the same root domain) using a shared cookie.

## Overview

When users visit multiple subdomains of your website (e.g., `www.example.com` and `hs.example.com`), they typically need to set their cookie consent preferences on each subdomain separately. This script solves that problem by:

1. Storing consent preferences in a shared cookie on the root domain
2. Automatically applying those preferences when users visit other subdomains
3. Syncing any consent changes back to the shared cookie

## How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Root Domain: .example.com                       │
│                                                                     │
│  ┌─────────────────────┐         ┌─────────────────────┐           │
│  │  www.example.com    │         │   hs.example.com    │           │
│  │    (WordPress)      │         │     (HubSpot)       │           │
│  │                     │         │                     │           │
│  │  User grants        │         │  User visits        │           │
│  │  consent ──────────►│ Cookie  │◄─── Consent auto-   │           │
│  │                     │ Shared  │     applied         │           │
│  └─────────────────────┘         └─────────────────────┘           │
│                                                                     │
│              Shared Cookie: termly_consent_sync                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Files Included

| File | Purpose |
|------|---------|
| `php-script.php` | WordPress implementation using Code Snippets plugin |
| `script.html` | Raw HTML/JS for HubSpot or other platforms |

---

## Implementation Instructions

### Prerequisites

- Termly consent banner must already be installed on both domains
- Both domains must share the same root domain (e.g., `www.example.com` and `hs.example.com`)
- SSL/HTTPS enabled on both domains (required for secure cookies)

---

## Step 1: Configure the Script

Before implementing, you need to update the `rootDomain` value in the script to match your domain.

### Find and Replace

In both `php-script.php` and `script.html`, locate this line:

```javascript
const CONFIG={rootDomain:"mywebsite.com",...
```

Change `mywebsite.com` to your root domain (without `www.` or subdomains):

```javascript
const CONFIG={rootDomain:"yourdomain.com",...
```

---

## Step 2: WordPress Installation (Code Snippets Plugin)

### 2.1 Install Code Snippets Plugin

1. In WordPress admin, go to **Plugins → Add New**
2. Search for "**Code Snippets**"
3. Install and activate the plugin by **Code Snippets Pro**

### 2.2 Add the Snippet

1. Go to **Snippets → Add New**
2. Give it a name: `Termly Cross-Domain Consent Sync`
3. Copy the entire contents of `php-script.php` into the code area
4. **Important Settings:**
   - Set **"Run snippet on"** to: `Only run on the front-end`
   - Or if using snippet scopes, select: `Run snippet everywhere (Frontend only)`
5. Click **Save Changes and Activate**

### 2.3 Verify Installation

1. Visit your WordPress site
2. Open browser Developer Tools (F12)
3. Go to **Elements** tab and search for `termly-consent-sync`
4. You should see the script in the page footer

---

## Step 3: HubSpot Installation

### Option A: Site-Wide via Settings (Recommended)

1. Log in to your HubSpot account
2. Go to **Settings** (gear icon) → **Website** → **Pages**
3. Scroll down to **Site Footer HTML**
4. Paste the contents of `script.html`
5. Click **Save**

### Option B: Via Custom Code Module

1. Go to **Marketing** → **Website** → **Website Pages**
2. Edit your page template
3. Add a **Custom HTML** module
4. Paste the contents of `script.html`
5. Save and publish

### Option C: Via HubSpot CMS

If using HubSpot CMS, add to your template's footer:

1. Go to **Marketing** → **Files and Templates** → **Design Tools**
2. Open your base template
3. Add the script before the closing `</body>` tag
4. Publish the template

---

## Step 4: Verify the Sync is Working

### Method 1: Browser Console

1. Visit your WordPress site
2. Open Developer Tools → Console
3. Type: `TermlyConsentSync.getDebugInfo()`
4. Press Enter

You should see output like:

```javascript
{
  config: {rootDomain: "yourdomain.com", cookieName: "termly_consent_sync", ...},
  currentHostname: "www.yourdomain.com",
  appliedFromOtherDomain: false,
  termlyConsent: {...},
  sharedConsentCookie: {...},
  builtConsentState: {...}
}
```

### Method 2: Cookie Inspection

1. Open Developer Tools → Application (Chrome) or Storage (Firefox)
2. Look under Cookies for your domain
3. Find `termly_consent_sync` cookie
4. Verify it's set on `.yourdomain.com` (with leading dot)

### Method 3: Cross-Domain Test

1. Visit your WordPress site and accept/configure cookies
2. Open a new tab and visit your HubSpot site
3. The same consent preferences should be automatically applied

---

## Configuration Options

### Enable Debug Mode

To troubleshoot issues, enable debug logging:

Find this line in the script:
```javascript
const CONFIG={rootDomain:"yourdomain.com",cookieName:"termly_consent_sync",cookieExpireDays:365,debug:false};
```

Change `debug:false` to `debug:true`:
```javascript
const CONFIG={rootDomain:"yourdomain.com",cookieName:"termly_consent_sync",cookieExpireDays:365,debug:true};
```

Then check the browser console for `[Termly Sync]` messages.

### Change Cookie Expiration

By default, the sync cookie expires after 365 days. To change this, modify `cookieExpireDays`:

```javascript
cookieExpireDays:30  // 30 days
```

---

## JavaScript API

The script exposes a global `TermlyConsentSync` object for manual control:

| Method | Description |
|--------|-------------|
| `TermlyConsentSync.sync()` | Manually trigger a consent sync |
| `TermlyConsentSync.clearSync()` | Delete the shared consent cookie |
| `TermlyConsentSync.getDebugInfo()` | Get current sync state and debug info |

### Example Usage

```javascript
// Force sync consent to shared cookie
TermlyConsentSync.sync();

// Clear synced consent (user will need to re-consent on each domain)
TermlyConsentSync.clearSync();

// Debug current state
console.log(TermlyConsentSync.getDebugInfo());
```

---

## Troubleshooting

### Consent Not Syncing Between Domains

1. **Check root domain configuration**: Ensure both sites use the same `rootDomain` value
2. **Verify HTTPS**: Both sites must use HTTPS for secure cookies
3. **Check cookie domain**: The cookie should be set on `.yourdomain.com` (with leading dot)
4. **Enable debug mode**: Check console for `[Termly Sync]` messages

### Cookie Not Being Set

1. Verify the script is loading (check for `termly-consent-sync` in page source)
2. Check browser console for JavaScript errors
3. Ensure Termly is properly configured and creating consent records

### Consent Applied But Not Persisting

1. Check if Termly is overwriting the applied consent
2. Verify the script loads **after** Termly initializes
3. Enable debug mode to see the sync flow

### Different Consent Formats

The script automatically converts between Termly's two consent formats:
- **API Cache format**: Used by Termly's native banner
- **GTM format**: Used when Termly integrates with Google Tag Manager

No configuration needed—conversion happens automatically.

---

## Technical Details

### Consent Data Structure

The shared cookie stores consent in this format:

```javascript
{
  "version": "3.0",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "source": "www.yourdomain.com",
  "consent": {
    "essential": true,
    "performance": true,
    "analytics": true,
    "advertising": false,
    "social_networking": false,
    "unclassified": false
  },
  "format": "api_cache",
  "rawKey": "TERMLY_API_CACHE",
  "rawValue": "..."
}
```

### Consent Categories Mapped

| Termly Category | GTM Consent Type |
|-----------------|------------------|
| essential | security_storage |
| performance | functionality_storage, personalization_storage |
| analytics | analytics_storage |
| advertising | ad_storage, ad_personalization, ad_user_data |
| social_networking | social_storage |
| unclassified | unclassified_storage |

### Event Listeners

The script listens for these Termly events:
- `termly:consentUpdated`
- `termly:consentSaved`
- `termly_consent_updated`

It also monitors:
- LocalStorage changes to Termly keys
- Termly consent modal buttons via MutationObserver

---

## Browser Support

- Chrome 60+
- Firefox 55+
- Safari 12+
- Edge 79+

Requires:
- LocalStorage API
- Document.cookie API
- MutationObserver API

---

## Security Considerations

- Cookie is set with `Secure` flag (HTTPS only)
- Cookie uses `SameSite=Lax` to prevent CSRF attacks
- Cookie is scoped to root domain only
- No sensitive data is stored (only consent preferences)

---

## License

MIT License - Feel free to use and modify for your needs.

---

## Support

If you encounter issues:

1. Enable debug mode and check console logs
2. Verify both scripts are loaded on respective sites
3. Ensure root domain configuration matches
4. Test with a fresh browser session (incognito mode)
