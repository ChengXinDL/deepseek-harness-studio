# Agent Note: Upstream DSH 0.1.1-rc.1 adaptation

Status: implemented

English | [中文](2026-08-21-upstream-dsh-0.1.1-rc.1-adaptation.zh.md)

## Problem

Studio branched from the official `dsh-v0.1.0-rc.8` release while the official Harness advanced to `dsh-v0.1.1-rc.1`. Pulling only the release-note features would leave package versions, credential events, Session Projection contracts, Web boot injection, build tooling, and security behavior on incompatible generations.

## Decision

Studio merges the immutable official `dsh-v0.1.1-rc.1` tag rather than following the moving upstream branch. The merge keeps Studio's Desktop version and product surfaces independent while adopting the official core version, native DeepSeek vision model, credential records and renamed reference event, Session Projection changes, structured Web boot injections, Windows-safe pnpm invocation, Bubblewrap PID isolation, and conversation improvements. Studio-specific permission copy, Plugin/Preset/Application surfaces, themes, recovery, and visual routing remain product-owned.

The Desktop packaged-Host deploy root now includes the new authorization package. Studio's visual bridge listens to `credentials/reference-updated`, and its browser fixture uses the official structured `bootInjections()` path. Obsolete generated JavaScript, declarations, and source maps under package `src/` directories were removed because they could shadow the merged TypeScript sources at runtime.

## Verification

The complete host, client, Web, and Desktop build passes at the merged core version. Focused official and Studio tests pass for DeepSeek vision, local visual enhancement, Session/API projections, static serving, Markdown, multiline questions, subagent headers, reference editing, and sandbox policy. The complete GUI lane passes 4,135 tests with one intentional skip; all 42 Desktop test files pass after updating the release fingerprint expectation; the assembled boot and visual-restart snapshots pass in replay mode.

The full Playwright Web matrix was not accepted as evidence on this machine because the newly resolved Playwright version has no downloaded Chromium Headless Shell. The run also exposed and led to removal of the stale generated source artifacts before that environment boundary was recorded. No browser runtime was downloaded and no Desktop installer was built.

## Alternatives considered

**Follow upstream `master`.** Rejected because its head can move during integration and cannot define a reproducible Desktop release baseline.

**Cherry-pick only the release-note commits.** Rejected because the visible changes depend on credential, projection, Web injection, build, and package-version contracts delivered by the full release tag.

**Replace Studio with the official tree.** Rejected because that would remove the Desktop control plane and the Plugin Center, Plugin Discovery, Preset Square, Application Center, themes, recovery, and visual-provider extensions.

## Consequences

Studio identifies its Harness core as `0.1.1-rc.1` while the Electron application identifies as `0.1.0-rc.15`. Native DeepSeek vision bypasses compatible visual providers; text-only models can still use Studio's verified cloud or self-hosted visual routes. Every package release restages and verifies the Desktop Host closure before publishing platform assets.
