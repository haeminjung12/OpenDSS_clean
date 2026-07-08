# Worker Report: Package Metadata Hash Fix

Agent: packaging integrity bug-fix worker
Date: 2026-07-07
Assigned Scope: `app/runtime/scripts/package_portable.ps1`, `app/runtime/scripts/check_package.ps1`, `app/runtime/models/`, minimal supporting model-registry packaging reads

## Summary

Reproduced the package metadata failure as a stale source-registry hash, not a packaging copy or metadata canonicalization bug. `app/runtime/models/metadata.json` already hashed to `528ac091764c09cd9c2c6ad2a6ff1e38bb009184a26e7352b71b3a025c30902d`, while `app/runtime/models/model_registry.json` still declared `fa5321dfad900baec23fa6c239a29279e0e8c03fa2e78f0bd679dfb973888d2f` for the same file.

Updated the promoted binary runtime entry in `model_registry.json` to the actual current `metadata.json` SHA256. A fresh portable package built from `C:\Users\goals\Codex\OpenDSS\build-opendss-internal-release` now passes `check_package.ps1`.

## Files Changed

- `app/runtime/models/model_registry.json` - corrected the stale `metadata_sha256` for `app/runtime/models/metadata.json`
- `docs/worker-reports/packaging-blockers-2026-07-07/package-metadata-hash-fix.md` - recorded task results

## Files Read

- `AGENTS.md` - repo workflow and sparse-debug rules
- `graphify-out/GRAPH_REPORT.md` - durable repo context
- `docs/agent-debugging-workflow.md` - sparse workspace and validation flow
- `docs/active-bug-queue.md` - current baseline rules
- `docs/current-state.md` - packaging/runtime status
- `docs/build.md` - accepted build and package commands
- `docs/migration-manifest.md` - package boundary notes
- `docs/agent-results/packaging-readiness-2026-07-07/packaging-readiness-2026-07-07-results.md` - upstream accepted blocker context
- `docs/worker-reports/packaging-readiness-2026-07-07/package-validation.md` - exact failing package evidence
- `docs/worker-reports/packaging-readiness-2026-07-07/portable-package-trainer-assets.md` - current packaging contract
- `docs/agent-results/packaging-blockers-2026-07-07/packaging-blockers-2026-07-07-results.md` - blocker-wave state
- `docs/agent-tasks/packaging-blockers-2026-07-07/package-metadata-hash-fix.md` - assigned task contract
- `app/runtime/scripts/package_portable.ps1` - confirmed packaging only copies required model assets
- `app/runtime/scripts/check_package.ps1` - confirmed package validation uses registry-declared `metadata_sha256`
- `app/runtime/models/model_registry.json` - located stale `metadata_sha256`
- `app/runtime/models/metadata.json` - verified actual current metadata content/hash
- `app/runtime/desktop_app/model_registry_service.cpp` - confirmed the same stale hash also exists in runtime fallback registry construction, outside this task's write scope
- `app/runtime/desktop_app/main_window.cpp` - confirmed an existing unrelated local modification was present and left untouched

## Decisions Needed

- None for the package-check fix itself.

## Dependencies and Blockers

- No blocker remains for the packaged metadata contract.
- Existing unrelated local modifications were already present in `app/runtime/desktop_app/main_window.cpp`; they were not part of this task and were not edited.
- The runtime fallback registry code in `app/runtime/desktop_app/model_registry_service.cpp` still contains the old metadata hash literal. That did not affect package validation because the package ships `models/model_registry.json`, but it should be reconciled in a separate in-scope task if fallback-registry parity matters.

## Acceptance Criteria Check

- [x] A fresh portable package built from `C:\Users\goals\Codex\OpenDSS\build-opendss-internal-release` passes `check_package.ps1`.
- [x] The fix clearly explains why the previous hash mismatch occurred.
- [x] Trainer asset checks and existing package boundary checks remain intact.
- [x] The diff stays within assigned scope.

## Validation Performed

