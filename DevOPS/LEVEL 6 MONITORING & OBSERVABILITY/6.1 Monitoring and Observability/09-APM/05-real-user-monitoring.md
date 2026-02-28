# Chapter 5: Real User Monitoring (RUM)

[← Previous: Performance Profiling](04-performance-profiling.md) | [Back to README](../README.md) | [Next: Unit 10 - Instrumentation Guidelines →](../10-Observability-Best-Practices/01-instrumentation-guidelines.md)

---

## Overview

Real User Monitoring (RUM) captures performance data from actual users' browsers and devices. Unlike synthetic monitoring which tests from controlled environments, RUM reveals the true user experience across diverse devices, networks, and geographies.

---

## 5.1 What RUM Measures

```
┌──────────────────────────────────────────────────────────┐
│           PAGE LOAD TIMELINE                             │
│                                                          │
│  Navigation ──► DNS ──► TCP ──► TLS ──► Request ──►    │
│    Start        Lookup   Connect  Handshk  Sent         │
│                                                          │
│  ──► Response ──► DOM Parse ──► Render ──► Interactive  │
│      First Byte    Processing    Paint      Page Ready  │
│                                                          │
│  KEY METRICS:                                            │
│  ├─ TTFB (Time to First Byte)     ████ 200ms           │
│  ├─ FCP (First Contentful Paint)  ████████ 800ms       │
│  ├─ LCP (Largest Contentful Paint)████████████ 1.5s    │
│  ├─ FID (First Input Delay)       █ 50ms               │
│  ├─ CLS (Cumulative Layout Shift) 0.05                  │
│  └─ TTI (Time to Interactive)     █████████████ 2.0s   │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Core Web Vitals

```
┌──────────────────────────────────────────────────────────┐
│  CORE WEB VITALS (Google's UX Metrics)                   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │ LCP — Largest Contentful Paint (Loading)         │   │
│  │ "How fast does the main content appear?"         │   │
│  │                                                   │   │
│  │ Good: < 2.5s  │ Needs Work: 2.5-4s │ Poor: > 4s │   │
│  │ ████████████████│██████████████████│████████████  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │ INP — Interaction to Next Paint (Responsiveness) │   │
│  │ "How fast does the page respond to clicks?"      │   │
│  │                                                   │   │
│  │ Good: < 200ms │ Needs Work: 200-500ms│ Poor: >500│   │
│  │ ████████████████│██████████████████│████████████  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │ CLS — Cumulative Layout Shift (Stability)        │   │
│  │ "Does content jump around while loading?"        │   │
│  │                                                   │   │
│  │ Good: < 0.1  │ Needs Work: 0.1-0.25│ Poor: > 0.25│   │
│  │ ████████████████│██████████████████│████████████  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Google uses Core Web Vitals as a ranking factor!       │
└──────────────────────────────────────────────────────────┘
```

---

## 5.3 RUM Implementation

### Grafana Faro (Open Source)

```html
<!-- Add to HTML <head> -->
<script>
  (function() {
    var script = document.createElement('script');
    script.src = 'https://unpkg.com/@grafana/faro-web-sdk@latest/dist/bundle/faro-web-sdk.iife.js';
    script.onload = function() {
      window.GrafanaFaroWebSdk.initializeFaro({
        url: 'https://faro-collector.example.com/collect',
        app: {
          name: 'my-web-app',
          version: '1.0.0',
          environment: 'production'
        },
        instrumentations: [
          ...window.GrafanaFaroWebSdk.getWebInstrumentations(),
          new window.GrafanaFaroWebTracingWebSdk.TracingInstrumentation()
        ]
      });
    };
    document.head.appendChild(script);
  })();
</script>
```

### Grafana Faro (React SDK)

```javascript
// npm install @grafana/faro-web-sdk @grafana/faro-react
import { initializeFaro, getWebInstrumentations } from '@grafana/faro-web-sdk';
import { TracingInstrumentation } from '@grafana/faro-web-tracing';
import { ReactIntegration } from '@grafana/faro-react';

const faro = initializeFaro({
  url: 'https://faro-collector.example.com/collect',
  app: {
    name: 'my-react-app',
    version: '1.0.0',
    environment: 'production',
  },
  instrumentations: [
    ...getWebInstrumentations({
      captureConsole: true,       // Capture console.error
      captureConsoleDisabledLevels: [], 
    }),
    new TracingInstrumentation(), // W3C traces from browser
    new ReactIntegration(),       // React error boundaries
  ],
});
```

### Datadog RUM

```html
<script>
  (function(h,o,u,n,d) {
    h=h[d]=h[d]||{q:[],onReady:function(c){h.q.push(c)}}
    d=o.createElement(u);d.async=1;d.src=n
    n=o.getElementsByTagName(u)[0];n.parentNode.insertBefore(d,n)
  })(window,document,'script','https://www.datadoghq-browser-agent.com/datadog-rum-v4.js','DD_RUM')
  
  DD_RUM.onReady(function() {
    DD_RUM.init({
      clientToken: '<CLIENT_TOKEN>',
      applicationId: '<APPLICATION_ID>',
      site: 'datadoghq.com',
      service: 'my-web-app',
      env: 'production',
      version: '1.0.0',
      sessionSampleRate: 100,
      sessionReplaySampleRate: 20,  // Session replay for 20%
      trackUserInteractions: true,
      trackResources: true,
      trackLongTasks: true,
      defaultPrivacyLevel: 'mask-user-input'
    });
  })
</script>
```

---

## 5.4 RUM Data Analysis

```
┌──────────────────────────────────────────────────────────┐
│  RUM DASHBOARD LAYOUT                                    │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Page Load Performance (p75)                     │    │
│  │  LCP: 2.1s ✓  │  INP: 150ms ✓  │  CLS: 0.08 ✓│    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  ┌───────────────────┐ ┌──────────────────────────┐     │
│  │ By Page           │ │ By Device               │     │
│  │ /home    1.8s     │ │ Desktop  1.5s            │     │
│  │ /product 2.5s     │ │ Mobile   3.2s     ⚠    │     │
│  │ /checkout 3.1s ⚠ │ │ Tablet   2.1s            │     │
│  └───────────────────┘ └──────────────────────────┘     │
│                                                          │
│  ┌───────────────────┐ ┌──────────────────────────┐     │
│  │ By Country        │ │ JS Errors                │     │
│  │ US     1.5s       │ │ TypeError: null   (1.2k) │     │
│  │ UK     2.0s       │ │ ChunkLoad error   (350)  │     │
│  │ India  3.8s  ⚠   │ │ Network error     (120)  │     │
│  │ Brazil 4.1s  ⚠   │ │                          │     │
│  └───────────────────┘ └──────────────────────────┘     │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │ LCP Trend (last 7 days)                         │    │
│  │  3s ┤                                           │    │
│  │     │     ╱╲                                    │    │
│  │  2s ┤────╱──╲───────────────────────            │    │
│  │     │   ╱    ╲──────────────────────            │    │
│  │  1s ┤──╱                                        │    │
│  │     └──┬──┬──┬──┬──┬──┬──┬───                  │    │
│  │        M  T  W  T  F  S  S                      │    │
│  └─────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

## 5.5 Frontend-to-Backend Trace Correlation

```
┌──────────────────────────────────────────────────────────┐
│  END-TO-END TRACE: Browser → API → Database             │
│                                                          │
│  Browser (RUM)                                           │
│  ├─ Page Load         ████████████████████████ 2.5s     │
│  │  ├─ DNS             █ 50ms                           │
│  │  ├─ TCP+TLS         ██ 100ms                         │
│  │  ├─ Document Load   ████ 200ms                       │
│  │  └─ XHR /api/data   ████████████████ 1.2s           │
│  │     │                                                 │
│  │     │ traceparent header propagated                   │
│  │     ▼                                                 │
│  │  Backend (APM)                                        │
│  │  ├─ api-gateway     ████████████████ 1.1s            │
│  │  │  ├─ auth         ██ 80ms                          │
│  │  │  └─ order-svc    ████████████ 900ms               │
│  │  │     ├─ db-query  ████████ 700ms   ← BOTTLENECK   │
│  │  │     └─ cache-set █ 30ms                           │
│  │  └─ render HTML     ██ 100ms                         │
│  ├─ JS Execution       ████ 300ms                       │
│  └─ Paint              █ 50ms                           │
│                                                          │
│  Value: One trace from user click to database query     │
└──────────────────────────────────────────────────────────┘
```

---

## 5.6 Session Replay

```
┌──────────────────────────────────────────────────────────┐
│  SESSION REPLAY                                          │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  ▶ Session #abc123 — User: john@example.com     │    │
│  │  Duration: 3:45  │  Pages: 5  │  Errors: 2     │    │
│  │                                                  │    │
│  │  [▶ Play]  ━━━━━━━━━━●━━━━━━━━━━━━━━  2:15     │    │
│  │                                                  │    │
│  │  ┌──────────────────────────────────────────┐   │    │
│  │  │                                          │   │    │
│  │  │     (Visual replay of user session -     │   │    │
│  │  │      DOM recording, not video)           │   │    │
│  │  │                                          │   │    │
│  │  │     Shows: clicks, scrolls, form fills,  │   │    │
│  │  │     page navigations, errors, network    │   │    │
│  │  │                                          │   │    │
│  │  └──────────────────────────────────────────┘   │    │
│  │                                                  │    │
│  │  Timeline:                                       │    │
│  │  0:00 - Page /home loaded                       │    │
│  │  0:15 - Clicked "Products"                      │    │
│  │  0:30 - ⚠ JS Error: TypeError                  │    │
│  │  1:00 - Rage click on "Add to Cart" button      │    │
│  │  1:15 - ❌ Network error on /api/cart            │    │
│  │  2:00 - User refreshed page                     │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  Privacy: Input masking, PII redaction required         │
│  Tools: Datadog Session Replay, FullStory, LogRocket   │
└──────────────────────────────────────────────────────────┘
```

---

## 5.7 RUM vs Synthetic Comparison

| Dimension | RUM | Synthetic |
|-----------|-----|-----------|
| Data Source | Real users | Scripted bots |
| Coverage | All pages users visit | Pre-defined test paths |
| Devices | All user devices | Controlled browsers |
| Availability | Only when users active | 24/7 testing |
| Geographic | Wherever users are | Pre-selected locations |
| Use Case | Actual user experience | Baseline & availability |
| Privacy | Must handle PII | No user data |
| Volume | High (every page view) | Low (scheduled tests) |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| High LCP | Large images, slow server | Optimize images, use CDN, improve TTFB |
| High CLS | Dynamic content, ads, fonts | Set explicit dimensions, preload fonts |
| High INP | Heavy JS on main thread | Code splitting, defer non-critical JS |
| RUM data gaps | SDK not loaded on all pages | Ensure SDK is in base template |
| Privacy concerns | Collecting PII | Enable input masking, anonymize IPs |

---

## Summary Table

| Concept | Details |
|---------|---------|
| RUM Purpose | Measure real user experience from their devices |
| Core Web Vitals | LCP (loading), INP (interactivity), CLS (stability) |
| Tools | Grafana Faro (OSS), Datadog RUM, New Relic Browser |
| Session Replay | Visual playback of user sessions |
| Trace Correlation | Browser spans linked to backend traces |
| Privacy | Must mask PII, comply with GDPR/regulations |

---

## Quick Revision Questions

1. **What are the three Core Web Vitals and what does each measure?**
2. **How does RUM differ from synthetic monitoring?**
3. **How is a browser-side trace connected to backend traces?**
4. **What is session replay and what privacy concerns does it raise?**
5. **What are common causes of poor LCP scores and how do you improve them?**

---

[← Previous: Performance Profiling](04-performance-profiling.md) | [Back to README](../README.md) | [Next: Unit 10 - Instrumentation Guidelines →](../10-Observability-Best-Practices/01-instrumentation-guidelines.md)
