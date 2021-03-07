# 描画サイクル

## Horizontal Dimensions

The drawing time for each dot is 4 CPU cycles.

```
  Visible     240 dots,  57.221 us,    960 cycles - 78% of h-time
  H-Blanking   68 dots,  16.212 us,    272 cycles - 22% of h-time
  Total       308 dots,  73.433 us,   1232 cycles - ca. 13.620 kHz
```

VRAM and Palette RAM may be accessed during H-Blanking. OAM can accessed only if "H-Blank Interval Free" bit in DISPCNT register is set.

## Vertical Dimensions

```
  Visible (*) 160 lines, 11.749 ms, 197120 cycles - 70% of v-time
  V-Blanking   68 lines,  4.994 ms,  83776 cycles - 30% of v-time
  Total       228 lines, 16.743 ms, 280896 cycles - ca. 59.737 Hz
```

All VRAM, OAM, and Palette RAM may be accessed during V-Blanking.

Note that no H-Blank interrupts are generated within V-Blank period.

(*) Even though vertical screen size is 160 lines, the upper 8 lines are not **really** visible, these lines are covered by a shadow when holding the GBA orientated towards a light source, the lines are effectively black - and should not be used to display important information.

## System Clock

The system clock is 16.78MHz (16x1024x1024 Hz), one cycle is thus approx. 59.59ns.

## Interlace

The LCD display is using some sort of interlace in which even scanlines are dimmed in each second frame, and odd scanlines are dimmed in each other frame (it does always render ALL lines in ALL frames, but half of them are dimmed).

The effect can be seen when displaying some horizontal lines in each second frame, and hiding them in each other frame: the hardware will randomly show the lines in dimmed or non-dimmed form (depending on whether the test was started in an even or odd frame).

Unknown if it's possible to determine the even/off frame state by software (or possibly to reset the hardware to this or that state by software).

Note: The NDS is applying some sort of frameskip to GBA games, about every 3 seconds there will by a missing (or maybe: inserted) frame, ie. a GBA game that is updating the display in sync with GBA interlace will get offsync on NDS consoles.