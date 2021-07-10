# 電源管理装置

## Power Management Device - Mitsumi 3152A (NDS) / Mitsumi 3205B (NDS-LITE)

The Power Management Device is accessed via SPI bus, DS Serial Peripheral Interface Bus (SPI)

To access the device, write the Index Register, then read or write the data register, and release the chipselect line when finished.

**Index Register**

```
  Bit0-6 Register Select          (0..3) (0..4 for DS-Lite) (0..7Fh for DSi)
  Bit7   Register Direction       (0=Write, 1=Read)
```

**Register 0 - Powermanagement Control (R/W)**

```
 Bit0   Sound Amplifier Enable   (0=Disable, 1=Enable)
         (Old-DS:  Disabled: Sound is very silent, but still audible)
         (DS-Lite: Disabled: Sound is NOT audible)
         (DSi in NDS Mode: R/W, but effect is unknown yet)
         (DSi in DSi Mode: Not used, Bit0 is always 1)
 Bit1   Sound Amplifier Mute     (0=Normal, 1=Mute) (Old-DS Only, not DS-Lite)
         (Old-DS:  Muted: Sound is NOT audible, that works only if Bit0=1)
         (DS-Lite: Not used, Bit1 is always zero)
         (DSi in NDS Mode: R/W, but effect is unknown yet)
         (DSi in DSi Mode: R/W, but effect is unknown yet)
 Bit2   Lower Backlight          (0=Disable, 1=Enable)
 Bit3   Upper Backlight          (0=Disable, 1=Enable)
 Bit4   Power LED Blink Enable   (0=Always ON, 1=Blinking OFF/ON)
 Bit5   Power LED Blink Speed    (0=Slow, 1=Fast) (only if Blink enabled)
         (DSi: Power LED Blinking isn't supported, neither in NDS nor DSi mode)
 Bit6   DS System Power          (0=Normal, 1=Shut Down)
 Bit7   Not used                 (always 0)
```

**Register 1 - Battery Status (R)**

```  
 Bit0   Battery Power LED Status (0=Power Good/Green, 1=Power Low/Red)
         (DSi: Usually 0, not tested if it changes upon Power=Low)
 Bit1-7 Not used
```

**Register 2 - Microphone Amplifier Control (R/W)**

```
  Bit0   Amplifier                (0=Disable, 1=Enable)
  Bit1-7 Not used                 (always 0)
  (DSi in NDS Mode: looks same as NDS, ie. only bit0 is R/W)
  (DSi in DSi Mode: Not used, always FFh)
```

**Register 3 - Microphone Amplifier Gain Control (R/W)**

```
  Bit0-1 Gain                     (0..3=Gain 20, 40, 80, 160)
  Bit2-7 Not used                 (always 0)
  (DSi in NDS Mode: looks same as NDS, ie. only bit0-1 are R/W)
  (DSi in DSi Mode: Not used, always FFh)
```

**Register 4 - DS-Lite and DSi Only - Backlight Levels/Power Source (R/W)**

```
  Bit0-1 Backlight Brightness (0..3=Low,Med,High,Max)   (R/W)
         (when bit2+3 are both set, then reading bit0-1 always returns 3)
  Bit2   Force Max Brightness when Bit3=1 (0=No, 1=Yes) (R/W)
  Bit3   External Power Present           (0=No, 1=Yes) (Read-Only)
  Bit4-7 Unknown (Always 4) (Read-Only)
  (DSi in NDS Mode: looks same as in DSi mode)
  (DSi in DSi Mode: Bit0-1 are R/W, but ignored, bit2-3 are always 0)
```

**Register 10h - DSi Only - Backlight Mirrors & Reset (R/W)**

```
  Bit0   Reset (0=No, 1=Reboot DSi) (same/similar as BPTWL reset feature?)
  Bit1   Unknown (R/W) (note: whatever it is, it isn't warmboot flag)
  Bit2-3 Mirror of Register 0, bit2-3 (backlight enable bits) (R/W)
  Bit4-7 Not used (always 0)
  Bit5   Not used (always 0) - but DSi bootrom sets that bit on boot error?
  (This register works in NDS mode and DSi mode, though it's mainly intended
  for NDS mode, eg. DS Download Play uses the Reset bit to return to DSi menu)
  (note: writing bit2 seems to affect BOTH bit1 and bit2 in register 0)
```

**Register 1Fh and 20h - DSi Only (?)**

DSi bootrom sets register 1Fh and 20h bit0-4 to value 1Fh on boot error, unknown purpose, seems to have no effect, maybe prototype backlight level?

## Notes

On Old-DS, registers 4..7Fh are mirrors of 0..3. On DS-Lite, registers 5,6,7 are mirrors of 4, register 8..7Fh are mirrors of 0-7.

On DSi (in DS mode), index 0,1,2,3,4,10h are used (reads as 0Fh,00h,00h,01h,41h,0Fh - regardless of backlight level, and power source), index 5..0Fh and 11h..7Fh return 00h (ie. unlike DS and DS-Lite, there are no mirrors; aside from the mirrored bits in register 10h).
