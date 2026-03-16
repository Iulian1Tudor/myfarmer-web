# Setup Notes (not served by GitHub Pages)

## Universal Links — TODO when Apple Developer account is created

1. Get your **Apple Team ID**:
   - Go to developer.apple.com → Account → Membership → Team ID (10-char code, e.g. ABC12DEF34)

2. Replace `YOUR_TEAM_ID` in `.well-known/apple-app-site-association` (appears twice):
   - Line 6:  `"appIDs": ["YOUR_TEAM_ID.com.myfarmer.app"]`
   - Line 16: `"apps": ["YOUR_TEAM_ID.com.myfarmer.app"]`
   - Then push to GitHub — the live file at myfarmerapp.co.uk/.well-known/apple-app-site-association updates automatically.

3. In MyFarmer app repo, `app.config.js` already has `associatedDomains: ["applinks:myfarmerapp.co.uk"]`
   — no change needed there.

4. EAS build is required for Universal Links to activate (Expo Go ignores associatedDomains).
   Custom scheme myfarmer:// continues to work in Expo Go during development.

## Supabase URL Configuration (already done)
- Site URL: https://myfarmerapp.co.uk
- Redirect URLs: https://myfarmerapp.co.uk/**  and  myfarmer:///set-password
