To build PhobGCC for Raspberry Pi Pico targets (RP2040 and RP2350):

1. Install the compiler toolchain as per the [Pico documentation](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf).
2. Git clone the `pico-sdk` repository from https://github.com/raspberrypi/pico-sdk
3. Check out a release that supports your target. RP2350 support requires Pico SDK 2.0.0 or newer.
4. Run `git submodule update --init` (omit `--init` if you're not doing it for the first time)
5. Export the `pico-sdk` directory as an environment variable `export PICO_SDK_PATH=[directory containing]/pico-sdk`
6. Clone the PhobGCC-SW repository.
7. Uncomment the appropriate `#include` line for your board in `../common/phobGCC.h`.
8. `cd` into the `PhobGCC/rp2040` subdirectory inside the `PhobGCC-SW` folder.
9. `mkdir build && cd build`
10. Configure for your board:
    - RP2040 / Pico: `cmake -DPICO_BOARD=pico ..`
    - RP2350 / Pico 2: `cmake -DPICO_BOARD=pico2 ..`
    - Or explicitly choose the chip family: `cmake -DPICO_PLATFORM=rp2040 ..` or `cmake -DPICO_PLATFORM=rp2350 ..`
11. Build with `cmake --build .`

If you switch between RP2040 and RP2350, use separate build directories or delete and recreate the old build directory before reconfiguring.
