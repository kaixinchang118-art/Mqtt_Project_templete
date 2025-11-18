This repository is an STM32F1 (HAL) project generated partly by STM32CubeMX and built via CMake/Ninja.

Keep guidance concise and actionable for code-generation and small embedded changes.

Quick architecture
- **Project Type:** STM32F1 HAL project (CubeMX generated sources under `cmake/stm32cubemx`).
- **Binary:** `1_LED` built by CMake; build outputs go to `build/<Preset>/` (presets in `CMakePresets.json`).
- **Key directories:** `Core/` (system and `main.c`), `Drivers/` (HAL/CMSIS), `Bsp/` (board drivers: `led.c`, `key.c`).

Developer workflows (explicit)
- **Configure & build:**
  - Configure + build with presets: `cmake --preset Debug` then `cmake --build --preset Debug`.
  - VS Code Tasks are configured: run the `build` or `build and flash` tasks from the Command Palette/Run Tasks. These use the `eide` commands defined in the workspace tasks.
- **Flash:** Use the workspace task `flash` or `build and flash` (they call the `eide` upload command). The builder produces `1_LED.hex` which the flasher uses (example from recent run: `1_LED.hex` at project root/build folder).
- **Debugging:** Use ST-Link / STM32CubeProgrammer or your usual debugger; logs show `STM32CubeProgrammer` was used in this workspace.

Project-specific patterns & conventions
- **HAL-first design:** The code uses STM32 HAL APIs (e.g., `HAL_GPIO_Init`, `HAL_Delay`, `HAL_UART_Transmit`). Prefer HAL calls unless adding low-level optimizations.
- **Board support in `Bsp/`:** Small drivers live in `Bsp/`. Example: `Bsp/led.c`
  - LED uses `GPIOA`, `PIN 2` (PA2) and is *active-low*: `Led_Set(LED_ON)` writes `GPIO_PIN_RESET`.
  - `Led_Status` is a global `uint8_t` used across `Bsp` files.
- **Input handling:** `Bsp/key.c` installs EXTI on `PA1` (`GPIO_MODE_IT_FALLING`, `GPIO_PULLUP`) and toggles LED inside `HAL_GPIO_EXTI_Callback` with a simple time-based debounce (uses `HAL_GetTick()`). Keep interrupt work short and use debouncing exactly as shown.
- **printf => UART:** `main.c` redirects `fputc` to `huart1`, so `printf()` output appears on USART1. Use `printf` for simple runtime logs.
- **Interrupts / Error handling:** `Error_Handler()` disables IRQs and loops. NVIC priorities are set directly (example: `HAL_NVIC_SetPriority(EXTI1_IRQn, 0, 0)`).

Build system notes for code edits
- **Where to add sources:** Add non-generated source files under `Bsp/` or `Core/Src/` and include them via the root `CMakeLists.txt` `target_sources()` or by updating `cmake/stm32cubemx` generated CMakeLists if those files should be managed with the CubeMX-generated set.
- **Toolchain:** `cmake/gcc-arm-none-eabi.cmake` is used by presets. Presets are `Debug` and `Release` (`CMakePresets.json`). The generator is `Ninja`.
- **Regenerating CubeMX code:** If you change `.ioc` and re-run CubeMX, the generated sources live in `cmake/stm32cubemx/`. Do not manually edit generated files unless you understand the CubeMX re-generation impact.

AI agent guidance (how to act here)
- When asked to add a peripheral driver: create `Bsp/<name>.c/.h`, implement Init/Deinit and simple API (e.g., `Led_Init`, `Led_Set`), then update `CMakeLists.txt` `target_sources()` to include the files.
- When editing IRQ or HAL callbacks: preserve the `HAL_GPIO_EXTI_Callback` signature and debounce logic pattern used in `Bsp/key.c`.
- For printf/logging: prefer `printf()` (already routed to `huart1`) for short messages. Do not assume a console is available.
- When changing clock or system files: update only under `Core/` and avoid manual changes to `cmake/stm32cubemx` unless regenerating from CubeMX.

Files to inspect first
- `CMakePresets.json` — build presets & generator.
- `cmake/stm32cubemx/` — CubeMX-generated CMake and source lists.
- `Core/Src/main.c`, `Core/Src/system_stm32f1xx.c` — main flow and system init.
- `Bsp/led.c`, `Bsp/key.c` — examples of board APIs and interrupt handling.

Prompts/examples for the maintainer
- "Add a button debouncer using software timer similar to `Bsp/key.c` and expose `Key_Pressed()` API." — create `Bsp/key.c` changes and update header and `CMakeLists.txt`.
- "Add a second LED on `PA3` with same active-low behavior as `PA2`" — add `Bsp/led2.c` and `led.h` updates, initialize in `main.c` after `Led_Init()`.

If anything here is unclear, ask for the specific change and I will update instructions or make a minimal patch.
