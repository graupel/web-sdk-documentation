# QuadraScan SDK — Getting Started

The QuadraScan SDK adds a full-body 3D scan to your site with one `<script>` tag, `init()`, and `startScan()`. The flow (camera, pose detection, mesh generation, measurements) runs in a hosted iframe — your page does not run the scan logic.

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
  units:     'imperial',          // optional: 'imperial' (default) | 'metric' — display unit system
  results: {                      // optional — controls which result sections are shown
    showPhv:             true,    // Body Maturation (PHV gauge)
    showBri:             true,    // Body Insights (BRI + WHtR gauges)
    showAnthropometrics: true,    // Anthropometrics tables (Full Body, Circumferences, Breadth)
  },
  tutorial: {                     // optional — controls pre-scan tutorial behavior
    enabled:          true,       // show/hide tutorial and tutorial buttons entirely
    required:         true,       // first-time users must watch before scanning
    requireEveryTime: false,      // if true, ignores localStorage and always shows tutorial
  },
  theme: {                        // optional — re-skin the iframe in your brand
    backgroundColor:        '#0b1320',
    surfaceCardColor:       '#141d2f',
    onSurfaceCardColor:     '#ffffff',
    popupBackgroundColor:   '#1f2533',
    popupTextColor:         '#efefef',
    scanTileBackgroundColor: '#141d2f',
    onScanTileColor:         '#c8c8c8',
    primaryColor:           '#e30613',
    onPrimaryColor:         '#ffffff',
    secondaryColor:         '#465064',
    onSecondaryColor:       '#f6f6f6',
    textColor:              '#ffffff',
    fontFamily:             "'Inter', -apple-system, sans-serif",
    buttonRadius:           '10px',
  },
});
```

**Pre-filling athlete data (optional)**

If you already have the athlete's information, pass an `athlete` object to pre-fill the **Athlete Info** step. Any field you omit is shown to the user during that step.

**TOU → Athlete Info → Scan Type → Camera/Upload → Viewer**

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
    email:      'jane.doe@example.com',  // optional pre-fill
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
  units:       'metric',         // optional: 'imperial' (default) | 'metric'
  results: {                     // optional: hide individual result sections
    showPhv:             false,  // hide Body Maturation (PHV)
    showBri:             true,
    showAnthropometrics: true,
  },
  width:       '1000px',         // optional: max width of the overlay (default: '1000px')
  height:      '90vh',           // optional: max height of the overlay (default: '90vh')
});
```

