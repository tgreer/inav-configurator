# Deployment

## macOS code signing and notarization

macOS builds are signed and notarized when credentials are provided via
environment variables. Without them, builds are produced unsigned (the
previous behaviour), so development and fork-PR builds need no setup.

### Local release builds

```bash
export OSX_SIGN_IDENTITY="Developer ID Application: Your Name (TEAMID)"
export APPLE_KEYCHAIN_PROFILE="your-notary-profile"  # created with: xcrun notarytool store-credentials
yarn make --platform darwin --arch arm64
yarn make --platform darwin --arch x64
```

Notes:

- The signing identity must exist in your keychain (`security find-identity -v -p codesigning`).
- Notarization uploads the app to Apple and waits; this can take minutes to hours.
- Do not run two macOS builds concurrently on one machine: `@electron/packager`
  wipes its shared temp directory on every run (deleting the other build's
  staged app before stapling), and the DMG maker writes to a fixed intermediate
  path (`out/make/INAV Configurator.dmg`), so parallel runs corrupt each other.
  Run architectures sequentially, or isolate them with separate `TMPDIR`s.

### CI (GitHub Actions)

The macOS jobs in `.github/workflows/ci.yml` sign and notarize automatically
when the following repository secrets are configured. If they are absent
(e.g. fork PRs), the signing steps are skipped and builds are unsigned.

| Secret | Content |
|---|---|
| `MACOS_CERT_P12` | base64-encoded Developer ID Application certificate (.p12, with private key) |
| `MACOS_CERT_PASSWORD` | password for the .p12 |
| `MACOS_SIGN_IDENTITY` | e.g. `Developer ID Application: Your Name (TEAMID)` |
| `APPLE_API_KEY_P8` | App Store Connect API key file contents (.p8) |
| `APPLE_API_KEY_ID` | App Store Connect API key ID |
| `APPLE_API_ISSUER` | App Store Connect issuer ID |

To create the credentials:

1. Export the Developer ID Application certificate (with private key) from
   Keychain Access as a `.p12`, then `base64 -i cert.p12` for the secret value.
2. Create an App Store Connect API key (Users and Access → Integrations →
   App Store Connect API, role: Developer) and download the `.p8`.

### Verification

Signed builds are verified in CI (`stapler validate` + `spctl --assess`).
To check a build manually:

```bash
xcrun stapler validate "out/INAV Configurator-darwin-arm64/INAV Configurator.app"
spctl --assess --type execute -v "out/INAV Configurator-darwin-arm64/INAV Configurator.app"
```

Expected output: `accepted`, `source=Notarized Developer ID`.

### Rollback

Signing is fully controlled by the environment variables/secrets above.
Removing them (or the secrets from the repository) reverts to unsigned builds
with no code changes required.
