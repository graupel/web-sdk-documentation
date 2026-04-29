# QuadraScan SDK — Getting Started

The QuadraScan SDK embeds a full-body 3D scan and measurement experience into any website with a single `<script>` tag and two function calls. The entire scan flow — camera access, pose detection, 3D model generation, and measurements — runs inside a hosted iframe overlay. No processing code runs on your site.

---

## Prerequisites

- A **license key** issued by QuadraScan for your domain.
- Your domain URL registered alongside the key (requests from unregistered origins are rejected with HTTP 403).
- Your site must be served over **HTTPS** in production (required for camera access).

---

## Integration (3 steps)

### 1. Add the script tag

Place this in your `<head>` or just before `</body>`:

```html
<script src="https://quadrascan.fitmatch.ai/sdk/quadrascan.js"></script>
```

### 2. Initialize the SDK

Call `QuadraScan.init()` once — before any scan is started. A good place is a `DOMContentLoaded` handler or immediately after the script tag.

```js
QuadraScan.init({
  apiKey:    'YOUR_LICENSE_KEY',  // required — key issued to your domain
  teamId:    'YOUR_TEAM_ID',      // required — associates scans with your team (never shown to the user)
  club:      'YOUR_CLUB_NAME',    // optional — club name (e.g. 'FC Dallas')
  athleteId: 'YOUR-OWN-UUID',     // optional — caller-supplied UUID (v4) to identify the athlete
  scanType:  'both',              // optional: 'self' | 'manual' | 'both' (default: 'both')
  tutorial: {                     // optional — controls pre-scan tutorial behavior
    enabled:          true,       // show/hide tutorial and tutorial buttons entirely
    required:         true,       // first-time users must watch before scanning
    requireEveryTime: false,      // if true, ignores localStorage and always shows tutorial
  },
});
```

**Pre-filling athlete data (optional)**

If you already have the athlete's information, pass it via the `athlete` option to pre-fill the **Athlete Info** step in the scan flow:

**TOU -> Athlete Info -> Scan Type -> Camera/Upload -> Viewer**

```js
QuadraScan.init({
  apiKey:        'YOUR_LICENSE_KEY',
  teamId:        'YOUR_TEAM_ID',
  club:          'FC Dallas',        // optional — club name stored with the athlete record
  clientName:    'FC Dallas',        // optional — shown in the client TOU screen
  clientTouUrl:  'https://fcdallas.com/terms',    // optional — your Terms of Use URL
  clientPpUrl:   'https://fcdallas.com/privacy',  // optional — your Privacy Policy URL
  athlete: {
    firstName:  'Jane',
    lastName:   'Doe',
    gender:     'Female',        // case-insensitive: 'male' | 'female' | 'other'
    dob:        '2005-08-14',    // YYYY-MM-DD (must be a real calendar date)
    heightCm:   170,             // metric height (cm). Converted to ft/in internally.
    // OR use imperial instead of heightCm:
    // heightFt: 5,
    // heightIn: 7,
    weight:     59,              // weight as a number
    weightUnit: 'metric',        // 'imperial' (lbs) | 'metric' (kg) — default: 'imperial'
  },
  scanType:    'both',           // optional: 'self' | 'manual' | 'both' (default: 'both')
  showResults: true,             // optional: show measurements screen after scan (default: true)
  width:       '1000px',         // optional: max width of the overlay (default: '1000px')
  height:      '90vh',           // optional: max height of the overlay (default: '90vh')
});
```

The SDK trims string fields, normalizes `gender` to lowercase (`Male` and `male` both work),
validates `dob` as a real `YYYY-MM-DD` date, and always converts `height` and `weight` to 
imperial units (ft/in and lbs) before sending athlete data to the API.

### 3. Start a scan

Attach `QuadraScan.startScan()` to any button or event.

```js
document.getElementById('scan-btn').addEventListener('click', () => {
  QuadraScan.startScan({
    onComplete(data) {
      // 'data' contains the full result payload.
      console.log('Scan complete:', data);
    },
    onError(err) {
      console.error('Scan error:', err);
    },
    onCancel() {
      console.log('User closed the scan.');
    },
  });
});
```

