# WRAM

## 4000247h - NDS9 - WRAMCNT - 8bit - WRAM Bank Control (R/W)
## 4000241h - NDS7 - WRAMSTAT - 8bit - WRAM Bank Status (R)

Nintendoの公式資料によるとこのレジスタの値は変更しない方がよさそうです。

 bit  |  内容
---- | ----
0-1 | ARM9/ARM7(0-3 = 32K/0K, 2nd 16K/1st 16K, 1st 16K/2nd 16K, 0K/32K)
2-7 | 不使用

ARM9では WRAMは `0x0300_0000-0x03FF_FFFF`の16MB、ARM7では WRAMは `0x0300_0000-0x037F_FFFF` の 8MBです。

The allocated 16K or 32K are mirrored everywhere in the above areas.

De-allocation (0K) is a special case: At the ARM9-side, the WRAM area is then empty (containing undefined data). At the ARM7-side, the WRAM area is then containing mirrors of the 64KB ARM7-WRAM (the memory at 3800000h and up).