The SDK trims string fields, normalizes `gender` to lowercase (`Male` and `male` both work),
validates `dob` as a real `YYYY-MM-DD` date, validates `email` format when provided, and always converts `height` and `weight` to 
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
      // Fired only when the user dismissed without completing.
      console.log('User closed the scan without completing.');
    },
    onScanStart() {
      console.log('Scan overlay opened.');
    },
    onPhotoCapture({ pose }) {
      // pose is 'front' | 'back' | 'left' | 'right'
      analytics.track('photo_captured', { pose });
    },
    onUpload() {
      analytics.track('scan_upload_started');
    },
    onClose() {
      // Always fires when the overlay is removed, regardless of how it closed.
      console.log('Scan overlay closed.');
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
| `athlete` | object | No | `{}` | Pre-supplied athlete data used to pre-fill Athlete Info. Only fields that are **not** provided are shown to the user. See the **`athlete` fields** table below. |
| `scanType` | string | No | `'both'` | Controls which option appears on the landing screen. `'self'` = Guided Scan button only (Upload Photos hidden). `'manual'` = Upload Photos button only (Guided Scan hidden). `'both'` = both options shown, user chooses. |
| `tutorial` | object | No | `{ enabled: true, required: true, requireEveryTime: false }` | Controls the pre-scan tutorial shown before Guided Scan and Upload Photos. See the **`tutorial` fields** table and **Pre-scan Tutorial** section below. |
| `showResults` | boolean | No | `true` | Whether to show the measurements screen after the scan completes. If `false`, `onComplete` still fires with the full result payload, and then the overlay closes automatically (no action required by the user). |
| `allowUserEdit` | boolean | No | `true` | When `false`, hides the **Edit** button on the post-scan viewer and blocks the edit modal. Does not affect first-time collection of missing athlete fields. |
| `showUseCredit` | boolean | No | `false` | When `true`, shows the upload credit notice on the viewer screen. |
| `units` | string | No | `'imperial'` | Display unit system for the measurements screen: `'imperial'` (lbs, inches) or `'metric'` (kg, cm). Does not affect the raw values in the `onComplete` payload — API values are always SI. |
| `results` | object | No | all `true` | Controls which result sections are visible when `showResults` is `true`. See the **`results` fields** table below. If all three sections are disabled the empty-state screen is shown. |
| `theme` | object | No | dark default | Re-skins the iframe UI in your brand. All sub-keys are optional and individually overridable. See the **`theme` fields** table below. The theme is frozen at `init()` — to change themes between sessions, call `init()` again with a new theme before the next `startScan()`. |
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
| `email` | string | Optional. Athlete's email address. When provided, pre-fills the record; not collected from the end user in SDK embed mode. |
| `gender` | string | Case-insensitive: `'male'` \| `'female'` \| `'other'`. |
| `dob` | string | Date of birth in strict `YYYY-MM-DD` format (must be a real calendar date). |
| `heightCm` | number | Height in centimeters (e.g. `170`). Converted to ft/in internally. Use instead of `heightFt/In`. |
| `heightFt` | number | Height — feet component (e.g. `5`). Use if `heightCm` is omitted. |
| `heightIn` | number | Height — inches component (e.g. `7`). Defaults to `0` if omitted. |
| `weight` | number | Body weight as a number; combined with `weightUnit`, then normalized to imperial for the API. |
| `weightUnit` | string | Input hint: `'imperial'` (lbs) or `'metric'` (kg). Defaults to `'imperial'`. |

### `tutorial` fields

| Field | Type | Default | Description |
|---|---|---|---|
| `enabled` | boolean | `true` | Shows or hides the tutorial screen and tutorial buttons entirely. Set to `false` to disable the tutorial feature completely. |
| `required` | boolean | `true` | When `true`, first-time users must watch the tutorial before entering a scan. When `false`, the tutorial is bypassed on scan start (users can still watch via the tutorial button). |
| `requireEveryTime` | boolean | `false` | When `true`, the localStorage watched flag is ignored and the tutorial is shown before every scan. Takes precedence over `required`. |

### `results` fields

Controls which sections appear in the measurements screen. All default to `true`. Omitting the `results` option entirely is the same as passing all `true`.

| Field | Type | Default | Description |
|---|---|---|---|
| `showPhv` | boolean | `true` | Show the **Body Maturation** section (PHV gauge). Only rendered when the athlete is ≤ 18, gender is known, and the API returned a PHV value — so hiding it is only relevant for those athletes. |
| `showBri` | boolean | `true` | Show the **Body Insights** section (BRI and Waist-to-Height Ratio gauges). |
| `showAnthropometrics` | boolean | `true` | Show the **Anthropometrics** section (Full Body, Circumferences, and Breadth collapsible tables). |

> If all three are set to `false`, the measurements screen shows the "scan uploaded" empty-state, but `onComplete` still receives the full measurement payload.

### `theme` fields

Each field is optional. Any field you omit falls back to the default — so passing `theme: { primaryColor: '#e30613' }` only changes the primary color, nothing else. Invalid values are silently dropped, so malformed colors never break a scan.

| Field | Type | Default | Description |
|---|---|---|---|
| `backgroundColor` | CSS color | `#000` | Page background. |
| `surfaceCardColor` | CSS color | `#1c1c1e` | Card-shaped panels on the measurements screen and scan flow (for example status messages). Nested rows tint automatically. |
| `onSurfaceCardColor` | CSS color | `#ffffff` | Copy on **`surfaceCardColor`** (measurements section titles, BRI/WHtR cards, Anthropometrics rows). Separate from **`textColor`** — use when the page uses one ink color but the surfaced cards still use **`surfaceCardColor`** (often darker). |
| `popupBackgroundColor` | CSS color | `#333333` | Centered dialogs (alerts, exit confirmation, Manual Scan, …). Pair with **`popupTextColor`**. |
| `popupTextColor` | CSS color | `#ffffff` | Text on **`popupBackgroundColor`**. |
| `scanTileBackgroundColor` | CSS color | `#222222` | Tile background on the landing screen (**Guided Scan** / **Upload Photos**), upload grid empty slots, and the Manual Scan pose strip. |
| `onScanTileColor` | CSS color | `#a8a8a8` | Ink on those tiles: landing icons and labels, pose silhouettes, and **Front** / **Back** / **Left** / **Right** labels. Pair with **`scanTileBackgroundColor`**. |
| `primaryColor` | CSS color | `#eeeeee` | Primary action buttons (**Continue**, **Upload**, **OK**, exit dialog **Cancel**, …). |
| `onPrimaryColor` | CSS color | `#1a1a1a` | Text/icons on **`primaryColor`**. |
| `secondaryColor` | CSS color | `#a78bfa` | Secondary actions (e.g. **Select Photos** on Manual Scan, exit dialog **Exit**). |
| `onSecondaryColor` | CSS color | `#ffffff` | Text/icons on **`secondaryColor`**. |
| `textColor` | CSS color | `#ffffff` | Main text on the scan flow, headers outside the surfaced cards (e.g. measurements **Scan Results** title), and viewer chrome tied to **`backgroundColor`**. Muted helpers use ~60% opacity. |
| `avatarColor` | CSS color | OBJ default | Tint for the 3D avatar mesh in the post-scan viewer. Accepts any valid CSS color (`#e30613`, `rgb(...)`, …). Omit to keep the generated model's default material. |
| `fontFamily` | CSS font stack | system stack | Valid CSS `font-family`. Fonts from the parent page **do not** apply inside the iframe — use web-safe / system stacks. |
| `buttonRadius` | CSS length | `12px` | Primary button corner radius. |

Pick contrasting pairs for `textColor` / `backgroundColor`, **`surfaceCardColor` / `onSurfaceCardColor`**, **`scanTileBackgroundColor` / `onScanTileColor`**, `popup*` together, and primary/secondary button pairs; the SDK does not enforce contrast. Gauges, status strips, validation messaging, camera focus highlight, and the loader use fixed colors. The **Powered by Fit:Match** footer is not themed.

**Example: a few preset themes**

```js
// "Brand red" — most common pattern for sports clubs.
QuadraScan.init({
  apiKey: '...', teamId: '...',
  theme: {
    backgroundColor:        '#0b1320',
    surfaceCardColor:       '#141d2f',
    onSurfaceCardColor:     '#ffffff',
    popupBackgroundColor:   '#1f2533',
    popupTextColor:         '#efefef',
    scanTileBackgroundColor: '#141d2f',
    onScanTileColor:         '#c8c8c8',
    primaryColor:           '#e30613',
    onPrimaryColor:         '#ffffff',
    secondaryColor:         '#465064',
    onSecondaryColor:       '#f6f6f6',
    textColor:              '#ffffff',
    buttonRadius:           '10px',
  },
});

// "Light mode" — works for daytime-leaning brand palettes.
QuadraScan.init({
  apiKey: '...', teamId: '...',
  theme: {
    backgroundColor:        '#ffffff',
    surfaceCardColor:       '#f4f5f7',
    onSurfaceCardColor:     '#0f172a',
    popupBackgroundColor:   '#fafafa',
    popupTextColor:         '#111111',
    scanTileBackgroundColor: '#e8eaf0',
    onScanTileColor:         '#64748b',
    primaryColor:           '#0f172a',
    onPrimaryColor:         '#ffffff',
    secondaryColor:         '#e8eaf0',
    onSecondaryColor:       '#0f172a',
    textColor:              '#0f172a',
    buttonRadius:           '8px',
  },
});

// "Just tweak the buttons" — keep defaults for everything else.
QuadraScan.init({
  apiKey: '...', teamId: '...',
  theme: {
    primaryColor:   '#5b5bd6',
    onPrimaryColor: '#ffffff',
    buttonRadius:   '999px',     // pill buttons
  },
});
```

---

## Pre-scan Tutorial

The tutorial is a full-screen instructional screen shown before the user enters Guided Scan or Upload Photos. It contains a video (Guided Scan) or a static image (Upload Photos) with a brief description and a **Continue** button.

A **Tutorial** button appears on the scan options screen at all times (unless `enabled: false`) so users can rewatch whenever they want.

The watched flag lives in `localStorage` so it survives closing the tab. In the embed, the key is scoped by parent origin (e.g. `__qs_tutorial_watched:https://your-site.com`) so different embedding sites do not share the same flag.

---

## `startScan()` Options

| Option | Type | Required | Description |
|---|---|---|---|
| `onComplete` | function | **Yes** | Called with the result payload when processing finishes successfully. If `showResults` is `true` (default), the overlay stays open on the measurements screen until the user dismisses it. If `showResults` is `false`, `onComplete` still fires with the payload, then the overlay closes automatically. |
| `onError` | function | No | Called with an error message string if something goes wrong (invalid key, network failure, etc.). The overlay remains open so the user can see the error inside the scan experience. |
| `onCancel` | function | No | Called when the user leaves without completing (parent backdrop click, **Exit** in the iframe, or `QuadraScan.close()`). Before upload, **Exit** shows a confirmation dialog; after a successful upload, **Exit** closes immediately. **Not fired if the scan already completed** (`onComplete` ran). |
| `onScanStart` | function | No | Called immediately when the scan overlay is opened, before the iframe loads. |
| `onPhotoCapture` | function | No | Called each time a photo is confirmed, with `{ pose }` where `pose` is `'front'`, `'back'`, `'left'`, or `'right'`. |
| `onUpload` | function | No | Called when the user clicks the upload button, before the HTTP request completes. No arguments. |
| `onClose` | function | No | Called every time the overlay is fully removed, regardless of how it closed (complete, cancel, or error). |

---

## `close()` Method

```js
QuadraScan.close();
```

Programmatically closes the overlay. Fires `onCancel` (if the scan was not already completed) followed by `onClose`.

Inside the iframe, the top-right **Exit** control uses the same confirmation rules as above (skipped once the scan has been uploaded to Reflect).

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

The `onComplete` payload always contains raw API values in SI units, regardless of the `units` display option. The `units` option only affects what is rendered inside the iframe measurements screen.

| `unit` value | Meaning |
|---|---|
| `"m"` | Meters — multiply by `39.3701` for inches, or by `100` for cm |
| `"kg"` | Kilograms — multiply by `2.20462` for lbs |
| `"lb"` / `"lbs"` | Pounds (user input) |
| `"ft"` | Feet (user input) |
| `"in"` | Inches (user input) |
| `"yr"` | Years (e.g. estimated age at Peak Height Velocity) |
| `""` | Unitless ratio or index (e.g. BMI, BRI, WHtR) |

### Accessing Common Measurements

All circumference and breadth values are in meters. Example helper that respects an imperial/metric preference:

```js
function formatMeasurement(entry, units = 'imperial') {
  if (!entry || entry.value == null) return '—';
  const { value, unit } = entry;
  const metric = units === 'metric';
  if (unit === 'm')  return metric ? `${(value * 100).toFixed(1)} cm` : `${(value * 39.3701).toFixed(1)}"`;
  if (unit === 'kg') return metric ? `${value.toFixed(1)} kg`          : `${(value * 2.20462).toFixed(1)} lbs`;
  if (unit === 'lb' || unit === 'lbs')
                     return metric ? `${(value / 2.20462).toFixed(1)} kg` : `${value} lbs`;
  return value + (unit ? ' ' + unit : '');
}