---

## `init()` Options

| Option | Type | Required | Default | Description |
|---|---|---|---|---|
| `apiKey` | string | **Yes** | — | License key issued for your domain. |
| `teamId` | string | **Yes** | — | Team ID to associate all scans with. Never shown to the end user. |
| `club` | string | No | `''` | Club name associated with the athlete (e.g. `'FC Dallas'`). Stored with the athlete record. |
| `athleteId` | string | No | `''` | Caller-supplied UUID (v4) to identify the athlete. If an athlete with this UUID already exists it is returned as-is (idempotent); if not, a new athlete is created using this UUID instead of an auto-generated one. Must be a valid UUID format. |
| `athlete` | object | No | `{}` | Pre-supplied athlete data used to pre-fill Athlete Info. See the **`athlete` fields** table below. |
| `scanType` | string | No | `'both'` | `'self'` = guided camera scan only. `'manual'` = upload photos from library only. `'both'` = user chooses Guided Scan vs Upload Photos on the landing screen. |
| `tutorial` | object | No | `{ enabled: true, required: true, requireEveryTime: false }` | Controls the pre-scan tutorial shown before Guided Scan and Upload Photos. See the **`tutorial` fields** table and **Pre-scan Tutorial** section below. |
| `showResults` | boolean | No | `true` | Whether to show the measurements screen after the scan completes. If `false`, `onComplete` still fires with the full result payload, and then the overlay closes automatically (no action required by the user). |
| `width` | string | No | `'1000px'` | Max width of the iframe overlay, e.g. `'720px'`. |
| `height` | string | No | `'90vh'` | Max height of the iframe overlay, e.g. `'85vh'`. |
| `embedUrl` | string | No | auto-detected | Override the hosted embed base URL. Used for staging or feature-branch testing only. |
| `clientName` | string | No | `''` | Your organization name shown in the client TOU screen (e.g. `'FC Dallas'`). Only relevant when `clientTouUrl` or `clientPpUrl` are also set. |
| `clientTouUrl` | string | No | `''` | URL for your Terms of Use. When provided, a second TOU screen is shown after the Fit:Match one. Can be used independently of `clientPpUrl`. |
| `clientPpUrl` | string | No | `''` | URL for your Privacy Policy. When provided, a second TOU screen is shown after the Fit:Match one. Can be used independently of `clientTouUrl`. |

### `athlete` fields

| Field | Type | Description |
|---|---|---|
| `firstName` | string | Athlete's first name. Leading/trailing spaces are trimmed. |
| `lastName` | string | Athlete's last name. Leading/trailing spaces are trimmed. |
| `gender` | string | Case-insensitive: `'male'` \| `'female'` \| `'other'`. |
| `dob` | string | Date of birth in strict `YYYY-MM-DD` format (must be a real calendar date). |
| `heightCm` | number | Height in centimeters (e.g. `170`). Converted to ft/in internally. Use instead of `heightFt/In`. |
| `heightFt` | number | Height — feet component (e.g. `5`). Use if `heightCm` is omitted. |
| `heightIn` | number | Height — inches component (e.g. `7`). Defaults to `0` if omitted. |
| `weight` | number | Body weight as a number. Sent to the API in imperial lbs when provided. |
| `weightUnit` | string | Input hint: `'imperial'` (lbs) or `'metric'` (kg). Defaults to `'imperial'`. |

### `tutorial` fields

| Field | Type | Default | Description |
|---|---|---|---|
| `enabled` | boolean | `true` | Shows or hides the tutorial screen and tutorial buttons entirely. Set to `false` to disable the tutorial feature completely. |
| `required` | boolean | `true` | When `true`, first-time users must watch the tutorial before entering a scan. When `false`, the tutorial is bypassed on scan start (users can still watch via the tutorial button). |
| `requireEveryTime` | boolean | `false` | When `true`, the localStorage watched flag is ignored and the tutorial is shown before every scan. Takes precedence over `required`. |

---

## Pre-scan Tutorial

The tutorial is a full-screen instructional screen shown before the user enters Guided Scan or Upload Photos. It contains a video (Guided Scan) or a static image (Upload Photos) with a brief description and a **Continue** button.

