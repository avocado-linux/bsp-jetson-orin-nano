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

## Rubicon

The BSP ships Rubicon run profiles (`rubicon.run` in `avocado.yaml`), so
`avocado provision --rubicon` and `avocado rubicon run` work with no
Rubicon config in the project. They expect this wiring of the Rubicon's
generic outputs:

| Rubicon (Arduino header) | Function | DevKit |
| --- | --- | --- |
| D2, `power` | power relay | the DevKit's supply |
| D3, `ssr1` | solid-state relay | across FC_REC and GND (J14): force recovery |
| D0 (RX), D1 (TX) | console UART, 3.3 V | the debug UART's TXD and RXD (crossed) |
| GND | ground | GND |

All of it fits an Arduino proto shield; the Rubicon's full pinout is in
rubicon-micro's `docs/pinout.md`. With the debug UART on D0/D1 no USB serial
adapter is needed: the hub reads the console from the Rubicon itself (a USB
adapter exported by the Rubicon, if present, is used instead).

Wired differently? Define a run profile of the same name in the project:
it replaces the BSP's whole.
