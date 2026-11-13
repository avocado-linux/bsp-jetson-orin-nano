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

To enable it, use a local copy of this extension (`source: { type: path }`),
uncomment the `CARRIER_FV_OVERLAY_DTB_FILE` line in
`stone/carrier-bsp/carrier.env`, then `avocado build` and `avocado provision`.
UEFI applies the overlay to the kernel device tree at boot. This was verified on
the 2026/next feed only; the 2024 feed's provisioner has not been tested. The source is
`dts/tegra234-p3767-sdmmc1-sdwire.dts`; rebuild the `.dtbo` with
`dtc -@ -I dts -O dtb -o stone/carrier-bsp/tegra234-p3767-sdmmc1-sdwire.dtbo dts/tegra234-p3767-sdmmc1-sdwire.dts`.