A **Tutorial** button appears on the scan options screen at all times (unless `enabled: false`) so users can rewatch whenever they want.

### localStorage key

The "watched" flag is stored in `localStorage` (not `sessionStorage`) so it persists across browser sessions on the same device/browser.
---

## `startScan()` Options

| Option | Type | Required | Description |
|---|---|---|---|
| `onComplete` | function | **Yes** | Called with the result payload when processing finishes successfully. If `showResults` is `true` (default), the overlay stays open on the measurements screen until the user dismisses it. If `showResults` is `false`, `onComplete` still fires with the payload, then the overlay closes automatically. |
| `onError` | function | No | Called with an error message string if something goes wrong (invalid key, network failure, etc.). The overlay remains open so the user can see the error inside the scan experience. |
| `onCancel` | function | No | Called when the user dismisses the overlay without completing a scan. Not fired if the scan already completed. |

---

## `close()` Method

```js
QuadraScan.close();
```

Programmatically closes the overlay. Fires `onCancel` if a scan was not already completed.

---

## Result Payload (`onComplete`)

```js
{
  scanId: "464f27fb-e5d6-46ed-87c7-19b56c86136c",

  athlete: {
    age:    15,           // number, derived from date of birth entered during scan
    gender: "female",     // string
    height: "5' 7\"",     // pre-formatted imperial string (ft' in")
    weight: "100 lb"      // pre-formatted string — unit is "lb"
  },

  measurements: {
    // Nested measurement categories — see structure below.
    breadth_measurements:       { display_name, measurements: { ... } },
    full_body_measurements:     { display_name, measurements: { ... } },
    lower_body_circumferences:  { display_name, measurements: { ... } },
    upper_body_circumferences:  { display_name, measurements: { ... } },
    user_inputs:                { display_name, measurements: { ... } }
  }
}
```

Each measurement entry inside a category follows this shape:

```js
{
  display_name: "Chest",    // human-readable label
  value: 0.9918,            // numeric value
  unit: "m"                 // unit string — see table below
}
```

### Units

| `unit` value | Meaning |
|---|---|
| `"m"` | Meters — multiply by `39.3701` to get inches |
| `"kg"` | Kilograms — multiply by `2.20462` to get lbs |
| `"lb"` | Pounds (user input) |
| `"ft"` | Feet (user input) |
| `"in"` | Inches (user input) |
| `"yr"` | Years (e.g. estimated age at Peak Height Velocity) |
| `""` | Unitless ratio or index (e.g. BMI, BRI, WHtR) |

### Accessing Common Measurements

All circumference and breadth values are in meters. Example helper:

```js
function getMeasurement(measurements, category, key) {
  const entry = measurements[category]?.measurements?.[key];
  if (!entry || entry.value == null) return null;
  return entry; // { display_name, value, unit }
}

const m = data.measurements;
const chest = getMeasurement(m, 'upper_body_circumferences', 'chest_circumference');
const waist = getMeasurement(m, 'upper_body_circumferences', 'narrowest_waist_circumference');
const hips  = getMeasurement(m, 'lower_body_circumferences', 'hip_circumference');

// Convert meters → inches
const toInches = (val) => (val * 39.3701).toFixed(1) + '"';
console.log('Chest:', toInches(chest.value)); // e.g. "39.1\""
```

### Key Measurement Paths

| Label | Category | Key |
|---|---|---|
| Chest | `upper_body_circumferences` | `chest_circumference` |
| Waist (narrowest) | `upper_body_circumferences` | `narrowest_waist_circumference` |
| Low Waist | `upper_body_circumferences` | `waist_circumference` |
| Upper Arm (L) | `upper_body_circumferences` | `upper_arm_circumference_left` |
| Forearm (L) | `upper_body_circumferences` | `forearm_circumference_left` |
| Hip | `lower_body_circumferences` | `hip_circumference` |
| Upper Thigh (L) | `lower_body_circumferences` | `upper_thigh_circumference_left` |
| Calf (L) | `lower_body_circumferences` | `calf_circumference_left` |
| Leg Length (L) | `full_body_measurements` | `leg_length_left` |
| BMI | `full_body_measurements` | `bmi` |
| BRI | `full_body_measurements` | `bri` |
| Waist-to-Height Ratio | `full_body_measurements` | `waist_to_height_ratio` |
| Lean Mass | `full_body_measurements` | `lean_mass` |
| Total Fat Mass | `full_body_measurements` | `total_fat` |
| Est. Age at PHV | `full_body_measurements` | `phv` |

