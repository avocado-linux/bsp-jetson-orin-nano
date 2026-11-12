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

## Idle frequency scaling

The extension ships `avocado-idle-governors.service`, which sets the GPU to `simple_ondemand` and
the CPUs to `schedutil` at boot, after `nvpmodel.service`. The kernels otherwise boot with the
performance governors, which hold the GPU at its maximum clock at idle.

To keep the performance governors (a tight control loop, a benchmark, latency-sensitive inference
that cannot wait for the GPU to ramp up), mask the unit from your own extension's overlay by
shipping a symlink at `/etc/systemd/system/avocado-idle-governors.service` pointing to
`/dev/null`. A unit in `/etc` takes precedence over the one in `/usr/lib`. Masking it does not
stop `nvpmodel.service`, which is a separate unit.
