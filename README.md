# developerburakgul.github.io

GitHub Pages site used as the Universal Links (AASA) host and Smart App Banner provider for iOS apps.

---

## How It Works

When a user taps a link like `https://developerburakgul.github.io/templateapp/home`:

- **App installed** → iOS opens the app directly at the correct screen (Universal Link via AASA)
- **App not installed** → Safari shows a fallback page with a Smart App Banner and an App Store download button

---

## Repository Structure

```
.
├── .well-known/
│   └── apple-app-site-association   # Universal Links config (one file, all apps)
├── _layouts/
│   └── applink.html                 # Shared layout: Smart App Banner + App Store button
├── templateapp/
│   ├── home/index.html
│   ├── favorites/index.html
│   └── settings/index.html
├── tapzero/
│   ├── play/index.html
│   ├── leaderboard/index.html
│   ├── history/index.html
│   ├── settings/index.html
│   └── game/index.html
├── 404.html                         # Universal fallback for unknown paths
├── index.html
└── _config.yml                      # Jekyll config: url + defaults per app namespace
```

---

## Registered Apps

| App | Bundle ID | Path Namespace |
|-----|-----------|----------------|
| TemplateApp (prod) | `com.BurakGul.TemplateApp` | `/templateapp/*` |
| TemplateApp (dev) | `com.BurakGul.TemplateApp-Dev` | `/templateapp/*` |
| TemplateApp (mock) | `com.BurakGul.TemplateApp-Mock` | `/templateapp/*` |
| TapZero (prod) | `com.BurakGul.TapZero` | `/tapzero/*` |
| TapZero (dev) | `com.BurakGul.TapZero-Dev` | `/tapzero/*` |
| TapZero (mock) | `com.BurakGul.TapZero-Mock` | `/tapzero/*` |

Team ID: `ZBYGRL25JU`

---

## Adding a New App

### 1. AASA — `.well-known/apple-app-site-association`

Add a new `details` record with the app's bundle IDs and path namespace:

```json
{
    "appIDs": [
        "ZBYGRL25JU.com.BurakGul.NewApp",
        "ZBYGRL25JU.com.BurakGul.NewApp-Dev",
        "ZBYGRL25JU.com.BurakGul.NewApp-Mock"
    ],
    "components": [
        {
            "/": "/newapp/feed",
            "comment": "Feed ekranı"
        },
        {
            "/": "/newapp/profile",
            "comment": "Profil ekranı"
        }
    ]
}
```

### 2. Jekyll defaults — `_config.yml`

Add a `defaults` block for the new path namespace:

```yaml
defaults:
  - scope: { path: "newapp" }
    values:
      layout: applink
      app_id: "NEW_APP_STORE_ID"
      app_name: "NewApp"
```

### 3. Fallback pages

Create one `index.html` per deep-link path. Each file is 3 lines — everything else comes from the layout and defaults:

```
newapp/
├── feed/index.html
└── profile/index.html
```

Contents of each file:

```yaml
---
title: Feed
app_argument: /newapp/feed
permalink: /newapp/feed
---
```

> `permalink` zorunlu — olmadan GitHub Pages `folder/index.html` yapısı nedeniyle
> `/newapp/feed` → `/newapp/feed/` şeklinde 301 redirect döner ve sayfa 200 vermez.

Push and GitHub Pages handles the rest.

---

## Updating an App Store ID

All pages for an app namespace share a single `app_id` value defined in `_config.yml`. To update:

```yaml
# _config.yml
defaults:
  - scope: { path: "templateapp" }
    values:
      app_id: "1234567890"   # replace TEMPLATEAPP_APP_STORE_ID with the real numeric ID
```

Find the numeric ID in App Store Connect — it's the number in the app's URL.

---

## Verification

**Check Apple's CDN has picked up the AASA:**
```bash
curl https://app-site-association.cdn-apple.com/a/v1/developerburakgul.github.io
```
Should return `200 OK` with your JSON and `Apple-Origin-Format: json`.

**Check a fallback page returns 200 (not 404):**
```bash
curl -I https://developerburakgul.github.io/templateapp/home
```

**Test Universal Link on simulator:**
```bash
xcrun simctl openurl booted "https://developerburakgul.github.io/templateapp/home"
```
