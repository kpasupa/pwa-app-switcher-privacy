# Conversation handoff — chronological reconstruction

Date: November 25, 2026 (conversation context). This is a faithful condensed reconstruction of the project discussion, not a verbatim export of every message or tool result. Direct user observations are distinguished from assistant proposals. Read SPEC.md for the consolidated engineering contract.

| Stage | User request / observation | Discussion outcome |
|---|---|---|
| Origin | HTML keyframe animation for PWA privacy. Stopwatch https://codepen.io/_Billy_Brown/pen/nLxKpO stops when swiping up, catches up rapidly on return. Hypothesis JS cannot execute while Safari closed/backgrounded. | rAF can suspend while elapsed time advances; catch-up does not prove missed callbacks replay. Proposed renewable CSS watchdog, not an OS state detector. |
| V1 | Need proof of concept. | Full-page opaque overlay using opacity keyframes, 500ms lease renewed every 100ms; lifecycle and gap fallbacks. |
| V2 | Add reset button to repeat testing. | Reset/unlock on mask, lifecycle switch. Comeback lock worked, App Switcher masking did not. |
| V3 | User observed CSS keyframes continue on swipe-up. | Dedicated transform overlay, translate3d from 110% to 0, steps(1,end) forwards, 150ms duration, rAF renewals, rotating CSS probe and JS counter. Both optional checkboxes disabled by default. |
| V3 result | User reports it works with neither checkbox checked. | Lifecycle checkbox triggers JS lock on hidden/pagehide; comeback checkbox forces mask on detected gap; unchecked still stops renewal on expired gap. Keep both off for isolated test. |
| Desktop issue | PC locks, frame gaps 80–120ms. | 150ms is aggressive; occasional delays and forced synchronous layout possible causes. Suggested 300ms test. |
| V4 | Dynamically support different PCs. | P95 adaptive lease, clamp 100–1000ms, 3x factor, 30 calibration samples, rolling 60. Historical HTML provided and embedded preview. |
| Visibility comparison | User asked to try Page Visibility API. | Independent visibility-only overlay with event log and reset; optional auto unlock. |
| Method B | User requested opacity watchdog preview then opacity + visibility only. | Separate versions; user reports Method B or visibility-only does not mask App Switcher. Independent opacity watchdog not unambiguously tested; do not conflate. |
| Methods C/D | Article proposed bottom swipe gesture and window.blur. User requested independent previews. | C detects touch within 20px of bottom and 10px upward movement (also multi-touch); D uses blur only; both reported failed in App Switcher. Embedded frame caveat. |
| Research | Asked if technique already discovered. | Assistant reported no exact match found, closest related JS wake detection, native snapshot concealment, PWA suspension research and heartbeat patent. No exhaustive prior-art verification; not proof of novelty. |
| Naming | Asked repository name, official Apple wording, preferred problem-focused name. | Settled discussion on pwa-app-switcher-privacy; App Switcher official UI term, snapshot developer term. Availability not checked. |
| Comparator | Asked whether Capawesome demo native only. | Native bridge approach is distinct from pure web; verify official plugin docs before publishing claims. |
| Capture | Asked screenshot/recording detection. | No reliable web OS capture detection; separate possible signal research, not project scope. |
| Hosting | Asked OpenAI temporary website. | Embedded preview differs from standalone; suggested GitHub Pages, no deployment performed. |
| Library | Asked simple class, mask only existing object, screen mask, callback while JS paused; unbiased developer perspective. | One controller, declarative .privacy-mask, screen/element modes, custom UI; callbacks only when JS available. |
| Resume | Asked auto reveal or developer-defined login/passcode; observed hidden then visible events consecutively on return. | Hybrid watchdog + visibility + heartbeat; state machine, deduplication, no unconditional unlock on visible, auto/manual resume only, manual developer controls unlock. No authentication coupling. |
| Element masks | Asked specific element demo. | Mark text/images/cards, one shared controller, dynamic DOM observation, existing layout preserved. No per-element auth policy initially. |
| Appearance | Asked blur, black, opacity, custom or guide. | Opaque transform mask first, custom color/CSS vars/overlay; blur and opacity experimental. Provide working default and guide for customization. |
| Packaging | Asked script vs API vs library. | One zero-runtime-dependency library, script tag and npm ESM with same API; TypeScript types. No backend. |
| Settings | Asked page vs global. | Global engine, page-level mode/resume override, element-level appearance. configure/resetConfig. |
| Scope | Asked how users choose full mask vs element. | mode: screen/element independent of resume: auto/manual; proposed screen+auto defaults. |
| Layout | Observed account-number parent box height changed slightly in preview. | Preview substituted fixed-height blocks; library must preserve layout with independent overlay, zero layout shift. Need test wrappers/geometry; pseudo-elements alone not universal. |
| Styling | Asked where to set opacity, solid, blue etc. | Global mask configuration, page override, element CSS variables; solid color default, opacity/blur experimental, no partial concealment as security guarantee. |
| Handoff | Asked full detail and chat session file for local Codex to develop. | This bundle contains consolidated specification, conversation reconstruction, original available HTML experiments and agent instructions. |

## Key direct observations (user wording)

- “I test this ‘start’ and it stop count when i swipe up / When i get back the sec run really fast to reach current time”
- “This method is work for detect comeback / But not for mask in ios multitask app view”
- “One inspection i know that animation keyframe not freeze or interrupt by swipe up”
- “Oh v3 work as expected” and “I did not check those box and its work”
- “i test on pc but its lock why? / fram gap is 80-120”
- “Method b or visibility api only not make privacy mask appear when swipeup(ios multitask)”
- “Both c and d also not work when swipe multitask”
- “visibility api work for detect for coming back (after come back its will trigger 2 even continuly > not visible > visible)”
- “i cannot fix developer resume auth method then it should dynamically like resume: auto or manual”
- “eg. Account number when mask unmask why parent box height is diff abit?”

## Open questions for development

Exact iOS device/version, standalone vs embedded environment for each result, V4 success separately, whether independent opacity watchdog was tested, package name availability, license, exact supported host elements, release defaults, strict CSP distribution, performance thresholds and device test criteria. The owner did not choose a license or authorize publication. Avoid presenting suggested API as implemented.
