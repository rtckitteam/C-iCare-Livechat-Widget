# C-iCare Livechat Widget — Installation Guide

How to install the C-iCare livechat widget on your website or in your app.

The widget is a floating chat button that connects your visitors to C-iCare
agents. Installing it takes **two tags** on a page — no package to install, no
backend changes, and no dependency on any framework.

---

## Contents

1. [What you need](#1-what-you-need)
2. [Quick install](#2-quick-install)
3. [Two modes: anonymous visitor vs signed-in customer](#3-two-modes-anonymous-visitor-vs-signed-in-customer)
4. [Full attribute reference](#4-full-attribute-reference)
5. [Web platform examples](#5-web-platform-examples)
6. [Installing in a mobile app (WebView)](#6-installing-in-a-mobile-app-webview)
7. [What the widget does](#7-what-the-widget-does)
8. [Sessions, storage, and sign-out](#8-sessions-storage-and-sign-out)
9. [Security and privacy](#9-security-and-privacy)
10. [Content Security Policy (CSP)](#10-content-security-policy-csp)
11. [Troubleshooting](#11-troubleshooting)
12. [Go-live checklist](#12-go-live-checklist)

---

## 1. What you need

| Requirement | Notes |
| --- | --- |
| **Widget key** | Your channel token, issued by the C-iCare team. One key = one livechat channel. |
| **Widget URL** | Where `widget.js` and `widget.css` are served from, given to you by the C-iCare team. |
| **A page served over HTTP/HTTPS** | The widget is an ES module — it does not run from `file://`. |

Throughout this guide the widget URL is written as `https://cdn.example.com/widget/`.
**Replace it with the URL you were given.**

### The files being served

```
widget.js      the application bundle (ES module)
widget.css     the widget styles
assets/        fonts and images
```

`assets/` is resolved automatically, relative to `widget.js`, so you never
reference it yourself — just make sure the folder is served next to `widget.js`.

### Ready-to-run examples

Two example files sit in the same folder as this guide:

| File | For |
| --- | --- |
| `example-guest.html` | Anonymous visitors — `data-widget-key` only |
| `example-logged-in.html` | Signed-in customers — identity passed from the page |

Open them through a web server (not by double-clicking), paste your widget key
into the test box on the page, and press **Load widget**.

---

## 2. Quick install

Paste inside `<head>`:

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.css">
```

Paste just before `</body>`:

```html
<div id="livechat-app"
     class="livechat-widget"
     data-widget-key="YOUR_WIDGET_KEY"></div>

<script type="module" crossorigin
        src="https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.js"></script>
```

That's it. The chat icon appears in the bottom-right corner of the page.

### Four things that must not change

| Rule | Why |
| --- | --- |
| `id="livechat-app"` | The widget looks for this id. Only **one** per page. |
| `class="livechat-widget"` | The widget's styles are written against `#livechat-app.livechat-widget`. Without it, your page layout can break. |
| `type="module"` | The bundle is an ES module. Without this attribute the browser refuses to run it. |
| A valid `data-widget-key` | Without one, the widget does not render at all. |

Where you put the `<div>` does not affect the result — the widget is always
`position: fixed` in the corner of the screen. Placing it before `</body>` is
simply the safest habit.

---

## 3. Two modes: anonymous visitor vs signed-in customer

The widget behaves in one of two ways, and what decides it is **whether the page
passes `data-email` or `data-phone`**.

### Mode A — anonymous visitor (pre-chat form)

The page does not know who the visitor is, so the widget asks first.

```html
<div id="livechat-app"
     class="livechat-widget"
     data-widget-key="YOUR_WIDGET_KEY"></div>
```

The visitor sees a form asking for **Name + Email**, and the chat starts when
they submit it.

`data-form` controls which fields are asked for:

| `data-form` | The form asks for |
| --- | --- |
| *(omitted)* | Name, Email |
| `name,email,phone` | Name, Email, Phone |
| `name,phone` | Name, Phone |
| `email` | Email only |

The form **must** ask for at least `email` or `phone` — a chat cannot start
without one of those contacts.

### Mode B — signed-in customer (no form)

The page already knows the customer, so there is nothing to ask.

```html
<div id="livechat-app"
     class="livechat-widget"
     data-widget-key="YOUR_WIDGET_KEY"
     data-name="Budi Santoso"
     data-email="budi@example.com"
     data-phone="081234567890"></div>
```

The customer sees a single **Start Live Chat** button. Tapping it opens a session
with that identity straight away.

### When the form disappears

| What the page passes | What the user sees |
| --- | --- |
| *(nothing)* | Pre-chat form |
| `data-name` only | Pre-chat form, name pre-filled |
| `data-email` | **Start Live Chat** button |
| `data-phone` | **Start Live Chat** button |
| `data-email` + `data-phone` + `data-name` | **Start Live Chat** button |

`data-name` alone is not enough: a name is a label, not a contact. In Mode B,
`data-form` is ignored.

Empty values count as absent — `data-email=""` is the same as leaving the
attribute out. So one template can serve both anonymous visitors and signed-in
customers: print each attribute only when you actually have a value for it.

---

## 4. Full attribute reference

All attributes go on the `<div id="livechat-app">` element.

| Attribute | Required | Values | Notes |
| --- | --- | --- | --- |
| `data-widget-key` | **Yes** | channel token | Issued by the C-iCare team |
| `data-pos` | No | `right` (default), `left` | Which corner the chat icon sits in |
| `data-form` | No | any of `name`, `email`, `phone` | Fields the pre-chat form asks for. Default: `name,email`. Ignored in Mode B |
| `data-name` | No | customer name | A label only. Does not trigger Mode B |
| `data-email` | No | customer email | Triggers Mode B |
| `data-phone` | No | customer phone | Triggers Mode B |

### A note on language

The widget's language follows your **channel setting on the server**, not an
attribute on the page. Ask the C-iCare team to change your channel's language.
Available languages: English, Indonesian, Japanese, Korean, Russian, and Chinese.

### Colours, logo, bot name, and greeting

All of these are configured per channel on the C-iCare side, not in your page:

- the widget's primary colour,
- the bot's name and avatar,
- the opening greeting message,
- the footer text, link, and logo,
- the satisfaction survey (CSAT) shown after a chat is closed.

Changes take effect on the next page load — you do not need to change any code or
reinstall anything. Send change requests to the C-iCare team.

---

## 5. Web platform examples

### Static HTML / WordPress / Shopify

Paste the [quick install](#2-quick-install) snippet into your theme's footer
template. In WordPress: **Appearance → Theme File Editor → footer.php**, just
before `</body>`. Or use any script-injection plugin (for example *Insert Headers
and Footers*) so the snippet survives theme updates.

For signed-in WordPress users:

```php
<?php $u = wp_get_current_user(); ?>
<div id="livechat-app"
     class="livechat-widget"
     data-widget-key="YOUR_WIDGET_KEY"
     <?php if ( $u->ID ) : ?>
       data-name="<?php echo esc_attr( $u->display_name ); ?>"
       data-email="<?php echo esc_attr( $u->user_email ); ?>"
     <?php endif; ?>></div>

<script type="module" crossorigin
        src="https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.js"></script>
```

### Laravel / Blade

```blade
<div id="livechat-app"
     class="livechat-widget"
     data-widget-key="{{ config('services.cicare.widget_key') }}"
     @auth
       data-name="{{ auth()->user()->name }}"
       data-email="{{ auth()->user()->email }}"
       data-phone="{{ auth()->user()->phone }}"
     @endauth></div>

<script type="module" crossorigin
        src="https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.js"></script>
```

### React / Next.js

The widget reads its attributes **once**, when the bundle runs, so they must be in
place before the script is loaded.

```jsx
import { useEffect } from 'react';

export default function LivechatWidget({ user }) {
  useEffect(() => {
    const el = document.getElementById('livechat-app');
    if (!el || document.getElementById('cicare-widget-script')) return;

    el.setAttribute('data-widget-key', process.env.NEXT_PUBLIC_CICARE_KEY);
    if (user?.name)  el.setAttribute('data-name',  user.name);
    if (user?.email) el.setAttribute('data-email', user.email);
    if (user?.phone) el.setAttribute('data-phone', user.phone);

    const css = document.createElement('link');
    css.rel = 'stylesheet';
    css.href = 'https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.css';
    document.head.appendChild(css);

    const script = document.createElement('script');
    script.id = 'cicare-widget-script';
    script.type = 'module';
    script.crossOrigin = 'anonymous';
    script.src = 'https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.js';
    document.body.appendChild(script);
  }, [user]);

  return <div id="livechat-app" className="livechat-widget" />;
}
```

The `document.getElementById('cicare-widget-script')` guard matters: React Strict
Mode runs effects twice in development, and the bundle must not load twice.

> **Important for SPAs:** the identity is read once, at startup. If the user signs
> in *after* the widget is running, the new attributes are not picked up — reload
> the page after sign-in and sign-out. See
> [Sessions, storage, and sign-out](#8-sessions-storage-and-sign-out).

### Vue / Nuxt

```vue
<template>
  <div id="livechat-app" class="livechat-widget"></div>
</template>

<script setup>
import { onMounted } from 'vue';

const props = defineProps({ user: Object });

onMounted(() => {
  const el = document.getElementById('livechat-app');
  el.setAttribute('data-widget-key', import.meta.env.VITE_CICARE_KEY);
  if (props.user?.email) el.setAttribute('data-email', props.user.email);
  if (props.user?.name)  el.setAttribute('data-name',  props.user.name);

  const css = document.createElement('link');
  css.rel = 'stylesheet';
  css.href = 'https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.css';
  document.head.appendChild(css);

  const script = document.createElement('script');
  script.type = 'module';
  script.crossOrigin = 'anonymous';
  script.src = 'https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.js';
  document.body.appendChild(script);
});
</script>
```

### Angular

Simplest route: paste the quick-install snippet directly into `src/index.html`.
For signed-in mode, set the attributes from `AppComponent` before loading the
script, using the same pattern as the Vue example above.

---

## 6. Installing in a mobile app (WebView)

The widget is a web application, so inside a mobile app it runs in a **WebView**.
The pattern is the same on every platform:

1. Host **one HTML page** on your own domain — say `https://your-site.com/chat` —
   containing nothing but the widget.
2. Open that page in a WebView, passing the user's identity from the app.
3. Enable JavaScript and DOM storage on the WebView.

### 6.1 The page the WebView opens

Create a simple page on your server. Example (Laravel, route `/chat`):

```blade
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
  <title>Support</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.css">
  <style>
    html, body { margin: 0; height: 100%; background: #fff; }
  </style>
</head>
<body>
  <div id="livechat-app"
       class="livechat-widget"
       data-widget-key="{{ config('services.cicare.widget_key') }}"
       @auth
         data-name="{{ auth()->user()->name }}"
         data-email="{{ auth()->user()->email }}"
         data-phone="{{ auth()->user()->phone }}"
       @endauth></div>

  <script type="module" crossorigin
          src="https://cdn.jsdelivr.net/gh/rtckitteam/C-iCare-Livechat-Widget@{RELEASE_TAG}/widget.js"></script>
</body>
</html>
```

**Fill the identity from your own server-side session**, not from URL parameters.
If you must pass something through the URL (`/chat?token=...`), pass **your own
session token** and resolve it to an identity on the server — do not put raw
emails or phone numbers in a query string, where anyone can edit them and where
they end up in access logs.

### 6.2 Android — WebView (Kotlin)

```kotlin
val webView: WebView = findViewById(R.id.webview)

webView.settings.apply {
    javaScriptEnabled = true          // required
    domStorageEnabled = true          // required - the chat session lives in localStorage
    mediaPlaybackRequiresUserGesture = false
    loadWithOverviewMode = true
    useWideViewPort = true
}

// Attachments and survey links open with target="_blank".
// Without this handling, tapping them does nothing at all.
webView.settings.setSupportMultipleWindows(true)
webView.webChromeClient = object : WebChromeClient() {
    override fun onCreateWindow(
        view: WebView, isDialog: Boolean, isUserGesture: Boolean, resultMsg: Message
    ): Boolean {
        val transport = resultMsg.obj as WebView.WebViewTransport
        val popup = WebView(view.context)
        popup.webViewClient = object : WebViewClient() {
            override fun shouldOverrideUrlLoading(
                v: WebView, request: WebResourceRequest
            ): Boolean {
                startActivity(Intent(Intent.ACTION_VIEW, request.url))  // open in browser
                return true
            }
        }
        transport.newWebView = popup
        resultMsg.sendToTarget()
        return true
    }

    // Required if you want the attachment (paperclip) button to work.
    override fun onShowFileChooser(
        view: WebView, filePathCallback: ValueCallback<Array<Uri>>,
        params: FileChooserParams
    ): Boolean {
        // Launch your file picker, return the result through filePathCallback.
        return true
    }
}

webView.loadUrl("https://your-site.com/chat")
```

The manifest needs internet permission:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Android notes:

- Use **HTTPS**. Since Android 9, plain HTTP is blocked unless explicitly allowed
  through a network security configuration.
- The device needs a reasonably current Android System WebView — the widget is an
  ES module. The WebView on Android 5.0+, kept updated through the Play Store,
  handles it.
- Do not load the page from `file://` — ES modules are rejected by CORS rules there.

### 6.3 iOS — WKWebView (Swift)

```swift
import WebKit

let config = WKWebViewConfiguration()
config.allowsInlineMediaPlayback = true
// Use a non-persistent data store ONLY if you actually want the chat session
// discarded every time the app closes. The default is already persistent.
config.websiteDataStore = .default()

let webView = WKWebView(frame: view.bounds, configuration: config)
webView.uiDelegate = self
view.addSubview(webView)

webView.load(URLRequest(url: URL(string: "https://your-site.com/chat")!))
```

`target="_blank"` links (attachments and surveys) need handling, or tapping them
has no effect:

```swift
extension ChatViewController: WKUIDelegate {
    func webView(_ webView: WKWebView,
                 createWebViewWith configuration: WKWebViewConfiguration,
                 for navigationAction: WKNavigationAction,
                 windowFeatures: WKWindowFeatures) -> WKWebView? {
        if let url = navigationAction.request.url {
            UIApplication.shared.open(url)   // open in Safari
        }
        return nil
    }
}
```

### 6.4 React Native

Using [`react-native-webview`](https://github.com/react-native-webview/react-native-webview):

```jsx
import { WebView } from 'react-native-webview';
import { Linking } from 'react-native';

export default function ChatScreen() {
  return (
    <WebView
      source={{ uri: 'https://your-site.com/chat' }}
      javaScriptEnabled                       // required
      domStorageEnabled                       // required
      thirdPartyCookiesEnabled
      setSupportMultipleWindows={false}       // route _blank to the handler below
      onShouldStartLoadWithRequest={(req) => {
        // Keep the chat page inside; send everything else to the system browser.
        if (req.url.startsWith('https://your-site.com/chat')) return true;
        Linking.openURL(req.url);
        return false;
      }}
    />
  );
}
```

### 6.5 Flutter

Using [`webview_flutter`](https://pub.dev/packages/webview_flutter):

```dart
final controller = WebViewController()
  ..setJavaScriptMode(JavaScriptMode.unrestricted)   // required
  ..setNavigationDelegate(
    NavigationDelegate(
      onNavigationRequest: (request) {
        if (request.url.startsWith('https://your-site.com/chat')) {
          return NavigationDecision.navigate;
        }
        launchUrl(Uri.parse(request.url));   // url_launcher package
        return NavigationDecision.prevent;
      },
    ),
  )
  ..loadRequest(Uri.parse('https://your-site.com/chat'));
```

DOM storage is on by default in recent `webview_flutter` versions; do not clear
WebView data on app exit if you want chats to be resumable.

### 6.6 WebView requirements at a glance

| Requirement | What breaks without it |
| --- | --- |
| JavaScript enabled | The widget never appears |
| DOM storage / `localStorage` enabled | Chats cannot be resumed after the screen is closed |
| Page served over HTTPS | The ES module bundle fails to load |
| `target="_blank"` handling | Attachments and survey links cannot be opened |
| `onShowFileChooser` (Android) | The attachment button does nothing |
| A modern WebView (Chrome 61+ / iOS 11+) | The ES module bundle is rejected |

---

## 7. What the widget does

Everything below works with no extra configuration on your side.

| Feature | Notes |
| --- | --- |
| **Text chat** | Two-way conversation with an agent, messages arrive in real time |
| **Attachments** | Visitors can send files, up to **5 MB** |
| **Emoji** | Built-in emoji picker in the composer |
| **Bot greeting** | Automatic opening message, configured per channel |
| **Session resume** | An unfinished chat continues when the page is opened again |
| **Satisfaction survey (CSAT)** | Shown after an agent closes the chat — star rating, or a link to your own form |
| **Multilingual** | English, Indonesian, Japanese, Korean, Russian, Chinese |

Allowed attachment types:

```
jpg  jpeg  png  gif  webp  pdf  doc  docx  xls  xlsx  csv  txt  zip
```

SVG is deliberately excluded — it can carry script.

---

## 8. Sessions, storage, and sign-out

The widget stores the chat session in the browser's `localStorage`, under the
`cwidget_` prefix. That is what lets a conversation continue after a reload or
when the visitor moves to another page.

Two consequences you need to handle:

**When a user signs out or switches account**, clear that storage so the previous
user's conversation does not carry over:

```js
Object.keys(localStorage)
  .filter((k) => k.startsWith('cwidget_'))
  .forEach((k) => localStorage.removeItem(k));

location.reload();
```

**When a user signs in**, reload the page. The widget reads `data-*` only once, at
startup, so an identity set after it is running will not be picked up.

The widget uses no cookies and requires no CORS configuration on your side.

---

## 9. Security and privacy

**The widget key is public.** It sits in your page's HTML, so anyone viewing the
source can read it. That is expected and safe for this kind of key — all it does
is identify which channel to use. What you must not do is put any other
credential (an API key, a user token, any secret) into a `data-*` attribute.

**Identity from the page is trusted as-is.** The widget sends `data-email` and
`data-phone` to the server without verifying them, because verifying identity is
your application's job. So:

- Fill those three attributes from **your server-side session**, never from user
  input, URL parameters, or anything else editable on the client.
- Do not pass emails or phone numbers through a user-editable query string — use
  your own session token and resolve it server-side.

**Keep sensitive data out of `data-*`.** Account numbers, national ID numbers and
the like do not belong there. These attributes exist only to identify the customer.

The widget talks to `https://api.omni.c-icare.cc` over HTTPS. No cookies are sent
(`withCredentials` is off), so the widget cannot read your site's login session.

---

## 10. Content Security Policy (CSP)

If your site enforces a CSP, add the directives below so the widget can run.
Replace `https://cdn.example.com` with your widget URL.

```
script-src  'self' https://cdn.example.com;
style-src   'self' 'unsafe-inline' https://cdn.example.com https://fonts.googleapis.com;
font-src    'self' https://fonts.gstatic.com;
img-src     'self' data: https:;
connect-src 'self' https://api.omni.c-icare.cc;
```

What each one is for:

| Directive | Covers |
| --- | --- |
| `script-src` | Loading `widget.js` |
| `style-src` | `widget.css`, the Rubik font from Google Fonts, and the channel colours applied as inline styles |
| `font-src` | The Rubik font files |
| `img-src` | Icons, the bot avatar, and image attachments sent inside the chat |
| `connect-src` | API calls and the real-time message stream (SSE) |

`'unsafe-inline'` on `style-src` is needed because channel colours are applied
through a `style` attribute. If your policy is strict, `style-src-attr
'unsafe-inline'` is enough and is narrower in scope.

The server delivering `widget.js` must also send an `Access-Control-Allow-Origin`
header — ES modules are always fetched in CORS mode, even from the same origin
under a permissive policy.

---

## 11. Troubleshooting

### The chat icon never appears

Open Developer Tools (F12) → **Console** tab and look for messages prefixed
`[cicare-widget]`.

| What the console says | What it means |
| --- | --- |
| `verify-token failed (401)` or `(403)` | The widget key is wrong, expired, or not for this channel |
| `verify-token failed (no response)` | The API is unreachable — check the network, a firewall, or your CSP `connect-src` |
| `Failed to load module script` | The `<script>` tag is missing `type="module"`, or the `widget.js` URL is wrong |
| *(nothing at all)* | The script never loaded — check the URL, ad blockers, and that `<div id="livechat-app">` really exists |

### The page layout breaks

`class="livechat-widget"` is missing from the mount element. That class is what
keeps the widget container from taking up space in your layout.

### The widget appears twice, or stops responding

The bundle was loaded more than once — usually an SPA that injects the script on
every route change, or React Strict Mode running an effect twice. Give the script
tag an id and check for it before appending (see the
[React example](#react--nextjs)).

### The customer's identity is ignored

The `data-*` attributes were set after the bundle had already run. Set the
attributes first, then load `widget.js`.

### The pre-chat form shows even though the customer is signed in

Both `data-email` and `data-phone` are empty. `data-name` on its own does not
remove the form.

### The attachment button does nothing in the mobile app

The WebView has no file-chooser handling. See
[WebView requirements at a glance](#66-webview-requirements-at-a-glance).

### Chats disappear every time the mobile app is reopened

DOM storage is off, or WebView data is being cleared on app exit.

### Opening the example files straight from disk doesn't work

`file://` does not support ES modules. Serve the folder over a simple web server:

```sh
npx serve .
# or
python3 -m http.server 8080
```

---

## 12. Go-live checklist

- [ ] Production widget key in place (not a test key)
- [ ] `widget.js` and `widget.css` point at the production URL from the C-iCare team
- [ ] `class="livechat-widget"` present on the mount element
- [ ] `type="module"` present on the script tag
- [ ] On signed-in pages: `data-email` or `data-phone` filled from the server session
- [ ] `cwidget_` storage cleared on sign-out
- [ ] Your CSP includes all five directives from [section 10](#10-content-security-policy-csp)
- [ ] Tested in Chrome, Safari, and Firefox
- [ ] Tested on a phone — both Android and iOS
- [ ] For mobile apps: JavaScript on, DOM storage on, `target="_blank"` handled
- [ ] A test chat reached an agent, and the agent's reply reached the visitor
- [ ] A test attachment was sent successfully
- [ ] Reloaded the page mid-chat — the conversation continued instead of restarting

---
## Latest Release

Our latest release version is v2.0.1

---
## License

The widget is proprietary software, licensed rather than sold. Your active
C-iCare subscription grants you the right to embed it, unmodified, on properties
you own or operate. Redistribution, resale, and reverse-engineering are not
permitted.

The full terms are in [LICENSE](LICENSE), next to this guide. Section 5 lists the
open-source components bundled inside the widget, which remain under their own
licenses.

---

## Need help?

Include the following so we can look into it quickly:

- the URL of the page the widget is installed on,
- a screenshot of the **Console** tab in Developer Tools,
- the widget key in use (safe to send — the key is public),
- the browser or device, and its version.