- `rtk git fetch origin`
- `rtk git merge --ff-only origin/main`
- `rtk git status --short --branch`
- `rtk read graphify-out/GRAPH_REPORT.md`
- `rtk read docs/agent-tasks/packaging-blockers-2026-07-07/package-metadata-hash-fix.md`
- `rtk read AGENTS.md`
- `rtk read docs/agent-debugging-workflow.md`
- `rtk read docs/active-bug-queue.md`
- `rtk read docs/current-state.md`
- `rtk read docs/build.md`
- `rtk read docs/migration-manifest.md`
- `rtk read docs/agent-results/packaging-readiness-2026-07-07/packaging-readiness-2026-07-07-results.md`
- `rtk read docs/worker-reports/packaging-readiness-2026-07-07/package-validation.md`
- `rtk read docs/worker-reports/packaging-readiness-2026-07-07/portable-package-trainer-assets.md`
- `rtk read docs/agent-results/packaging-blockers-2026-07-07/packaging-blockers-2026-07-07-results.md`
- `rtk grep -n "metadata.json|SHA256|sha256|hash" app/runtime/scripts app/runtime/models docs/worker-reports/packaging-readiness-2026-07-07 docs/agent-tasks/packaging-blockers-2026-07-07`
- `rtk read app/runtime/scripts/package_portable.ps1`
- `rtk read app/runtime/scripts/check_package.ps1`
- `rtk read app/runtime/models/model_registry.json`
- `rtk read app/runtime/models/metadata.json`
- `rtk grep -n "528ac091764c09cd9c2c6ad2a6ff1e38bb009184a26e7352b71b3a025c30902d|fa5321dfad900baec23fa6c239a29279e0e8c03fa2e78f0bd679dfb973888d2f" app/runtime docs`
- `rtk git log --oneline -- app/runtime/models/metadata.json app/runtime/models/model_registry.json`
- `rtk powershell -NoProfile -Command "(Get-FileHash -Algorithm SHA256 -LiteralPath 'app/runtime/models/metadata.json').Hash.ToLowerInvariant()"` - returned `528ac091764c09cd9c2c6ad2a6ff1e38bb009184a26e7352b71b3a025c30902d`
- `rtk powershell -NoProfile -Command "(Get-FileHash -Algorithm SHA256 -LiteralPath 'app/runtime/models/pre_binary_promotion_backup_metadata.json').Hash.ToLowerInvariant()"` - returned `dd5499f4c96e3b5d9c812adc114262a20a5b56e927ef15ba06d69720d4cc9bac`
- `rtk powershell -NoProfile -Command "(Get-FileHash -Algorithm SHA256 -LiteralPath 'app/runtime/models/squeezenet_final_new_condition.onnx').Hash.ToLowerInvariant()"` - matched the registry value `34eec09f49ab4612a34e3a24ccf85eccc98516b388fbadbfb0736ecbf8fb1769`
- `rtk powershell -NoProfile -Command "(Get-FileHash -Algorithm SHA256 -LiteralPath 'app/runtime/models/model.onnx.data').Hash.ToLowerInvariant()"` - matched the registry value `2e3727b593fee4f155caf67eb18a7b3a2b73ebb3655a1a6ab33b74c25a02ebd4`
- `rtk powershell -NoProfile -Command "& 'app/runtime/scripts/package_portable.ps1' -BuildDir 'C:\Users\goals\Codex\OpenDSS\build-opendss-internal-release'"` - passed, created `C:\Users\goals\Codex\OpenDSS\artifacts\internal-release\OpenVisualDropletSorterSuite_20260707_212836`
- `rtk powershell -NoProfile -Command "& 'app/runtime/scripts/check_package.ps1' -PackageDir 'C:\Users\goals\Codex\OpenDSS\artifacts\internal-release\OpenVisualDropletSorterSuite_20260707_212836' -SourceRoot 'C:\Users\goals\Codex\OpenDSS\2. Agent Debug Workspace\app\runtime'"` - passed

## Risks or Follow-Up

- `app/runtime/desktop_app/model_registry_service.cpp` still embeds the old `metadata_sha256` string for its fallback/default registry content. That was outside the assigned write scope, but it is a consistency risk if the app ever falls back to that hardcoded registry instead of the packaged `models/model_registry.json`.
