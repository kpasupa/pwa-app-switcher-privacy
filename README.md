# PWA App Switcher Privacy

Experimental web-only privacy masking for the iOS App Switcher using a renewable CSS transform animation watchdog and configurable resume policies.

## Status

This project currently contains a specification and historical browser experiments. The production library has not been implemented. The owner reported success with the V3 transform approach; broader physical-device validation remains outstanding.

The planned library is framework-independent, client-only, and has zero runtime dependencies. It will support screen and element masking with automatic or developer-controlled resume.

## Project files

- [SPEC.md](SPEC.md): proposed API, implementation stages, limitations, and acceptance criteria.
- [CONVERSATION.md](CONVERSATION.md): discussion history and reported observations.
- [experiments/](experiments/): original standalone HTML proofs of concept, not production code.
- [AGENTS.md](AGENTS.md): development instructions.

Open the experiment HTML files in a browser for local exploration. App Switcher behavior must be tested on physical iOS devices using a top-level page or installed PWA.

## Limitations

Masking is best-effort visual concealment. It does not provide authentication, screenshot or recording prevention, or a guarantee that the operating system will capture a masked snapshot. JavaScript callbacks cannot be relied on while execution is suspended.

The extracted handoff folder (`zip/`) and reverse-engineering output (`rea-output/`) are excluded from Git. All example sensitive data is synthetic.

No license has been selected yet.