---

## WebView Integration (iOS / Android / Flutter)

The recommended approach for mobile apps is **not** to load `quadrascan.fitmatch.ai` directly in a WebView. Instead:

1. **Host your own integration page** on your domain (e.g. `https://app.yoursite.com/scan`). This page loads the QuadraScan SDK like any normal web integration.
2. **Point the WebView to that page.** The SDK handles the full scan flow inside an iframe overlay inside your page, inside the WebView.
3. **Pass results back to native code** from the `onComplete` callback using the platform's JavaScript bridge.

This approach is correct because:
- Your license key is registered to your domain — requests from `app.yoursite.com` will pass origin validation automatically.
- Your native app stays in control of the page lifecycle, navigation, and any app-level UI around the scan.
- The scan result lands in `onComplete` as a JavaScript object, which you then forward to native code.

---

### Passing Results Back to Native Code

#### iOS (Swift / Objective-C)

Register a `WKScriptMessageHandler` in your `WKWebViewConfiguration`, then call it from `onComplete`:

```js
// On your integration page
QuadraScan.startScan({
  onComplete(data) {
    // Send the full result payload to native Swift/ObjC code
    window.webkit.messageHandlers.scanComplete.postMessage(JSON.stringify(data));
  },
  onError(err) {
    window.webkit.messageHandlers.scanError.postMessage(err);
  },
});
```

```swift
// In your WKScriptMessageHandler
func userContentController(_ ucc: WKUserContentController,
                           didReceive message: WKScriptMessage) {
    if message.name == "scanComplete",
       let body = message.body as? String,
       let data = body.data(using: .utf8) {
        let result = try? JSONDecoder().decode(ScanResult.self, from: data)
        // Use result in your app
    }
}
```

#### Android (Kotlin / Java)

Expose a `@JavascriptInterface` and call it from `onComplete`:

```js
// On your integration page
QuadraScan.startScan({
  onComplete(data) {
    Android.onScanComplete(JSON.stringify(data));
  },
  onError(err) {
    Android.onScanError(err);
  },
});
```

```kotlin
// In your Activity / Fragment
webView.addJavascriptInterface(object {
    @JavascriptInterface
    fun onScanComplete(json: String) {
        // Parse json and use result on main thread
    }
    @JavascriptInterface
    fun onScanError(message: String) { }
}, "Android")
```

#### Flutter

Declare a `JavascriptChannel` in your `WebViewController`, then call it from `onComplete`:

```js
// On your integration page
QuadraScan.startScan({
  onComplete(data) {
    ScanBridge.postMessage(JSON.stringify(data));
  },
  onError(err) {
    ScanBridge.postMessage(JSON.stringify({ error: err }));
  },
});
```

```dart
// In your Flutter widget
WebViewController()
  ..addJavaScriptChannel(
    'ScanBridge',
    onMessageReceived: (JavaScriptMessage message) {
      final data = jsonDecode(message.message);
      // Use data in your app
    },
  );
```

---

### WebView Requirements

| Requirement | Detail |
|---|---|
| HTTPS | Your integration page **must** be served over `https://`. Camera access is blocked on non-secure origins. |
| Camera permission | Grant camera permission to the WebView in your app before or when the user initiates a scan. |
| `allow="camera"` | The SDK sets this on the iframe automatically — no extra config needed on your side. |
| JavaScript enabled | Must be enabled (it is by default on all platforms). |
| Origin | The URL you point the WebView to must match the origin registered for your license key. |

---

