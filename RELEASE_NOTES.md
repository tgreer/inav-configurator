Configurator-only patch release. **No firmware changes** — continues to work with INAV 9 firmware (9.0.x or 9.1.x), same as 9.1.0.

---

## Fixes since 9.1.1

* **CLI** — Fixed MSP status polls leaking into the CLI session when opening the tab, which showed up as garbled output (raw MSP2 frame bytes mixed into the console). #2688
* **CLI** — Stopped deferred MSP retries from firing after CLI mode was already active on a slow or loaded serial link. #2688

---

Since 9.0.1: the 9.1.1 totals (**32 fixes** and **39 new features/enhancements**), plus the items below. For full PR-by-PR detail, see the wiki release notes.

---

## Highlights

### Signed and notarized macOS builds

Official tagged builds are now signed with a Developer ID certificate and notarized by Apple, so Gatekeeper accepts the app instead of blocking it as unsigned. Pull-request and fork CI builds stay unsigned on purpose — signing credentials are not available to PR-controlled build config.

### Tagged release builds

Pushing a version tag (`9.1.3` or `v9.1.3`) builds every platform and **requires** signed, notarized macOS artifacts. Incomplete signing or notarization secrets fail the job instead of shipping an unsigned official build.

The GitHub Release body is taken from `RELEASE_NOTES.md` on the tagged commit.

---

## Localization

No new translations in this patch.

---

**Firmware Release Notes:** https://github.com/iNavFlight/inav/releases/tag/9.1.0
