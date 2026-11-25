# PWA App Switcher Privacy — engineering handoff

Status: proposed library specification, based on a working user-reported proof of concept. Not a released package. Project owner intends to use Codex locally to develop it.

## 1. Goal and scope

Provide best-effort visual concealment of sensitive web content in the iOS App Switcher without a native bridge. Use CSS transform keyframes as a renewable visual lease: the mask is opaque by default, an animation moves it out of the protected area at the start and returns it at expiry, and JavaScript continuously renews the lease. When callbacks stop, the already-scheduled animation may finish independently. A separate controller reconciles lifecycle signals on return. The library is framework-independent and client-only; no telemetry, network service, authentication implementation, Capacitor, Cordova or native code.

The official Apple term is App Switcher; app snapshot is developer terminology. “App Switcher privacy” is descriptive, not an official API. Suggested repo/package name `pwa-app-switcher-privacy` (availability not verified). Suggested description: Experimental web-only privacy masking for the iOS App Switcher using a renewable CSS animation watchdog and configurable resume policies.

## 2. Evidence and limitations

The owner reports V3 transform masking worked with both lifecycle and comeback checkboxes off. The conversation later refers to V3/V4 as successful, but there is no recorded device/browser matrix or separately confirmed V4 result. Desktop frame gaps of 80–120 ms and unwanted locking were reported with the 150 ms threshold. Visibility-only overlay, opacity plus visibility, bottom-edge touch and window.blur did not mask the iOS multitasking view in the owner's tests. Some tests used embedded previews; these are not equivalent to a top-level Safari page or installed PWA. Owner observed CSS keyframes continuing during swipe-up and hidden/visible events arriving close together after returning. Exact ordering, iOS version, model, snapshot timing and compositor behavior are not independently measured. Independent opacity-keyframe watchdog is not established as failed or successful. No claim of novelty or patentability; prior search assertions are not a formal prior-art review.

An app switcher may show a live surface or a previously captured snapshot. The OS can capture before lease expiry, and WebKit can suspend rendering or discard layers. A CSS mask is not access control, DOM data removal, screenshot prevention or screen recording detection. Native capture notifications are unavailable to ordinary PWAs. A CSS animation may mask while JS is suspended; callbacks, logout, network requests and authentication logic cannot be relied on until JS resumes. Server-side session security belongs to the host app. Fail-closed means the library's intended local rendering/state default, not an OS guarantee.

## 3. Proposed public contract (not implemented)

Use one controller per document. Global defaults, route/page overrides, element-level appearance. Proposed default `mode: 'screen'`, `resume: 'auto'` for visual-only convenience (manual explicitly selected for auth-sensitive pages). No per-element resume policies in V1. Choose mode explicitly in examples.

```html
<link rel="stylesheet" href="./privacy.css">
<script defer src="./privacy.min.js"></script>
<span class="balance privacy-mask">$25,000</span>
<script defer src="./app.js"></script>
```

```js
// app.js, after DOM available
const privacy = Privacy.init({
  mode: 'element', // screen | element
  selector: '.privacy-mask',
  resume: 'manual', // auto | manual
  watchdog: { adaptive: true, minTimeout: 100, maxTimeout: 1000, safetyFactor: 3 },
  lifecycle: { visibility: true, blur: false },
  mask: { color: '#171717', borderRadius: '0px' },
  onResume: async (context) => {
    const ok = await appAuthenticate({ signal: context.signal });
    if (ok && !context.signal.aborted) privacy.unlock(context.interruptionId);
  }
});
```

`appAuthenticate` is an application-defined placeholder. The token argument on unlock is a proposed safety improvement over earlier examples. No bundled passcode, login, biometric UI or server calls. `resume: auto` reveals only after a confirmed eligible return; `manual` keeps masks until explicit unlock. Manual means developer controlled, not necessarily user interaction. `onResume` fires for both modes once per detected interruption episode, not per event. Do not await arbitrary callbacks in auto mode or interpret return values as authorization. Callback exceptions must not unlock manual mode. Expose `onLock` only as JS-observed notification, not exact visual masking timestamp. Provide `onStateChange` if useful. `onResume` context: interruptionId, interruptionMs (estimated, not actual background duration), signals (visibility/heartbeat/pagehide/pageshow as supported), visibilityState, timeoutMs, AbortSignal. Snapshot context; document semantics if later signals arrive.

