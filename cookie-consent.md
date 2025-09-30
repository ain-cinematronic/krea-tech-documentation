# Cookie Consent Implementation

This project uses the Orest Bida CookieConsent v3 banner with consent-based GA4 loading

## Behavior Matrix

| Analytics Settings | GA Measurement ID | Admin UI | Banner Shows Analytics |
|-------------------|-------------------|----------|------------------------|
| Disabled | - | Toggle hidden | No analytics category |
| Enabled | Missing/Invalid | Toggle hidden | No analytics category |
| Enabled | Valid (G-XXXX) | Toggle visible & clickable | Shows if admin enables |

## Usage

### Admin Configuration
1. Navigate to **System Globals > Analytics Settings**
2. Enable GA and add measurement ID
3. Go to **System Globals > Cookie Settings**
4. Toggle "Enable Optional Analytics Category" (now visible)
5. Customize banner text as needed

### Footer Integration
Users can reopen preferences via footer "Cookie settings" link:
```tsx
window.showCookieSettings()
```

## Technical Notes

- **Data Consistency**: Server-side hooks prevent analytics category from being enabled without proper GA config
- **GraphQL Integration**: Both globals exposed via `fetchGlobals()` for SSR
- **Client Safety**: GA config validated client-side before script injection
- **Privacy First**: IP anonymization enabled by default, consent required for any tracking
- **Debug Mode**: `debug_mode` is sent to GA automatically in development or when `NEXT_PUBLIC_GA_DEBUG=true`

### Environment Variables

| Variable | Purpose | Default | Notes |
|----------|---------|---------|-------|
| `NEXT_PUBLIC_VERBOSE_LOGGING` | Enables extra client logging if referenced | unset/false |
| `NEXT_PUBLIC_GA_DEBUG` | Forces GA debug mode outside dev | false | Only set in staging when verifying events. |
| `NEXT_PUBLIC_ENABLE_COOKIE_BANNER` | (Optional) Disable banner entirely | true (implicit) | If false, consent UI + GA gating skipped. |

`NEXT_PUBLIC_GA_DEBUG` is helpful on staging to surface events in GA DebugView without relying on the browser extension. Do NOT enable it permanently in production; it is only for diagnostic sessions.

## Maintenance

Library updates:
```bash
yarn add vanilla-cookieconsent@latest
```
Verify configuration compatibility with [upstream docs](https://cookieconsent.orestbida.com/). Orest Bida CookieConsent v3](https://cookieconsent.orestbida.com/) library with dynamic Google Analytics integration.

## Architecture

### Core Components
- `src/app/_components/CookieConsent/CookieConsentInitializer.tsx` - Client-side banner initialization
- `src/payload/globals/CookieSettings.ts` - Admin-configurable banner content and behavior
- `src/payload/globals/AnalyticsSettings.ts` - Google Analytics configuration
- Footer integration via `window.showCookieSettings()` trigger

### Admin Configuration
Two CMS globals under **System Globals**:

1. **Cookie Settings** (SystemAdmin only) - Controls banner text, titles, descriptions; consumed by the frontend at runtime.
2. **Analytics Settings** (SystemAdmin only) - Controls GA integration (measurement ID, anonymization); consumed by the frontend at runtime.

### Dynamic Behavior
The analytics category only appears when:
- Analytics Settings has `enableGA: true`
- Valid GA4 measurement ID provided (G-XXXXXXXXXX format)
- Cookie Settings has `enableAnalytics: true`

## Categories

1. **Necessary** (always enabled, read-only)
   - Signed device identifier (for rating deduplication)
   - Signed device identifier used to prevent duplicate ratings
   - Security and session essentials

2. **Analytics** (optional, consent-based)
   - Google Analytics 4 with Consent Mode
   - Only loads after explicit user consent
   - IP anonymization enforced by default

## Google Analytics Integration

### Setup Process (SystemAdmin)
1. **Enable in Analytics Settings**
   - Set `enableGA: true`
   - Add valid GA4 measurement ID (G-XXXXXXXXXX)
   - Configure IP anonymization (default: enabled)

2. **Configure Cookie Banner**
   - Analytics Settings properly configured → "Enable Optional Analytics Category" appears
   - Admin can toggle whether to show analytics choice to users
   - Customize analytics category description text

### Technical Implementation & Security
- **Lazy Loading**: GA script only loads after user consent
- **Consent Mode**: Integrates with Google's consent framework
- **Server-side Validation**: Hooks ensure data consistency
- **Conditional UI**: Analytics toggle hidden until GA properly configured
 - **Role Enforcement**: Only SystemAdmin can modify or even see these globals in the admin; other roles consume read-only data indirectly.
 - **Debug Mode Injection**: `debug_mode` flag set when `process.env.NODE_ENV==='development'` or `NEXT_PUBLIC_GA_DEBUG==='true'`.


## Manual Reopen

`window.showCookieSettings()` is attached for optional programmatic reopening.

## Accessibility & Simplicity

The banner uses the default “bar” layout (bottom center) with concise wording focused on essential vs analytics only.

## Maintenance

If the library is updated, bump the version:

```bash
yarn add vanilla-cookieconsent@<newVersion>
```