## Full Working Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>My Site</title>
</head>
<body>
  <button id="scan-btn">Start Body Scan</button>
  <div id="result"></div>

  <script src="https://quadrascan.fitmatch.ai/sdk/quadrascan.js"></script>
  <script>
    QuadraScan.init({
      apiKey:       'YOUR_LICENSE_KEY',
      teamId:       'YOUR_TEAM_ID',
      club:         'FC Dallas',               // optional
      athleteId:    'YOUR-UUID-HERE',          // optional — caller-supplied UUID
      clientName:   'FC Dallas',               // optional — shown in client TOU screen
      clientTouUrl: 'https://fcdallas.com/terms',   // optional
      clientPpUrl:  'https://fcdallas.com/privacy',  // optional
      scanType:     'both',
      tutorial: {                              // optional — defaults shown
        enabled:          true,               // show tutorial + tutorial buttons
        required:         true,               // first-time users must watch
        requireEveryTime: false,              // skip for returning users via localStorage
      },
    });

    document.getElementById('scan-btn').addEventListener('click', () => {
      QuadraScan.startScan({
        onComplete(data) {
          const m = data.measurements;

          function fmt(category, key) {
            const e = m[category]?.measurements?.[key];
            if (!e || e.value == null) return '—';
            if (e.unit === 'm')  return (e.value * 39.3701).toFixed(1) + '"';
            if (e.unit === 'kg') return (e.value * 2.20462).toFixed(1) + ' lbs';
            return e.value + (e.unit ? ' ' + e.unit : '');
          }

          document.getElementById('result').innerHTML = `
            <p>Scan ID: ${data.scanId}</p>
            <p>Age: ${data.athlete.age} | Gender: ${data.athlete.gender}</p>
            <p>Height: ${data.athlete.height} | Weight: ${data.athlete.weight}</p>
            <p>Chest: ${fmt('upper_body_circumferences', 'chest_circumference')}</p>
            <p>Waist: ${fmt('upper_body_circumferences', 'narrowest_waist_circumference')}</p>
            <p>Hips:  ${fmt('lower_body_circumferences', 'hip_circumference')}</p>
          `;
        },
        onError(err) {
          alert('Scan failed: ' + err);
        },
        onCancel() {
          console.log('Scan cancelled.');
        },
      });
    });
  </script>
</body>
</html>
```

---

## Security Notes

- **License keys are intentionally client-visible** — the security model is origin enforcement, not key secrecy (same as Stripe's publishable key).
- Every API request from the iframe is validated against both the license key and the registered origin. Requests from unregistered origins are rejected with HTTP 403.
- Your upstream API credentials (`QUADRASCAN_API_KEY`, `REFLECT_API_KEY`) are server-side only and never sent to the client.
- The iframe requires `allow="camera; microphone"` — the SDK sets this automatically. No other permissions are needed.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Overlay opens but shows "Unable to start scan" | Invalid or missing license key | Check `apiKey` matches the key registered for your domain |
| HTTP 403 on scan API calls | Origin mismatch | Ensure the domain in your license key registration exactly matches `window.location.origin` of the embedding page |
| Camera not working | Page not served over HTTPS, or permission denied | Serve over HTTPS; ensure the user allows camera in the browser prompt |
| `onComplete` fires but measurements are all `null` | Accessing wrong path in `data.measurements` | Use the category + key paths from the table above — values are nested, not flat |
| `onComplete` never fires — overlay shows "Scan Uploaded" | Reflect completed the scan but returned no measurements | Verify the athlete's `dateOfBirth` and `gender` are correct, and that the `teamId` matches the team the athlete is registered under. The user can hit "Start New Scan" to retry without closing the overlay. |
| Second scan fails with missing license key | Navigating away from the iframe before `sessionStorage` is written | Ensure you are on a supported browser; this is handled automatically by the SDK |
| Tutorial keeps showing for returning users | `localStorage` cleared or blocked (e.g. private browsing) | Expected — private/incognito mode does not persist `localStorage`. Use `requireEveryTime: true` where this matters. |
| Tutorial never shows despite `required: true` | The watched flag was set in a previous session | Clear the flag from the browser console: `Object.keys(localStorage).filter(k => k.startsWith('__qs_tutorial_watched')).forEach(k => localStorage.removeItem(k))` |
| Tutorial button not visible | `enabled: false` was passed to `init()` | Set `tutorial: { enabled: true }` or omit the `tutorial` option entirely |

---
---