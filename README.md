# AORURA

AORURA LED library, CLI, and emulator.

## Table of contents

- [Protocol](#protocol)
- [Library](#library)
- [CLI](#cli)
- [Emulator](#emulator)
- [Hardware](#hardware)

## Protocol

AORURA exposes a serial connection (19200n8). Every command is exactly two bytes:

- `XX` turns the LED off
- `A<` puts the LED into its signature shimmering "aurora" state
- a color byte + `!` makes the LED light up with the given color
- a color byte + `*` makes the LED flash with the given color at a half-second
  interval

AORURA responds to these commands with one byte: `Y` if successful, `N` if not.

There's one more: `SS`. AORURA responds to this command with two bytes
representing the command for its current state.

AORURA's initial state is `B*` (flashing blue).

Valid color bytes:

- `B`: blue
- `G`: green
- `O`: orange
- `P`: purple
- `R`: red
- `Y`: yellow

## Library

[`aorura`](lib) is a Rust crate that implements [the AORURA protocol](#protocol).

## Usage

### Example

```rust
use aorura::*;
use failure::*;

fn main() -> Fallible<()> {
  let mut led = Led::open("/dev/ttyUSB0")?;

  led.set(State::Flash(Color::Red))?;
  led.set(State::Off)?;

  assert_eq!(led.get()?, State::Off);
  assert_eq!(State::try_from(b"B*")?, State::Flash(Color::Blue));

  Ok(())
}
```

## CLI

[`aorura-cli`](cli) is a CLI built on top of [the library](#library).

### Usage

```
Usage: aorura-cli <path> [--set STATE]
       aorura-cli --help

Gets, and optionally, sets AORURA LED state.

Options:
  --set STATE  set LED to given state

States: aurora, flash:COLOR, off, static:COLOR
Colors: blue, green, orange, purple, red, yellow
```

#### Example

```shell
path=/dev/ttyUSB0
original_state=$(aorura-cli $path)

# Enable flashing yellow LED:
aorura-cli $path --set flash:yellow

# Do something time-consuming:
sleep 10

# Revert back to the original LED state:
aorura-cli $path --set $original_state
```

## Emulator

[`aorura-emu`](emu) is a PTY-based AORURA emulator. It can be used with
[the CLI](#cli) in lieu of [the AORURA hardware](#hardware).

### Usage

```
Usage: aorura-emu <path>
       aorura-emu --help

Emulates AORURA LED device over a PTY symlinked to the given path.
```

## Hardware

- AORURA-3 (HoloPort and HoloPort+)

  ![AORURA-3 photo](res/aorura-3.jpg)

- AORURA-UART-1 (HoloPort Nano)

  ![AORURA-UART-1 photo](res/aorura-uart-1.jpg)
