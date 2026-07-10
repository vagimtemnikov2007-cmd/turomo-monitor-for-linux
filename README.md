# TURMO Linux UI Modular Alpha

A modular Linux application for small USB screens such as **TURMO / UsbMonitor / Turing Smart Screen 3.5"**. The project can send a system dashboard, static images, test patterns, and GIF animations to the display through a USB serial port.

This is a non-commercial experimental project. The main goal is to build a working pipeline for a 3.5-inch USB screen and gradually expand it without turning the whole codebase into one huge file.

## Features

- **PySide6** GUI.
- CLI launcher through `turmo_lite.py`.
- Frame transmission through a serial port.
- **RevA** protocol support for UsbMonitor/Turing 3.5" screens.
- System dashboard rendering: CPU, RAM, disk, temperatures, network, and GPU via `nvidia-smi` when available.
- PNG/JPG/WebP image sending.
- GIF animation playback.
- PNG preview generation without sending anything to the device.
- Test patterns for checking colors, orientation, and pixel formats.
- Helper scripts for testing stride, rotation, RGB565, and other display parameters.

## Tested screen profile

By default, the project is configured for this mode:

```text
Port:         /dev/ttyACM0
Baud:         4000000
Protocol:     reva
Resolution:   320x480
Pixel format: rgb565le
```

If your screen is detected differently, replace the port with `/dev/ttyUSB0` or a path from `/dev/serial/by-id/...`.

## Installation

Extract the archive and enter the project directory:

```bash
cd ~/Downloads/turmo-linux-ui-modular
chmod +x *.sh
./install.sh
```


The script creates a `.venv` virtual environment and installs dependencies from `requirements.txt`.

## Dependencies

Main Python dependencies:

```text
psutil
pillow
pyserial
PySide6
```

GPU metrics use `nvidia-smi` if it is installed. If it is not available, the application will still run, but GPU data may be shown as `N/A`.

## Quick GUI start

```bash
./run_gui.sh
```

The GUI lets you choose the mode, image, GIF, screen parameters, pixel format, rotation, background, and other settings.

## Quick dashboard start

```bash
./run_reva_dashboard.sh
```

This sends the default system dashboard to the screen using the RevA mode.

## Test pattern

```bash
./run_cli_reva_test.sh
```

Or directly:

```bash
python turmo_lite.py \
  --protocol reva \
  --port /dev/ttyACM0 \
  --baud 4000000 \
  --width 320 \
  --height 480 \
  --pixel-format rgb565le \
  --test-pattern \
  --once
```

## USB port permissions

If you get `Permission denied: /dev/ttyACM0`, temporarily allow access to the port:

```bash
sudo chmod a+rw /dev/ttyACM0
```

Permanent option:

```bash
sudo usermod -aG dialout "$USER"
sudo usermod -aG uucp "$USER"
reboot
```

After adding yourself to these groups, fully log out and back in, or reboot.

## Finding the screen port

Show serial ports:

```bash
python turmo_lite.py --list
```

If the screen is not found, check USB devices:

```bash
lsusb
ls -l /dev/ttyACM* /dev/ttyUSB* /dev/serial/by-id/* 2>/dev/null
```

For connection diagnostics, start `dmesg`, then reconnect the screen:

```bash
sudo dmesg -w
```

A good sign is a new device such as `ttyACM0` or `ttyUSB0`.

## Sending an image

Example with the included image:

```bash
./send_cat_reva.sh picture.png
```

Or directly:

```bash
python turmo_lite.py \
  --protocol reva \
  --port /dev/ttyACM0 \
  --baud 4000000 \
  --width 320 \
  --height 480 \
  --pixel-format rgb565le \
  --image picture.png \
  --fit contain \
  --clear \
  --once
```

## Generate a PNG preview without sending to the screen

```bash
python turmo_lite.py \
  --image picture.png \
  --fit contain \
  --dry-run preview.png
```

This is useful when you want to check how the frame will look at 320×480 before sending it to the device.

## GIF animations

Play the included sample GIF:

```bash
./play_sample_gif.sh
```

Play your own GIF:

```bash
./play_gif_cli.sh file.gif
```

Or directly:

```bash
python turmo_lite.py \
  --gif file.gif \
  --gif-loop \
  --gif-fps 8 \
  --protocol reva \
  --port /dev/ttyACM0 \
  --baud 4000000 \
  --width 320 \
  --height 480 \
  --pixel-format rgb565le
```

Full-screen GIFs can be heavy because one 320×480 RGB565 frame is about 300 KB. Smaller GIFs or lower FPS values such as `3–8` usually work better.