Proposed methods: `Privacy.init(options)` starts singleton; `lock()` explicit persistent lock; `unlock(interruptionId?)` rejects hidden/stale authorization; `configure(overrides)` atomic page override; `resetConfig()` restores initial defaults; `refresh()` rescan; `getState()` JS logical state; `getMetrics()` diagnostic timing; `destroy()` cleanup. Consider token-required unlock in async manual flow; tokenless unlock reserved for intentional current-state UI action. Idempotent initialization or explicit error; do not silently create duplicate loops. `destroy()` is explicit removal of protection; document whether it keeps shields until caller opts out. Do not let config changes unlock already locked content. Avoid a stop() API unless safe semantics are documented (stop renewal should mask, not reveal).

ESM usage `import { Privacy } from 'pwa-app-switcher-privacy'`; classic script exposes `window.Privacy` (consider more specific global to avoid collisions; owner has not decided). Type declarations, no runtime dependencies, optional separate CSS for strict CSP and pre-init concealment. npm/CDN distribution planned, not published. No invented CDN URL.

## 4. Global/page/element settings

One shared heartbeat. App-level timing and lifecycle settings, global mode/resume and appearance defaults. Page overrides via configure/resetConfig; host router controls when, not library route detection. On route transitions apply more protective configuration before inserting sensitive content. No automatic unlock when switching manual to auto. Multi-page navigation creates new document; no cross-document authorization persistence by default. SPA navigation should not create duplicate controllers. Page overrides replace/reset predictably; document merge rules, e.g. nested option merge against initial defaults and validate atomically.

Screen mode generates an opaque viewport overlay with customizable safe content or a developer-owned overlay element. Element mode discovers `.privacy-mask` or configurable selector, leaves unmarked elements visible. Support text, card/container, image and dynamic additions; inputs, tables, multiline inline, SVG and shadow DOM need explicit validation or documented exclusions. Avoid silently pretending unsupported nodes are protected. MutationObserver batched registration/removal; synchronous refresh API for callers needing protection before painting; baseline CSS loaded before sensitive content. Prevent flashes during initialization/new registration. Nested matching targets should generally collapse to outermost cover or have explicit semantics.

## 5. Appearance and layout

Use solid fully opaque transform-driven covers first. Black default, custom opaque colors (blue etc.), CSS variables `--privacy-mask-color`, `--privacy-mask-radius`. Config defaults < page override < element CSS vars. Radius default zero or ensure rounded corners cannot reveal data; do not allow translucent covers in privacy-safe configuration. Custom overlay branding/UI is optional; keep motion, z-index and coverage library-controlled. Custom authentication UI should be outside masked content / inside safe overlay, accessible while protected. Distinguish opacity of content (0 hides) vs overlay opacity (1 covers). Blur and opacity watchdog are separate experimental rendering engines; do not advertise as supported until independent device tests. Blur/partial opacity can reveal content. Provide styling guide, not a requirement for developers to implement their own masking engine.

Zero layout shift on lock/unlock is an acceptance criterion. Do not replace text with arbitrary-height blocks, set display:none, or change original transform. Dedicated absolute overlay layer, clipped host region or a carefully designed external overlay registry. Pseudo-elements conflict with host styles and cannot reliably support replaced elements (img/input); blanket `display:inline-block`, `position:relative`, `overflow:hidden` can change layout/containing blocks/clip existing content. Evaluate wrappers vs external geometry-based overlay, including scrolling, resizing, transforms, clipping, multiline inline fragments, stacking contexts, top-layer dialogs and popovers. External overlays require keeping geometry current, adding complexity; wrappers can change flex/grid/table semantics. Start with documented supported host types and require explicit wrapper markup for unsupported cases instead of unsafe automatic DOM rewrites. Measure element and parent bounding boxes before/after. Use inert or equivalent interaction/focus management once JS observes lock; preserve prior attributes and restore safely. CSS-only interval cannot guarantee interaction or accessibility blocking. Mask is decorative by default; accessible lock status/controls must remain reachable. Respect reduced-motion without disabling protection (instant step, no slide effect).

