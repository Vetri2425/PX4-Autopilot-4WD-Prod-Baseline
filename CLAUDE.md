# PX4-Autopilot-4WD-Prod-Baseline — DYX 4WD production baseline

Pristine PX4 firmware baseline for a **Holybro Pixhawk 6X 4WD production rover**.
Safety-critical C/C++. This file is the only preloaded context.

## Repo facts

- **Repo:** `Vetri2425/PX4-Autopilot-4WD-Prod-Baseline` (plain new repo, NOT a GitHub fork —
  the account's fork network for `PX4/PX4-Autopilot` is already occupied by the customized
  `Vetri2425/PX4-Autopilot` used for the 3WD rover, so a second "fork" would have inherited
  that history. Content/history here is a byte-identical mirror of upstream instead).
- **Pinned base:** PX4 `v1.17.0` stable == commit `d6f12ad1c4f70ad3230afd7d86e971421e02fef4`.
  Verified via `git describe --tags` == `v1.17.0` exactly (not RC/beta), present on
  `release/1.17` and `stable`, not `main`.
- **Production branch:** `dyx-4wd-production`, branched from the pinned commit.
- **Target hardware:** Holybro Pixhawk 6X — `px4_fmu-v6x_default`.
- **Upstream remote:** `upstream` → `https://github.com/PX4/PX4-Autopilot.git` (for reviewing
  future PX4 fixes selectively — never merge `upstream/main` wholesale).
- No firmware source, drivers, DDS topics, parameters, or board config have been modified yet.
  This is a clean baseline for later DDS + Rover Speed/Rate/Attitude control work.

## Build — CI only, one commit → one trigger → one artifact

```sh
make format               # if/when local source changes are made
git add -A && git commit  # ONE commit per change
git push origin dyx-4wd-production   # auto-triggers exactly one workflow: Build px4_fmu-v6x_default
```

`.github/workflows/build_fmu_v6x.yml` is the only active workflow in this repo — every other
stock PX4 workflow (Build all targets, Checks, EKF Update Change Indicator, SITL Tests, etc.)
was disabled via `gh workflow disable` (not deleted/edited — files are untouched, so upstream
source stays pristine). This guarantees exactly one CI run and one firmware artifact per push.

Build container: `ghcr.io/px4/px4-dev:v1.16.0-rc1-258-g0369abd556` — the same pinned toolchain
PX4's own release CI uses for this target. **Do not build with a bare `ubuntu-*` runner +
`Tools/setup/ubuntu.sh`** — Ubuntu's apt-packaged `gcc-arm-none-eabi` produces larger code and
overflows `px4_fmu-v6x_default`'s FLASH by ~16KB even on unmodified source (confirmed
2026-09-03: apt toolchain → FLASH 100.82%, build fails; pinned container → FLASH 99.70%, builds).

Watch + download, mirroring the 3WD rover fork's proven flow:
```sh
gh run watch <run-id> --repo Vetri2425/PX4-Autopilot-4WD-Prod-Baseline --exit-status
gh run download <run-id> --repo Vetri2425/PX4-Autopilot-4WD-Prod-Baseline --dir /tmp/dl
```

## Local artifact archive

After every successful build, copy the `.px4` into:
```
PX4-Firmware/<short-sha>-<slugified-commit-message>/
  px4_fmu-v6x_default.px4
  build_info.txt   # SHA, branch, message, target, CI run URL
```
`PX4-Firmware/` sits alongside this repo at `/Users/dyx_a1/Vetri/Way_to_Mark/PX4-Firmware/`,
one subfolder per successful build — never overwrite a prior build's folder.

⚠ **No local build.** Local submodules are fully deinitialized (`git submodule deinit --all`)
on purpose — this machine's network repeatedly stalls on PX4's larger submodules
(`Tools/simulation/*`), which GitHub's runners don't hit. Building locally re-litigates that
pain for no benefit; CI is the only build path. If you need to inspect submodule source,
`git submodule update --init <specific-path>` one at a time rather than `--recursive`.

## Commit rules

- One commit per logical change. One push == one CI trigger == one artifact folder.
- Conventional commits: `type(scope): description`.
- No Claude attribution in commit messages for this repo (matches the 3WD rover fork's rule).

## Do not (yet)

- Do not change rover control logic, DDS topics, parameters, drivers, or board configuration.
- Do not merge code from `Vetri2425/PX4-Autopilot` (the 3WD fork) or any other custom fork.
- Do not re-enable the disabled stock workflows unless you specifically need one of them —
  each re-enabled workflow adds another trigger per push, breaking the 1:1 commit→artifact rule.
