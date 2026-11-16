# bsp-jetson-orin-nano

Board support for the Nvidia Jetson Orin Nano devkit

## Using this extension

`bsp-jetson-orin-nano` is an [Avocado](https://avocadolinux.org) extension — a reusable fragment of
build- and runtime-configuration that you compose into your own Avocado project. To use it,
declare it as a package-sourced extension in your `avocado.yaml` and add it to a runtime:

```yaml
extensions:
  avocado-bsp-jetson-orin-nano:
    source:
      type: package
      version: "*"        # or pin an exact version

runtimes:
  my-runtime:
    extensions:
      - avocado-bsp-jetson-orin-nano
```

Then install and build:

```sh
avocado install   # fetches + installs the SDK, extensions and runtime deps from your config
avocado build     # builds the SDK compile steps, extensions and runtime images
```

`avocado install` pulls the extension from your target's package feed and merges its
config into your project; `avocado build` then produces the runtime.

## microSD behind an SD mux (SDWire3)

With an SD mux tongue in the microSD slot, reads through the mux fail at the
stock SDR104 clock (195 MHz) and pass at 65 MHz and below. The slot's
card-detect switch also does not follow a mux switch. The opt-in overlay
`stone/carrier-bsp/tegra234-p3767-sdmmc1-sdwire.dtbo` sets `max-frequency =
<65000000>` and `non-removable` on `mmc@3400000`: the card is probed at boot
and whenever the host is rebound
(`echo 3400000.mmc > /sys/bus/platform/drivers/sdhci-tegra/unbind`, then `bind`),
never on insertion. Expect about 20 MiB/s reads instead of about 75 MiB/s.
To enable it, use a local copy of this extension. Under the extension declaration
in the consuming project, set
`source: { type: path, path: ../bsp-jetson-orin-nano }`, where `path` points at
the local checkout of this repository. Then uncomment the `CARRIER_FV_OVERLAY_DTB_FILE` line in
`stone/carrier-bsp/carrier.env`, then `avocado build` and `avocado provision`.
UEFI applies the overlay to the kernel device tree at boot. This was verified on
the 2026/next feed only; the 2024 feed's provisioner has not been tested. The source is
`dts/tegra234-p3767-sdmmc1-sdwire.dts`; rebuild the `.dtbo` with
`dtc -@ -I dts -O dtb -o stone/carrier-bsp/tegra234-p3767-sdmmc1-sdwire.dtbo dts/tegra234-p3767-sdmmc1-sdwire.dts`.

## Idle frequency scaling

The extension ships `avocado-idle-governors.service`, which sets the GPU to `simple_ondemand` and
the CPUs to `schedutil` at boot, after `nvpmodel.service`. The kernels otherwise boot with the
performance governors, which hold the GPU at its maximum clock at idle.

To keep the performance governors (a tight control loop, a benchmark, latency-sensitive inference
that cannot wait for the GPU to ramp up), mask the unit from your own extension's overlay by
shipping a symlink at `/etc/systemd/system/avocado-idle-governors.service` pointing to
`/dev/null`. A unit in `/etc` takes precedence over the one in `/usr/lib`. The extension enables
`nvpmodel.service` on its own, so masking the governors unit leaves the power mode and the GPU
devfreq node in place.

## Tested kernels

`tegra-nvpmodel` is installed on every kernel, including 5.15 and 6.6 on the 2024 feed. It has not been tested on a 2024 kernel, so `nvpmodel.service` on kernels 5.15 and 6.6 is untested.
