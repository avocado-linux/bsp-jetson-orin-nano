# Changelog

All notable changes to avocado-bsp-jetson-orin-nano are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