## 6. Watchdog baseline and timing

Historical V4 file in experiments. Core CSS:
```css
@keyframes pap-watchdog {
  from { transform: translate3d(0,110%,0); }
  to { transform: translate3d(0,0,0); }
}
.pap-mask-layer { transform: translate3d(0,0,0); will-change: transform; }
.pap-renew { animation: pap-watchdog var(--pap-timeout,500ms) steps(1,end) forwards; }
```
Full-screen mask fixed inset 0; for elements clip moving mask inside target region. Use namespaced classes/keyframes. Historical renewal removes animation class, forces layout with `void mask.offsetWidth`, re-adds class each rAF. Preserve as baseline, batch across layers (remove all, flush once, add all) only after testing; compare WAAPI currentTime renewal and other optimizations separately. One rAF loop, no unnecessary per-frame DOM diagnostics in release, avoid one timer per target. Compositor promotion hints do not guarantee acceleration. Many layers cost GPU memory; screen mode often cheaper.

Adaptive timeout proposal: collect 30 accepted foreground rAF gaps while masked; rolling window 60; nearest-rank P95; timeout clamp(ceil(3*P95),100,1000) ms; initial 500 ms. These are PoC heuristics, not validated universal defaults. Example 120ms P95 => 360ms. Detect expiry BEFORE sampling/adapting; use last armed timeout/deadline, never newly calculated timeout to classify previous gap. Track monotonic performance.now at actual renewal, not only rAF timestamp; compare Date.now conservatively for system sleep if monotonic clock behavior differs; clock jumps must not silently reveal. Ignore hidden/resume/outlier gaps for training. Reset calibration on interruptions; don't count a hidden-to-visible sample. Calibration must remain masked, and manual mode must not silently bypass auth during recalibration. Limit upward timeout changes, consider periodic rather than per-frame percentile computation, quantify tradeoff: more time reduces false locks but can miss snapshot. Severe CPU stall indistinguishable from suspension. If insufficient calibration, remain protected or use explicit documented fallback. No arbitrary timeouts advertised as safe.

Historical V4 restarts every frame, calculates/sorts on each accepted frame, updates DOM counters continuously, and can resume automatically after calibration. It lacks episode state, stale auth handling, production cleanup, dynamic elements, and robust hidden guards in the active loop. Treat as experiment, not drop-in library. `offsetWidth` may itself worsen frame gaps. No measured baseline performance supplied.

## 7. Lifecycle/state machine

Inputs: heartbeat deadline/gap (primary inference), visibilitychange (fallback and return hint), optionally pagehide/pageshow for bfcache/navigation; blur off by default and experimental. Do not depend on visibility events for App Switcher masking. No bottom gesture required. Never renew while document.hidden. Reconcile state before renewing after any gap. Read current document visibility, not assumed historical event values. A queued event may report current state rather than its original transition. `hidden` should explicitly protect when available; `visible` is a hint not unconditional unlock. Treat BFCache restore as potential interruption, test browser behavior. No unload dependency.

States: initializing/calibrating (masked), active (lease renewed), protected (JS-observed or explicit lock), resuming (reconciling signals), awaiting-unlock (manual), destroyed. Visual mask can precede JS logical state; getState cannot report OS snapshot. Separate explicit lock from interruption lock; auto resume must not undo explicit lock. On first interruption allocate generation id; force stable protected state before any renewal. Duplicate visibility and gap signals coalesce into episode; choose/test a bounded foreground stabilization window (e.g. consecutive healthy frames and short settling period), not a promise of exact dedup across arbitrary delays. No visibility event required for heartbeat-only recovery. Do not dispatch resume while hidden. In manual awaiting state continue monitoring heartbeats/visibility even though renewal stopped. A subsequent interruption invalidates pending auth (AbortController + generation check); prevent async callback from unlocking later episode. Handle callback exceptions, rejected promises, reentrancy, route changes/destroy, explicit lock during resume. On auto eligible return resume safely after reconciliation; on manual emit once and stay masked until valid unlock. Treat background duration as estimate. Design deterministic event ordering with tests before implementation.