const m = data.measurements;
const chest = m['upper_body_circumferences']?.measurements?.['chest_circumference'];
const waist = m['upper_body_circumferences']?.measurements?.['narrowest_waist_circumference'];
const hips  = m['lower_body_circumferences']?.measurements?.['hip_circumference'];

console.log('Chest:', formatMeasurement(chest, 'imperial')); // e.g. "39.1\""
console.log('Chest:', formatMeasurement(chest, 'metric'));   // e.g. "99.3 cm"
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
      units:        'imperial',                // optional: 'imperial' (default) | 'metric'
      results: {                               // optional — all true by default; omit to show everything
        showPhv:             true,
        showBri:             true,
        showAnthropometrics: true,
      },
      tutorial: {                              // optional — defaults shown
        enabled:          true,               // show tutorial + tutorial buttons
        required:         true,               // first-time users must watch
        requireEveryTime: false,              // skip for returning users via localStorage
      },
      allowUserEdit: true,                    // optional (default: true) — set false to hide Edit on viewer
      showUseCredit: false,                   // optional (default: false) — set true to show upload credit notice
    });

    document.getElementById('scan-btn').addEventListener('click', () => {
      QuadraScan.startScan({
        onScanStart() {
          document.getElementById('scan-btn').disabled = true;
        },
        onPhotoCapture({ pose }) {
          console.log('Photo accepted:', pose); // 'front' | 'back' | 'left' | 'right'
        },
        onUpload() {
          console.log('Upload started by user.');
        },
        onComplete(data) {
          const m = data.measurements;

          // The payload always contains raw SI values. Format for display yourself,
          // or mirror the units option you passed to init().
          function fmt(category, key, units = 'imperial') {
            const e = m[category]?.measurements?.[key];
            if (!e || e.value == null) return '—';
            const metric = units === 'metric';
            if (e.unit === 'm')  return metric ? `${(e.value * 100).toFixed(1)} cm`  : `${(e.value * 39.3701).toFixed(1)}"`;
            if (e.unit === 'kg') return metric ? `${e.value.toFixed(1)} kg`           : `${(e.value * 2.20462).toFixed(1)} lbs`;
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
          // Fires only if the scan was not completed.
          console.log('Scan cancelled by user.');
        },
        onClose() {
          // Always fires — re-enable the button regardless of outcome.
          document.getElementById('scan-btn').disabled = false;
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
| Measurements screen is blank / shows "Scan Uploaded" but scan succeeded | All result sections disabled via `results` config, or enabled sections have nothing to render (e.g. PHV hidden for athletes over 18) | Expected when hiding UI — `onComplete` still receives the full payload. To show measurements in the iframe, enable at least one section that applies to the athlete, or omit the `results` option entirely. |
| Measurements screen still shows imperial values despite `units: 'metric'` | `units` option not passed to `init()` | Ensure `units: 'metric'` is set in the `QuadraScan.init()` call before `startScan()`. Note: the `onComplete` payload always contains raw SI values regardless of `units`. |
| Theme isn't applied — iframe shows defaults | Theme passed to `startScan()` instead of `init()`, or all values were invalid | The `theme` option lives on `init()`, not `startScan()`. Invalid color/length values are dropped silently — verify each value with `CSS.supports('color', '<your value>')` (or `CSS.supports('border-radius', …)` for `buttonRadius`). |
| Theme applied but my web font doesn't load | Web fonts loaded on the parent page do not propagate into the iframe | Choose a system font stack for `fontFamily` (e.g. `"-apple-system, BlinkMacSystemFont, 'Inter', 'Segoe UI', sans-serif"`), or use a font your end users are likely to have installed. |
| Button text is unreadable on the primary color | `onPrimaryColor` doesn't contrast with `primaryColor` | The SDK does not enforce contrast. Pair a dark `primaryColor` with `onPrimaryColor: '#ffffff'`, or a light `primaryColor` with `onPrimaryColor: '#1a1a1a'`. |
| Measurements card copy is unreadable (dark-on-dark inside BRI/WHtR cards) | `textColor` matches the sticky header/page, but **`surfaceCardColor`** is still a dark inset panel | Pass **`onSurfaceCardColor`** for copy that sits **on** those panels (`#ffffff` on dark teal, `#0f172a` on light gray `#f4f5f7`, …). |
| Second scan fails with missing license key | Navigating away from the iframe before `sessionStorage` is written | Ensure you are on a supported browser; this is handled automatically by the SDK |
| Tutorial keeps showing for returning users | `localStorage` cleared or blocked (e.g. private browsing) | Expected — private/incognito mode does not persist `localStorage`. Use `requireEveryTime: true` where this matters. |
| Tutorial never shows despite `required: true` | The watched flag was set in a previous session | Clear the flag from the browser console: `Object.keys(localStorage).filter(k => k.startsWith('__qs_tutorial_watched')).forEach(k => localStorage.removeItem(k))` |
| Tutorial button not visible | `enabled: false` was passed to `init()` | Set `tutorial: { enabled: true }` or omit the `tutorial` option entirely |
