# Changelog

All notable changes to avocado-bsp-jetson-orin-nano are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- `avocado-idle-governors.service`, shipped in the confext overlay at `/usr/lib/systemd/system` and enabled through
  `enable_services`. The kernels boot with the performance CPU and GPU governors, which keeps
  the GPU at its maximum clock at idle. The unit sets the GPU to `simple_ondemand` and the
  CPUs to `schedutil` after `nvpmodel.service`, which the unit also pulls in. Idle power on
  an Orin Nano at 25 W: 5.5 W with performance governors, 4.75 W with the unit's settings.
- `tegra-nvpmodel` on every kernel (5.15 and 6.6 on the 2024 feed, 6.8 and 6.18 on the 2026 feed). Neither `nvpmodel.service` nor
  `nvpower.service` was enabled on an Avocado runtime (package presets are not applied), so no
  power mode was set at boot and the GPU devfreq node did not exist until they ran. The
  governors unit now starts them.
- `/usr/libexec/avocado-idle-governors`, the script the unit runs. It logs the governor it set
  for the GPU and for each cpufreq policy, skips with a message when the GPU node is absent or
  `schedutil` is not available, and exits non-zero only when a write fails.
- `package_files` lists `overlay`, `stone` and `avocado.yaml`; the package previously carried
  only `avocado.yaml`.

### Changed
- `nvpmodel.service` now runs at boot on every kernel and sets the power
  mode. Before this release no runtime started it, so no power-mode clock caps applied; now
  the device boots into the default mode in `/etc/nvpmodel.conf`, which can be lower than the
  clocks the board ran at before, so throughput can drop on upgrade with no change to your
  `avocado.yaml`. Select a different mode with `nvpmodel -m <id>`; nvpmodel saves the choice in
  `/var/lib/nvpmodel/status`. The default mode per SKU is not listed here yet.

## [0.2.1]

### Changed
- The 2024 feed now uses the target name `jetson-orin-nano`, as the 2026 feed does:
  meta-avocado scarthgap renamed `jetson-orin-nano-devkit` to `jetson-orin-nano`.
  `supported_targets` lists the one name, and CI builds and releases it for
  both feeds. A 2024 project must set `default_target: jetson-orin-nano`.

## [0.2.0]

### Changed
- **The extension is now `avocado-bsp-jetson-orin-nano`** (was `avocado-bsp-jetson-orin-nano-devkit`),
  matching the repository rename. A project pinning the old name must update
  its `avocado.yaml`; the old package stays published at 0.1.0 on the 2024 feed
  and will not receive further updates.
- **Publishes to both feeds under each feed's own target name.** The 2026 feed
  calls this target `jetson-orin-nano`, the 2024 feed calls it `jetson-orin-nano-devkit`;
  `supported_targets` lists both and each release/test matrix leg names the one
  its feed uses. Previously both legs published to `jetson-orin-nano-devkit`, which
  is not a target in the 2026 feed — so nothing ever landed in
  `2026/next/target/jetson-orin-nano-ext`.

### Added
- Package set for the wrynose kernels: `kernel-6.18.*` (linux-yocto, upstream
  rtw88 + mainline btusb/btrtl) and `kernel-6.8.*` (L4T r39.2, NVIDIA's
  downstream Realtek shim and rtk_btusb).
- The scarthgap blocks (`kernel-5.15.*`, `kernel-6.6.*`) are retained so the
  2024 feed keeps building. `tegra-nvphs`, `tegra-nvstartup` and
  `kernel-module-crct10dif-ce` moved from the common list into them, since they
  exist in the 2024 feed but not on the wrynose kernels.
- The audio modules are referenced unprefixed (`kernel-module-snd-soc-*`) rather
  than `nv-kernel-module-snd-soc-*`: the unprefixed capability is Provided in
  both feeds, the `nv-` one only in 2024.

## [0.1.0]

### Added
- Initial release: Board support for the Nvidia Jetson Orin Nano devkit.
- CI via the shared `avocado-linux/actions` reusable workflows: PR build check
  (`test.yml`) and tag-driven package + publish to the Avocado feed (`release.yml`).
