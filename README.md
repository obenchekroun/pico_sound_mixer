# sound_fx_mixer

This code implements a triggered sound FX capability for a Pi Pico. Ten push buttons (GP0 to GP9) each trigger a sound, up to four sounds can play at the same time and are mixed in real time, then sent over I2S (GP13 DIN, GP14 BCLK, GP15 LRCK) to a MAX98357 amplifier. This project makes use of pico-extras (https://github.com/raspberrypi/pico-extras.git).

## Adding or changing sounds

The sounds are stored in flash as C arrays, generated from `.wav` files by `wav_convert/wav_convert.py`. Every `.wav` must be mono, 16-bit PCM, 24000 Hz. The script refuses any file that does not match.

1. Put the `.wav` files in `wav_convert/wav_files/`. The file name becomes the C identifier, so use letters, digits and underscores only (for example `cow1.wav`).
2. Generate straight into the firmware folder:

   ```
   python3 wav_convert/wav_convert.py -o sounds
   ```

   This writes one `.c` per sound, plus `sounds/sounds.h` (the `enum sound_id` with `CAT1`, `COW1`... and the extern declarations) and `sounds/sounds_table.c` (the `sound_array[]` and `sound_array_len[]` tables). Remove the `.c` of any sound you deleted.
3. Map buttons to sounds in `button_sound_map[]` in `main.c`, using the enum names.
4. Rebuild. `CMakeLists.txt` picks up every `.c` in `sounds/` automatically.

Without `-o`, the script writes into `wav_convert/output/` so you can inspect the result first.

## Build

```
mkdir build && cd build
cmake ..
make -j8
```

Copy `build/pico_sound_mixer.uf2` to the Pico while holding BOOTSEL.

## Tuning

In `main.c`:

- `NUM_VOICES`: number of simultaneous sounds. When all voices are busy, a new press steals the oldest one.
- `MIX_SHIFT`: per-voice attenuation in bits (1 = -6 dB, 2 = -12 dB). The mix is accumulated in 32 bits and clipped.
- `DEBOUNCE_LOOPS`: button lockout after an edge, in main-loop iterations of about 10.7 ms.