## 8. Project layout and delivery

Suggested structure: src/index.ts, controller.ts, watchdog.ts, timing.ts, lifecycle.ts, masks/screen.ts, masks/element.ts, styles.css, types.ts; examples/screen.html, elements.html, manual.html, spa.html; experiments/ historical controls; tests/unit and browser; README.md, package.json, tsconfig, build scripts, LICENSE (owner must choose license; do not assign automatically), dist ignored or release policy documented. Build ESM, browser IIFE/minified, type declarations and CSS. Dev tools may use TypeScript, bundler and browser test runner; runtime dependency count zero. Support SSR import without touching DOM until init. Consider strict CSP (external stylesheet, no eval), cross-origin iframes out of scope, no analytics. Keep namespaced CSS, clean up listeners/observers/rAF/layers, restore host state. No package publishing or remote creation without owner approval.

Implementation order: (1) reproduce baseline screen demo, (2) pure timing/state reducer with unit tests, (3) screen controller + hybrid resume, (4) element overlays and dynamic targets, (5) page settings/styling, (6) build packaging/docs, (7) physical-device validation, (8) optional optimizations/experimental engines. Prioritize proven screen engine and correctness over fancy effects.

## 9. Acceptance and test plan

Unit tests with injected clock/scheduler: percentile boundaries, invalid config, deadline comparison using previous armed timeout, calibration/hidden sample filtering, missed frames, long gaps, clock change, manual stop, explicit lock persistence, delayed hidden/visible sequences in different orders, duplicate signals, gap without visibility, foreground event without gap, repeated interruption during auth, stale token/aborted signal, callback failure, destroy and config changes. No old authorization may reveal newly locked content. No auto reveal while hidden. Tests must distinguish actual CSS mask state from simulated controller state.

Browser tests: mask expiry when renewal stops (not merely calling JS lock); actual animation computed transform; no flash on resume/initialization/dynamic insertion; auto and manual modes, custom UI controls, route changes, cleanup; compare bounding boxes and CLS on inline text, flex/grid, images, tables, responsive sizes, scroll; high z-index/top-layer limitations; keyboard and accessibility; reduced motion; CSP; 1/10/100 masks benchmark CPU, GPU/layer count if available, frame gaps and false locks. Headless WebKit is useful but not iOS App Switcher evidence.

Physical iOS matrix: record model, iOS version, Safari tab vs installed home-screen PWA, standalone top-level URL vs embedded preview, timeout, P95, mode, mask count, method and fallback flags. Test partial slow swipe held, release into app switcher without returning, quick swipe, switch app, return, lock/unlock device, Control/Notification Center, rapid repeated transitions, rotation, low-power mode. Check preview immediately and after hold, first frame on return, any flash, callback count, auth cancellation. Repeat each scenario multiple times; report pass rate and false locks. Screen recordings or external camera as appropriate (recording may affect performance). Use synthetic data. Compare original V3/V4 with baseline and optimized implementation. Desktop Chrome/Edge/Firefox/Safari where available, 60/120Hz and throttled CPU; Android optional, no claims without tests. Record actual measurements; no invented results. Suggested sample record: run ID, build commit, environment, action, timeout, visibility log, gap, mask seen before return?, snapshot exposure?, return flash?, callbacks, notes.

## 10. References for verification

Apple Support: https://support.apple.com/guide/iphone/switch-between-open-apps-iph1a1f981ad/ios
Apple UIKit: https://developer.apple.com/documentation/uikit/preparing-your-ui-to-run-in-the-background
Apple older Q&A: https://developer.apple.com/library/archive/qa/qa1838/_index.html
MDN visibility: https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilitychange_event
MDN rAF: https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame
Original stopwatch: https://codepen.io/_Billy_Brown/pen/nLxKpO
Native comparator: https://github.com/capawesome-team/capacitor-privacy-screen-demo
These are reference links from the discussion or standard docs; recheck current contents if citing in public README. Do not claim first discovery or native-equivalent protection.