## Useful CLI flags

```text
--list                  show serial ports
--port                  set the screen port
--baud                  set the serial baud rate
--protocol              reva or legacy
--width / --height      framebuffer size
--pixel-format          pixel format
--test-pattern          send a test image
--image                 send a PNG/JPG/WebP image
--gif                   play a GIF
--gif-loop              loop the GIF
--gif-fps               limit GIF FPS
--fit                   contain / cover / stretch
--bg                    background: black, white, gray, #RRGGBB
--rotate                rotate the final framebuffer
--content-rotate        rotate the source image before fitting
--safe-margin           add inner padding
--clear                 clear the screen before sending the frame
--once                  send one frame and exit
--dry-run               save the frame to PNG without sending it
--save-prepared         save the prepared frame
--prepare-only          prepare only, do not send
--brightness            brightness 0..100
```

Show the full help:

```bash
python turmo_lite.py --help
```

## Calibration scripts

The archive includes helper scripts for experimenting with the device:

```text
try_reva_baud.sh          test different baud rates
probe_stride.sh           test one stride value
probe_all_strides.sh      scan multiple stride values
probe_roll_y.sh           test vertical shifting
probe_tail_pad.sh         test tail padding
test_reva_rgb565.sh       RevA + RGB565 test
test_rgb565_locked.sh     RGB565 test without GUI
test_stride480.sh         480x320 mode test
```

These scripts are not needed for normal use. They are for debugging cases where the image shifts, wraps, becomes distorted, or uses the wrong colors.

## Project structure

```text
turmo-linux-ui-modular/
  install.sh                 environment setup
  run_gui.sh                 launch GUI
  run_reva_dashboard.sh      launch system dashboard
  run_cli_reva_test.sh       RevA test pattern
  play_sample_gif.sh         play sample_spinner.gif
  play_gif_cli.sh            play a custom GIF
  send_cat_reva.sh           send picture.png through RevA
  turmo_lite.py              compatible CLI entry point
  turmo_gui.py               compatible GUI entry point
  turmo/
    cli.py                   argparse CLI
    gui.py                   PySide6 GUI
    constants.py             default screen parameters
    serial_device.py         TURMO/RevA serial protocols
    codecs.py                RGB/RGB565/BGRA encoding
    image_ops.py             resize, rotate, fit, background, margin
    gif.py                   GIF frame preparation
    metrics.py               system metrics collection
    renderers.py             dashboard and test pattern renderers
    frame.py                 frame source selection
    ports.py                 serial port discovery
    core.py                  compatibility layer
    sources/                 placeholder for future frame sources
  ARCHITECTURE.md            architecture notes
  README.md                  this file
```

## Architecture

Older versions kept almost all logic inside one `turmo_lite.py` file. In this version, the logic is split into the `turmo/` package:

1. CLI or GUI selects the mode and settings.
2. `frame.py` chooses the frame source: dashboard, image, GIF, or test pattern.
3. `image_ops.py` converts the image to the required size and layout.
4. `codecs.py` encodes pixels.
5. `serial_device.py` sends the frame to the USB screen.

This makes it easier to add new frame sources later: screen streaming, terminal widgets, system monitoring, or custom scenes.

## Note about screen capture

The project contains `turmo/sources/screen.py`, but screen capture is not implemented yet. It is a placeholder for future desktop streaming through PipeWire, X11, MSS, or another backend.

## Troubleshooting

### `No .venv found`

Run the installer first:

```bash
./install.sh
```

### `Permission denied: /dev/ttyACM0`

```bash
sudo chmod a+rw /dev/ttyACM0
```

Or add the user to the `dialout` and `uucp` groups.

### Screen is not found

Check the cable. Some USB cables are charge-only and do not transfer data.

```bash
lsusb
sudo dmesg -w
```

### Image is shifted, wrapped, or distorted

Use the RevA mode:

```bash
--protocol reva --pixel-format rgb565le --width 320 --height 480
```

If that does not help, try the `probe_*` calibration scripts.

### Wrong colors

Try another `--pixel-format`, but for the tested RevA profile the usual value is:

```bash
--pixel-format rgb565le
```

### GIF is slow

Lower the FPS:

```bash
--gif-fps 3
```

Or use a smaller GIF.

## Minimal working flow

```bash
cd ~/Downloads/turmo-linux-ui-modular
chmod +x *.sh
./install.sh
sudo chmod a+rw /dev/ttyACM0
./run_cli_reva_test.sh
./run_reva_dashboard.sh
```


If the test pattern appears on the screen, the basic frame-sending pipeline works.
